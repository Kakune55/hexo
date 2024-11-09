---
layout: _posts
title: DockerHub镜像服务
date: 2024-11-09 13:45:03
tags: ["kaku的小玩意"]
---
## 域名

```bash
http://docker.mirrors.kakunet.top
```

### 基于 Cloudflare Workers 的 Docker 镜像代理

它能够中转对 Docker 官方镜像仓库的请求，解决一些访问限制和加速访问的问题。

## 如何使用？

### 1.官方镜像路径前面加域名

```bash
docker pull docker.mirrors.kakunet.top/stilleshan/frpc:latest
```

```bash
docker pull docker.mirrors.kakunet.top/library/nginx:stable-alpine3.19-perl
```

### 2.一键设置镜像加速

修改文件 `/etc/docker/daemon.json`（如果不存在则创建）

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["docker.mirrors.kakunet.top"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 注意

本镜像基于Cloudflare Works 每日有请求量限制。

**请勿滥用**

**请勿滥用**

## 特别鸣谢

### [Cloudflare](https://www.cloudflare.com/)

![](https://cdn.cookielaw.org/logos/6b10d640-dc80-4fbf-a462-ae81dbad56e4/f2b3f698-2a83-400c-ab3d-ae88a0a1d3c4/fca68c5f-051b-4269-9463-b0ba60c90bde/Logo.png)

### [gh-hunshcn](https://github.com/hunshcn)/[gh-proxy](https://github.com/hunshcn/gh-proxy) Github
