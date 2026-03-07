---
标题：Lighthouse × OpenClaw：手把手教你搞定浏览器配置与启动
来源：腾讯云开发者社区 - 时来之笔
原文链接：https://cloud.tencent.com/developer/article/2626186
捕获日期：2026-03-06
标签：[OpenClaw, 浏览器配置，Lighthouse, 无头浏览器，服务器部署]
---

# Lighthouse × OpenClaw：手把手教你搞定浏览器配置与启动

**作者**：时来之笔  
**发布**：腾讯云开发者社区  
**修改时间**：2026-02-03  
**阅读量**：13.1K

---

## 核心问题

很多用户安装完 OpenClaw 后，运行 `openclaw browser status` 发现状态一直是 `running: false` 而且 `browser: unknown`。

> 这其实是我们在 linux 服务器环境下并没有安装浏览器的原因。

OpenClaw 的浏览器组件在无桌面环境（Headless）下需要特定的"打开方式"。

---

## 环境说明

- **服务器**：Lighthouse 轻量应用服务器
- **系统**：OpencloudOS 9
- **终端**：OrcaTerm（Lighthouse 自带 Web Shell）

---

## 步骤 1：安装 Chrome 浏览器

大多数 Linux 服务器未预置 Google 软件源，直接使用包管理器安装容易失败。最稳妥的方式是直接下载官方安装包。

### OpencloudOS/CentOS/Fedora (.rpm)

```bash
# 下载安装包
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm

# 安装依赖包
yum install -y liberation-fonts
yum install -y xdg-utils

# 安装字体，防止浏览器文字加载不出来
yum install -y google-noto-sans-cjk-fonts

# 安装 google 包（用 yum localinstall，自动补齐依赖）
yum localinstall -y ./google-chrome-stable_current_x86_64.rpm

# 确认安装路径
which google-chrome
# 通常返回：/usr/bin/google-chrome
```

### Debian/Ubuntu (.deb)

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

---

## 步骤 2：OpenClaw 浏览器配置

OpenClaw 的安装脚本并没有帮我们安装 Chrome，它不知道浏览器在哪，所以需要手动配置。

```bash
# 设置默认使用 openclaw 模式
openclaw config set browser.defaultProfile "openclaw"

# 开启无桌面模式
openclaw config set browser.headless true

# 强制开启 NoSandbox（否则 root 用户运行时 Chrome 拒绝启动）
openclaw config set browser.noSandbox true

# 确认 chrome 路径
openclaw config set browser.executablePath "/usr/bin/google-chrome"

# 重启 OpenClaw Gateway
openclaw gateway restart

# 启动浏览器
openclaw browser start

# 验证状态
openclaw browser status
# 预期输出：running: true
```

---

## 常见问题

### 问题 1：which google-chrome 找不到

检查安装是否成功，重新执行安装步骤。

### 问题 2：配置后飞书机器人仍提示无法使用浏览器

检查配置文件中 `profile` 是否为 `messaging`，将其改成 `full`。

### 问题 3：浏览器打开后分辨率很低

需要调整 viewport 配置。

### 问题 4：无法访问网络

检查服务器网络配置和防火墙设置。

---

## 核心金句

> OpenClaw 的浏览器配置并不难，只要避开默认配置的坑，它就能在 Lighthouse 里稳定运行。

> 现在的你已经拥有了一个可以 24 小时自动上网冲浪的 AI 助手了。

---

## 可提炼选题

1. **OpenClaw 无头服务器浏览器配置完全指南** - 针对 Linux 服务器的浏览器安装与配置
2. **Lighthouse 部署 OpenClaw 避坑手册** - 腾讯云服务器专属部署教程
3. **OpenClaw 浏览器常见报错排查清单** - 汇总评论区高频问题与解决方案

---

## 评论区高频问题

| 问题 | 状态 |
|------|------|
| which google-chrome 找不到 | 安装路径问题 |
| 如何处理登录账号与密码问题 | 待解答 |
| 配置后飞书仍提示无法使用浏览器 | profile 配置问题 |
| 浏览器分辨率低 | viewport 配置问题 |
| 无法访问网络 | 网络/防火墙问题 |
