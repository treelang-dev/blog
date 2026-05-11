---
title: 青龙面板自动化任务部署记录
author: Treelang
pubDatetime: 2026-05-13T12:00:00Z
slug: qinglong-automation-deployment
featured: false
draft: false
tags:
  - Qinglong
  - Automation
  - Docker
  - Cloudflare Tunnel
description: 记录青龙面板的部署和自动化任务配置过程，包括 Docker 部署、环境变量、定时任务、通知推送、远程访问方式和安全注意事项。
---

## 前言

青龙面板是一个常见的自动化任务管理工具，可以用来管理定时任务、脚本、环境变量和日志。对于个人服务器来说，它比较适合作为自动化任务中心，例如定时执行脚本、检查服务状态、发送通知或触发一些日常任务。

本篇文章记录的是青龙面板的基础部署和使用过程，包括 Docker 部署、任务配置、环境变量管理和远程访问方式。同时也会记录一些安全注意事项，避免将后台服务直接暴露到公网。

## 部署环境

本次部署环境大致如下：

- Debian 12
- Docker
- Docker Compose
- Cloudflare Tunnel
- Cloudflare Access
- Qinglong

青龙面板部署在个人微型服务器上，通过 Docker 运行。由于青龙面板中可能保存脚本、环境变量和日志信息，因此不建议直接暴露公网端口。

## 创建部署目录

首先创建青龙面板的部署目录：

```bash
mkdir -p ~/docker/qinglong
cd ~/docker/qinglong
```

在这个目录中存放 `docker-compose.yml` 和数据目录，方便后续备份和维护。

## 编写 Docker Compose 文件

创建文件：

```bash
nano docker-compose.yml
```

写入以下内容：

```yaml
services:
  qinglong:
    image: whyour/qinglong:latest
    container_name: qinglong
    restart: unless-stopped
    ports:
      - "5700:5700"
    volumes:
      - ./data:/ql/data
```

保存后启动：

```bash
docker compose up -d
```

查看容器状态：

```bash
docker ps
```

如果容器正常运行，可以在局域网访问：

```text
http://服务器IP:5700
```

## 初始化青龙面板

第一次访问青龙面板时，需要完成初始化设置，包括账号和密码配置。建议使用强密码，不要使用过于简单的默认密码。

初始化完成后，可以进入面板查看以下功能：

- 定时任务
- 环境变量
- 配置文件
- 脚本管理
- 任务日志
- 系统设置

## 添加定时任务

在青龙面板中，可以通过“定时任务”添加脚本任务。任务通常包括以下内容：

- 任务名称
- 执行命令
- 定时规则
- 是否启用
- 日志输出

例如，如果要定时执行一个 Python 脚本，可以设置命令：

```bash
python3 /ql/data/scripts/example.py
```

也可以执行 Shell 脚本：

```bash
bash /ql/data/scripts/example.sh
```

定时规则可以使用 cron 表达式。比如每天凌晨 1 点执行：

```text
0 1 * * *
```

## 环境变量管理

青龙面板支持集中管理环境变量，这对于脚本运行比较方便。比如某些脚本需要 token、cookie 或 API key，就可以放在环境变量中，而不是直接写进脚本。

需要注意的是，环境变量中可能包含敏感信息，因此：

- 不要把环境变量截图公开
- 不要把 token 写进博客
- 不要把 cookie 提交到 GitHub
- 不要在日志中输出敏感内容

写博客或记录部署过程时，所有密钥都应该用占位符代替。

## 通知推送

青龙面板可以结合通知工具实现任务结果推送。常见方式包括：

- Telegram Bot
- Server酱
- Bark
- 邮箱通知
- 企业微信机器人

如果使用 Telegram Bot，需要保存 bot token 和 chat id。相关内容也应该放在环境变量中，避免直接写到脚本或公开文章中。

## 远程访问方式

青龙面板运行在内网服务器上，如果只在局域网使用，直接访问：

```text
http://服务器IP:5700
```

即可。

如果需要在外网访问，不建议直接开放端口。更安全的方式有两种。

### 方式一：Tailscale 或 ZeroTier

这种方式适合个人使用。只有自己的设备加入同一个私有网络后，才能访问青龙面板。

优点是安全性较高，不需要公开后台服务。

### 方式二：Cloudflare Tunnel + Access

如果已经有自己的域名，可以使用 Cloudflare Tunnel 将青龙面板映射到子域名，例如：

```text
ql.treelang.me
```

但必须加上 Cloudflare Access。这样访问青龙面板前，会先经过 Cloudflare 的身份验证，再进入青龙自己的登录页。

基本结构如下：

```text
浏览器
  ↓
ql.treelang.me
  ↓
Cloudflare Access
  ↓
Cloudflare Tunnel
  ↓
内网青龙面板
```

这样相当于多了一层保护。

## 不建议的做法

不建议直接在路由器上把 5700 端口映射到公网，也不建议让青龙面板裸露在公网 IP 上。

青龙面板可以执行脚本，也可能保存敏感环境变量。如果后台被扫描或爆破，风险比较高。

因此，远程访问青龙时至少应该满足以下条件：

- 使用强密码
- 不开放裸端口
- 使用 Cloudflare Access 或私有网络
- 定期更新镜像
- 不运行来源不明的脚本
- 不在日志中输出敏感信息

## Webhook 自动触发任务

后续可以使用 Cloudflare Workers 做一个中间层，用来触发青龙任务。这样外部只访问 Worker 地址，由 Worker 校验密钥后再调用青龙 API。

结构如下：

```text
外部请求
  ↓
Cloudflare Worker
  ↓
校验 Webhook Secret
  ↓
调用青龙 OpenAPI
  ↓
执行指定任务
```

这样可以避免直接暴露青龙 API 和 token。

## 数据备份

青龙面板的数据目录在：

```text
~/docker/qinglong/data
```

这个目录中保存了青龙的配置、脚本、环境变量和任务信息。建议定期备份这个目录。

可以简单使用 tar 打包：

```bash
tar -czvf qinglong-backup.tar.gz ~/docker/qinglong/data
```

也可以写成定时备份脚本，定期保存到其他磁盘或云存储中。

## 常见问题

### 1. 容器启动失败

可以查看日志：

```bash
docker logs qinglong
```

根据日志判断是否是端口占用、权限问题或镜像拉取失败。

### 2. 访问不了面板

先检查容器是否运行：

```bash
docker ps
```

再检查端口是否监听：

```bash
ss -tulnp | grep 5700
```

如果是在局域网访问，还需要确认服务器 IP 是否正确。

### 3. 外网访问失败

如果使用 Cloudflare Tunnel，需要检查 tunnel 是否在线，以及 public hostname 是否配置正确。

如果使用 Cloudflare Access，需要确认访问策略是否允许当前账号登录。

## 总结

青龙面板适合作为个人自动化任务中心，但它不是普通静态网页，而是可以执行脚本和保存敏感信息的后台系统。因此部署时不能只关注能不能访问，更要关注访问是否安全。

本次部署完成后，青龙面板可以用于管理定时任务、自动化脚本和通知推送。后续可以继续结合 Cloudflare Workers、Webhook 和个人服务器，实现更加完整的自动化工作流。
