---
标题：Claude Code 项目记忆机制详解 - Auto Memory + CLAUDE.md 配置指南
阶段：待写
关联参考：[[20260306-Claude Code 项目记忆机制 Auto-Memory+CLAUDE.md]]
标签：[Claude Code, Auto Memory, CLAUDE.md, 项目记忆，AI 编程配置]
创建日期：2026-03-06
来源：微信公众号 - 兔兔 AGI 技术极简主义
---

# Claude Code 项目记忆机制详解 - Auto Memory + CLAUDE.md 配置指南

## 选题背景

Anthropic 为 Claude Code 上线了自动记忆功能，但很多人还不知道：
- Auto Memory 和 CLAUDE.md 有什么区别
- 如何配置多层级的规则文件
- 如何让子智能体拥有持久化记忆

需要一篇完整的配置指南。

## 核心架构

### 两种记忆组件

| 组件 | 编写者 | 内容 | 存储位置 | 共享范围 |
|------|--------|------|----------|----------|
| Auto Memory | Claude 自动记录 | 使用模式、调试经验、偏好 | `~/.claude/projects/.../memory/` | 仅当前用户 |
| CLAUDE.md | 开发者手动编写 | 项目规则、开发规范 | 项目目录 | 可团队共享 |

### CLAUDE.md 四层作用域

| 作用域 | 位置 | 优先级 |
|--------|------|--------|
| 组织级 | 系统目录 | 最低 |
| 项目级 | `./CLAUDE.md` | 中 |
| 用户级 | `~/.claude/CLAUDE.md` | 中 |
| 本地级 | `./CLAUDE.local.md` | 最高 |

## 核心观点

> Auto Memory 用来自动记录经验，而 CLAUDE.md 用来明确规则。

> 有了项目记忆之后，AI 更像是一个熟悉项目的协作伙伴，而不是每次都需要重新培训的新人。

> 每次会话启动时，只会加载 MEMORY.md 的前 200 行。

## 文章结构

1. **项目记忆是什么** - Auto Memory + CLAUDE.md 的分工
2. **CLAUDE.md 配置详解** - 四层作用域、@语法导入、路径特定规则
3. **Auto Memory 工作机制** - 200 行限制、存储结构、启用禁用
4. **子智能体持久化记忆** - user/project/local 作用域选择
5. **实战最佳实践** - 项目初始化、团队协作、大型项目组织

## 实施步骤

1. 运行 `/init` 自动生成基础配置
2. 编写项目级 CLAUDE.md（不超过 200 行）
3. 使用 .claude/rules/ 模块化管理大型项目
4. 配置 Auto Memory（默认开启）
5. 为子智能体添加 memory 字段

## 待验证要点

1. OpenClaw 是否兼容 CLAUDE.md 格式
2. Auto Memory 在 OpenClaw 中的实现方式
3. 与 SOP_MYOS.md 内容工厂流程的结合点

## 参考

- [[20260306-Claude Code 项目记忆机制 Auto-Memory+CLAUDE.md]] - 完整配置说明
