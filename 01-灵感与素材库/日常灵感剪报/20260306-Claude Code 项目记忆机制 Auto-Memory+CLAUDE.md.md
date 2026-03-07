---
标题：深入理解 Claude Code 的项目记忆机制：Auto-Memory + CLAUDE.md
来源：微信公众号 - 兔兔 AGI 技术极简主义
原文链接：https://mp.weixin.qq.com/s/igvGpFSN01OGx57jmQ6suA
捕获日期：2026-03-06
标签：[Claude Code, Auto Memory, CLAUDE.md, 项目记忆，AI 编程，子智能体，记忆系统]
---

# 深入理解 Claude Code 的项目记忆机制：Auto-Memory + CLAUDE.md

**来源**：兔兔 AGI 技术极简主义  
**发布**：2026 年 3 月 6 日

---

## 核心更新

Anthropic 为 Claude Code 上线了**自动记忆**（Auto Memory）功能。

> 让模型在长期项目中逐步积累对项目的理解，减少开发者反复提供背景信息的负担。

---

## 什么是项目记忆

Claude Code 的项目记忆功能让 AI 助手逐渐「了解」你的项目：
- 目录结构
- 编码风格
- 架构约定

**每次会话启动时，这些保存下来的信息都会自动加载**。

### 两个组成部分

| 组件 | 说明 |
|------|------|
| **Auto Memory**（自动记忆） | Claude 自动记录使用模式、调试经验、项目架构细节 |
| **CLAUDE.md 文件**（显式指导） | 开发者手动编写的项目规则、开发规范、上下文 |

> Auto Memory 用来自动记录经验，而 CLAUDE.md 用来明确规则，两者一起构成项目记忆体系。

---

## Auto Memory vs CLAUDE.md

| 特性 | Auto Memory | CLAUDE.md |
|------|-------------|-----------|
| **编写者** | Claude 自动记录 | 开发者手动编写 |
| **内容性质** | 使用模式、调试经验、偏好 | 项目规则、开发规范、上下文 |
| **存储位置** | `~/.claude/projects/.../memory/` | 项目目录或根目录 |
| **管理方式** | Claude 自动维护 | 由开发者维护 |
| **共享范围** | 仅当前用户（按项目） | 可以团队共享 |
| **适用场景** | 日常使用中自动积累 | 需要明确规范时 |

---

## 为什么需要项目记忆

**没有项目记忆的问题**：
- 每次启动会话都要重新说明：「我们使用 TypeScript」「代码缩进是 2 个空格」
- 浪费时间，打断开发节奏
- AI 像每次都需要重新培训的新人

**有了项目记忆之后**：
- 信息在会话开始时自动加载
- Claude 从一开始就能理解项目基本情况
- AI 更像是一个熟悉项目的协作伙伴

---

## CLAUDE.md 文件指令机制

### 文件位置与作用域

| 作用域 | 位置 | 用途 | 共享给 |
|--------|------|------|--------|
| **组织级** | 系统目录 | 公司级编码规范、安全策略 | 组织内所有用户 |
| **项目级** | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 项目架构、编码规范、开发流程 | 团队成员（通过版本控制共享） |
| **用户级** | `~/.claude/CLAUDE.md` | 个人偏好、常用工具配置 | 仅你自己（所有项目） |
| **本地级** | `./CLAUDE.local.md` | 当前项目的私人配置（通常不提交到 Git） | 仅你自己 |

> Claude 会从当前工作目录开始向上遍历目录树，并加载沿途的所有 CLAUDE.md 文件。路径越具体，优先级越高。

### 快速生成项目配置

```bash
/init
```

Claude 会扫描项目结构，识别常见的构建命令、测试方式以及项目约定，并生成一份 CLAUDE.md 草稿。

### 编写有效指令的 4 个原则

#### 1. 控制篇幅

> 每个 CLAUDE.md 文件最好不要超过 200 行。文件过长会消耗大量 token，可能导致 Claude 难以完全遵循指令。

#### 2. 结构清晰

```markdown
# 项目约定

## 构建命令
- 开发环境：`pnpm dev`
- 构建：`pnpm build`
- 测试：`pnpm test`

## 代码规范
- 使用 2 空格缩进
- 组件放在 `src/components/`
- API 路由放在 `src/api/routes/`
```

#### 3. 具体明确

| ❌ 模糊指令 | ✅ 明确指令 |
|------------|------------|
| "格式化代码" | "使用 2 空格缩进" |
| "测试你的改动" | "提交前运行 pnpm test" |
| "保持文件组织" | "API 处理函数放在 src/api/handlers/" |

#### 4. 避免冲突

如果多个 CLAUDE.md 文件中存在相互矛盾的指令，Claude 可能会选择其中之一执行。

### 使用 @ 语法导入文件

```markdown
# 项目概述
参见 @README 获取项目简介

# 可用命令
参见 @package.json 了解所有 npm scripts

# 工作流程
遵循 @docs/git-workflow.md 中的 Git 分支策略
```

- 相对路径是相对于包含 `@` 的文件解析
- 导入最多支持 **5 层嵌套**
- 首次导入外部文件时，Claude 会弹出批准对话框

### 使用 .claude/rules/ 组织大型项目

```
your-project/
├── .claude/
│   ├── CLAUDE.md           # 主项目指令
│   └── rules/
│       ├── code-style.md   # 代码风格指南
│       ├── testing.md      # 测试约定
│       ├── api-design.md   # API 设计规范
│       └── security.md     # 安全要求
```

所有 .md 文件会被递归加载。

### 路径特定规则

规则文件可以在 YAML frontmatter 中使用 `paths` 字段来限定作用范围：

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API 开发规则

- 所有 API 端点必须包含输入校验
- 使用标准错误响应格式
- 包含 OpenAPI 文档注释
```

| 模式 | 匹配内容 |
|------|----------|
| `**/*.ts` | 任意目录下的 TypeScript 文件 |
| `src/**/*` | src/ 目录下的所有文件 |
| `*.md` | 项目根目录下的 Markdown 文件 |
| `src/components/*.tsx` | 特定目录下的 React 组件 |

### 跨项目共享规则

```bash
# 链接整个共享规则目录
ln -s ~/shared-claude-rules .claude/rules/shared

# 链接单个规则文件
ln -s ~/company-standards/security.md .claude/rules/security.md
```

### 用户级规则

```
~/.claude/rules/
├── preferences.md    # 编码偏好
└── workflows.md      # 工作流程习惯
```

用户级规则的优先级低于项目级规则。

---

## Auto Memory 自动学习机制

### 工作原理：200 行限制

> 每次会话启动时，只会加载 MEMORY.md 的前 200 行。

**限制的作用**：
- 控制上下文消耗：防止记忆文件无限增长
- 保持信息新鲜：Claude 会将详细内容拆分到主题文件
- 按需加载：主题文件仅在需要时才被读取

### 存储位置与文件结构

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 简洁索引，每次会话加载
├── debugging.md       # 调试模式详细笔记
├── api-conventions.md # API 设计决策
└── ...                # 其他主题文件
```

`<project>` 路径从 git 仓库路径推导而来，同一仓库的所有 worktree 和子目录共享同一个 Auto Memory 目录。

### 启用与禁用

**方法 1：在会话中使用 /memory**
```bash
/memory
```

**方法 2：修改项目配置**
```json
{
  "autoMemoryEnabled": false
}
```

**方法 3：使用环境变量**
```bash
CLAUDE_CODE_DISABLE_AUTO_MEMORY=1
```

### 审查和编辑记忆

运行 `/memory` 命令可以：
- 查看当前会话加载的所有 CLAUDE.md 和规则文件
- 切换 Auto Memory 开关
- 打开 Auto Memory 文件夹
- 在编辑器中打开任意文件

> 当你告诉 Claude"记住用 pnpm 而不是 npm"时，它会自动保存到 Auto Memory。

---

## 子智能体的持久化记忆

### 配置方法

在子智能体的 markdown 文件中添加 `memory` 字段：

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---

你是一个代码审查员。在审查代码时，将发现的模式、约定和常见问题更新到你的代理记忆中。
```

### 作用域选择

| 作用域 | 位置 | 适用场景 |
|--------|------|----------|
| **user** | `~/.claude/agent-memory/<agent-name>/` | 智能体跨所有项目共享学习内容 |
| **project** | `.claude/agent-memory/<agent-name>/` | 智能体的知识仅与当前代码库相关 |
| **local** | `.claude/agent-memory-local/<agent-name>/` | 项目特定且不应提交到版本控制 |

### 启用记忆后，子智能体会

- 在系统提示中包含对记忆目录的读写指令
- 自动加载记忆目录下 MEMORY.md 的前 200 行
- 自动启用 Read、Write 和 Edit 工具来管理记忆文件

### 最佳实践

- 默认使用 `user` 作用域
- 在子智能体开始任务前，让它先查询记忆
- 完成任务后，让子智能体更新记忆

---

## 实战最佳实践

### 项目初始化：自动生成配置

```bash
/init
```

Claude 会自动识别：
- 构建系统（npm、pnpm、yarn、make、cargo 等）
- 测试框架与测试命令
- 项目目录结构
- 常见代码模式

### 团队协作：共享规则 vs 个人偏好

| 团队共享（放入 CLAUDE.md） | 个人偏好（放入个人配置） |
|---------------------------|-------------------------|
| 项目架构与目录结构 | 编辑器快捷键习惯 |
| 编码规范（缩进、命名约定） | 调试偏好 |
| 构建和测试命令 | 私有开发环境 URL |
| API 设计规范 | 个人工具链配置 |
| 安全要求 | |

### 大型 monorepo 组织策略

1. **模块化管理 .claude/rules/**
2. **路径特定规则** - 让规则只在相关文件被处理时加载
3. **排除无关配置** - 在 settings.local.json 中使用 claudeMdExcludes
4. **符号链接共享通用规则**

---

## 常见问题

### 问题 1：Claude 没有按预期执行 CLAUDE.md 中的指令

- 运行 `/memory`，确认文件已被加载
- 检查 CLAUDE.md 是否放在 Claude 会读取的位置
- 确保指令具体明确
- 检查是否存在冲突或重复的指令

### 问题 2：不清楚 Auto Memory 保存了什么

运行 `/memory` 并打开 Auto Memory 文件夹即可查看内容。

### 问题 3：CLAUDE.md 文件过大

超过 200 行的 CLAUDE.md 会占用过多上下文。解决方法：
- 使用 `@path/to/file` 导入，将详细内容移到单独文件
- 拆分成 .claude/rules/ 目录下的多个主题文件
- 删除过时或重复的指令

### 问题 4：执行 /compact 后指令丢失

CLAUDE.md 在执行 /compact 后不会丢失内容。如果指令消失，说明它们只是出现在对话中，并未写入 CLAUDE.md。

---

## 核心金句

> 让模型在长期项目中逐步积累对项目的理解，减少开发者反复提供背景信息的负担。

> Auto Memory 用来自动记录经验，而 CLAUDE.md 用来明确规则。

> 有了项目记忆之后，AI 更像是一个熟悉项目的协作伙伴，而不是每次都需要重新培训的新人。

> 每次会话启动时，只会加载 MEMORY.md 的前 200 行。

> 在 AI 编程工具快速发展的背景下，「会话即失忆」一直是影响生产效率的问题。

---

## 可提炼选题

1. **Claude Code 项目记忆机制详解** - Auto Memory + CLAUDE.md 的完整配置指南
2. **AI 编程的"会话失忆"问题解决方案** - 从 Claude Code 记忆机制看行业趋势
3. **CLAUDE.md 编写最佳实践** - 200 行限制下的高效指令设计
4. **子智能体持久化记忆配置** - 让 AI 团队跨项目共享学习成果
