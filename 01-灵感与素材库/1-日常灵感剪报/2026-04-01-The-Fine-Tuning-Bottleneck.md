---
title: "The Fine-Tuning Bottleneck Isn't the Algorithm"
date: "2026-04-01"
tags: ["AI", "Fine-Tuning", "Fireworks-AI", "Agent", "SFT", "RFT", "DPO"]
source: "https://fireworks.ai/blog/fine-tuning-bottlenecks
关联参考:
  - "https://fireworks.ai/blog/fine-tuning-bottlenecks"
description: "Fireworks AI 观点：微调的真正瓶颈不是算法，而是集成、迭代速度和选择合适方法"
---

# The Fine-Tuning Bottleneck Isn't the Algorithm

> 来源：Fireworks AI Blog
> 发布日期：2026-03-28

## 核心观点

团队往往把微调表现不佳归咎于算法，但实际的摩擦点在于：

- 奖励函数与内部 API 的连接
- 两周才出一轮实验结果
- 确定 SFT、RFT、DPO 哪种方法适合

**每个微调项目的最终目标**：构建特定领域的 Agent——代码修复、客服、深度研究、金融运营。

## 案例：Genspark Deep Research

Genspark 在封闭前沿模型上达到 0.76 奖励分数天花板后，转向在开放模型上用 RFT，最终达到 **0.82**——这是单纯提示词工程无法实现的跃升。

## 三大瓶颈

### 1. 集成与数据主权

许多组织要求敏感业务逻辑和专有数据留在内部环境。

Fireworks 的解决方案：
- **Training API**：数据永不离开客户环境
- **BYOB 存储**：客户 VPC 内的安全远程执行环境

### 2. 迭代速度

训练任务本身很少是瓶颈——**周期时间**才是。

问题：团队花数周搭建基础设施、策划数据集、运行任务，然后通过离线评估发现模型仍不达标。

成功团队的秘诀：
- 高频任务提交 + 快速反馈循环
- **Cursor Composer 2**：约每 5 小时出一个新 checkpoint

> 有一个客户在数周内跑了 **100+ 个任务**

### 3. 选择正确的方法

方法应该跟随问题走，不是反过来：

| 方法 | 适用场景 |
|------|----------|
| **SFT** | 高质量示范数据 |
| **RFT** | 奖励信号比正确答案更清晰 |
| **DPO** | 存在强偏好对时对齐行为 |

常见模式：**组合使用**——先用 SFT 建立基线，再用 RFT 细化 Agent 行为。

## 从托管到完全控制

团队通常的演进路径：

1. **托管微调**（通过 API）
2. **Training API**（需要自定义损失函数、optimizer-step 访问或全参数更新）

## 未来方向：Agentic Fine-Tuning Loops

愿景：
- 自动观察生产环境错误
- 选择合适的训练方法
- 配置运行、验证结果、部署改进
- 无需人工干预

> "人类定义什么是好的，设置护栏。剩下的由系统完成。"

## 金句

> 微调的瓶颈不是算法，而是集成、迭代速度和选择合适方法。
