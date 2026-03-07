---
标题：子智能体持久化记忆配置 - 让 AI 团队跨项目共享学习成果
阶段：待写
关联参考：[[20260306-Claude Code 项目记忆机制 Auto-Memory+CLAUDE.md]]
标签：[子智能体，持久化记忆，AI Agent, Claude Code, 记忆配置]
创建日期：2026-03-06
来源：微信公众号 - 兔兔 AGI 技术极简主义
---

# 子智能体持久化记忆配置 - 让 AI 团队跨项目共享学习成果

## 选题背景

Claude Code 的子智能体（subagent）也可以拥有持久化记忆：
- 代码审查员可以记住常见代码模式
- 测试专家可以记住项目测试约定
- 跨项目共享学习成果

很多人还不知道这个功能。

## 配置方法

### 在子智能体 markdown 文件中添加 memory 字段

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---

你是一个代码审查员。在审查代码时，将发现的模式、约定和常见问题更新到你的代理记忆中。
```

### 三种作用域选择

| 作用域 | 位置 | 适用场景 |
|--------|------|----------|
| **user** | `~/.claude/agent-memory/<agent-name>/` | 智能体跨所有项目共享学习内容 |
| **project** | `.claude/agent-memory/<agent-name>/` | 智能体的知识仅与当前代码库相关 |
| **local** | `.claude/agent-memory-local/<agent-name>/` | 项目特定且不应提交到版本控制 |

## 启用记忆后的行为

- 在系统提示中包含对记忆目录的读写指令
- 自动加载记忆目录下 MEMORY.md 的前 200 行
- 自动启用 Read、Write 和 Edit 工具来管理记忆文件

## 最佳实践

- 默认使用 `user` 作用域
- 在子智能体开始任务前，让它先查询记忆
- 完成任务后，让子智能体更新记忆

## 核心观点

> 子智能体也可以像主智能体一样积累和复用经验。

> 跨项目共享学习成果，让 AI 团队越用越聪明。

## 文章结构

1. **子智能体记忆是什么** - 与主智能体记忆的区别
2. **配置方法** - memory 字段与作用域选择
3. **三种作用域对比** - user vs project vs local
4. **启用后的行为** - 自动加载与更新
5. **最佳实践** - 如何最大化记忆价值
6. **实战案例** - 代码审查员、测试专家等角色配置

## 待验证要点

1. OpenClaw 是否支持子智能体记忆配置
2. 与 Tableau AI 文章中的"图书管理员"角色的对比
3. 记忆文件的格式规范

## 参考

- [[20260306-Claude Code 项目记忆机制 Auto-Memory+CLAUDE.md]] - 子智能体记忆配置说明
