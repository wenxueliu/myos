---
created: 2026-03-06
tags: [OpenClaw, 多 Agent, 飞书，腾讯云服务器，AI 团队协作，agentToAgent]
source: https://juejin.cn/post/7609481369001492499
author: 风象南
---

# 用 OpenClaw + 飞书，快速搭建 5 个可协作的 AI 助理团队

> 多个飞书机器人 + 独立工作空间 + 互相协作 = 专业化分工的 AI 助理团队

---

## 🎯 为什么需要多 Agent？

### 单一 AI 的局限

| 局限 | 表现 |
|------|------|
| **记忆容量有限** | 难以同时维护多个专业领域的信息 |
| **上下文混乱** | 不同类型的数据在同一上下文中互相干扰 |
| **缺乏持续追踪** | 无法长期独立跟踪特定领域的任务进度 |

### 多 Agent 的优势

> 多 Agent 架构的核心是 **"专业化分工 + 协作"**：每个 Agent 专注自己的领域，通过 `agentToAgent` 实现高效协作。

---

## 📋 5 个 Agent 角色设计

| Agent ID | 名称 | 职责 |
|----------|------|------|
| `aiboss` | 大总管 | 总协调助手，负责协调其他 Agent 和日常任务 |
| `ainews` | 资讯助理 | AI 行业资讯收集、每日 8:00 和 18:00 推送 |
| `aicontent` | 内容助理 | 文章写作、视频脚本、社交媒体内容 |
| `aicode` | 代码助理 | 代码审查、技术方案、问题解决 |
| `aitask` | 任务助理 | 任务跟踪、提醒、进度管理 |

---

## 📦 前置准备

### 1. 腾讯云轻量应用服务器

- **购买地址**：腾讯云 OpenClaw 镜像
- **建议配置**：前期选最低配 2 核 2GB 即可
- **镜像选择**：OpenClaw

### 2. 飞书开发者账号

- 注册飞书开放平台账号：open.feishu.cn
- 创建企业（如果没有的话）

---

## 🤖 配置飞书多应用

### 步骤 1：创建 5 个飞书应用

1. 登录飞书开放平台
2. 进入"应用管理" → "创建应用" → "自建应用"
3. 填写应用信息：
   - 应用名称：如"AI 大总管"、"AI 资讯助理"
   - 应用描述：简短描述该 Agent 的职责
   - 应用图标：建议每个 Agent 用不同图标
4. 点击"创建"

**重复以上步骤，创建 5 个独立应用。**

### 步骤 2：获取应用凭证

对于每个应用，获取并记录以下信息（在"凭证与基础信息"页面）：

- **App ID**：如 `cli_xxx`
- **App Secret**：如 `i63Qyyyyy`

**建议表格整理**：

| Agent | 应用名称 | App ID | App Secret |
|-------|----------|--------|------------|
| aiboss | AI 大总管 | cli_xxx | your_secret |
| aicontent | AI 内容助理 | cli_xxx | your_secret |
| ainews | AI 资讯助理 | cli_xxx | your_secret |
| aicode | AI 代码助理 | cli_xxx | your_secret |
| aitask | AI 任务助理 | cli_xxx | your_secret |

### 步骤 3：配置应用能力（⚠️ 关键步骤）

#### 3.1 开启机器人能力

1. 进入应用详情 → "权限管理"
2. 开启"机器人能力"
3. 添加必要的权限：
   - `im:message` (接收消息)
   - `im:message:group_at_msg` (接收群组 @ 消息)
   - `im:message:send_as_bot` (发送消息)

#### 3.2 配置事件订阅（⚠️ 长连接）

1. 进入"事件订阅"
2. 选择"长连接"模式
3. 启用以下事件：
   - `im.message.receive_v1` (接收消息)
   - `im.message.message_read_v1` (消息已读)

> ⚠️ **重点**：必须配置"长连接事件订阅"，否则 Bot 无法上线！

#### 3.3 发布应用

1. 进入"版本管理与发布"
2. 创建新版本
3. 填写更新日志
4. 发布（可选择"开发版"或"正式版"）

---

## ⚙️ OpenClaw 多 Agent 配置

### 步骤 1：创建独立 Workspace

```bash
# 创建 5 个独立工作区
mkdir -p /root/.openclaw/workspace-boss
mkdir -p /root/.openclaw/workspace-news
mkdir -p /root/.openclaw/workspace-content
mkdir -p /root/.openclaw/workspace-code
mkdir -p /root/.openclaw/workspace-task
```

### 步骤 2：编辑 openclaw.json

```bash
vi /root/.openclaw/openclaw.json
```

#### 2.1 配置 agents 数组

```json
{
  "agents": {
    "list": [
      {
        "id": "aiboss",
        "default": true,
        "name": "aiboss",
        "workspace": "/root/.openclaw/workspace-boss",
        "model": { "primary": "glmcode/glm-4.7" }
      },
      {
        "id": "aicontent",
        "name": "aicontent",
        "workspace": "/root/.openclaw/workspace-aicontent",
        "model": { "primary": "glmcode/glm-4.7" }
      },
      {
        "id": "ainews",
        "name": "ainews",
        "workspace": "/root/.openclaw/workspace-ainews",
        "model": { "primary": "glmcode/glm-4.7" }
      },
      {
        "id": "aicode",
        "name": "aicode",
        "workspace": "/root/.openclaw/workspace-aicode",
        "model": { "primary": "glmcode/glm-4.7" }
      },
      {
        "id": "aitask",
        "name": "aitask",
        "workspace": "/root/.openclaw/workspace-aitask",
        "model": { "primary": "glmcode/glm-4.7" }
      }
    ]
  }
}
```

> 💡 说明：
> - `id`: Agent 的唯一标识
> - `default`: 标记默认 Agent（只有一个为 true）
> - `workspace`: 独立工作目录路径
> - `model.primary`: 使用的模型

#### 2.2 配置飞书多账户

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "accounts": {
        "aiboss": { "appId": "cli_xxx", "appSecret": "your_secret" },
        "aicontent": { "appId": "cli_xxx", "appSecret": "your_secret" },
        "ainews": { "appId": "cli_xxx", "appSecret": "your_secret" },
        "aitask": { "appId": "cli_xxx", "appSecret": "your_secret" },
        "aicode": { "appId": "cli_xxx", "appSecret": "your_secret" }
      }
    }
  }
}
```

> 💡 注意：`accounts` 的 key（如 `aiboss`）要与 Agent ID 对应

#### 2.3 配置 bindings 路由

```json
{
  "bindings": [
    { "match": { "channel": "feishu", "accountId": "aiboss" }, "agentId": "aiboss" },
    { "match": { "channel": "feishu", "accountId": "aicontent" }, "agentId": "aicontent" },
    { "match": { "channel": "feishu", "accountId": "ainews" }, "agentId": "ainews" },
    { "match": { "channel": "feishu", "accountId": "aicode" }, "agentId": "aicode" },
    { "match": { "channel": "feishu", "accountId": "aitask" }, "agentId": "aitask" }
  ]
}
```

> 💡 说明：
> - `match.channel`: 固定为 `"feishu"`
> - `match.accountId`: 对应飞书账户的 key
> - `agentId`: 消息路由到哪个 Agent

#### 2.4 开启 agentToAgent 通信

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["aiboss", "aicontent", "ainews", "aicode", "aitask"]
    }
  }
}
```

### 步骤 3：为每个 Agent 创建核心文件

#### 3.1 IDENTITY.md（身份信息）

```bash
cat > /root/.openclaw/workspace-boss/IDENTITY.md << 'EOF'
# IDENTITY.md - AIBoss

- **Name**: AIBoss
- **Role**: 大总管，团队协调者
- **Emoji**: 👔
- **Vibe**: 专业、高效、有条理
EOF
```

#### 3.2 SOUL.md（人设和行为准则）

```bash
cat > /root/.openclaw/workspace-boss/SOUL.md << 'EOF'
# SOUL.md - AIBoss

你是 AIBoss，大总管，负责团队协调和任务管理。

## 核心职责

- 团队协调和任务分发
- 项目进度跟踪
- 跨 Agent 协作调度

## 工作流程

1. 接收用户需求
2. 分析任务类型
3. 分发给对应的 Agent
4. 跟踪任务进度
5. 汇总结果给用户

## 协作方式

需要其他 Agent 协作时，使用 sessions_send 工具：

- 需要最新资讯？→ sessions_send(agentId="ainews", message="...")
- 需要内容产出？→ sessions_send(agentId="aicontent", message="...")
- 需要技术支持？→ sessions_send(agentId="aicode", message="...")
- 需要任务提醒？→ sessions_send(agentId="aitask", message="...")
EOF
```

#### 3.3 AGENTS.md（团队成员通讯录）

```bash
cat > /root/.openclaw/workspace-boss/AGENTS.md << 'EOF'
# AGENTS.md - 团队成员

- **AIBoss** (你) - 大总管
  - agentId: aiboss
  - 职责：团队协调、任务分发

- **AINews** - 资讯助理
  - agentId: ainews
  - 职责：AI 行业资讯收集、每日推送

- **AIContent** - 内容助理
  - agentId: aicontent
  - 职责：文章写作、视频脚本、社交媒体内容

- **AICode** - 代码助理
  - agentId: aicode
  - 职责：代码审查、技术方案、问题解决

- **AITask** - 任务助理
  - agentId: aitask
  - 职责：任务跟踪、提醒、进度管理
EOF
```

#### 3.4 MEMORY.md（长期记忆）

```bash
cat > /root/.openclaw/workspace-boss/MEMORY.md << 'EOF'
# MEMORY.md - AIBoss 长期记忆

## 项目记录

### 2026-02-23
- 完成飞书多 Agent 系统搭建
- 5 个 Agent 全部上线

## 重要决策

- 使用 OpenClaw 框架
- 部署在腾讯云服务器
- 飞书作为主要沟通渠道
EOF
```

**重复以上步骤，为其他 4 个 Agent 创建对应文件。**

### 步骤 4：重启 OpenClaw Gateway

```bash
# 重启 Gateway 使配置生效
openclaw gateway restart

# 查看 Gateway 状态
openclaw gateway status

# 查看日志（确认 Agent 启动）
openclaw logs --follow
```

---

## 🐛 踩坑与解决方案

### 坑 1：Bot 无法上线

**症状**：飞书应用配置完成，但 Bot 状态一直是离线。

**原因**：未配置"长连接事件订阅"。

**解决方案**：
1. 进入飞书开放平台 → 应用详情 → "事件订阅"
2. 选择"长连接"模式
3. 启用 `im.message.receive_v1` 事件
4. 保存并发布应用

> 💡 提示：这是最容易遗漏的步骤，配置事件订阅后务必重新发布应用。

---

### 坑 2：Agent 无法协作

**症状**：Agent 之间无法通信，协作请求失败。

**原因**：未配置 `AGENTS.md` 团队成员列表，Agent 不知道彼此的存在。

**解决方案**：在每个 Agent 的 workspace 中创建 `AGENTS.md`，列出所有团队成员。

---

### 坑 3：Workspace 数据混乱

**症状**：不同 Agent 的数据出现混乱或覆盖。

**原因**：多个 Agent 共用同一个 workspace 路径。

**解决方案**：确保每个 Agent 的 `workspace` 路径独立且不重复。

---

### 坑 4：消息路由错误

**症状**：发给某个 Agent 的消息被路由到了其他 Agent。

**原因**：`bindings` 配置中的 `accountId` 和 `agentId` 不匹配。

**解决方案**：检查 `bindings` 数组，确保每个飞书账户的 `accountId` 正确对应到目标 `agentId`。

---

### 坑 5：ID 大小写导致配置失效

**症状**：配置完成后，Agent 无法启动或消息无法路由。

**原因**：agent、channels 等 ID 定义使用了大小写混合（如 `AIContent`、`aIBoss`），OpenClaw 不能正常处理。

**解决方案**：确保所有 ID 定义都是**纯小写字母**：

```json
// ✅ 正确 - 全小写
{ "id": "aiboss", "name": "aiboss" }

// ❌ 错误 - 大小写混合
{ "id": "AIBoss", "name": "AIContent" }
```

**影响范围**：
- `agents.list[].id` - Agent ID 必须小写
- `channels.feishu.accounts` 的 key - 账户标识必须小写
- `bindings[].agentId` - Agent ID 引用必须小写

**检查清单**：
- [ ] 所有 Agent ID 都是纯小写（如 `aiboss`、`aicontent`）
- [ ] 飞书账户标识都是纯小写（如 `aiboss`、`ainews`）
- [ ] bindings 中的 `accountId` 和 `agentId` 都是纯小写

---

## ✅ 验证与测试

### 1. 检查 Agent 状态

```bash
openclaw status
```

期望输出：
```
Agent: aiboss    Status: running ✅
Agent: aicontent Status: running ✅
Agent: ainews    Status: running ✅
Agent: aicode    Status: running ✅
Agent: aitask    Status: running ✅
```

### 2. 单 Agent 测试

**首次使用需要配对**：

第一次向 Bot 发送消息时，会收到配对提示：
```
OpenClaw: access not configured.
Your Feishu user id: ou_xxx
Pairing code: xxxx
Ask the bot owner to approve with:
openclaw pairing approve feishu xxxx
```

在服务器上执行批准命令：
```bash
openclaw pairing approve feishu xxxx
```

**测试消息**：

| Bot | 测试消息 |
|-----|----------|
| AIBoss | "你好，你是谁？" |
| AINews | "今天有什么 AI 资讯？" |
| AIContent | "帮我写一个文章大纲" |
| AICode | "这段代码有什么问题？" |
| AITask | "创建一个任务提醒" |

### 3. Agent 间协作测试

在飞书中 @AIBoss，让它调用其他 Agent：

```
@AIBoss 帮我让 AINews 推送今天的 AI 资讯
```

AIBoss 应该能够：
1. 接收你的指令
2. 调用 `sessions_send` 联系 AINews
3. AINews 执行并返回结果
4. AIBoss 汇总结果给你

### 4. 检查清单

- [ ] 5 个飞书应用全部发布
- [ ] OpenClaw Gateway 运行正常
- [ ] 5 个 Agent 状态全部显示 `running`
- [ ] 单独向每个 Bot 发送测试消息
- [ ] 测试 Agent 间协作

---

## 📎 附录：完整 openclaw.json 示例

```json
{
  "meta": { "lastTouchedVersion": "2026.2.9", "lastTouchedAt": "2026-02-21T06:24:18.113Z" },
  "tools": {
    "agentToAgent": { "enabled": true, "allow": ["aiboss", "aicontent", "ainews", "aicode", "aitask"] }
  },
  "agents": {
    "list": [
      { "id": "aiboss", "default": true, "name": "aiboss", "workspace": "/root/.openclaw/workspace-boss", "model": { "primary": "glmcode/glm-4.7" } },
      { "id": "aicontent", "name": "aicontent", "workspace": "/root/.openclaw/workspace-aicontent", "model": { "primary": "glmcode/glm-4.7" } },
      { "id": "ainews", "name": "ainews", "workspace": "/root/.openclaw/workspace-ainews", "model": { "primary": "glmcode/glm-4.7" } },
      { "id": "aicode", "name": "aicode", "workspace": "/root/.openclaw/workspace-aicode", "model": { "primary": "glmcode/glm-4.7" } },
      { "id": "aitask", "name": "aitask", "workspace": "/root/.openclaw/workspace-aitask", "model": { "primary": "glmcode/glm-4.7" } }
    ]
  },
  "bindings": [
    { "match": { "channel": "feishu", "accountId": "aiboss" }, "agentId": "aiboss" },
    { "match": { "channel": "feishu", "accountId": "aicontent" }, "agentId": "aicontent" },
    { "match": { "channel": "feishu", "accountId": "ainews" }, "agentId": "ainews" },
    { "match": { "channel": "feishu", "accountId": "aicode" }, "agentId": "aicode" },
    { "match": { "channel": "feishu", "accountId": "aitask" }, "agentId": "aitask" }
  ],
  "channels": {
    "feishu": {
      "enabled": true,
      "accounts": {
        "aiboss": { "appId": "cli_xx", "appSecret": "***" },
        "aicontent": { "appId": "cli_xx", "appSecret": "***" },
        "ainews": { "appId": "cli_xx", "appSecret": "***" },
        "aitask": { "appId": "cli_xx", "appSecret": "***" },
        "aicode": { "appId": "cli_xx", "appSecret": "***" }
      }
    }
  }
}
```

---

**原文**：[掘金 - 用 OpenClaw + 飞书，快速搭建 5 个可协作的 AI 助理团队](https://juejin.cn/post/7609481369001492499)
**归档**：2026-03-06
