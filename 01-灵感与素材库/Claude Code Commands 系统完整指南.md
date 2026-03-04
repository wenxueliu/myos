---
tags: [Claude Code, Commands, Slash 命令，自定义命令，AI 编程，教程]
created: 2026-03-04
source: https://my.feishu.cn/wiki/FbDowO2hQi8mtFk8ww5cB9bbnSd
author: 老金
status: 素材
---

# Claude Code Commands 系统完整指南

> Slash 命令就是 Markdown 提示词文件 —— 一次配置，永久使用。

## 核心概念

**自定义命令 = `.claude/commands/` 目录下的 Markdown 文件**

- **命令名** = 文件名（不含.md 后缀）
- **命令内容** = Markdown 文件内容（作为提示词注入）
- **参数** = `$ARGUMENTS` 变量接收用户输入

```
/hello 张三
  │      │
  │      └── $ARGUMENTS = "张三"
  └── 读取 hello.md
```

## 命令作用域

| 作用域 | 存放位置 | 生效范围 |
|--------|----------|----------|
| 项目级 | `.claude/commands/` | 仅当前项目 |
| 用户级 | `~/.claude/commands/` | 所有项目共享 |

**优先级**: 项目级 > 用户级 > 内置命令

⚠️ **核心系统命令受保护**（如 `/clear`、`/help`）不可被自定义命令覆盖。

## frontmatter 配置详解

```yaml
---
description: 公众号文章全自动创作
argument-hint: <主题关键词>
allowed-tools:
  - Read
  - Write
  - WebSearch
  - Grep
  - Bash
model: claude-sonnet-4-6-20250929
disable-model-invocation: false  # 纯文本替换时设为 true
---
```

### 可用工具列表

| 工具 | 功能 | 通俗解释 |
|------|------|----------|
| Read | 读取文件 | 打开文件看内容 |
| Write | 写入文件 | 新建文件并写入 |
| Edit | 编辑文件 | 修改已有文件 |
| Bash | 执行命令 | 在终端运行命令 |
| WebSearch | 网络搜索 | 像 Google 一样搜网页 |
| WebFetch | 抓取网页 | 下载网页内容来分析 |
| Glob | 按名称查找文件 | 找所有*.md 文件 |
| Grep | 按内容搜索 | 找包含"TODO"的代码 |
| Task | 启动子代理 | 派出分身帮你干活 |

## 命令命名空间

子目录命令使用 `目录名：命令名` 格式：

```
.claude/commands/
├── dev/
│   ├── code-review.md    → /dev:code-review
│   └── debug.md          → /dev:debug
├── test/
│   └── generate.md       → /test:generate
└── deploy/
    └── prepare.md        → /deploy:prepare
```

## 实战案例：公众号写作命令

```markdown
---
description: 公众号文章全自动创作 - 从选题到成稿的完整流程
argument-hint: <主题关键词>
allowed-tools:
  - Read
  - Write
  - WebSearch
  - Grep
  - Bash
---

# 公众号文章创作系统

## 角色定义
你是一位资深的公众号写作专家，擅长创作接地气、有深度的技术科普文章。

## 任务
根据用户提供的主题，创作一篇高质量的公众号文章。

**主题**：$ARGUMENTS

## 执行流程

### 步骤 1：选题可行性检查
- 是否有足够的信息支撑？
- 受众是否感兴趣？
- 是否与账号定位匹配？

### 步骤 2：信息收集
WebSearch(query="$ARGUMENTS 最新资讯 2025")

### 步骤 3：构思大纲
一、金句开头（1 段）
二、问题引入（2-3 段）
三、核心内容（5-8 段）
四、总结升华（1-2 段）

### 步骤 4：撰写文章
风格要求：说人话、用类比、多短句、口语化
格式要求：标题≤30 字、正文 1500-2000 字、段落≤150 字

### 步骤 5：保存文章
Write → articles/drafts/[日期]_[主题].md

### 步骤 6：生成标题
生成 5 个备选标题（包含数字、引发好奇心、≤30 字）
```

## 命令组合与工作流

```markdown
# 每日内容创作工作流

## 执行流程
1. 执行 `/hotspot` 获取今日 AI 热点
2. 选择适合的写作主题
3. 执行 `/write [选定主题]` 创作文章
4. 执行 `/title-gen` 生成更多标题
5. 执行 `/pre-check` 进行发布前检查
6. 输出完整工作报告
```

## 故障排查速查

| 问题 | 排查步骤 |
|------|----------|
| 命令不存在 | 检查文件是否存在、文件名正确、目录位置正确 |
| frontmatter 解析错误 | 检查 YAML 语法、缩进、`---` 标记完整 |
| 命令执行失败 | 检查 `allowed-tools` 是否包含所需工具 |
| 命令执行慢 | 减少不必要的工具调用、限制搜索结果数量 |

## 社区资源

- **Claude Command Suite**: https://github.com/qdhenry/Claude-Command-Suite (148+ 专业命令)
- **Awesome Claude Code**: https://github.com/hesreallyhim/awesome-claude-code

## 与 OpenClaw 的关联

1. **coding-agent skill** 可参考 Commands 系统设计 agent 任务委托
2. **模块化设计** 思路可用于 OpenClaw skill 开发
3. **工作流组合** 理念适用于 OpenClaw 多 agent 协作
4. **frontmatter 配置** 模式可借鉴到 OpenClaw 技能配置

---

**原文档版本**: v1.1（2026-02-25 更新）

**预计学时**: 4-6 小时

**难度等级**: ⭐⭐ 入门级
