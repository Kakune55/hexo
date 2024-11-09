---
layout: _posts
title: Docker的服务器安装
date: 2023-10-08 16:39:52
tags: ["教程"]
---
Docker 一键安装脚本（2023/10/8更新）

## Docekr简介

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows操作系统的机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

使用Docker可以大大提高应用的部署效率，和提供更好的应用间隔离所提供的安全性。

## Docker 安装要求

官方对于安装要求的介绍

https://docs.docker.com/engine/install/

Docker 需要安装在 64 位的 x86 平台或 ARM 平台上（如树莓派） ，并且要求内核 版本不低于 3.10。但实际上内核越新越好，过低的内核版本可能会出现部分功能无 法使用，或者不稳定。

用户可以通过如下命令检查自己的内核版本详细信息：
~~~ bash
uname -a
~~~
## 使用脚本快速安装docker

官方安装脚本  
这是Docker官方提供的安装脚本，但是在国内因为网络原因经常无法正常使用
~~~ bash
curl -sSL https://get.docker.com/ | sh
~~~
阿里云镜像 官方安装脚本（推荐）
~~~ bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
~~~ 
DaoCloud 的安装脚本（部分网络环境可能有问题）
~~~ bash
curl -sSL https://get.daocloud.io/docker | sh
~~~
## 为Docker配置国内镜像源

在国内因为网络原因常常会不能正常访问Docker Hub或者访速度过慢，导致镜像拉取速度过慢甚至失败。

所以我们要为Docker配置国内镜像源

针对Docker客户端版本大于1.10.0的用户 您可以通过修改daemon配置文件/etc/docker/daemon.json（没有时新建该文件）来使用加速器：
~~~ bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["将此处替换为镜像源地址"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
~~~
文件修改后应为
~~~ json
{
    "registry-mirrors": ["<your accelerate address>"]
}
~~~
也可以配置多个镜像源，具体格式如下
~~~ json
{
    "registry-mirrors" : [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://cr.console.aliyun.com/"
  ]
}
~~~
以下是一些常用的镜像源
~~~
网易：http://hub-mirror.c.163.com
中科大镜像地址：http://mirrors.ustc.edu.cn/
中科大github地址：https://github.com/ustclug/mirrorrequest
Azure中国镜像地址：http://mirror.azure.cn/
Azure中国github地址：https://github.com/Azure/container-service-for-azure-china
DockerHub镜像仓库: https://hub.docker.com/ 
阿里云镜像仓库： https://cr.console.aliyun.com   （需要自行配置，获取私人地址）
google镜像仓库： https://console.cloud.google.com/gcr/images/google-containers/GLOBAL （如果你本地可以翻墙的话是可以连上去的 ）
coreos镜像仓库： https://quay.io/repository/ 
RedHat镜像仓库： https://access.redhat.com/containers
~~~
## Docker的启动与停止
~~~ bash
systemctl命令是系统服务管理器指令
启动docker：
systemctl start docker
停止docker：
systemctl stop docker
重启docker：
systemctl restart docker
查看docker状态：
systemctl status docker
开机启动：
systemctl enable docker
查看docker概要信息
docker info
查看docker帮助文档
docker ‐‐help
~~~