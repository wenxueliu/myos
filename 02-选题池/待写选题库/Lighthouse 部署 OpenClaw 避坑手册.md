---
标题：Lighthouse 部署 OpenClaw 避坑手册
阶段：待写
关联参考：[[20260306-Lighthouse-OpenClaw 浏览器配置教程]]
标签：[OpenClaw, Lighthouse, 腾讯云服务器，部署教程]
创建日期：2026-03-06
来源：腾讯云开发者社区
---

# Lighthouse 部署 OpenClaw 避坑手册

## 选题背景

Lighthouse 是腾讯云轻量应用服务器，很多人用它部署 OpenClaw：
- 系统预置环境不同
- OrcaTerm 终端有特殊配置
- 网络环境与本地不同

需要一篇 Lighthouse 专属的部署避坑手册。

## 核心环境

- **服务器**：Lighthouse 轻量应用服务器
- **系统**：OpencloudOS 9
- **终端**：OrcaTerm（Lighthouse 自带 Web Shell）
- **成本**：约 99 元/年（参考腾讯云活动）

## 核心坑点

### 坑 1：系统无预置浏览器
**表现**：`openclaw browser status` 显示 `running: false`  
**解决**：手动安装 Chrome

### 坑 2：Google 软件源缺失
**表现**：yum install google-chrome 失败  
**解决**：直接下载官方 .rpm 包

### 坑 3：Root 用户无法启动 Chrome
**表现**：Chrome 拒绝启动  
**解决**：配置 `browser.noSandbox: true`

### 坑 4：中文字体缺失
**表现**：浏览器文字显示为方块  
**解决**：安装 `google-noto-sans-cjk-fonts`

## 核心观点

> 避坑的本质是提前知道哪里会踩坑。

> 用对方法，OpenClaw 在 Lighthouse 上可以稳定运行，成为 24 小时 AI 助手。

## 待补充

1. Lighthouse 购买与初始化步骤
2. 完整部署时间估算
3. 与其他云服务器的对比（阿里云、华为云等）

## 参考

- [[20260306-Lighthouse-OpenClaw 浏览器配置教程]] - Lighthouse 专属教程
