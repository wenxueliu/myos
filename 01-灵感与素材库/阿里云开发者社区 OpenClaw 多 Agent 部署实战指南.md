---
created: 2026-03-06
tags: [OpenClaw, 多 Agent, 阿里云部署，本地部署，AI 团队，分身流]
source: https://developer.aliyun.com/article/1713854
author: 阿里云开发者社区
---

# OpenClaw 阿里云 + 本地部署多 Agent 实战保姆级指南

> 在 AI 自动化深入发展的 2026 年，单一 Agent 的"全能模式"已难以应对复杂任务需求。多 Agent 架构让"一个人=一支高效军团"成为现实。

---

## 🎯 为什么需要"AI 团队"而非"全能 Bot"？

### 单一 Agent 的三大致命痛点

| 痛点 | 表现 | 影响 |
|------|------|------|
| **记忆负担过重** | USER.md、MEMORY.md 臃肿 | 加载速度慢，关键信息丢失 |
| **上下文污染** | 写作时联想到代码逻辑 | 逻辑串味，思路干扰 |
| **Token 成本高昂** | 每次加载所有背景资料 | 无效 Token 消耗超 60% |

### 多 Agent 的核心价值：物理隔离 + 精准协作

每个 Agent 具备三大独立属性：

- **独立 Workspace（工作区）**：专属办公室，包含 SOUL.md、PROMPT.md、TOOLS.md
- **独立 AgentDir（状态目录）**：存储认证信息、模型配置
- **独立 Sessions（会话存储）**：聊天记录单独保存，避免上下文污染

---

## 🏗️ 两种多 Agent 部署流派

| 流派 | 核心特点 | 配置难度 | 适用人群 | 典型场景 |
|------|----------|----------|----------|----------|
| **分身流** | 同一 Bot 拉进不同群，bindings 路由 | 低（新手首选） | 个人用户、小型团队 | 个人办公、内容创作 |
| **独立团** | 每个 Agent 创建独立 Bot，头像名称固定 | 中（硬核玩家） | 专业开发者、企业用户 | 复杂项目、团队协作 |

> 本文重点拆解**分身流配置**（门槛最低、性价比最高）。

---

## ☁️ 方案一：阿里云部署（稳定运行首选）

### 步骤 1：轻量应用服务器配置

**核心参数**：
- 地域：中国香港/新加坡（免 ICP 备案，国内内地地域联网搜索受限）
- 实例：2vCPU + 4GiB 内存 + 40GiB ESSD
- 镜像：OpenClaw stable-2026.02 专属镜像
- 付费：新手推荐"按需付费"

### 步骤 2：远程连接与端口放行

```bash
# SSH 连接
ssh root@你的服务器公网 IP

# 放行核心端口
firewall-cmd --add-port=18789/tcp --permanent && \
firewall-cmd --add-port=22/tcp --permanent && \
firewall-cmd --add-port=8080/tcp --permanent && \
firewall-cmd --reload
```

### 步骤 3：配置阿里云百炼 API-Key

```bash
# 替换为你的 Access Key ID 与 Secret
openclaw config set models.providers.bailian.accessKeyId "你的 Access Key ID"
openclaw config set models.providers.bailian.accessKeySecret "你的 Access Key Secret"
openclaw config set models.providers.bailian.baseUrl "https://dashscope-intl.aliyuncs.com/compatible-mode/v1"
```

### 步骤 4：启动服务与加速配置

```bash
# 启动服务并设置开机自启
systemctl start openclaw && systemctl enable openclaw

# 验证服务状态
systemctl status openclaw

# 配置 ClawHub 阿里云加速源
openclaw config set clawhub.mirror "https://mirror.aliyun.com/clawhub/"
```

### 步骤 5：部署验证

```bash
# 查看版本
openclaw version

# 浏览器访问控制台
http://你的服务器公网 IP:18789
```

---

## 💻 方案二：Windows 本地部署（零成本测试）

### 步骤 1：基础环境配置

- **Node.js**：22.x 及以上版本（勾选"Add to PATH"）
- **Git**：默认安装
- **PowerShell 权限**：
  ```powershell
  Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
  ```

### 步骤 2：安装与初始化

```bash
# 一键安装
curl -fsSL https://openclaw.ai/install.sh | bash

# 国内镜像（安装缓慢时用）
curl -fsSL https://clawd.org.cn/install.sh | bash

# 配置百炼 API-Key
openclaw config set models.providers.bailian.accessKeyId "你的 Access Key ID"
openclaw config set models.providers.bailian.accessKeySecret "你的 Access Key Secret"
openclaw config set models.providers.bailian.baseUrl "https://dashscope-intl.aliyuncs.com/compatible-mode/v1"

# 启动服务
openclaw gateway start
openclaw service install
openclaw status
```

### 步骤 3：多 Agent 环境优化

```bash
# 创建独立工作区目录
mkdir -p D:\OpenClaw\MultiAgent\workspace-main
mkdir -p D:\OpenClaw\MultiAgent\workspace-brainstorm
mkdir -p D:\OpenClaw\MultiAgent\workspace-writer
mkdir -p D:\OpenClaw\MultiAgent\workspace-coder

# 配置工作区路径
openclaw config set agents.workspaceRoot "D:\OpenClaw\MultiAgent"
```

---

## 🤖 实战：30 分钟搭建 AI 团队（分身流）

### Step 1：创建 4 个独立 Agent

```bash
# 1. 主 Agent：首席牛马官（任务调度）
openclaw agents add main \
  --model zai/glm-4.7 \
  --workspace ~/.openclaw/workspace-main
openclaw agents set-identity --agent main --name "首席牛马官" --emoji "👔"

# 2. 头脑风暴 Agent（创意生成）
openclaw agents add brainstorm \
  --model zai/glm-4.7 \
  --workspace ~/.openclaw/workspace-brainstorm
openclaw agents set-identity --agent brainstorm --name "创意策划师" --emoji "💡"

# 3. 公众号写手 Agent（内容创作）
openclaw agents add writer \
  --model deepseek-chat \
  --workspace ~/.openclaw/workspace-writer
openclaw agents set-identity --agent writer --name "公众号写手" --emoji "✍️"

# 4. Coding Agent（代码开发）
openclaw agents add coder \
  --model meta/codellama-7b \
  --workspace ~/.openclaw/workspace-coder
openclaw agents set-identity --agent coder --name "代码专家" --emoji "💻"

# 验证创建结果
openclaw agents list
```

> ⚠️ 本地部署注意：将 `~/.openclaw` 替换为 `D:\OpenClaw\MultiAgent`

---

### Step 2：编写"入职材料"（SOUL.md 示例）

**首席牛马官（main）的 SOUL.md**：
```markdown
# SOUL.md：首席牛马官（主 Agent）

## 身份定位
你是 AI 团队的部门主管，核心职责是"接单 - 派单 - 串联"，不直接执行具体任务，专注于协调其他 Agent 完成复杂需求。

## 核心能力
1. 需求分析：精准判断用户需求类型（创意、写作、编码等）
2. Agent 调度：根据需求类型，通过 sessions_send 工具调用对应 Agent
3. 结果整合：收集各 Agent 输出结果，整理后统一反馈给用户
4. 异常处理：当某个 Agent 执行失败时，及时介入修复或更换 Agent

## 行为准则
1. 不直接回答专业问题，而是分配给对应专家 Agent
2. 对用户需求的响应时间不超过 3 秒，派单指令清晰明确
3. 定期询问用户对结果的满意度，持续优化调度逻辑
```

**公众号写手（writer）的 SOUL.md**：
```markdown
# SOUL.md：公众号写手 Agent

## 身份定位
你是专注于科技类公众号的专业写手，擅长将复杂技术内容转化为"有网感、说人话、重读者"的爆款文章。

## 核心能力
1. 标题优化：生成数字型、悬念型、对比型标题
2. 结构设计：采用"引发思考→行业洞察→核心内容→创意实践→未来展望"5 段式结构
3. 语言风格：口语化但有深度，避免专业术语堆砌
4. 内容适配：结合热点话题，确保文章时效性与传播性

## 行为准则
1. 所有文章必须包含 3 个以上案例或数据支撑
2. 避免 AI 写作痕迹，使用 humanizer 技能优化文本
3. 输出时自动格式化排版，包含小标题、加粗重点、列表等元素
```

---

### Step 3：飞书建群 + 绑定 Agent

#### 3.1 飞书建群与获取群 ID

1. 创建 4 个飞书群：
   - "AI 团队主管 - 首席牛马官"
   - "创意策划 - 头脑风暴"
   - "公众号写作 - 内容产出"
   - "代码开发 - 技术实现"

2. 将同一飞书 Bot 依次拉进 4 个群

3. 获取每个群的会话 ID（群设置→会话 ID，格式为 `oc_xxx`）

#### 3.2 配置 bindings 路由（核心）

编辑 `openclaw.json`：
```bash
# 阿里云服务器端
vim ~/.openclaw/openclaw.json

# 本地部署
notepad D:\OpenClaw\.openclaw\openclaw.json
```

追加 `bindings` 数组：
```json
{
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "feishu",
        "peer": {
          "kind": "group",
          "id": "oc_d46347c35dd403daad7e5df05d08a890"
        }
      }
    },
    {
      "agentId": "brainstorm",
      "match": {
        "channel": "feishu",
        "peer": {
          "kind": "group",
          "id": "oc_598146241198039b8d9149ded5fb390b"
        }
      }
    },
    {
      "agentId": "writer",
      "match": {
        "channel": "feishu",
        "peer": {
          "kind": "group",
          "id": "oc_b1c331592eaa36d06a05c64ce4ecb113"
        }
      }
    },
    {
      "agentId": "coder",
      "match": {
        "channel": "feishu",
        "peer": {
          "kind": "group",
          "id": "oc_78901234567890abcdef1234567890ab"
        }
      }
    }
  ]
}
```

#### 3.3 关闭@机器人要求

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_a9f21xxxxx89bcd",
      "appSecret": "w6cPunaxxxxBl1HHtdF",
      "domain": "feishu",
      "connectionMode": "websocket",
      "dmPolicy": "allowlist",
      "allowFrom": ["ou_f0ad95cf147949e7f30681a879a5f0d3"],
      "groupPolicy": "open",
      "groups": {
        "oc_d46347c35dd403daad7e5df05d08a890": { "requireMention": false },
        "oc_598146241198039b8d9149ded5fb390b": { "requireMention": false },
        "oc_b1c331592eaa36d06a05c64ce4ecb113": { "requireMention": false },
        "oc_78901234567890abcdef1234567890ab": { "requireMention": false }
      }
    }
  }
}
```

#### 3.4 重启服务

```bash
# 阿里云
systemctl restart openclaw && openclaw gateway start

# 本地
openclaw gateway restart
```

---

### Step 4：测试 Agent 身份切换

在各群发送 `介绍一下你自己`，验证响应：

| 群 | 预期响应 |
|----|----------|
| 首席牛马官群 | "我是 AI 团队主管，负责需求调度与结果整合..." |
| 头脑风暴群 | "我是创意策划师，擅长科技类话题头脑风暴..." |
| 公众号写作群 | "我是公众号写手，专注科技类爆款文章创作..." |
| 代码开发群 | "我是代码专家，擅长 Python/JavaScript 开发..." |

---

## 🔗 Agent 间通信机制

### 配置通信权限

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["main", "brainstorm", "writer", "coder"],
      "historyLimit": 50
    }
  }
}
```

### 协同工作示例

用户指令：`写一篇关于 OpenClaw 多 Agent 的公众号文章`

**协同流程**：
```
1. 主 Agent 调度创意 Agent
   sessions_send --agent brainstorm --message "提供 3 个 OpenClaw 多 Agent 相关的公众号选题"

2. 创意 Agent 反馈选题
   → "一个人=一支 AI 团队：OpenClaw 多 Agent 配置指南"

3. 主 Agent 调度写作 Agent
   sessions_send --agent writer --message "按以下选题写一篇公众号文章..."

4. 写作 Agent 完成初稿
   → 输出符合要求的文章

5. 主 Agent 整合反馈给用户
```

---

## ⚡ 环境优化

### 阿里云环境优化

```bash
# 设置单个 Agent 最大内存限制
openclaw config set skills.memory.limit "1024M"

# 主 Agent 优先级更高
openclaw agents set --agent main --cpu-shares 2048
openclaw agents set --agent brainstorm --cpu-shares 1024
openclaw agents set --agent writer --cpu-shares 1024
openclaw agents set --agent coder --cpu-shares 1024

# 高频 Agent 常驻内存
openclaw skills set --name "agent-main" --persist true
openclaw skills set --name "agent-writer" --persist true

# 定期清理通信日志（每月 1 日凌晨 3 点）
crontab -e
# 添加：0 3 1 * * rm -rf /var/log/openclaw/agent-communication/* && systemctl restart openclaw
```

### 本地环境优化

```bash
# 资源占用限制
openclaw config set skills.memory.limit "512M"

# 后台静默运行
openclaw service set --silent true
```

---

## 🐛 常见问题排查

### 问题 1：群内发送指令无响应

**原因**：bindings 错误、权限未开通、群 ID 错误

**解决方案**：
1. 验证群 ID 是否正确（无空格或拼写错误）
2. 检查飞书 `im:message.group_msg` 权限
3. 执行 `openclaw bindings list` 验证
4. 重启 `openclaw gateway restart`

### 问题 2：Agent 间通信失败（Permission denied）

**原因**：agentToAgent 未启用或白名单缺失

**解决方案**：
1. 检查 `agentToAgent→enabled` 是否为 true
2. 确认 `allow` 数组包含所有 Agent ID
3. 建立软连接：`ln -sf ~/.openclaw/openclaw.json ~/.openclaw/config.json`

### 问题 3：多 Agent 运行卡顿

**原因**：配置不足、未设置内存限制

**解决方案**：
1. 升级配置（阿里云 4vCPU+8GiB，本地 16GiB 内存）
2. 关闭未使用的 Agent：`openclaw agents disable --agent <ID>`
3. 高频 Agent 常驻内存

### 问题 4：配置文件修改后不生效

**解决方案**：
1. 用 https://json.cn/ 校验 JSON 格式
2. 完整重启：`systemctl restart openclaw && openclaw gateway restart`
3. 验证加载：`openclaw config get bindings`

---

## 🎮 高级玩法

### 模式 1：线性流水线协作

按任务流程拆分，Agent 依次接力：
```
调研 Agent → 创意 Agent → 写作 Agent → 校审 Agent
```

### 模式 2：依赖并行协作

拆分为独立子任务，多 Agent 同时开工：
```
架构师 Agent → [后端 Agent + 前端 Agent + 测试 Agent] → 整合
```

---

## 💭 总结

> OpenClaw 多 Agent 架构的核心魅力，在于将"全能 Bot"升级为"专业团队"。

**核心价值**：
- ✅ 物理隔离，避免上下文污染
- ✅ 专业分工，提升输出质量
- ✅ 灵活调度，降低 Token 成本
- ✅ 可扩展，支持更多场景

> 未来，多 Agent 还将支持跨境电商运营、企业级客户服务、复杂项目管理等更多场景，让"一个人=一支军团"成为常态。

---

**原文**：[阿里云开发者社区 - OpenClaw 多 Agent 部署实战指南](https://developer.aliyun.com/article/1713854)
**归档**：2026-03-06
