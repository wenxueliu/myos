---
标题：AI Agent 自动化工作流实战 - 从需求到 PR 的 8 步流程
阶段：待写
关联参考：[[20260306-OpenClaw-Claude Code 超强教程一个人搭建开发团队]]
标签：[AI Agent, 自动化工作流，PR 流程，Code Review, CI/CD]
创建日期：2026-03-06
来源：微信公众号 - Datawhale
---

# AI Agent 自动化工作流实战 - 从需求到 PR 的 8 步流程

## 选题背景

很多人用 AI 写代码还是"手动模式"：
- 自己写 prompt
- 自己运行测试
- 自己创建 PR
- 自己找 reviewer

需要一篇完整的自动化工作流指南，展示从需求到 PR 合并的全流程。

## 8 步完整流程

| 步骤 | 名称 | 执行者 | 耗时 |
|------|------|--------|------|
| 1 | 需求理解与拆解 | OpenClaw | 即时 |
| 2 | 启动代理 (worktree + tmux) | OpenClaw | 1 分钟 |
| 3 | 自动监控 (每 10 分钟检查) | cron 脚本 | 自动 |
| 4 | Agent 创建 PR | Codex/Claude | 30-60 分钟 |
| 5 | 自动化 Code Review (3 个 Agent) | Codex/Gemini/Claude | 并行 |
| 6 | 自动化测试 (CI 管道) | CI 系统 | 并行 |
| 7 | 人工 Review | 开发者 | 5-10 分钟 |
| 8 | 合并 + 清理 | cron 任务 | 自动 |

**总耗时**：1-2 小时（从需求到上线）  
**人工投入**：10 分钟（仅 Step 7）

## "完成"的定义

- ✅ PR 已创建
- ✅ 分支已同步到 main（没有冲突）
- ✅ CI 通过（lint、类型检查、单元测试、E2E 测试）
- ✅ Codex reviewer 通过
- ✅ Claude Code reviewer 通过
- ✅ Gemini reviewer 通过
- ✅ 如果有 UI 改动，必须包含截图

## 自动化 Code Review 策略

| 审查者 | 特点 | 使用策略 |
|--------|------|----------|
| Codex Reviewer | 最靠谱，擅长发现边界情况 | 主力审查者 |
| Gemini Code Assist | 免费，能发现安全问题 | 不用白不用 |
| Claude Code Reviewer | 过度谨慎，建议多为过度设计 | 除非"critical"，否则跳过 |

## CI 检查清单

- Lint 和 TypeScript 检查
- 单元测试
- E2E 测试
- Playwright 测试（预览环境）
- **新规则**：UI 改动必须附截图，否则 CI 失败

## 核心观点

> 作者的 review 只需要 5-10 分钟。很多 PR 他甚至不看代码，只看截图就直接合并了。

> 这个监控脚本是 100% 确定性的，非常省 token，只在需要人工介入时才会通知作者。

## tmux 干预技巧

```bash
# 代理方向错了
tmux send-keys -t codex-templates "停一下。先做 API 层，别管 UI。" Enter

# 代理需要更多上下文
tmux send-keys -t codex-templates "类型定义在 src/types/template.ts，用那个。" Enter
```

## 实施步骤

1. 设置 git worktree 隔离环境
2. 配置 tmux 会话管理
3. 编写 cron 监控脚本
4. 配置多 Agent Code Review
5. 设置 CI 检查清单（含截图规则）
6. 配置 Telegram 通知

## 待验证要点

1. 国内 CI 工具的选择（GitHub Actions vs 自建）
2. 截图自动生成的实现方式
3. 多 Agent Review 的 token 成本

## 参考

- [[20260306-OpenClaw-Claude Code 超强教程一个人搭建开发团队]] - 完整工作流与 Code Review 策略
