---
layout: _posts
title: nload网络监控工具的使用
date: 2023-3-19 16:54:49
tags: ["教程"]
---
## 前言

如果你想在命令行界面监控网络吞吐量，nload 应用程序是个不错的选择。它是一个实时监控网络流量和带宽使用的控制台应用程序，使用两个图表可视化地展示接收和发送的流量，并提供诸如数据交换总量、最小/最大网络带宽使用量等附加信息。

## 安装

### 在 CentOS/RHEL/Red Hat/Fedora Linux 上安装 nload
~~~ bash
yum install -y epel-release  #先安装epel软件库
yum install -y nload         #再安装nload
~~~
### 在Debian/Ubuntu Linux上安装
~~~ bash
apt update
apt install nload
~~~ 
## 使用

nload的基本语法为
~~~ bash
nload
nload device
nload [options] device1 device2
~~~
例如
~~~ bash
nload
nload eth0
nload em0 em2
~~~ 
nload 命令一旦执行就会开始监控网络设备，你可以使用下列快捷键操控 nload 应用程序。
~~~ bash
你可以按键盘上的 ← → 或者 Enter/Tab 键在设备间切换。

按 F2 显示选项窗口。

按 F5 将当前设置保存到用户配置文件。

按 F6 从配置文件重新加载设置。

按 q 或者 Ctrl+C 退出 nload。
~~~ 
默认每 100 毫秒刷新一次显示数值，下面的例子将时间间隔设置成 500 毫秒
~~~ bash
nload -t 500
~~~
程序界面如下
![nload](https://img.linux.net.cn/data/attachment/album/201404/18/133554o1q91yym98zpmamd.gif)
也可以手动指定网速单位

语法如下：
~~~ bash
nload -u h|H|b|B|k|K|m|M|g|G
nload -U h|H|b|B|k|K|m|M|g|G
nload -u h
nload -u G
nload -U G
~~~
释义：

**小写选项** -u: h 意为自动格式化为人类易读的单位，b 意为 Bit/s，k 意为 kBit/s，m 意为 MBit/s，g 意为 GBit/s。大写字母意为使用 Byte 替代 Bit。默认为 k。

**大写选项** -U 与小写选项 -u 非常相似，不同之处在于它展示的是数据量，比如 Bit, kByte, GBit 等等。（没有 "/s"）。默认值是 M