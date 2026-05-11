---
title: Debian 12 微型服务器部署记录
author: Treelang
pubDatetime: 2026-05-10T12:00:00Z
slug: debian-12-mini-server
featured: false
draft: false
tags:
  - Debian
  - Linux
  - Server
  - Docker
description: 记录在低功耗设备上部署 Debian 12 微型服务器的过程，包括系统安装、基础配置、SSH、Docker、Cloudflare Tunnel 和后续服务规划。
---

## 前言

除了个人主页和博客之外，我还希望搭建一个长期运行的个人微型服务器，用来部署一些轻量服务，例如青龙面板、状态监控、自动化脚本、内网服务入口等。

相比租用云服务器，低功耗设备更适合做个人实验环境。它可以放在家里长期运行，功耗低，成本可控，也方便折腾系统和网络配置。本篇文章记录的是 Debian 12 微型服务器的基础部署过程。

## 设备用途规划

这台服务器主要用于以下场景：

- 部署青龙面板
- 运行 Docker 服务
- 配置 Cloudflare Tunnel
- 搭建个人自动化任务中心
- 作为内网服务入口
- 进行 Linux 和网络环境学习

由于设备性能有限，所以不适合部署非常重的服务。更适合运行轻量级 Web 服务、定时任务、监控工具和自动化脚本。

## 安装 Debian 12

首先需要准备 Debian 12 系统镜像，并制作启动盘。安装时建议选择较轻量的系统环境，可以不安装完整桌面环境，只保留基础系统和 SSH 服务。

安装过程中需要注意以下几点：

- 设置一个容易记住但足够安全的用户名
- 设置强密码
- 确认网络连接正常
- 如果设备长期放在固定位置，可以配置静态 IP
- 建议开启 SSH 服务，方便远程管理

安装完成后，进入系统并更新软件包。

## 更新系统

登录系统后，先更新软件源和软件包：

```bash
sudo apt update
sudo apt upgrade -y
```

安装一些常用工具：

```bash
sudo apt install -y curl wget git vim nano htop unzip ca-certificates gnupg lsb-release
```

这些工具在后续安装 Docker、配置服务和排查问题时都会用到。

## 配置 SSH

如果需要远程管理服务器，可以使用 SSH。

检查 SSH 服务状态：

```bash
systemctl status ssh
```

如果没有安装 SSH 服务，可以执行：

```bash
sudo apt install -y openssh-server
```

启动 SSH：

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

在局域网内可以通过以下命令连接：

```bash
ssh 用户名@服务器IP
```

为了提高安全性，后续可以配置 SSH 密钥登录，并关闭密码登录。

## 查看服务器 IP

可以使用以下命令查看 IP 地址：

```bash
ip addr
```

或者：

```bash
hostname -I
```

如果服务器 IP 经常变化，可以考虑在路由器中为设备绑定固定 IP，或者在系统中配置静态 IP。

## 安装 Docker

很多服务都可以通过 Docker 部署，因此 Docker 是个人服务器上比较重要的基础环境。

安装前先更新依赖：

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
```

创建 keyrings 目录：

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

添加 Docker GPG key：

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

添加 Docker 软件源：

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

安装 Docker：

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

检查 Docker 是否安装成功：

```bash
docker --version
docker compose version
```

启动 Docker：

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

## 配置当前用户使用 Docker

默认情况下，普通用户直接执行 Docker 命令可能会提示权限不足。可以把当前用户加入 docker 组：

```bash
sudo usermod -aG docker $USER
```

然后退出重新登录，再测试：

```bash
docker ps
```

如果能正常显示容器列表，说明配置成功。

## 部署青龙面板

可以使用 Docker Compose 部署青龙面板。先创建目录：

```bash
mkdir -p ~/docker/qinglong
cd ~/docker/qinglong
```

创建 `docker-compose.yml`：

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

启动：

```bash
docker compose up -d
```

查看运行状态：

```bash
docker ps
```

在局域网内访问：

```text
http://服务器IP:5700
```

青龙面板涉及脚本、环境变量和日志内容，不建议直接暴露到公网。如果需要远程访问，应结合 Cloudflare Tunnel、Cloudflare Access 或 Tailscale 等方式进行保护。

## 配置 Cloudflare Tunnel

如果已经有自己的域名，可以通过 Cloudflare Tunnel 将内网服务发布出去。例如：

```text
ql.treelang.me → http://localhost:5700
```

但对于青龙面板这类后台服务，不能只做内网穿透，还应该使用 Cloudflare Access 增加一层身份验证。

如果不想使用域名，也可以使用 Tailscale 或 ZeroTier 搭建私有网络，这样青龙面板只允许自己的设备访问，安全性更好。

## 服务器维护建议

长期运行的微型服务器需要注意几个问题。

第一是电源稳定。低功耗设备虽然功耗小，但如果经常断电，可能会造成系统或数据损坏。

第二是磁盘空间。Docker 日志和容器数据可能会逐渐占用磁盘空间，需要定期检查：

```bash
df -h
```

第三是系统更新。可以定期执行：

```bash
sudo apt update
sudo apt upgrade -y
```

第四是备份。青龙配置、脚本、环境变量和重要数据应该定期备份，不要只保存在服务器本地。

## 后续计划

这台服务器后续计划部署以下服务：

- 青龙面板
- Uptime Kuma 状态监控
- Cloudflare Tunnel
- 自动化 Webhook
- 文件备份脚本
- 个人服务导航页

所有公网访问的服务都应该尽量加上身份验证，尤其是后台管理类服务，不能直接裸露到公网。

## 总结

这次 Debian 12 微型服务器部署主要完成了系统安装、基础软件配置、SSH、Docker 和青龙面板部署。它为后续的个人自动化服务和内网服务管理提供了基础环境。

相比云服务器，微型服务器更适合个人长期折腾和学习。后续重点会放在服务安全、自动备份、内网穿透和自动化任务管理上。
