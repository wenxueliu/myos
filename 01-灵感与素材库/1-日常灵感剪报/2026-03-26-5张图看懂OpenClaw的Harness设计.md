---
title: "5张图看懂OpenClaw的Harness设计"
source: "微信公众号 - 诗与沅方"
url: "https://mp.weixin.qq.com/s/JtECR8pMopRSec_FF3S20g"
author: "FloraCat"
date: 2026-03-24
tags:
  - OpenClaw
  - AI Agent
  - 架构设计
  - Claude Code
  - Harness
type: 灵感原文
关联参考: []
---

# 5张图看懂OpenClaw的Harness设计

近来Harness这个词非常火，而且Claude Code最近也在龙虾化，我就想深扒下OpenClaw的Harness是如何设计的呢？

---

## 核心观点

**OpenClaw是个Gateway-First的项目。**

它上接多渠道入口，下连会话路由、插件扩展、记忆系统和运行时，中间是一条统一的执行主链路。

**Harness = OpenClaw面向LLM的结构化运行壳**

它负责：
- 组装 prompt
- 挂载 tools
- 接入 skills 和 memory
- 处理策略与安全限制
- 通过 Provider Adapter 与不同厂商的 LLM API 交互

因为有Harness，OpenClaw才不是"直接把文本丢给模型"，而是真正具备了可扩展、可控制、可落地的Agent运行能力。

---

## 图1：整体架构图

**OpenClaw整体分层架构：**
- 用户交互层
- 网关控制层
- 能力层与扩展层
- 底层状态存储

---

## 图2：Harness 核心运行链路

### 7层结构：

| 层次 | 职责 |
|------|------|
| 输入与上下文 | 用户消息/命令/会话历史/工作区文件/bootstrap上下文 |
| Prompt装配器 | 将system prompt、skills prompt、docs、bootstrap文件、运行时信息拼成最终提示词 |
| 模型解析与策略 | 决定用哪个模型、什么thinking档位、哪个认证身份；处理fallback、hooks干预 |
| 工具与安全壳 | 限制模型可调用能力边界，增强安全性 |
| Agent会话与执行循环 | 创建Agent Session、接收流式输出、处理Tool Call |
| 厂商适配器 | 统一封装不同模型厂商的API调用 |
| 传输与认证 | HTTP、SSE、WebSocket等传输方式与认证机制 |

---

## 图3：消息传递时序图

**消息从"收进来"到"发回去"的全过程：**

1. 消息进入 → 去重、顺序控制、基础校验
2. 结合账号、会话和话题线程，定位到正确的Agent和Session
3. 整理上下文（正文、媒体、回复引用）
4. 交给Agent Runtime执行
5. 策略判断、hooks、typing状态、skills、工具调用、记忆检索共同参与
6. 根据replyTo和线程关系，选择正确投递目标发回

**核心设计：无论消息来自CLI、WebChat还是外部渠道，最终都落到同一条执行主线。**

---

## 图4：记忆系统

**记忆由三部分组成：**

### 1. 工作区里的记忆文件
- `MEMORY.md` 和 `memory/*.md`
- 属于**长期记忆**

### 2. Agent运行时里的记忆工具
- `MEMORY.md` 直接注入上下文（只覆盖会话启动时）
- `memory/*.md` 通过 `memory_search / memory_get` 按需读取

### 3. 后台索引与检索层
- 把 `MEMORY.md` 和 `memory/*.md` 建成每个 agent 一份的 SQLite 索引
- 负责 chunk/embedding/检索，不是记忆内容本身

**注意：** SOUL.md / IDENTITY.md / USER.md 等人设文件虽然也是bootstrap persona上下文，但**不属于严格意义上的记忆**，属于persona/identity注入。

---

## 图5：插件系统设计

**Gateway为中枢，插件系统是无限能力扩展。**

### 插件生命周期：
发现 → 加载 → 注册 → 运行时激活

### 三类插件：
- **渠道插件**：负责接消息入口
- **平台插件**：负责扩展 tools、hooks、providers 和 skills
- **Gateway注入点**：负责将这些能力接到 methods、routes、services 上

**关键设计：** 插件能力不会直接散落在系统各处，而是先汇总到"**运行时激活注册表**"。

---

## 金句

> "因为有Harness，OpenClaw才不是'直接把文本丢给模型'，而是真正具备了可扩展、可控制、可落地的Agent运行能力。"

---

原文链接：https://mp.weixin.qq.com/s/JtECR8pMopRSec_FF3S20g
