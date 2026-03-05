---
created: 2026-03-06
tags: [OpenClaw, 多 Agent, 架构设计, Session 管理，术哥无界]
source: https://mp.weixin.qq.com/s/TiVIweyZEKivTYxpXvZFSw
author: 术哥无界
---

# 用 OpenClaw 打造你的 AI 助手军团：Gateway -> Agent -> Session 多 Agent 架构完全拆解与实战

> 无界探索，有术而行。

---

## 🎯 OpenClaw 定位

**自托管网关**，连接多个聊天应用（WhatsApp、Telegram、Discord、飞书、企微等）与 AI Agent，让你的一套 AI 能力同时服务多个渠道。

> 说得更直白一点：你有一个 AI 助手，你想在多个平台上都能用它，你不想为每个平台单独部署一套，你还想要不同平台用不同的"人格"。OpenClaw 就是干这事的。

---

## 🏗️ 三层架构

```
聊天应用层（WhatsApp/Telegram/Discord）
       ↓
Gateway（守护进程）
       ↓
┌──────┼──────┐
↓      ↓      ↓
Agent  Agent  Agent（多个独立人格）
↓      ↓      ↓
Session Session Session（各自的对话上下文）
```

---

## 🔑 核心概念

### Gateway：大管家

运行在自己机器或服务器上的守护进程：

- **维护连接**：保持与 WhatsApp、Telegram 等平台的长连接
- **消息路由**：根据规则把消息分发到对应的 Agent
- **协议转换**：把各平台的消息格式统一成标准格式
- **安全验证**：通过 JSON Schema 验证入站消息

> 一个 Gateway 可以同时服务十几个 Agent，每个 Agent 独立运行，互不干扰。

### Agent：独立人格

一个 Agent 就是一个"完整的大脑"，拥有自己的：

- **Workspace（工作区）**：文件、笔记、人格配置（AGENTS.md、SOUL.md）
- **State Directory（状态目录）**：认证配置、模型设置，路径在 `~/.openclaw/agents/<agentId>/agent/`
- **Session Store（会话存储）**：聊天历史，路径在 `~/.openclaw/agents/<agentId>/sessions/`

> ⚠️ 关键点：**每个 Agent 的认证配置是独立的**。千万别跨 Agent 重用 `agentDir` —— 我第一次配置时踩了这个坑，结果会话全乱了。

### Session：上下文管理器

每个 Session 有独立的：

**Session Key（会话键）**：
- 直接聊天：`agent:<agentId>:main`
- 群组：`agent:<agentId>:whatsapp:group:<id>`
- Sub-Agent：`agent:<agentId>:subagent:<uuid>`

**DM Scope（私聊作用域）**：
| 模式 | 说明 |
|------|------|
| `main` | 所有私聊共享一个会话（默认，保持对话连续性） |
| `per-peer` | 每个联系人独立会话 |
| `per-channel-peer` | 按渠道 + 联系人隔离（多用户场景推荐） |

**生命周期**：可以设置每日重置、空闲重置，或手动 `/reset`

> Session 机制是设计得非常优雅的部分。

---

## 🔄 Agent 运行循环

```
Intake（接收消息）
  ↓
Context Assembly（组装上下文）
  ↓
Model Inference（模型推理）
  ↓
Tool Execution（工具执行）
  ↓
Streaming Replies（流式回复）
  ↓
Persistence（持久化）
```

**WebSocket 协议**：
- Request：`{type:"req", id, method, params}`
- Response：`{type:"res", id, ok, payload|error}`
- Event：`{type:"event", event, payload}`

**上下文管理**：
- **Compaction（压缩）**：把较旧的对话总结成摘要，支持手动 `/compact`
- **Session Pruning（修剪）**：只修剪旧的工具结果，保留对话文本
- **Memory System**：`memory/YYYY-MM-DD.md`（每日日志）+ `MEMORY.md`（长期记忆）

---

## 🎛️ 多 Agent 路由规则

路由是**确定性的，最具体优先**：

```
peer 匹配（精确的群组/频道 ID）
  ↓
guildId（Discord 服务器）
  ↓
teamId（Slack 团队）
  ↓
accountId（账号级别）
  ↓
频道级别匹配（accountId: "*"）
  ↓
回退到默认 Agent
```

**配置示例**：
```javascript
{
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363424282127706@g.us" },
      },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

**效果**：
- 特定 WhatsApp 群组 → family Agent（受限权限）
- 其他 WhatsApp 私聊 → chat Agent（日常助手）
- Telegram 所有消息 → opus Agent（深度工作）

---

## 🤖 Sub-Agent：后台任务

启动后台任务，不阻塞主对话：

```
用户："帮我研究一下最新的 Node.js 发布说明"
  ↓
主 Agent 调用 sessions_spawn 工具
  ↓
创建新 Session：agent:main:subagent:<uuid>
  ↓
Sub-Agent 在后台运行研究任务
  ↓
完成后向主聊天公告结果
```

主 Agent 会立即返回 `{status: "accepted", runId}`，然后 Sub-Agent 自己跑。

**配置示例**：
```javascript
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",  // 用更便宜的模型
        thinking: "low",
        maxConcurrent: 4,
        archiveAfterMinutes: 30,
      },
    },
  },
}
```

---

## 💼 实战场景

### 场景 1：个人助手 + 家庭 Agent

```javascript
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Personal Assistant",
        workspace: "~/.openclaw/workspace",
        sandbox: { mode: "off" },  // 主 Agent 不沙箱化
      },
      {
        id: "family",
        name: "Family Bot",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent" },  // 完全沙箱化
        tools: {
          allow: ["read"],
          deny: ["exec", "write", "edit", "apply_patch"],  // 只读
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "your-group-id@g.us" },
      },
    },
  ],
}
```

### 场景 2：工作 Agent + 日常 Agent

```javascript
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",  // 快速响应
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",  // 深度思考
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

---

## 📊 与其他框架对比

| 特性 | OpenClaw | LangChain | AutoGen | Claude Code |
|------|----------|-----------|---------|-------------|
| 部署方式 | 自托管 Gateway | 库/框架 | 库/框架 | IDE 集成 |
| 多渠道支持 | ✅ 原生支持 | ❌ 需要自己集成 | ❌ 需要自己集成 | ❌ 仅 IDE |
| Session 管理 | ✅ 内置多种作用域 | ⚠️ 需要自己实现 | ⚠️ 需要自己实现 | ✅ 简单管理 |
| 多 Agent 路由 | ✅ 原生支持 | ⚠️ 需要自己实现 | ✅ 支持 | ❌ 不支持 |
| Sub-Agent | ✅ 内置非阻塞 | ⚠️ 需要自己实现 | ✅ 支持 | ⚠️ 受限 |
| 沙箱隔离 | ✅ Docker 集成 | ⚠️ 需要自己实现 | ⚠️ 需要自己实现 | ❌ 不支持 |
| 开源 | ✅ MIT | ✅ MIT | ✅ MIT | ❌ 闭源 |

---

## ⚠️ 最佳实践：踩坑总结

### Session 设计
- **主 Session 专用于 1:1 流量**：让群组保持自己的 Session Key
- **多用户场景必须启用 `dmScope`**：否则所有人的消息会混在一起
- **定期压缩**：用 `/compact` 保持上下文新鲜

### 多 Agent 路由
- **最具体优先**：Peer 匹配 > Account 匹配 > Channel 匹配
- **明确绑定**：不要依赖隐式默认，写清楚
- **每个 Agent 独立工作区**：不要共享 `agentDir`

### 安全建议
- **默认启用沙箱**：`non-main` 模式保护非主 Session
- **限制工具权限**：特别是 `exec`、`write`、`edit`
- **备份工作区**：放入私有 git 仓库

### 记忆管理
- **每日日志**：`memory/YYYY-MM-DD.md`
- **长期记忆**：`MEMORY.md` 用于持久事实
- **启用自动刷新**：让模型在压缩前写入重要信息

---

## 📝 常用命令

```bash
# 查看状态
openclaw status

# 列出所有 Session
openclaw sessions --json

# 查看上下文使用
/status

# 压缩上下文
/compact Focus on decisions and open questions

# Sub-Agent 管理
/subagents list
/subagents stop <id>
/subagents log <id> 10 tools
```

---

## 💭 写在最后

> OpenClaw 给我的感觉，像是一把精心设计的瑞士军刀——不是每个功能都行业领先，但组合在一起，解决了一个真实存在的问题。

**多 Agent 架构范式**：
- Gateway 统一接入
- Agent 独立隔离
- Session 精细管理
- 路由灵活配置

> 在这个 AI 遍地的时代，**能不能让 AI 无处不在，可能是下一个竞争焦点**。

---

**原文**：[术哥无界 - 用 OpenClaw 打造你的 AI 助手军团](https://mp.weixin.qq.com/s/TiVIweyZEKivTYxpXvZFSw)
**归档**：2026-03-06
