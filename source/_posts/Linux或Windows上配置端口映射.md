---
layout: _posts
title: Linux或Windows上配置端口映射
date: 2023-4-10 17:32:20
tags: ["教程"]
---
在多网卡服务器环境中，为了使隔离网络中的服务能够互相通信，可以通过配置服务器来实现数据包的转发功能。以下是在Windows和Linux操作系统下实现端口映射的方法。

## Windows 下实现端口映射

### 1. 查询端口映射情况

使用`netsh interface portproxy show v4tov4`命令可以查看当前所有的IPv4到IPv4的端口映射情况。

```cmd
netsh interface portproxy show v4tov4
```

### 2. 查询某一个 IP 的所有端口映射情况

若要查询特定IP的所有端口映射情况，可以使用`find`命令过滤结果。

```cmd
netsh interface portproxy show v4tov4 | find "192.168.1.1"
```

### 3. 增加一个端口映射

通过`netsh interface portproxy add v4tov4`命令可以添加新的端口映射规则。

```cmd
netsh interface portproxy add v4tov4 listenaddress=2.2.2.2 listenport=8080 connectaddress=192.168.1.50 connectport=80
```

### 4. 删除一个端口映射

若需删除已存在的端口映射规则，可执行以下命令。

```cmd
netsh interface portproxy delete v4tov4 listenaddress=2.2.2.2 listenport=8080
```

## Linux 下实现端口映射

### 1. 允许数据包转发

首先需要开启系统的IP转发功能，并设置相应的iptables规则以允许数据包的转发。

```bash
echo 1 >/proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -j MASQUERADE
iptables -A FORWARD -i ens33 -j ACCEPT
iptables -t nat -A POSTROUTING -s 192.168.50.0/24 -o ens37 -j MASQUERADE
```

### 2. 设置端口映射

接下来，通过添加iptables规则来实现端口的映射。

```bash
iptables -t nat -A PREROUTING -p tcp --dport 6080 -j DNAT --to-destination 10.0.0.100:6090
```