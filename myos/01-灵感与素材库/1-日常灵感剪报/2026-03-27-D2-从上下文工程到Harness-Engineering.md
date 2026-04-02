---
title: "【D2 演讲实录】从上下文工程到 Harness Engineering"
公众号: "ByteDance Web Infra"
date: "2026-03-27"
source: "https://mp.weixin.qq.com/s/ERSjcq9YURHvlsdTUv_Paw"
tags:
  - AI Coding
  - Harness Engineering
  - 上下文工程
  - 工具链设计
  - 字节跳动
关联参考:
  - "myos/02-选题池/待写选题库/2026-03-27-选题-从上下文工程到Harness-Engineering-3个角度.md"
---

> 本文为第 20 届 D2 技术大会的演讲实录。分享人：周晓 | Web Infra AI Coding 负责人
> PDF 和视频版本：https://github.com/zhoushaw/Context-Engineering-to-Harness-Engineering

## 核心观点

**为什么 AI Coding 让工程师更累了？**
- AI 只加速了写代码（30%），但验证/测试/部署占 70%
- 编码提速并未打破系统性天花板，非编码工作量正在爆发

**Harness Engineering 三条核心结论：**
1. **自回归** → 上下文越长，重复计算越多
2. **KV Cache** → 追加便宜，修改昂贵（保护前缀 = 复用缓存）
3. **上下文是最贵资源** → 治理它，不是堆砌它

**范式转移：**
- 从「写代码的人」→「给 Agent 设计环境、设计反馈的人」
- 好工具标准：快、准、结构化
- Agents Team：用结构化通信突破单 Agent 智能上限

## 金句

> 不要只把大模型看作'大脑'，必须为它打造专属的'工作室'。

> 工程师角色转变：从「编写代码」转向「设计环境、明确意图」

> 工具不是给人用的 UI — 是给 Agent 用的 API。快、准、结构化，缺一不可。

> 人机协作核心：Spec 答 What，Plan 答 How — 不确定时在第 1 轮就向人求助，而非猜错后浪费 20 轮。
