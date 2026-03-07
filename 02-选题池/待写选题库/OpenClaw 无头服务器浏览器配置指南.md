---
标题：OpenClaw 无头服务器浏览器配置指南
阶段：待写
关联参考：[[20260306-Lighthouse-OpenClaw 浏览器配置教程]]
标签：[OpenClaw, 无头浏览器，服务器部署，Chrome, Linux]
创建日期：2026-03-06
来源：腾讯云开发者社区
---

# OpenClaw 无头服务器浏览器配置指南

## 选题背景

很多人在 Linux 服务器上部署 OpenClaw 后，浏览器状态一直是 `running: false`：
- 不知道需要单独安装 Chrome
- 不知道需要配置 NoSandbox
- 不知道路径怎么填

需要一篇完整的无头服务器浏览器配置指南。

## 核心步骤

### 1. 安装 Chrome 浏览器
- Debian/Ubuntu 用 .deb 包
- CentOS/OpencloudOS 用 .rpm 包
- 关键：安装字体防止文字加载失败

### 2. OpenClaw 配置
- `browser.defaultProfile`: "openclaw"
- `browser.headless`: true
- `browser.noSandbox`: true
- `browser.executablePath`: "/usr/bin/google-chrome"

### 3. 验证
- `openclaw gateway restart`
- `openclaw browser start`
- `openclaw browser status` → running: true

## 核心观点

> OpenClaw 的浏览器配置并不难，只要避开默认配置的坑，它就能在服务器里稳定运行。

> 大多数 Linux 服务器未预置 Google 软件源，直接包管理器安装容易失败，最稳妥的方式是直接下载官方安装包。

## 待验证要点

1. 不同 Linux 发行版的安装命令差异
2. 字体包是否必须（对中文页面的影响）
3. 非 root 用户运行时的权限配置

## 参考

- [[20260306-Lighthouse-OpenClaw 浏览器配置教程]] - 完整安装步骤
