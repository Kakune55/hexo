---
layout: _posts
title: Nginx的配置与使用-上
date: 2023-03-8 16:03:22
tags: ["教程"]
---
## 前言
Nginx是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的http://Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。 其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。2011年6月1日，nginx 1.0.4发布。 Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，作为反代服务器的性能最为优秀。

套话说完了，那么nginx到底可以用来干什么呢。简单来说主要是一下几点：

正向代理

反向代理

负载均衡

动静分离

## nginx的安装
### 使用包管理器进行安装
对于Debian/Ubuntu及其同系的系统，使用apt进行安装
~~~ bash
sudo apt-get update
sudo apt-get install nginx
~~~
启动nginx与设置开机启动

~~~ bash
sudo systemctl start nginx
sudo systemctl enable nginx
~~~
---
对于RedHat/Centos及其同系的系统，使用yum进行安装  
在CentOS7中默认软件源没有nginx，所以首先我们要添加nginx官方的yum源

***此步骤仅为CentOS7需要使用***
~~~ bash
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
~~~
**安装Nginx**  
通过 ```yum search nginx``` 看看是否已经添加源成功。如果成功则执行下列命令安装Nginx
~~~ bash
yum install -y nginx
~~~
**启动nginx与设置开机启动**
~~~ bash
sudo systemctl start nginx
sudo systemctl enable nginx
~~~
### 编译安装(演示环境Debian12)
#### 1.在官网下载相应的源码包
Nginx官网：https://nginx.org/en/

下载页面： https://nginx.org/en/download.html

将源码包下载至服务器
~~~ bash
wget https://nginx.org/download/nginx-1.22.1.tar.gz #以nginx1.22.1为例
~~~
解压并进入nginx源码目录
~~~ bash
tar -xvzf ./nginx-1.22.1.tar.gz
cd ./nginx-1.22.1
~~~
#### 2.安装编译环境
~~~ bash
sudo apt-get update && sudo apt-get upgrade #更新系统和软件源

sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev libssl-dev build-essential #安装nginx的依赖包 zlib pcre openssl（可以源码安装也可以直接系统安装）
~~~
**安装openssl(为nginx提供ssl功能)**
~~~ bash
sudo apt-get install -y openssl
~~~
**配置并编译安装nginx**
~~~ bash
./configure #检查依赖并生成编译用配置文件
make && make install #编译并安装
cd /usr/local/nginx #进入安装目录
cd ./sbin #进入可执行二进制文件目录
./nginx -v #查看nginx版本
./nginx #启动nginx
~~~
**查看效果**  
浏览器输入: 你服务器的ip地址 当看到下图所示内容代表安装成功

![nginx默认页面](https://devpress-image.s3.cn-north-1.jdcloud-oss.com/a/f0ffdfd9c6_default_page.jpg)
如果访问失败可以检查一下服务器的80端口是否开启。

### Nginx的常用命令
~~~ bash
nginx -h #查看帮助信息
nginx -v #查看版本信息
nginx -V #查看版本信息和配置参数信息
nginx  #启动nginx
nginx -c “配置文件” #启动nginx并制定配置文件
nginx -s reload #重新加载nginx配置文件
nginx -s quit #完整有序地退出nginx
nginx -s stop #快速停止nginx
nginx -s reopen #重新加载配置文件
nginx -t -c 配置文件 #检测配置文件是否正确
~~~