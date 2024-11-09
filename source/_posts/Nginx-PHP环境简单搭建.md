---
layout: _posts
title: Nginx+PHP环境简单搭建
date: 2024-7-01 17:55:50
tags: ["教程"]
---
## 前言

采用nginx+php作为webserver的架构模式，在现如今运用相当广泛。然而第一步需要实现的是如何让nginx正确的调用php。由于nginx调用php并不是如同调用一个静态文件那么直接简单，是需要动态执行php脚本。所以涉及到了对nginx.conf文件的配置。这一步对新手而言需要动点脑筋，对于一般的熟手而言，也有不少同学并没有搞透彻为何要如此这般配置。本文的主要内容为如何在nginx server中正确配置php调用方法，以及配置的基本原理。

## 原理

### nginx实现php动态解析原理

nginx 是一个高性能的http服务器和反向代理服务器。即nginx可以作为一个HTTP服务器进行网站的发布处理，也可以作为一个反向代理服务器进行负载均衡。但需要注意的是：nginx本身并不会对php文件进行解析。对PHP页面的请求将会被nginx交给FastCGI进程监听的IP地址及端口，由php-fpm(第三方的fastcgi进程管理器)作为动态解析服务器处理，最后将处理结果再返回给nginx。即nginx通过反向代理功能将动态请求转向后端php-fpm，从而实现对PHP的解析支持，这就是Nginx实现PHP动态解析的基本原理。

首先需要了解一些概念。**（nginx + php-fpm +fastcgi）**

**Nginx** 是非阻塞IO & IO复用模型，通过操作系统提供的类似 epoll 的功能，可以在一个线程里处理多个客户端的请求。Nginx 的进程就是线程，即每个进程里只有一个线程，但这一个线程可以服务多个客户端。

**PHP-FPM** 是阻塞的单线程模型，pm.max_children 指定的是最大的进程数量，pm.max_requests 指定的是每个进程处理多少个请求后重启(因为 PHP 偶尔会有内存泄漏，所以需要重启)。PHP-FPM 的每个进程也只有一个线程，但是一个进程同时只能服务一个客户端。

**fastCGI** ：为了解决不同的语言解释器(如php、python解释器)与webserver的通信，于是出现了cgi协议。只要你按照cgi协议去编写程序，就能实现语言解释器与webwerver的通信。如php-cgi程序。但是webserver每收到一个请求，都会去fork一个cgi进程，请求结束再kill掉这个进程。这样有10000个请求，就需要fork、kill php-cgi进程10000次。 fastcgi是cgi的改良版本。fast-cgi每次处理完请求后，不会kill掉这个进程，而是保留这个进程，使这个进程可以一次处理多个请求。大大提高了效率。

## 安装

**安装Nginx**

**使用软件包管理器安装**

Debian系:
~~~ bash
sudo apt install nginx -y
~~~
Red Hat系：
~~~ bash
sudo yum install nginx -y
~~~
**安装PHP**

为了使用php我们需要安装php-fpm

Debian系:
~~~ bash
sudo apt install php-fpm -y
~~~
Red Hat系：
~~~ bash
sudo yum install php-fpm -y
~~~

PHP-FPM（PHP FastCGI Process Manager），是用于管理 PHP 进程池的软件，用于接收和处理来自 Web 服务器（如Nginx）的请求。PHP-FPM会创建一个主进程（通常以操作系统中根用户的身份运行），控制何时以及如何把 HTTP 请求转发给一个或多个子进程处理。PHP-FPM 主进程还控制着什么时候创建和销毁 PHP 子进程。PHP-FPM 进程池中的每个进程存在的时间都比单个 HTTP 请求长，可以处理10、50、100或更多的 HTTP 请求。

**验证安装**

简单的验证以下安装的软件
~~~ bash
nginx -v && php -v
~~~
如安装无误应该可以正确输出版本信息
~~~ bash
root@debian01:~# nginx -v && php -v
nginx version: nginx/1.22.1
PHP 8.2.5 (cli) (built: Apr 27 2023 08:13:47) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.2.5, Copyright (c) Zend Technologies
with Zend OPcache v8.2.5, Copyright (c), by Zend Technologies
~~~
## 配置  

### 配置php与nginx进程间通讯

[（引用nginx - Nginx中fastcgi_pass的配置问题 - SegmentFault 思否）](https://segmentfault.com/q/1010000004854045)

Nginx和PHP-FPM的进程间通信有两种方式,一种是TCP,一种是UNIX Domain Socket.
其中TCP是IP加端口,可以跨服务器.而UNIX Domain Socket不经过网络,只能用于Nginx跟PHP-FPM都在同一服务器的场景.用哪种取决于你的PHP-FPM配置:
方式1:
~~~
php-fpm.conf: listen = 127.0.0.1:9000
nginx.conf: fastcgi_pass 127.0.0.1:9000;
~~~
方式2:
~~~
php-fpm.conf: listen = /tmp/php-fpm.sock
nginx.conf: fastcgi_pass unix:/tmp/php-fpm.sock;
~~~
其中php-fpm.sock是一个文件,由php-fpm生成,类型是srw-rw----.

UNIX Domain Socket可用于两个没有亲缘关系的进程,是目前广泛使用的IPC机制,比如X Window服务器和GUI程序之间就是通过UNIX Domain Socket通讯的.这种通信方式是发生在系统内核里而不会在网络里传播.UNIX Domain Socket和长连接都能避免频繁创建TCP短连接而导致TIME_WAIT连接过多的问题.对于进程间通讯的两个程序,UNIX Domain Socket的流程不会走到TCP那层,直接以文件形式,以stream socket通讯.如果是TCP Socket,则需要走到IP层,对于非同一台服务器上,TCP Socket走的就更多了.

**UNIX Domain Socket:**  
> Nginx <=> socket <=> PHP-FPM  

**TCP Socket(本地回环):**  
> Nginx <=> socket <=> TCP/IP <=> socket <=> PHP-FPM

**TCP Socket(Nginx和PHP-FPM位于不同服务器):**  
> Nginx <=> socket <=> TCP/IP <=> 物理层 <=> 路由器 <=> 物理层 <=> TCP/IP <=> socket <=> PHP-FPM

像mysql命令行客户端连接mysqld服务也类似有这两种方式:
**使用Unix Socket连接(默认):**  
> mysql -uroot -p --protocol=socket --socket=/tmp/mysql.sock

**使用TCP连接:**  
> mysql -uroot -p --protocol=tcp --host=127.0.0.1 --port=3306

 如果你没有更改默认的```php-frm.conf```配置文件 默认为使用```UNIX Domain Socket   ```

### 配置nginx

接下来我们来修改nginx的配置文件```nginx.conf```

我们只需要对虚拟主机部分进行修改

如果你不懂nginx的配置文件如何操作可以翻看我之前的博客 [Nginx的配置与使用-上](https://www.kakunet.top/2023/03/08/Nginx的配置与使用-上)

以下为最基础的配置示例
~~~ nginx
server {
    listen       80; #nginx web服务监听的服务端口
    server_name  47.113.146.28; # 此处可更换为监听域名
    root   /var/website/; #网站根目录
    index  index.php index.html index.htm; #网站主页

        location ~ \.php$ { #php网页部分
        #fastcgi_pass 127.0.0.1:9000; #与php-frm的通讯接口配置详见上文
        fastcgi_pass    unix:/run/php/php-fpm.sock; #本示例使用默认的unix sock
        fastcgi_index index.php; #fastcgi主页
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        }
    location ~ /\.ht {
        deny  all;
    }
~~~

较完整配置可参考这篇文章 [Nginx和PHP的配置 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/97208252)

### 重启nginx以加载配置
~~~ bash
nginx -s reload
~~~
### 测试/使用

我们可以通过下面的方法判断Nginx与php配置是否成功。

在Nginx的网站根目录(本例中为 /var/website/)下创建一个php文件，随便起名我的是php_info.php

内容如下：
~~~ php
<?php

    // 顺便可以看一下php的扩展全不全
    phpinfo();
~~~
然后再浏览器中尝试访问它  
![img](\images\pageuse\2023-07-01-020719.png)  
如图正常访问

### 完成

至此nginx + php 的环境已经配置完成

