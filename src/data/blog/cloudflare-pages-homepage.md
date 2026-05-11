---
title: 使用 Cloudflare Pages 搭建个人主页
author: Treelang
pubDatetime: 2026-05-08T12:00:00Z
slug: cloudflare-pages-homepage
featured: true
draft: false
tags:
  - Cloudflare
  - Pages
  - Astro
  - Portfolio
description: 记录使用 Astro 作品集模板和 Cloudflare Pages 搭建个人主页的过程，包括模板选择、本地修改、GitHub 自动部署、自定义域名绑定和重定向配置。
---

## 前言

之前一直想搭建一个属于自己的个人主页，用来整理项目、技术方向和一些正在折腾的东西。相比直接把所有内容放在 GitHub README 里，独立的个人主页会更加直观，也更适合作为长期展示入口。

这次我选择使用 Cloudflare Pages 部署个人主页，原因主要有三个。第一，Cloudflare Pages 对静态站点非常友好，可以直接连接 GitHub 仓库，代码更新后自动构建和部署。第二，Cloudflare 自带 CDN、HTTPS 和 DNS 管理功能，后续绑定自己的域名比较方便。第三，个人主页本身不需要复杂后端，用 Pages 部署足够简单，也便于后期维护。

本篇文章记录的是从选择模板到完成部署的完整过程，主要包括本地运行、修改配置、推送到 GitHub、Cloudflare Pages 部署，以及绑定自定义域名等内容。

## 项目选择

个人主页使用的是一个基于 Astro 和 Tailwind CSS 的作品集模板。这个模板的结构比较简单，主要通过配置文件控制个人信息、技能标签、项目展示、经历和教育背景。

我选择这种模板的原因是它不需要从零开始写界面，整体风格比较简洁，适合做个人作品集。后续如果想要调整颜色、图标或者项目内容，也只需要修改少量配置和组件文件。

主页主要展示以下内容：

- 个人简介
- 技能标签
- 项目经历
- 教育背景
- 联系方式
- GitHub、Telegram、微信等社交入口

由于网站目前主要用于个人展示，所以没有加入复杂的后端功能。

## 本地环境准备

在 Windows 上搭建这个项目之前，需要先安装几个基础工具：

- Git
- Node.js
- npm
- VS Code

安装 Git 后，可以在 PowerShell 中检查：

```powershell
git --version
```

安装 Node.js 后，可以检查：

```powershell
node -v
npm -v
```

如果 PowerShell 提示无法运行 `npm.ps1`，可以临时使用：

```powershell
npm.cmd install
npm.cmd run dev
```

或者修改当前用户的 PowerShell 执行策略：

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

## 克隆模板项目

进入用户目录：

```powershell
cd C:\Users\Treelang
```

克隆项目：

```powershell
git clone https://github.com/RyanFitzgerald/devportfolio.git treelang-home
cd treelang-home
```

安装依赖：

```powershell
npm.cmd install
```

启动本地开发服务器：

```powershell
npm.cmd run dev
```

正常情况下，终端会显示一个本地访问地址，例如：

```text
http://localhost:4321
```

打开浏览器访问这个地址，就可以看到本地预览页面。

## 修改个人信息

模板的主要内容集中在配置文件中，例如：

```text
src/config.ts
```

在这个文件中，可以修改个人名称、标题、简介、技能、项目和经历。

例如，我将首页标题修改为：

```ts
title: "Developer & Builder"
```

这样不会过度强调学校或专业，而是更偏向个人开发者主页的表达方式。

个人简介也进行了调整，重点放在工程仿真、Android 开发、个人服务器和自动化实践上，而不是反复强调学校背景。

## 增加社交图标

模板默认支持 Email、GitHub、LinkedIn、Twitter 等链接。如果需要加入 Telegram 或微信图标，需要在配置文件和组件中增加对应字段。

例如，在 `social` 中加入：

```ts
telegram: "https://t.me/treelang_dev",
wechat: "/wechat.jpg",
```

然后在组件中增加对应的 SVG 图标。微信图标可以设置为点击后打开二维码图片，因此需要把二维码放到：

```text
public/wechat.jpg
```

这样部署后可以通过以下路径访问：

```text
https://treelang.me/wechat.jpg
```

## 构建项目

修改完成后，需要先在本地测试构建：

```powershell
npm.cmd run build
```

如果没有报错，会生成：

```text
dist
```

这个文件夹就是最终要部署到 Cloudflare Pages 的静态文件。

也可以本地预览构建后的效果：

```powershell
npm.cmd run preview
```

## 推送到 GitHub

在 GitHub 新建仓库，例如：

```text
treelang-home
```

然后在本地执行：

```powershell
git remote remove origin
git remote add origin https://github.com/treelang-dev/treelang-home.git

git add .
git commit -m "init treelang homepage"
git branch -M main
git push -u origin main
```

如果推送时出现网络连接重置，可以先尝试：

```powershell
git config --global http.version HTTP/1.1
```

如果仍然无法推送，也可以先使用 Cloudflare Pages 的 Direct Upload，直接上传 `dist` 文件夹。

## 部署到 Cloudflare Pages

进入 Cloudflare 后台：

```text
Workers & Pages
→ Create application
→ Pages
→ Connect to Git
```

选择刚才的 GitHub 仓库后，构建配置如下：

```text
Framework preset: Astro
Build command: npm run build
Build output directory: dist
```

部署成功后，Cloudflare 会生成一个默认访问地址，例如：

```text
treelang-home.pages.dev
```

确认默认地址可以访问后，再绑定自己的域名。

## 绑定自定义域名

进入 Pages 项目的 Custom domains 页面，添加：

```text
treelang.me
```

如果需要让 `www.treelang.me` 跳转到 `treelang.me`，可以在 Cloudflare DNS 中添加：

```text
Type: A
Name: www
IPv4 address: 192.0.2.1
Proxy status: Proxied
```

然后在 Redirect Rules 中设置：

```text
https://www.treelang.me/*
```

跳转到：

```text
https://treelang.me/${1}
```

状态码选择：

```text
301 Permanent Redirect
```

这样访问 `www.treelang.me` 时，就会自动跳转到主域名。

## 最终效果

完成后，个人主页可以通过以下地址访问：

```text
https://treelang.me
```

页面主要包括个人简介、技能标签、项目展示、教育经历和社交链接。后续可以继续增加博客入口、项目详情页面、工具入口页面等内容。

## 总结

这次个人主页搭建主要完成了几个目标：使用 Astro 模板快速搭建页面，通过 Cloudflare Pages 实现自动部署，绑定自定义域名，并完成了基础的重定向配置。

整体过程并不复杂，比较适合个人作品集和技术主页。后续如果需要博客、文档站、短链接或 Webhook 接口，可以继续结合 Cloudflare Pages、Workers、Tunnel 和 Access 逐步扩展。
