---
layout: _posts
title: 为Hyper-V虚拟网络设置NAT
date: 2023-5-28 17:09:15
tags: ["教程"]
---
## 前言

因为ip资源往往是有限的，通常我们不能为每一台虚拟机都分配上独立的公网ip地址，这时我们就需要用到NAT（Network Address Translation，网络地址转换）技术

在Hyper-V中我们可以创建一个NAT虚拟网络用于虚拟机正常访问网络

## NAT虚拟网络概述

NAT 使用主计算机的 IP 地址和端口通过内部 Hyper-V 虚拟开关向虚拟机授予对网络资源的访问权限。

网络地址转换 (NAT) 是一种网络模式，旨在通过将一个外部 IP 地址和端口映射到更大的内部 IP 地址集来转换 IP 地址。 基本上，NAT 使用流量表将流量从一个外部（主机）IP 地址和端口号路由到与网络上的终结点（虚拟机、计算机和容器等）关联的正确内部 IP 地址

此外，NAT 允许多个虚拟机托管需要相同（内部）通信端口的应用程序，方法是将它们映射到唯一的外部端口。

出于所有这些原因，NAT 网络对于容器技术是很常见的（请参阅[Windows容器网络](https://learn.microsoft.com/zh-cn/virtualization/windowscontainers/container-networking/architecture)）。

## 要求与限制

### 要求

Windows 10 周年更新或更高版本

安装并开启Hyper-V

### 限制

**注意**：目前，每台主机仅限一个 NAT 网络。 有关 Windows NAT (WinNAT) 实现、功能和限制的更多详细信息，请参考 [WinNAT capabilities and limitations blog（WinNAT 功能和限制日志）](https://techcommunity.microsoft.com/t5/Virtualization/Windows-NAT-WinNAT-Capabilities-and-limitations/ba-p/382303)

## 创建NAT虚拟网络

让我们来实际操作一边NAT虚拟网络的创建

首先让我们以管理员身份打开Powershell控制台

创建一个虚拟交换机，类型为内部
~~~
New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
~~~
其中 ```SwitchName``` 为要创建的虚拟交换机名称, ```-SwitchType``` 后的 ```Internal``` 为虚拟交换机类型，即``内部``

查找刚创建的虚拟交换机的接口索引

可以通过运行 ```Get-NetAdapter``` 来查找接口索引

你的输出应类似下面的形式：
~~~ ps
PS C:\> Get-NetAdapter

Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
----                  --------------------               ------- ------       ----------           ---------
vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps
~~~
其中ifIndex所对应的数字为NIC的接口索引，你需要记住你所创建的虚机交换机的接口索引

## 配置NAT网关

使用 ```New-NetIPAddress``` 配置 NAT 网关

下面是本命令的语法
~~~ ps
New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
~~~
若要配置网关，你将需要一些有关你的网络的信息：
- **IPAddress** - NAT 网关 IP 指定要用作 NAT 网关 IP 的 IPv4 或 IPv6 地址。 常规形式将为 a.b.c.1（例如 172.16.0.1）。 尽管最后一个位置不一定是 .1，但通常是 1（基于前缀长度）。 此 IP 地址位于来宾虚拟机使用的地址范围。 例如，如果来宾 VM 使用 IP 范围 172.16.0.0，则可以使用 IP 地址 172.16.0.100 作为 NAT 网关。

**通用网关 IP 为 192.168.0.1**

- **PrefixLength** -- NAT 子网前缀长度定义的 NAT 本地子网大小（子网掩码）。 子网前缀长度将介于 0 到 32 之间的一个整数值。
0 将映射整个 Internet，32 则只允许一个映射的 IP。 常用值范围从 24 到 12，具体要取决于多少 IP 需要附加到 NAT。

**常用 PrefixLength 为 24 -- 这是子网掩码 255.255.255.0**

- **InterfaceIndex** -- ifIndex 是你在上一步中确定的虚拟交换机的接口索引。

下面是一个例子
~~~ ps
New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
~~~
## 配置NAT网络

使用 ```New-NetNat``` 配置 NAT 网络

下面是本命令的语法
~~~ ps
New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
~~~
若要配置网关，你将需要提供一些有关网络和 NAT 网关的信息：

- **Name** - NATOutsideName 描述 NAT 网络的名称。 将使用此参数删除 NAT 网络。

- **InternalIPInterfaceAddressPrefix** - NAT 子网前缀同时描述上述 NAT 网关 IP 前缀和上述 NAT 子网前缀长度。

常规形式将为 a.b.c.0/NAT 子网前缀长度

**综上所述，对于本示例，我们将使用 192.168.0.0/24**

对于我们的示例，运行以下命令以设置 NAT 网络：

~~~ ps
New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
~~~
恭喜你，至此你已经拥有了一个NAT虚拟网络

## 连接虚拟机

若要将虚拟机连接到新的 NAT 网络，请使用“VM 设置”菜单将你在 NAT 网络设置部分的第一步中创建的内部交换机连接到虚拟机。

**注意：**

由于 WinNAT 本身不会将 IP 地址分配给某个终结点（例如，VM），因此，你将需要从 VM 内手动完成此操作，即设置 NAT 内部前缀范围内的 IP 地址、设置默认网关 IP 地址，以及设置 DNS 服务器信息。 唯一需要注意的是终结点何时连接到容器。 在这种情况下，主机网络服务 (HNS) 分配并使用主机计算服务 (HCS) 直接向容器分配 IP 地址、网关 IP 和 DNS 信息。