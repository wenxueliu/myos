---
title: "BMAD，gstack or Superpowers，从零开发产品的设计架构工具选型策略"
公众号: "硅基鹿鸣"
date: "2026-03-27"
source: "https://mp.weixin.qq.com/s/_rmH9PzWGAWX24zgp7Uadw"
tags:
  - AI Coding
  - Claude Code
  - 设计框架
  - 产品设计
  - Superpowers
  - gstack
  - BMAD
关联参考: []
---

> 作者：陆徐洲，LIMS 公司 AI 算法负责人

## 核心内容

同一需求（检验科 AI 科研智能体）横向对比三个 Claude Code 框架：

### Superpowers（11万 star）
- **特点**：流程强制，7 阶段流水线 + TDD
- **体验**：选择题式追问，1.5 小时出 331 行设计文档
- **局限**：不挑战你，也不帮你多想一步

### BMAD（4.2万 star）
- **特点**：文档驱动，12+ Agent 角色分工
- **体验**：5 小时，角色扮演式脑暴，挖出 39 个产品概念
- **产出**：5 份 500 行规范化 md 文档，PRD 评分 8.6/10
- **代价**：烧 token 厉害，Max 20x 订阅 5 小时用掉 20%

### gstack（4.4万 star）
- **特点**：产品设计和 QA 见长
- **在需求挖掘上**有独到洞察（主动推送研究方向）

## 结论

- **BMAD 95 分**：适合项目最前期，把需求和产品设计做扎实
- **gstack 80 分**：做产品设计和 QA
- **Superpowers**：TDD 编码实现
- 组合用法：**gstack 做产品设计和 QA，Superpowers 做 TDD 编码实现，BMAD 在项目最前期用一次**
