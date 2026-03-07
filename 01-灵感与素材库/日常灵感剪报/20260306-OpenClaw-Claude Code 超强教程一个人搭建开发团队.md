---
标题：OpenClaw + Claude Code 超强教程：一个人就能搭建完整的开发团队！
来源：微信公众号 - Datawhale
原文链接：https://mp.weixin.qq.com/s/gtxM1f3JmfXqDuxGIa3-ng
捕获日期：2026-03-06
标签：[OpenClaw, Claude Code, Codex, AI Agent, 多 Agent 协作，自动化工作流，独立开发]
---

# OpenClaw + Claude Code 超强教程：一个人就能搭建完整的开发团队！

**来源**：Datawhale  
**发布**：2026 年 2 月 25 日  
**作者**：Elvis（独立开发者）

---

## 核心数据：一个人，一天 94 次代码提交

过去 4 周的真实数据：

- **单日最高 94 次提交**（平均每天 50 次提交）
- **30 分钟内完成 7 个 PR**
- 从想法到上线的速度快到可以"当天交付客户需求"
- 这一天他还开了 3 个客户会议，**编辑器都没打开过**

**成本**：每月 $190（Claude $100 + Codex $90），新手起步 $20 就能跑起来。

> 作者的 Git 历史看起来像是"刚招了一个开发团队"，但实际上只有他一个人。关键变化是：他从"管理 Claude Code"变成了"管理一个 AI 管家，这个管家再去管理一群 Claude Code"。

---

## 核心架构：双层系统（编排层 + 执行层）

### 1 月之前 vs 1 月之后

| 时间 | 方式 | 问题 |
|------|------|------|
| 1 月之前 | 直接用 Codex 或 Claude Code 写代码 | 上下文窗口有限，业务信息缺失 |
| 1 月之后 | OpenClaw 作为编排层，调度 Codex/Claude Code/Gemini | 系统能自动完成几乎所有小到中等复杂度任务 |

### 为什么需要双层系统？

**Codex/Claude Code 的根本限制**：
- 上下文窗口是固定的，你只能二选一
- 塞满代码 → 没空间放业务上下文
- 塞满客户历史 → 没空间放代码库

**OpenClaw 改变等式**：

| 层级 | 角色 | 职责 |
|------|------|------|
| **编排层 (OpenClaw)** | 主厨 | 持有所有业务上下文，翻译 prompt，调度 Agent |
| **执行层 (Agent)** | 专业厨师 | 读写代码库、运行测试、提交代码、创建 PR |

> 通过上下文的专业化分工，而不是换更强的模型。

### 安全边界设计

- **编排层**：读取 Obsidian 会议记录、访问生产数据库（只读）、管理员 API 权限
- **执行层**：永远不会接触生产数据库，也不会看到客户敏感信息

---

## 完整工作流：从客户需求到 PR 合并的 8 个步骤

### 第 1 步：客户需求 → OpenClaw 理解并拆解

**神奇之处**：零解释成本。所有会议记录自动同步到 Obsidian，OpenClaw 已经读过了通话内容。

**OpenClaw 做的三件事**：
1. 给客户充值 — 用管理员 API 立即解除客户的使用限制
2. 拉取客户配置 — 从生产数据库（只读）获取客户现有的设置
3. 生成 prompt 并启动代理 — 把所有上下文打包，喂给 Codex

### 第 2 步：启动代理

```bash
# 创建 worktree + 启动代理
git worktree add ../feat-custom-templates -b feat/custom-templates origin/main
cd ../feat-custom-templates && pnpm install

# 启动 tmux 会话（可中途干预）
tmux new-session -d -s "codex-templates" \
  -c "/Users/elvis/Documents/GitHub/medialyst-worktrees/feat-custom-templates" \
  "$HOME/.codex-agent/run-agent.sh templates gpt-5.3-codex high"
```

**为什么用 tmux？** 因为可以中途干预：

```bash
# 代理方向错了
tmux send-keys -t codex-templates "停一下。先做 API 层，别管 UI。" Enter

# 代理需要更多上下文
tmux send-keys -t codex-templates "类型定义在 src/types/template.ts，用那个。" Enter
```

### 第 3 步：自动监控

一个 cron 任务每 10 分钟检查一次所有代理的状态：

- tmux 会话还活着吗？
- 有没有创建 PR?
- CI 状态如何？
- 如果失败了，是否需要重试？（最多重试 3 次）

> 这个监控脚本是 100% 确定性的，非常省 token，只在需要人工介入时才会通知作者。

### 第 4 步：Agent 创建 PR

Agent 写完代码，提交，推送，然后用 `gh pr create --fill` 创建 PR。

**"完成"的定义**：
- ✅ PR 已创建
- ✅ 分支已同步到 main（没有冲突）
- ✅ CI 通过（lint、类型检查、单元测试、E2E 测试）
- ✅ Codex reviewer 通过
- ✅ Claude Code reviewer 通过
- ✅ Gemini reviewer 通过
- ✅ 如果有 UI 改动，必须包含截图

### 第 5 步：自动化 Code Review

| 审查者 | 特点 | 使用策略 |
|--------|------|----------|
| **Codex Reviewer** | 最靠谱，擅长发现边界情况 | 主力审查者 |
| **Gemini Code Assist** | 免费，能发现安全问题 | 不用白不用 |
| **Claude Code Reviewer** | 过度谨慎，建议多为过度设计 | 除非标记"critical"，否则跳过 |

### 第 6 步：自动化测试

CI 管道会跑：
- Lint 和 TypeScript 检查
- 单元测试
- E2E 测试
- Playwright 测试（在和生产环境一模一样的预览环境里跑）

**新规则**：如果 PR 改了 UI，必须在描述里附上截图，否则 CI 直接失败。

### 第 7 步：人工 Review

作者收到 Telegram 通知："PR #341 准备好了，可以 review。"

这时候：
- CI 全绿
- 三个 AI 审查者都批准了
- 截图展示了 UI 变化
- 所有边界情况都在 review 评论里记录了

> 作者的 review 只需要 5-10 分钟。很多 PR 他甚至不看代码，只看截图就直接合并了。

### 第 8 步：合并

PR 合并。每天有个 cron 任务清理孤立的 worktree 和任务记录。

**完整流程走完**：从客户需求到代码上线，可能只用了 1-2 小时，而作者的实际投入可能只有 10 分钟。

---

## 三个让系统更聪明的机制

### 机制 1：改进版 Ralph Loop — 不只是重复，而是学习

> 当 Agent 失败时，Zoe（OpenClaw）不会用同样的 prompt 重启。她会带着完整的业务上下文，分析失败原因，然后重写 prompt。

**坏例子（静态 prompt）**：
```
"实现自定义模板功能"
```

**好例子（动态调整）**：
```
"停。客户要的是 X，不是 Y。这是他们在会议里的原话：
'我们希望保存现有配置，而不是从头创建新的。'
重点做配置复用，不要做新建流程。"
```

**Zoe 主动找活干**：
- **早上**：扫描 Sentry → 发现 4 个新错误 → 启动 4 个 Agent 去调查和修复
- **会议后**：扫描会议记录 → 发现 3 个客户提到的功能需求 → 启动 3 个 Codex
- **晚上**：扫描 git log → 启动 Claude Code 更新 changelog 和客户文档

> 成功的模式会被记录下来。时间越长，Zoe 写的 prompt 越好，因为她记得什么能成功。

### 机制 2：Agent 选择策略 — 不同任务找不同的专家

| Agent | 擅长领域 | 占比 |
|-------|----------|------|
| **Codex (gpt-5.3-codex)** | 后端逻辑、复杂 bug、多文件重构 | 90% |
| **Claude Code (claude-opus-4.5)** | 前端工作、权限问题少 | 速度型 |
| **Gemini** | 设计师，有设计审美 | UI 设计 |

> Gemini 设计，Claude 建造。

### 机制 3：瓶颈在哪？RAM

每个 Agent 需要：
- 自己的 worktree
- 自己的 node_modules
- 运行构建、类型检查、测试

**5 个 Agent 同时跑** = 5 个并行的 TypeScript 编译器 + 5 个测试运行器 + 5 套依赖加载到内存。

作者的 Mac Mini (16GB RAM) 最多同时跑 4-5 个 Agent。他买了一台 Mac Studio M4 Max (128GB RAM, $3500)。

---

## 如何搭建：从零到运行只需 10 分钟

**最简单的方法**：
把这整篇文章复制给 OpenClaw，告诉它："按照这个架构，给我的代码库实现一套 Agent 集群系统。"

**你需要准备**：
- OpenClaw 账号
- Codex 和/或 Claude Code 的 API 访问
- 一个 git 仓库
- （可选）Obsidian 用于存储业务上下文

---

## 核心金句

> 关键变化是：他从"管理 Claude Code"变成了"管理一个 AI 管家，这个管家再去管理一群 Claude Code"。

> 通过上下文的专业化分工，而不是换更强的模型。

> 这个监控脚本是 100% 确定性的，非常省 token，只在需要人工介入时才会通知作者。

> 作者的 review 只需要 5-10 分钟。很多 PR 他甚至不看代码，只看截图就直接合并了。

> 当 Agent 失败时，Zoe 不会用同样的 prompt 重启。她会带着完整的业务上下文，分析失败原因，然后重写 prompt。

> 成功的模式会被记录下来。时间越长，Zoe 写的 prompt 越好，因为她记得什么能成功。

> 下一代创业者不会雇 10 个人去做一个人加一套系统就能做的事。他们会这样构建——保持小规模，快速行动，每天发布。

> 2026：一个人的百万美元公司。杠杆是巨大的，属于那些理解如何构建递归自我改进 AI 系统的人。

---

## 可提炼选题

1. **一个人的百万美元公司 -AI Agent 驱动的独立开发模式** - 从 0 到 1 搭建自动化开发系统
2. **OpenClaw 编排层设计指南** - 如何设计业务上下文与 Agent 调度系统
3. **AI Agent 自动化工作流实战** - 从需求到 PR 的 8 步流程详解
4. **多 Agent 协作的成本与性能优化** - RAM 瓶颈、Agent 选择策略、监控设计
