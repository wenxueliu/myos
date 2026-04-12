---
title: 下一代Agent架构——Pi Agent Core 设计逻辑深度解析
date: 2026-03-28
source: https://zhuanlan.zhihu.com/p/2004665077618458930
tags:
  - Pi Agent
  - Agent架构
  - 极简主义
  - Mario Zechner
关联参考:
  - https://github.com/badlogic/pi-mono
状态: 灵感待整理
---

> **核心哲学**: *"An autonomous agent is just an LLM + tools + a loop."*
> — Mario Zechner

## 一、Pi 的反直觉立场

在当前 Agent 框架生态中，大多数项目在做**加法**：更多工具、更长提示词、更复杂的规划链、更多子 Agent。Pi 的创作者 Mario Zechner 认为这是一条弯路。他的核心论点是：

> *"前沿模型已经被 RL 训练得足够理解'编码 Agent'是什么。你不需要 10,000 token 的系统提示词。"*

| 对比维度 | Claude Code | Codex | Pi |
| --- | --- | --- | --- |
| 系统提示词 | 约 10,000+ tokens | 适中 | < 1,000 tokens |
| 内置工具数 | 数十个 | 适中 | 4 个 (read/write/edit/bash) |
| Plan Mode | 有（黑盒子 Agent） | 有 | 无（用文件代替） |
| MCP 支持 | 有 | 有 | 无（用 CLI 工具代替） |
| Sub-Agent | 有（不可观测） | — | 无（通过 bash 自我调用） |

## 二、架构分层：5 个文件构成的运行时

`pi-agent-core` 的全部源码只有 **5 个文件、约 1,500 行代码**。

## 三、类型系统：少即是多

### 3.1 AgentMessage — 应用状态与模型上下文的分离

```
// 空接口 —— 通过 Declaration Merging 扩展
export interface CustomAgentMessages {
    // Empty by default - apps extend via declaration merging
}

// AgentMessage = LLM 消息 + 自定义消息
export type AgentMessage = Message | CustomAgentMessages[keyof CustomAgentMessages];
```

**为什么不直接用 LLM 的** **`Message`** **类型？** 因为真实应用中存在大量"非 LLM 消息"：

| 场景 | 消息类型 | 交给 LLM？ |
| --- | --- | --- |
| 用户提问 | UserMessage | ✅ |
| Agent 回复 | AssistantMessage | ✅ |
| 工具结果 | ToolResultMessage | ✅ |
| UI 通知 | { role: "notification" } | ❌ 过滤掉 |
| 文件变更事件 | { role: "artifact" } | ❌ 过滤掉 |
| 会话分支标记 | { role: "branch" } | ❌ 过滤掉 |

这就是 Zechner 所称的"**最晚转换**"（late conversion）策略。

### 3.2 AgentLoopConfig — 可插拔的行为注入

注意这里**没有** `maxSteps`、`maxTokens`、`temperature` 等"常见"配置项。循环只关心两件事：

1. **如何把 AgentMessage 转为 LLM 能懂的格式**（`convertToLlm`）
2. **工具执行间隙有没有人要说话**（`getSteeringMessages`/`getFollowUpMessages`）

### 3.3 AgentEvent — 细粒度的生命周期事件

三层嵌套的生命周期：**Agent > Turn > Message/Tool**。每一层都有 start/end 事件，构成完整的可观测性。

> *"Claude Code 的 Plan Mode 会 spawn 一个子 Agent，你对子 Agent 内部的运作零可见性。这是黑盒中的黑盒。"*

## 四、核心循环：双层 While 的精密设计

### 4.1 入口：`agentLoop` vs `agentLoopContinue`

| 函数 | 用途 | 前置条件 |
| --- | --- | --- |
| agentLoop(prompts, context, config) | 用户发了新消息 | 可以从空上下文开始 |
| agentLoopContinue(context, config) | 重试/恢复 | 上下文最后一条非 assistant |

### 4.2 双层循环结构

**1. 外层循环: FollowUp 驱动**

```
while (true) {
    // ... 内层循环处理 tool calls + steering ...

    // Agent 即将停止，检查 follow-up
    const followUpMessages = (await config.getFollowUpMessages?.()) || [];
    if (followUpMessages.length > 0) {
        pendingMessages = followUpMessages;
        continue; // 重启内层循环！
    }
    break; // 真正停止
}
```

**2. 内层循环: ToolCall + Steering 驱动**

```
while (hasMoreToolCalls || pendingMessages.length > 0) {
    // 注入 pending messages
    // 流式调用 LLM
    // 执行工具
    // 检查 steering
}
```

### 4.3 Steering 中断的实现细节

当用户在工具执行期间发送 steering message 时，`executeToolCalls` 会**立即停止执行后续工具**，将剩余工具全部标记为"Skipped due to queued user message"并返回错误结果。

## 五、Agent 类：状态容器 + 消息队列

### 5.1 两种队列模式

```
// Steering mode: "all" = 一次性发送全部 | "one-at-a-time" = 每次只发一条
private steeringMode: "all" | "one-at-a-time";
private followUpMode: "all" | "one-at-a-time";
```

**为什么需要 one-at-a-time？** 考虑这个场景：用户在 Agent 工作时快速发送了 3 条修正。`one-at-a-time` 模式让 Agent 逐条处理，每条都能得到充分响应。

### 5.2 `prompt()` vs `continue()` vs `steer()` vs `followUp()`

| 方法 | 调用时机 | 效果 |
| --- | --- | --- |
| prompt(msg) | Agent 空闲时 | 开一轮新对话 |
| continue() | Agent 空闲时 | 从当前状态续接（重试/消费队列） |
| steer(msg) | Agent 工作时 | 中断当前工具链，插入消息 |
| followUp(msg) | 任何时候 | 排队到 Agent 完成后执行 |
| abort() | Agent 工作时 | 取消当前 LLM 调用 |

## 六、Proxy Stream：带宽优化的客户端重建

核心创新是 **带宽优化**：服务器不传输 `partial` 字段，仅传输轻量的 delta 事件。

## 七、设计哲学总结

### 7.1 "不做什么"比"做什么"更重要

| 刻意不做 | 理由 |
| --- | --- |
| 无 Plan Mode | 用文件 PLAN.md 替代。有完整可观测性，可版本控制，可跨会话共享 |
| 无 MCP 支持 | MCP 工具描述占 7-9% 上下文窗口。用 CLI + README 通过 bash 调用，按需加载 |
| 无 Sub-Agent | "黑盒中的黑盒"，失去可观测性。通过 bash 自我调用，保留完整输出可见性 |
| 无 maxSteps | 循环自然结束。"我从来没找到需要 maxSteps 的用例，所以为什么要加？" |
| 无权限检查 | "安全措施大多是安全剧场。一旦 Agent 能写代码和运行代码，就 game over。" |

### 7.2 核心设计原则

- **极简主义**: < 1000 token 系统提示、4 个核心工具、5 个源文件
- **可观测性**: 三层事件生命周期，所有工具执行可见
- **可干预性**: Steering 中断、FollowUp 排队、Abort 取消
- **最晚转换**: 内部用 AgentMessage，仅 LLM 边界转换
- **自我进化**: 通过 bash 自我调用，运行时动态扩展，不依赖预制 Skills

### 7.3 写给开发者：何时借鉴 Pi 的设计

- ✅ **你的 Agent 需要高可观测性** → 学习 Pi 的三层事件系统
- ✅ **你在构建编码/CLI Agent** → 学习"4 工具 + bash"的极简策略
- ✅ **你需要跨 Provider 会话迁移** → 学习 `convertToLlm` 的最晚转换模式
- ✅ **你需要用户中途干预** → 学习 Steering/FollowUp 双队列机制
- ❌ **你需要复杂的多 Agent 编排** → Pi 的设计理念与此相悖
- ❌ **你需要严格的安全沙箱** → Pi 明确选择了"YOLO by default"

## 相关链接

- 项目地址: https://github.com/badlogic/pi-mono
