---
title: "开源多平台 AI 智能体上下文驱动开发技能 Conductor 完整上手攻略"
author: "兔兔AGI / 技术极简主义"
source: "微信公众号"
url: "https://mp.weixin.qq.com/s/0hudGw8PyyHR1Q8o6YPrdQ"
date: "2026-02-28"
tags:
  - AI编程
  - Agent开发
  - Conductor
  - 上下文管理
  - 工程化
  - ClaudeCode
关联参考: []
---

# 开源多平台 AI 智能体上下文驱动开发技能 Conductor 完整上手攻略

> 在 AI 编程时代，我们很容易形成一种惯性思维：需求一来就直接写代码，默认 AI 能自动理解一切。但实际情况往往是，代码越写越多，才发现方向跑偏了，或者生成的实现和项目原有风格并不匹配。

**核心问题**：我们过度依赖临时对话来传递项目上下文。每次开启新会话，都要重新说明项目背景、技术选型和代码规范，不仅效率低，也让团队很难长期保持一致的标准。

**Conductor 的解法**：把项目上下文从临时对话中抽离出来，沉淀到代码库中，通过持久化的 Markdown 文件来管理规范和计划，确保每一次 AI 参与都有清晰、稳定的项目基础。

> 这更像一句老工程师常说的话：**三思而后行（Measure twice, code once）**。Conductor 把 AI 编程拉回工程本身——先把规划想清楚，再动手实现，方向始终掌握在人手里。

---

## 什么是 Conductor

Conductor 是一个开源的多平台 **AI 智能体技能（Agent Skills）**，基于 Google 的 [Conductor](https://github.com/gemini-cli-extensions/conductor)（Gemini CLI 扩展工具）项目移植而来，支持 **Claude Code**、**Gemini CLI**、**OpenCode**、**Codex** 等多种 CLI 工具。

它遵循「**上下文驱动开发**」的思路，通过固定流程推进需求澄清、方案规划以及功能或缺陷实现，让 AI 的输出始终建立在清晰、可追溯的项目上下文之上。

**Conductor 的目标**：让每个任务都经历完整、可控的生命周期。通过「**Context → Spec & Plan → Implement**」这一结构化流程，它把 AI 从单一的编码工具，提升为具备主动项目推进能力的工具。

---

## 核心理念

> 掌控你的代码。通过将上下文作为与代码同等管理的资产，代码库就能成为**单一可信源**，为每一次智能体交互提供深度、持久的项目感知。

团队可以在此基础上为所有 AI 交互建立一套可持续、具备项目感知能力的工作体系。系统会持续维护以下内容：

- **产品愿景**：作为可持续更新的文档，而不是一次性说明
- **技术决策**：以结构化形式记录，便于追溯和复用
- **工作单元（tracks）**：包含清晰的规范说明和分阶段的实施计划
- **TDD 工作流**：在关键阶段设置验证检查点，确保质量可控

**关键特点**：Conductor 本身不运行任何服务，只定义了一套 AI 需要遵循的工作流程。所有内容都以 **Markdown** 和 **TOML** 文件存在，不使用私有格式，也不依赖任何厂商 API。项目上下文随代码一起版本化、一起流转。

---

## 功能特性

- **规范与规划**：在开发前先整理需求和任务计划，让接下来的工作有明确方向
- **上下文管理**：保持代码风格、技术栈和产品目标的一致性
- **安全迭代**：在代码生成前对计划进行审查，关键决策由人掌控
- **团队协作**：项目文档作为共享基础，不同成员和 AI 都能在同一规范下工作
- **支持现有项目**：适用于新项目和已有项目
- **语义化回滚**：可按逻辑工作单元（track、phase、task）撤销变更
- **状态持久化**：设置和任务状态会保存到文件里，可跨会话继续工作

---

## 核心工作流

### 步骤一：项目设置（setup）

```bash
# Claude Code / OpenCode / Codex / Antigravity
set up conductor

# Gemini CLI
/conductor:setup
```

Conductor 会问你一系列问题，帮你把项目的核心上下文整理好：

- **产品定义**：项目背景、目标用户、主要功能概览
- **产品规范**：文案风格、品牌约束、视觉规范等
- **技术栈**：语言、数据库、框架等技术选型
- **工作流**：如 TDD 流程、提交策略、团队协作约定
- **代码风格**：根据所选语言制定的代码规范

生成的目录结构：
```
conductor/
├── product.md              # 产品愿景、目标与范围
├── product-guidelines.md   # 产品与设计相关规范
├── tech-stack.md           # 技术栈与工具选型
├── workflow.md             # 开发工作流（TDD、提交规范等）
├── code_styleguides/       # 各语言的代码风格指南
└── tracks.md               # 当前项目任务（tracks）索引
```

### 步骤二：创建任务（newTrack）

```bash
# Claude Code / OpenCode / Codex / Antigravity
create a new track for dark mode

# Gemini CLI
/conductor:newTrack "Add a dark mode toggle to the settings page"
```

创建完成后，会生成两个核心文件：
- **spec.md**：需求规范，明确要做什么以及为什么要做
- **plan.md**：实施计划，把工作按阶段（Phase）拆分为具体任务和子任务

生成的目录结构：
```
conductor/tracks/<track_id>/
├── spec.md          # 需求规范说明
├── plan.md          # 可执行的任务与阶段计划
└── metadata.json    # Track 元数据
```

### 步骤三：执行实施（implement）

```bash
# Claude Code / OpenCode / Codex / Antigravity
implement the next task

# Gemini CLI
/conductor:implement
```

工作流程：
1. 选择下一个要处理的任务
2. 按照项目里的工作流执行（例如 TDD：先写测试 → 测试失败 → 实现功能 → 测试通过）
3. 同步更新 plan.md 中对应任务的状态
4. 每个阶段完成后，提示进行必要的人工检查与确认
5. 按配置在 Task 或 Phase 完成后提交代码

---

## 常用命令

| 命令 | 用途 |
|------|------|
| `set up conductor` | 初始化项目上下文 |
| `create a new track for <任务>` | 创建新 Track |
| `implement the next task` | 开始实现 |
| `check status` | 查看项目进度 |
| `review my work` | 审查已完成的工作 |
| `revert` | 按逻辑单元回滚 |

---

## Task 状态标记

- `[ ]` 未开始
- `[~]` 进行中
- `[x]` 已完成

---

## 写到最后

Conductor 不只是一个工具，更像是一种工作方式的改变。它的重点不是写得有多快，而是让你能和一个懂工程的「AI 伙伴」协作，把软件踏踏实实做好。

**Github 地址**：https://github.com/jnorthrup/conductor2
