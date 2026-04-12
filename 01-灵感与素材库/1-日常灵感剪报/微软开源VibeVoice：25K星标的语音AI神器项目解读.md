---
title: "微软开源VibeVoice：25K星标的语音AI神器项目解读"
公众号: "世界那么哒"
date: "1970-01-01 08:33"
source: "https://mp.weixin.qq.com/s/nsMrbol6Gi7waWHYutBUmg"
---

![image](images/img_001.png)

**项目资源**：

- • 🏠 项目主页：aka.ms/VibeVoice
- • 💻 代码仓库：github.com/microsoft/VibeVoice
- • 🎮 在线体验：aka.ms/VibeVoice-Demo
- • 📒 arXiv报告: arxiv.org/pdf/2508.19205


## 论文摘要


> This report presents VibeVoice, a novel model designed to synthesize long-form speech with multiple speakers by employing next-token diffusion, which is a unified method for modeling continuous data by autoregressively generating latent vectors via diffusion. To enable this, we introduce a novel continuous speech tokenizer that, when compared to the popular Encodec model, improves data compression by 80 times while maintaining comparable performance.

> The tokenizer effectively preserves audio fidelity while significantly boosting computational efficiency for processing long sequences. Thus, VibeVoice can synthesize long-form speech for up to 90 minutes (in a 64K context window length) with a maximum of 4 speakers, capturing the authentic conversational "vibe" and surpassing open-source and proprietary dialogue models.

**核心要点**：

- • Next-Token Diffusion：统一方法，通过扩散自回归生成潜在向量来建模连续数据
- • 80倍压缩率：相比 Encodec 模型大幅提升数据压缩效率
- • 90分钟合成：64K 上下文窗口，最长可合成 90 分钟音频
- • 4人对话：支持最多 4 个说话人的多角色对话
- • 超越同类：在客观和主观指标上均超越开源和闭源对话模型


## 一、为什么VibeVoice值得你关注？


### 1. 来自微软的顶级技术背书

微软作为全球科技巨头，其开源项目通常代表着业界的最高水平。VibeVoice不仅技术先进，而且文档完善、社区活跃。


### 2. 真正解决痛点

- • 长音频处理难：传统模型只能处理短片段，VibeVoice支持90分钟单次处理
- • 多说话人难：4人对话无缝切换，告别"机器人腔"
- • 实时响应难：~200ms首音频延迟，流式输出


### 3. 完全开源可用

采用MIT协议，模型权重托管在HuggingFace，可以免费商用（需遵守协议）。


## 二、核心技术架构

VibeVoice 有两大核心创新：


### Feature 1: 超长音频合成能力

![image](images/img_002.png)

Feature 1: 超长音频合成能力论文中的图1展示了 VibeVoice 的核心能力：**VibeVoice 能够合成超过 5000 秒（即 83+ 分钟）的音频**，同时在主观评估中始终超越最强的开源和闭源系统。

关键数据：

- • 合成时长：5000+ 秒（超过 83 分钟）
- • 主观评估：在偏好度（Preference）、真实感（Realism）、丰富度（Richness）上均超越所有对比系统
- • 上下文窗口：64K tokens

这意味着 VibeVoice 可以一次性生成一集播客、一场会议纪要或一段完整有声书章节。


### Feature 2: Next-Token Diffusion 架构

![image](images/img_003.png)

Feature 2: Next-Token Diffusion 架构论文原文：*"Voice prompts and text scripts provide initial input. VibeVoice processes hybrid context features, and its hidden states condition a token level Diffusion Head (D), which predicts acoustic VAE for speech segments, subsequently recovered by acoustic decoder (A)."*

**中文解读**：

VibeVoice 的工作流程如下：

1. 1. 输入阶段：提供语音提示（Voice prompts）和文本脚本（Text scripts）作为初始输入
2. 2. 特征融合：VibeVoice 处理混合上下文特征（hybrid context features），将语音特征和文本特征无缝融合
3. 3. 条件生成：LLM 的隐藏状态（hidden states）作为条件，指导 Token 级 Diffusion Head (D) 进行生成
4. 4. 声学预测：Diffusion Head 预测声学 VAE 特征，用于生成语音片段
5. 5. 音频重建：声学分词器的解码器（Acoustic Decoder A）将 VAE 特征恢复为最终音频输出

**架构优势**：

- • 端到端统一：无需级联多个模型，消除误差累积
- • 高效去噪：仅需 10 步迭代（DPM-Solver++）
- • 高质量生成：CFG 引导系数 1.3，保证生成质量


## 三、两大核心技术详解


### 1. 超低帧率语音分词器 (7.5 Hz)

VibeVoice 创新性地采用 **7.5 Hz 帧率**的连续语音分词器，相比流行的 Encodec 模型实现 **80倍压缩率提升**。

**声学分词器 (Acoustic Tokenizer)**：

- • 采用 σ-VAE 变分自编码器架构
- • 镜像对称的编码器-解码器结构
- • 7 stage Modified Transformer blocks（含 1D 深度可分离因果卷积）
- • 6层下采样，从 24kHz 输入实现累积 3200X 下采样
- • 每个编码器/解码器约 340M 参数
- • 语音-文本比：约 2:1（两个语音token ≈ 一个 BPE 文本token）

**语义分词器 (Semantic Tokenizer)**：

- • 采用与声学分词器相同的层级架构
- • 使用 ASR（自动语音识别） 作为代理任务训练
- • 将语义编码器的表示与文本语义对齐

**音频重建性能**：

![image](images/img_004.png)

- 不同语音tokenizer在LibriTTS上的重建效果，关键指标是音质（PESQ）、可懂度（STOI）和整体质量（UTMOS），同时关注token压缩率（Token Rate）。


### 2. LLM + Diffusion Head 协同                                                组件规格LLM 骨干Qwen2.5（1.5B / 7B）扩散头4层训练策略课程学习（4096 → 65536 tokens）上下文窗口64K tokens去噪步数10 步（DPM-Solver++）CFG 引导系数1.3


## 四、VibeVoice-TTS：长文本语音合成


### 核心特性

- • 90分钟长文本生成：一次性合成最长90分钟对话或演讲音频
- • 4人对话支持：支持单次对话中最多4个说话人
- • 情感丰富：捕捉对话中的情感变化和自然节奏
- • 跨语言合成：支持英语、中文及多种语言


### 性能表现

VibeVoice 在播客场景下的表现优于业界顶级模型：

![image](images/img_005.png)


## 五、VibeVoice 模型家族

**三个模型，三种能力：**

**VibeVoice-ASR（7B参数）**：60分钟音频 → 结构化文本，用于会议纪要、字幕生成

**VibeVoice-TTS（1.5B参数）**：长文本 → 90分钟音频，用于有声书、播客制作

**VibeVoice-Realtime（0.5B参数）**：流式文本 → 实时音频，用于直播解说、虚拟人


## 六、VibeVoice-ASR：长语音识别


### 核心特性

- • 60分钟单次处理：支持在64K token长度内接受最长60分钟连续音频
- • 自定义热词：提供特定领域的专业词汇，显著提升识别准确率
- • 结构化转录（Who, When, What）：联合执行语音识别、说话人分离、时间戳标注
- • 50+语言原生支持：无需显式设置语言，原生支持多语言及语码切换


## 七、VibeVoice-Realtime：实时语音生成


### 核心特性

- • 参数量仅0.5B：轻量化设计，便于部署
- • 首音频延迟~200ms：流式输入，快速响应
- • 流式文本输入：支持边输入边生成
- • 长文本~10分钟：单次可生成约10分钟音频


## 八、典型应用场景

![image](images/img_006.png)

VibeVoice应用场景
## 九、快速上手

![image](images/img_007.png)

VibeVoice快速上手
### 安装代码


```
# 1. 克隆仓库git clone https://github.com/microsoft/VibeVoice.gitcd VibeVoice# 2. 安装依赖pip install -e .# 3. 启动ASR演示apt update && apt install ffmpeg -ypython demo/vibevoice_asr_gradio_demo.py \  --model_path microsoft/VibeVoice-ASR --share# 4. 或启动实时TTS演示python demo/vibevoice_realtime_demo.py \  --model_path microsoft/VibeVoice-Realtime-0.5B
```


### 模型下载                                            模型链接VibeVoice-ASR-7B🤗 HuggingFace[1]VibeVoice-TTS-1.5B🤗 HuggingFace[2]VibeVoice-Realtime-0.5B🤗 HuggingFace[3]

       
### 在线体验

- • ASR Playground[4]
- • Colab 实时TTS[5]


## 十、安全与责任

VibeVoice 作为研究框架，用户应当：

- • 确保转录内容的可靠性
- • 检查输出内容的准确性
- • 避免创建欺骗性内容
- • 遵守AI生成内容适当披露原则


## 写在最后

VibeVoice 的两大核心创新使其能够合成 **5000+ 秒（83+ 分钟）**的音频，同时在主观评估中始终超越所有开源和闭源系统。

无论是 **90分钟长音频合成**、**4人对话**，还是 **~200ms 实时生成**，VibeVoice 都交出了令人印象深刻的答卷。

随着 AI 技术的快速发展，语音交互正在成为人机协作的新范式。而 VibeVoice，正是这个新时代的开创者之一。

**相关链接**：

- • 项目主页[6]
- • GitHub 仓库[7]（25K+ Stars）
- • HuggingFace 模型[1]
- • arXiv 技术报告[8]

引用链接[1] 🤗 HuggingFace: *https://huggingface.co/microsoft/VibeVoice-ASR*
[2] 🤗 HuggingFace: *https://huggingface.co/microsoft/VibeVoice-1.5B*
[3] 🤗 HuggingFace: *https://huggingface.co/microsoft/VibeVoice-Realtime-0.5B*
[4] ASR Playground: *https://aka.ms/vibevoice-asr*
[5] Colab 实时TTS: *https://colab.research.google.com/github/microsoft/VibeVoice/blob/main/demo/vibevoice_realtime_colab.ipynb*
[6] 项目主页: *https://microsoft.github.io/VibeVoice/*
[7] GitHub 仓库: *https://github.com/microsoft/VibeVoice*
[8] arXiv 技术报告: *https://arxiv.org/pdf/2508.19205*

