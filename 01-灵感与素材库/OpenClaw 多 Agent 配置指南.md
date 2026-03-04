---
tags: [OpenClaw, 多 Agent, 配置教程，飞书，Agent 隔离]
created: 2026-03-04
source: https://cloud.tencent.com/developer/article/2632835
author: 嘉钰
status: 素材
---

# 玩转 OpenClaw｜如何配置多个相互独立的 Agent？

> 就像一个真正的公司或组织内的成员一样，大家各自处理不同类型的任务，分工明确、职能分化，这几乎是任何一个系统在复杂度达到一定程度后的必由之路。

## 核心问题

单个 Agent 处理多类任务会带来**严重的记忆负担**，极大拉高 Token 消耗，后期往往一个问题可以干掉数十万 Token。

**解决方案**: 创建多个相互隔离的 Agent，分别具有不同的人格、技能、记忆、工作空间，实现一个"AI 军团"。

## 原理解释

### 多 Agent 核心机制

1. **Agent 是隔离的"脑"** - 拥有独立 workspace（记忆、技能等）、独立 sessions
2. **路由规则** - 由配置里的 `bindings` 决定（最具体规则优先）
3. **模型配置** - 每个 Agent 可在 `agents.list[].model` 单独配置，运行时优先覆盖默认值

### 本文方案

- 默认主 Agent 继续处理全部私聊
- 每个群绑定到不同的群组 Agent
- 每个 Agent 的记忆、技能、模型均可完全隔离

## 配置步骤

### 前置条件

1. 已通过腾讯云 Lighthouse 一键部署 OpenClaw
2. 已完成飞书应用配置并接入 OpenClaw
3. 拥有飞书群组创建权限

### 步骤一：创建飞书群组并获取 ID

1. 打开飞书 → 创建群组
2. 群组设置 → 拉到最底端 → 复制会话 ID
3. 格式类似：`oc_5b6799cff4a754c15e5ff3025becc648`

### 步骤二：备份配置文件

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup.$(date +%Y%m%d_%H%M%S)
```

### 步骤三：配置新增 Agent

```bash
openclaw agents add --workspace /root/.openclaw/workspace-feishu-writer feishu-writer
```

**参数说明**:
- `--workspace`: 独立数据空间路径，建议格式 `/root/.openclaw/workspace-团队名`
- `新 Agent 的名称（ID）`: 推荐格式 `feishu-团队名 - 用途`
- `--model`: 可选，指定自定义模型

### 步骤四：配置飞书群组绑定

⚠️ **警告**: `openclaw config set --json bindings '[...]'` 会**整体替换**当前 bindings 配置！

**正确做法**:
1. 先导出当前绑定：`openclaw config get bindings`
2. 在原有数组基础上追加新绑定
3. 整体写回

```bash
openclaw config set --json bindings '[
  {
    "agentId": "feishu-writer",
    "match": {
      "channel": "feishu",
      "peer": {
        "kind": "group",
        "id": "oc_5b6799cff4a754c15e5ff3025becc648"
      }
    }
  }
]'
```

### 步骤五：配置飞书群组允许列表

```bash
openclaw config set channels.feishu.groupPolicy allowlist
openclaw config set --json channels.feishu.groupAllowFrom '["oc_5b6799cff4a754c15e5ff3025becc648"]'
```

### 步骤六：重启 Gateway

```bash
openclaw gateway restart
openclaw gateway status
```

### 步骤七：添加机器人到群组

飞书群组设置 → 群机器人 → 添加机器人

## 测试验证

### 测试 1：群组消息使用独立 Agent

在群组中提问：`你所在的工作空间路径是什么？`

✅ 预期：返回新 Agent 的 workspace 路径 `/root/.openclaw/workspace-feishu-writer`

### 测试 2：主 Agent 与独立 Agent 隔离

私聊提问：`你所在的工作空间路径是什么？`

✅ 预期：返回默认 workspace 路径 `/root/.openclaw/workspace`

### 测试 3：数据隔离验证

1. 群组中让机器人记住：`记住：我们团队的代号是"北极星"`
2. 私聊中询问：`你记得我们团队的代号是什么吗？`

✅ 预期：主 Agent 回答"我不知道"，说明数据完全隔离

## 进阶操作

### 为 Agent 配置自定义模型

```bash
openclaw agents add --workspace /root/.openclaw/workspace-feishu-writer --model kitcoding-openai/z-ai/glm4.7 feishu-writer
```

### 为多个群组配置不同 Agent

```bash
openclaw config set --json bindings '[
  {
    "agentId": "feishu-writer",
    "match": {
      "channel": "feishu",
      "peer": {
        "kind": "group",
        "id": "oc_e4dfb35658c81ce5100add124c3592a8"
      }
    }
  },
  {
    "agentId": "feishu-operations",
    "match": {
      "channel": "feishu",
      "peer": {
        "kind": "group",
        "id": "oc_9f23c45769d92df621Bee02350c4603d"
      }
    }
  }
]'
```

## 故障排查

| 问题 | 排查方法 |
|------|----------|
| Gateway 启动失败 | `cat ~/.openclaw/openclaw.json \| python3 -m json.tool` 检查 JSON 格式 |
| 群组中机器人无响应 | `openclaw logs --follow` 查看日志 |
| Agent 未起到隔离作用 | `openclaw agents list --bindings` 检查绑定配置 |
| 配置命令执行失败 | 使用在线 JSON 校验工具验证格式 |

## 常见问题

**Q：能否为一个群组配置多个 Agent？**

A：目前不支持。每个群组只能绑定一个 Agent，但可以为多个不同群组配置不同的 Agent。

**Q：独立 Agent 是否继承主 Agent 的配置？**

A：不继承。独立 Agent 拥有完全独立的 workspace、记忆和状态，但可以继承 `agents.defaults` 中的默认配置。

**Q：如何删除已配置的 Agent？**

```bash
openclaw agents delete feishu-writer
```

---

**原文**: 腾讯云开发者社区

**作者**: 嘉钰

**发布日期**: 2026-03-03

**专栏**: 玩转 Lighthouse

## 与 MyOS 的关联

1. **多 Agent 协作** - 可为不同内容创作环节配置专属 Agent（选题/写作/审核/发布）
2. **工作空间隔离** - 每个选题系列可用独立 workspace，避免上下文污染
3. **模型差异化** - 不同任务使用不同模型（创意用强模型， routine 用性价比模型）
4. **飞书群组集成** - 团队协作场景可直接复用本方案
