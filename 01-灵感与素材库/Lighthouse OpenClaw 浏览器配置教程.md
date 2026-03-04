---
tags: [OpenClaw, 浏览器配置, Lighthouse, 教程]
created: 2026-03-04
source: https://cloud.tencent.com/developer/article/2626186
status: 素材
---

# Lighthouse × OpenClaw：浏览器配置与启动指南

> 很多用户安装完 OpenClaw 后，运行 `openclaw browser status` 发现状态一直是 `running: false` 而且 `browser: unknown`。别慌，其实 OpenClaw 的浏览器组件在无桌面环境（Headless）下需要特定的"打开方式"。

## 核心问题

在无桌面环境（Headless）服务器上配置 OpenClaw 浏览器组件时，常见坑点：

- **依赖缺失** - 缺少必要的浏览器运行时依赖
- **Root 权限报错** - 权限配置不当导致启动失败
- **状态异常** - `running: false` / `browser: unknown`

## 适用环境

- 腾讯云 Lighthouse 轻量应用服务器
- OrcaTerm 终端
- 无桌面环境（Headless）

## 关键标签

`#openclaw` `#clawdbot` `#moltbot` `#Lighthouse` `#轻量应用服务器` `#服务器配置` `#人工智能`

---

**原文链接**: [腾讯云开发者社区](https://cloud.tencent.com/developer/article/2626186)

**作者**: 云煮鱼

**收录专栏**: 时来之笔
