---
title: "分享一个Claude Code宝藏网站Claude Code Templates"
source: "微信公众号 - 程序员小溪"
url: "https://mp.weixin.qq.com/s/JIBqh6oU5K7cgGrgWAXjEw
author: "mz小溪"
date: 2025-10-21
tags:
  - Claude Code
  - AI工具
  - 效率工具
  - 工作流
type: 灵感原文
---

# 分享一个Claude Code宝藏网站Claude Code Templates

## 前言

Claude Code Templates 是 Anthropic Claude Code 的即用型配置。提供全面的 AI Agents、自定义命令、Settings、Hooks、MCP服务 和项目模板集合，可极大增强开发工作流程。

- 官网地址：https://www.aitmpl.com/agents
- Github地址：https://github.com/davila7/claude-code-templates

## 基本使用

### 命令行工具

Claude Code Templates 提供了 CLI 命令行工具，可以一行指令添加模版。

**1）交互式安装**
```bash
# 交互式浏览和安装
$ npx claude-code-templates@latest
```

**2）安装特定模版**
```bash
# 安装特定组件
$ npx claude-code-templates@latest --agent business-marketing/security-auditor
$ npx claude-code-templates@latest --command performance/optimize-bundle
$ npx claude-code-templates@latest --setting performance/mcp-timeouts
$ npx claude-code-templates@latest --hook git/pre-commit-validation
$ npx claude-code-templates@latest --mcp database/postgresql-integration

# 以 fullstack-developer Agent为例：
$ npx claude-code-templates@latest --agent=development-team/fullstack-developer --yes
```

**3）一键安装**
```bash
# 一键安装所有模版类型
npx claude-code-templates@latest --agent development-team/frontend-developer --command testing/generate-tests --mcp development/github-integration
```

### 可视化配置

**1）快速检索**
Claude Code Templates 提供了检索和详细的分类用于快速查找模版。还提供了排序功能，可以根据已验证优先、下载量最多、按字母顺序进行排序。

**2）安装模版**
点击任意一个模板项目可以查看详情、复制安装命令以及添加到栈。

**3）一键安装**
Claude Code Templates 提供一键安装多个模版功能（购物车模式）。

**4）自定义模版**
支持自定义模版配置，点击首页不同模版插件的【Add New】根据提示完成配置、验证、提交。

## 其他工具

**1）Claude Code分析**
使用实时状态检测和性能指标实时监控 AI 驱动的开发会话。
```bash
$ npx claude-code-templates@latest --analytics
```
执行完成后会在浏览器打开 Claude Code 分析服务 http://localhost:3333/

**2）对话监视器**
移动优化界面，通过安全的远程访问实时查看 Claude 响应。
```bash
# Local access
$ npx claude-code-templates@latest --chats
# Secure remote access via Cloudflare Tunnel
$ npx claude-code-templates@latest --chats --tunnel
```
执行完成后会在浏览器打开对话监视器服务 http://localhost:9876/

**3）Agents仪表盘**
查看并分析使用代理工具的 Claude 对话。执行完成后会在浏览器打开 http://localhost:3333/#agents

**4）插件仪表盘**
从统一界面查看市场、已安装的插件并管理权限。
```bash
$ npx claude-code-templates@latest --plugins
```
执行完成后会在浏览器打开 http://localhost:3336/

**5）项目初始化配置**
为项目添加初始化配置，支持选择开发语言和框架，生成包含自定义命令、配置、MCP 和记忆文件的项目目录结构。

**6）健康检查**
全面的诊断，确保您的 Claude Code 安装得到优化。
```bash
$ npx claude-code-templates@latest --health-check
```

---

原文链接：https://mp.weixin.qq.com/s/JIBqh6oU5K7cgGrgWAXjEw
