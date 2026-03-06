---
标题：OpenAI GPT-5.4 官方发布核心能力拆解
阶段：灵感
关联参考：[]
标签：[OpenAI, GPT-5.4, Agent, 计算机使用，工具调用，多模态]
创建日期：2026-03-06
来源：OpenAI 官方博客
source: https://openai.com/index/introducing-gpt-5-4/
author: OpenAI
---

# OpenAI GPT-5.4 官方发布核心能力拆解

> GPT-5.4 是 OpenAI 最强大、最高效的专业工作前沿模型，集成了推理、编码和 Agent 工作流的最新进展。

---

## 🎯 核心定位

- **发布渠道**：ChatGPT、API、Codex
- **双版本**：GPT-5.4（标准版）+ GPT-5.4 Pro（高性能版）
- **关键突破**：首个具备**原生计算机使用能力**的通用模型

---

## 📊 核心性能对比

| 基准测试 | GPT-5.4 | GPT-5.3-Codex | GPT-5.2 |
|---------|---------|---------------|---------|
| **GDPval**（专业知识工作） | 83.0% | 70.9% | 70.9% |
| **SWE-Bench Pro**（软件工程） | 57.7% | 56.8% | 55.6% |
| **OSWorld-Verified**（计算机使用） | 75.0% | 74.0% | 47.3% |
| **Toolathlon**（工具调用） | 54.6% | 51.9% | 46.3% |
| **BrowseComp**（网页研究） | 82.7% | 77.3% | 65.8% |

> GDPval 测试覆盖 44 种职业的知识工作，GPT-5.4 在 83% 的情况下达到或超过行业专业人士水平。

---

## 🔑 六大核心能力升级

### 1. 原生计算机使用（Computer Use）

**首个通用模型具备原生计算机操作能力**

- 可通过 Playwright 等库编写代码操作电脑
- 支持截图驱动的鼠标/键盘命令
- 开发者可通过 custom confirmation policies 配置安全行为

**基准表现**：
- OSWorld-Verified：75.0%（超越人类 72.4%）
- WebArena-Verified：67.3%（浏览器使用）
- Online-Mind2Web：92.8%（仅用截图）

---

### 2. 工具搜索（Tool Search）

**解决多工具场景的 token 效率问题**

传统方式：所有工具定义 upfront 放入 prompt → 数千到数万 tokens

新方式：轻量级工具列表 + 按需查找定义

**效果**：
- MCP Atlas 基准测试（36 个 MCP 服务器）
- **Token 使用减少 47%**
- 准确率保持不变

---

### 3. 视觉与文档理解

**高密度、高分辨率图像处理**

- `original` 输入级别：支持最高 10.24M 像素或 6000 像素最大维度
- `high` 输入级别：支持最高 2.56M 像素或 2048 像素最大维度

**基准表现**：
- MMMU-Pro（视觉理解）：81.2%（无工具）
- OmniDocBench（文档解析）：0.109 错误率（优于 GPT-5.2 的 0.140）

---

### 4. 编码能力集成

**融合 GPT-5.3-Codex 的编码优势**

- 匹配或超越 GPT-5.3-Codex 在 SWE-Bench Pro 的表现
- 延迟更低
- Codex `/fast` 模式：**1.5 倍 token 速度**

**新增实验性技能**：
- `Playwright (Interactive)`：可视化调试 Web/Electron 应用
- 可在构建过程中实时测试应用

---

### 5. 深度网页研究

**GPT-5.4 Thinking 的增强能力**

-  upfront 提供思考计划，用户可中途调整方向
- 更持久的多轮搜索能力
- 更好的长上下文保持

**BrowseComp 表现**：
- GPT-5.4：82.7%
- GPT-5.4 Pro：89.3%（新 SOTA）
- GPT-5.2：65.8%

---

### 6. 事实准确性提升

**降低幻觉和错误**

在用户标记事实错误的去标识化 prompt 测试中：
- **单个声明虚假率降低 33%**
- **完整响应错误率降低 18%**

---

## 💼 专业工作场景优化

### 电子表格与演示文稿

| 任务 | GPT-5.4 | GPT-5.2 |
|------|---------|---------|
| 投行建模任务 | 87.3% | 68.4% |
| 演示文稿评估（人类偏好） | 68.0% | - |

**配套发布**：
- ChatGPT for Excel 插件（企业用户）
- 更新的 [spreadsheet skills](https://github.com/openai/skills/tree/main/skills/.curated/spreadsheet) 和 [presentation skills](https://github.com/openai/skills/tree/main/skills/.curated/slides)

---

## 🔒 安全与监控

### 网络安全能力分级

- 按 Preparedness Framework 归类为 **High cyber capability**
- 扩展网络安全栈：监控系统、可信访问控制、异步阻塞

### 思维链（CoT）可控性

**新开源评估**：[CoT controllability](https://openai.com/index/reasoning-models-chain-of-thought-controllability/)

- 测试模型是否能故意模糊推理以逃避监控
- **GPT-5.4 Thinking 的 CoT 控制能力低** → 正面安全属性
- 说明模型无法隐藏推理，CoT 监控仍然有效

---

## 💰 API 定价

| 模型 | 输入价格 | 缓存输入 | 输出价格 |
|------|---------|---------|---------|
| gpt-5.2 | $1.75/M | $0.175/M | $14/M |
| **gpt-5.4** | **$2.50/M** | **$0.25/M** | **$15/M** |
| gpt-5.2-pro | $21/M | - | $168/M |
| **gpt-5.4-pro** | **$30/M** | - | **$180/M** |

> 虽然单价更高，但 token 效率提升可降低总体使用量

---

## 📅 发布计划

| 平台 | 版本 | 可用性 |
|------|------|--------|
| **API** | gpt-5.4 / gpt-5.4-pro | 立即发布 |
| **ChatGPT** | GPT-5.4 Thinking | Plus/Team/Pro 用户（即日起） |
| **ChatGPT** | GPT-5.4 Pro | Pro/Enterprise 计划 |
| **Codex** | GPT-5.4 | 逐步发布 |

**GPT-5.2 Thinking 退役时间**：2026 年 6 月 5 日

---

## 🔗 相关资源

- [API 文档](https://developers.openai.com/api/docs/guides/latest-model)
- [系统卡](https://deploymentsafety.openai.com/gpt-5-4-thinking)
- [CoT 可控性研究](https://openai.com/index/reasoning-models-chain-of-thought-controllability/)
- [GDPval 基准](https://openai.com/index/gdpval/)

---

**原文**：[OpenAI - Introducing GPT-5.4](https://openai.com/index/introducing-gpt-5-4/)
**归档**：2026-03-06
