---
title: "5张图看懂OpenClaw的Harness设计"
source: "FloraCat-诗与沅方"
url: "https://mp.weixin.qq.com/s/JtECR8pMopRSec_FF3S20g"
author: "FloraCat"
date: 2026-03-24
tags:
  - OpenClaw
  - AI Agent
  - 架构设计
  - Claude Code
type: 待写选题
关联参考:
  - "myos/01-灵感与素材库/1-日常灵感剪报/2026-03-26-5张图看懂OpenClaw的Harness设计.md"
---

# 选题：5张图看懂OpenClaw的Harness设计

## 切入角度

**角度一：Gateway-First — 做个"好壳"比造轮子更重要**

- OpenClaw的核心理念：不是直接造LLM，而是做面向LLM的结构化运行壳
- 7层Harness架构：prompt装配、模型解析、工具安全壳、执行循环、厂商适配...
- 金句："因为有Harness，才不是直接把文本丢给模型"
- 可对比：Claude Code的Harness设计有何异同？

**角度二：记忆不是单一模块 — 三层结构才是正解**

- 工作区记忆文件（MEMORY.md / memory/*.md）= 长期记忆
- Agent运行时记忆工具 = 按需读取
- 后台SQLite索引层 = chunk/embedding/检索
- 关键洞察：人设文件（SOUL.md）不属于记忆，属于persona注入
- 对比：Claude Code的记忆设计是否也有类似分层？

**角度三：插件即能力 — 运行时激活注册表是精髓**

- 三类插件：渠道插件（接消息）、平台插件（扩展能力）、Gateway注入点
- 插件不散落各处，而是先汇总到"运行时激活注册表"
- 类比：Claude Code的MCP是不是也是这个思路？
- 适合开发者视角：如何设计自己的Agent扩展架构
