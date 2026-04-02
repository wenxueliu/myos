---
title: "JSSE: A JavaScript Engine Built by an Agent"
date: "2026-04-01"
tags: ["AI", "JavaScript", "Engine", "Rust", "Claude-Code"]
source: "https://p.ocmatos.com/blog/jsse-a-javascript-engine-built-by-an-agent.html"
description: "AI Agent 全程自主开发的 JavaScript 引擎 JSSE，42天达成 100% test262 兼容率"
关联参考:
  - "HackerNews Top 50 #37: JSSE: A JavaScript Engine Built by an Agent"
---

# JSSE: A JavaScript Engine Built by an Agent

> 原文作者：Paulo Matos
> 发布日期：2026-03-17
> 来源：Notes & Code Blog

## 核心结论

Paulo Matos 用 Claude Code AI Agent 全程自主开发了 JSSE（JavaScript Simple Engine），这是首个达成 100% test262 非分期（non-staging）合规的 JavaScript 引擎。

**关键数据：**
- 约 **170,000 行 Rust**，零行由人工编写
- 人工交互时间：**约 4 小时**（分布在项目周期内）
- 项目成本：**$4,619**（47天内 302 个会话）
- 约 **$0.03/行代码**

## 开发过程

JSSE 在项目启动后仅 **4 小时**就执行了第一行 JavaScript，并在 2026 年 3 月 9 日达成完全合规，历时 **42 天**。

开发流程：

1. Agent 遵循 `PLAN.md` 任务清单（从 ECMAScript 规范和 test262 子模块生成）
2. Claude 分析 test262 覆盖缺口，提出按影响力排序的方案
3. 自主实现功能

**亮点事件：**
- 整个 Temporal API（ES 最复杂的提案之一）在一次 16 小时的通宵会话中实现
- 修复了一个 `String.prototype` 接线 bug，一夜间解封约 **11,000 个测试**

## 技术成就

JSSE 在 test262 合规指标上超越了 Boa 和 Node.js（使用相同测试套件对比）。

Agent 自主处理了：
- 数组空洞语义（array hole semantics）
- 生成器（Generators）
- SharedArrayBuffer、Atomics
- 大量 Unicode/RegExp 支持

> **性能说明**：JSSE 是纯 tree-walking 解释器，性能比 Node.js 慢 1.2x ~ 703x（故意不优先优化性能）

## 意义

这个项目证明了：**在正确的测试驱动反馈循环下，当前的 AI 编码 Agent 能够成功实现复杂规范**。

## 相关链接

- 仓库：github.com/pmatos/jsse（MIT 许可）
- HackerNews 讨论：排名第 37
