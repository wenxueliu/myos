---
title: "Karpathy一夜6万star，AutoResearch疯狂进化一个月：扒完4个项目，有点慌"
date: 2026-03-30
source: https://mp.weixin.qq.com/s/CltIs_qpIlMUAlIfxMackQ
tags:
  - AutoResearch
  - Karpathy
  - AI Agent
  - 自动研究
  - OpenClaw
  - 自主循环
---

# Karpathy一夜6万star，AutoResearch疯狂进化一个月：扒完4个项目，有点慌

> Karpathy 扔出一个 600 行的脚本，GitHub 一夜之间冒出十几个"自动研究"项目。这股浪潮背后，藏着 AI Agent 最核心的进化方向。

---

## 核心结论

AI Agent 从"听指令干活"进化到"自己定目标、自己做实验、自己迭代"的分水岭。

核心循环：**修改 → 验证 → 保留或丢弃 → 重复**（科学方法本身）

---

## 一、Karpathy 原版：60.9k star

**三个文件**：
- `prepare.py` —— 数据准备，不动
- `train.py` —— 模型训练代码，AI 改这个
- `program.md` —— 给 AI 的指令，人改这个

**设计亮点**：
- **固定时间预算**：每次训练5分钟，睡一晚上跑100个实验
- **program.md 就是 Skill**：用自然语言"编程"研究组织
- **指标是铁律**：验证集 bits per byte，越低越好

83个实验，从 baseline 0.998 降到 0.978。

---

## 二、工程化扩展：从"训练模型"到"万物皆可循环"

### Codex-autoresearch（867 star）：7种模式

| 模式 | 干什么 | 典型场景 |
| --- | --- | --- |
| **loop** | 朝着指标迭代 | 提升测试覆盖率 |
| **plan** | 分析代码库，推荐指标 | "我想优化但不知道测什么" |
| **debug** | 假设-验证法找bug | API随机503 |
| **fix** | 逐个修复错误直到清零 | 12个测试挂了 |
| **security** | STRIDE+OWASP安全审计 | 上线前安全检查 |
| **ship** | 发布检查清单 | PR/部署/发版 |
| **exec** | CI/CD无人值守模式 | GitHub Actions里跑 |

**分级恢复策略**：
- 连续丢弃3次 → REFINE（微调当前策略）
- 连续丢弃5次 → PIVOT（换个方向）
- 2次PIVOT没用 → 上网搜
- 3次PIVOT无果 → 停下来找人

### Claude-autoresearch（2.8k star）：9种命令

新增三个模式：
- **scenario** —— 场景探索器，12个维度生成边界用例
- **predict** —— 多人格预测，5个"专家"同时分析
- **learn** —— 自动文档引擎

**Guard机制**：Verify验"指标有没有变好"，Guard验"别的东西有没有坏掉"。

---

## 三、AutoResearchClaw：终极形态（9.5k star）

23个阶段，从"研究XX"到输出可以投NeurIPS的论文，全程无人干预。

```
A: 确定研究方向 → B: 文献搜索 → C: 知识综合 → D: 实验设计
→ E: 跑实验 → F: 分析决策 → G: 写论文 → H: 最终出品
```

**核心设计**：
- **真文献，不瞎编**：四层引用验证，发现幻觉引用直接删
- **实验会自愈**：代码跑崩了？AST分析错误原因，LLM修复，最多试10轮
- **多Agent辩论**：假设生成、结果分析、同行评审，多视角达成共识
- **反编造系统**：VerifiedRegistry强制数据来自真实实验

**MetaClaw跨实验记忆**：
- 第N次运行 → 提取经验教训 → 转化为可复用Skill
- 第N+1次运行 → 自动注入历史经验

实测数据：阶段重试率降低24.8%，微调轮数减少40%。

---

## 四、四项目横向对比

| 维度 | Karpathy原版 | Codex版 | Claude版 | AutoResearchClaw |
| --- | --- | --- | --- | --- |
| Star数 | 60.9k | 867 | 2.8k | 9.5k |
| 定位 | ML训练优化 | 软件工程全域 | 软件+非技术 | 学术研究全流程 |
| Agent数 | 1 | 1（可并行3） | 1 | 多Agent协作 |
| 学习能力 | 无 | 跨session lessons | 无 | MetaClaw跨run |
| 输出物 | 优化后的train.py | 改进后的代码 | 改进后的代码 | 完整论文+实验+LaTeX |

---

## 五、四个值得关注的方向

**1. "循环"比"一次性"更重要**

闭环迭代——AI做了修改，自己验证效果，自己决定保留还是回滚。

**2. 机械验证是根基**

只信可量化的指标：测试通过率、loss值、type error数量。

**3. 跨session记忆是下一个战场**

Karpathy原版没有记忆。Codex版加了lessons文件。AutoResearchClaw做到了跨run学习。

**4. 多Agent协作会成为标配**

多角色分工：有人提假设，有人写代码，有人挑毛病，有人做决策。

---

## 核心金句

> "研究不再由肉脑完成...代码已经是一个超出人类理解的自修改二进制文件" —— Karpathy

> 真正的问题不是"AI能不能做研究"——它已经在做了。真正的问题是：**当AI能24小时不休息地跑实验、写论文、自我纠错，"人类研究者"这个角色会变成什么样？**

---

## 参考项目

- https://github.com/karpathy/autoresearch — 起源，60.9k star
- https://github.com/leo-lilinxiao/codex-autoresearch — Codex工程化版
- https://github.com/uditgoenka/autoresearch — Claude版
- https://github.com/aiming-lab/AutoResearchClaw — 全自动论文流水线，9.5k star
