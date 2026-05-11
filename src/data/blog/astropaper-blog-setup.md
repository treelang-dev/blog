---
title: 使用 AstroPaper 搭建个人技术博客
author: Treelang
pubDatetime: 2026-05-09T12:00:00Z
slug: astropaper-blog-setup
featured: true
draft: false
tags:
  - Astro
  - AstroPaper
  - Blog
  - Cloudflare
description: 记录使用 AstroPaper 模板搭建个人技术博客的过程，包括项目创建、站点配置、Markdown 文章编写、Cloudflare Pages 部署和 blog.treelang.me 子域名绑定。
---

## 前言

在完成个人主页之后，我希望再搭建一个独立的技术博客，用来记录平时的项目部署、问题排查、学习笔记和技术折腾过程。个人主页更适合展示“我是谁”和“我做过什么”，而博客更适合记录具体过程，例如 Cloudflare 配置、Android 自动构建、Debian 服务器部署、青龙面板和工程仿真相关内容。

这次博客使用 AstroPaper 模板搭建。AstroPaper 是一个基于 Astro 的博客模板，支持 Markdown 写作、标签分类、搜索、RSS、深浅色模式和分页功能。对于个人技术博客来说，它的功能已经比较完整，不需要自己从零写博客系统。

## 博客规划

博客计划主要记录以下几个方向：

- Cloudflare
- Android Development
- Battery Simulation
- Linux
- Personal Server
- Automation
- Life

这些分类基本覆盖了我目前主要折腾的内容。前期博客不会追求特别复杂的功能，重点是把真实的配置过程、报错信息和解决方法记录下来。

## 创建项目

进入用户目录：

```powershell
cd C:\Users\Treelang
```

使用 Astro 官方命令创建项目：

```powershell
npm.cmd create astro@latest -- --template satnaing/astro-paper
```

项目名称填写：

```text
treelang-blog
```

如果提示是否安装依赖，选择 Yes。如果提示是否初始化 Git 仓库，也可以选择 Yes。

创建完成后进入项目：

```powershell
cd C:\Users\Treelang\treelang-blog
```

启动本地开发服务器：

```powershell
npm.cmd run dev
```

浏览器打开：

```text
http://localhost:4321
```

即可看到博客模板页面。

## 修改站点信息

博客的主要配置文件通常在：

```text
src/config.ts
```

或者类似的配置文件中。需要修改站点标题、作者、描述、网站地址和社交链接。

例如可以将网站信息设置为：

```ts
export const SITE = {
  website: "https://blog.treelang.me",
  author: "Treelang",
  profile: "https://treelang.me",
  desc: "记录工程仿真、Android Development、Personal Server、Cloudflare 与 Automation 实践。",
  title: "Treelang Blog",
  lightAndDarkMode: true,
  postPerPage: 3,
};
```

具体字段可能会因为模板版本不同而略有差异。修改时不要盲目整段覆盖，应该按照原文件结构修改对应内容。

## 文章目录

AstroPaper 的文章通常放在：

```text
src/data/blog
```

每一篇文章都是一个 Markdown 文件，例如：

```text
cloudflare-pages-homepage.md
astropaper-blog-setup.md
debian-12-mini-server.md
```

每篇文章开头都需要写 frontmatter，用来声明标题、作者、发布时间、slug、标签和文章描述。

标准格式如下：

```md
---
title: 文章标题
author: Treelang
pubDatetime: 2026-05-09T12:00:00Z
slug: article-slug
featured: false
draft: false
tags:
  - Tag1
  - Tag2
description: 文章简短描述。
---
```

如果 `draft` 设置为 `true`，文章通常不会在正式页面中显示。写完准备发布时，需要改成：

```yaml
draft: false
```

## 写第一篇文章

可以在 `src/data/blog` 中新建：

```text
cloudflare-pages-homepage.md
```

文章内容用于记录个人主页的搭建过程。写技术文章时，我比较推荐采用以下结构：

```md
## 前言

说明为什么写这篇文章。

## 环境说明

列出系统、工具和项目版本。

## 实现过程

按步骤记录实际操作。

## 遇到的问题

记录报错、原因和解决方法。

## 最终效果

说明实现了什么。

## 总结

整理收获和后续计划。
```

这种结构比较适合部署类文章，因为后续自己查阅时会比较清楚。

## 删除模板自带文章

模板中通常会自带一些示例文章。正式使用前，可以进入：

```text
src/data/blog
```

删除不需要的示例文件，只保留自己写的文章。

如果担心以后忘记格式，可以保留一篇示例文章作为模板，或者新建一个不发布的草稿：

```text
example-draft-post.md
```

并设置：

```yaml
draft: true
```

## 本地构建测试

写完文章后，需要测试项目能否正常构建：

```powershell
npm.cmd run build
```

如果 frontmatter 缺少必要字段，Astro 会报错。例如：

```text
title: Required
description: Required
pubDatetime: Required
```

遇到这种错误时，需要检查文章开头的 `---` 是否正确，以及字段名是否写错。

特别注意，不建议用 Windows 自带记事本编辑 Markdown 文件，因为有时可能会自动改变符号格式。建议使用 VS Code。

## 部署到 Cloudflare Pages

在 GitHub 新建仓库：

```text
treelang-blog
```

然后推送代码：

```powershell
git add .
git commit -m "init treelang blog"
git branch -M main
git remote add origin https://github.com/treelang-dev/treelang-blog.git
git push -u origin main
```

进入 Cloudflare：

```text
Workers & Pages
→ Create application
→ Pages
→ Connect to Git
```

选择博客仓库后，构建配置如下：

```text
Framework preset: Astro
Build command: npm run build
Build output directory: dist
```

部署成功后，会获得默认地址：

```text
treelang-blog.pages.dev
```

## 绑定 blog.treelang.me

进入博客 Pages 项目的 Custom domains 页面，添加：

```text
blog.treelang.me
```

如果 Cloudflare 没有自动创建 DNS 记录，可以手动添加：

```text
Type: CNAME
Name: blog
Target: treelang-blog.pages.dev
Proxy status: Proxied
TTL: Auto
```

等待几分钟后，访问：

```text
https://blog.treelang.me
```

如果能够正常打开，说明博客部署成功。

## 和个人主页关联

博客部署成功后，可以回到个人主页项目，在项目列表或导航中添加：

```text
Blog → https://blog.treelang.me
```

这样个人主页和博客就形成了完整入口：

```text
treelang.me       个人主页
blog.treelang.me  技术博客
```

## 总结

这次博客搭建完成后，后续写文章就不需要再重复部署流程。每次只需要在 `src/data/blog` 中新建 Markdown 文件，写完后提交到 GitHub，Cloudflare Pages 会自动构建并部署。

相比把笔记分散放在本地文件夹里，博客可以让技术记录更加系统，也更方便长期积累。
