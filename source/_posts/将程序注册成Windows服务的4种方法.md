---
layout: _posts
title: 将程序注册成Windows服务的4种方法
date: 2024-04-08 17:20:00
tags: ["教程"]
---
Microsoft Windows 服务（过去称为 NT 服务）允许用户创建可在其自身的 Windows 会话中长时间运行的可执行应用程序。 这些服务可在计算机启动时自动启动，可以暂停和重启，并且不显示任何用户界面。 这些功能使服务非常适合在服务器上使用，或者需要长时间运行的功能（不会影响在同一台计算机上工作的其他用户）的情况。 还可以在与登录用户或默认计算机帐户不同的特定用户帐户的安全性上下文中运行服务。 有关服务和 Windows 会话的详细信息，请参阅 Windows SDK 文档。

## 使用windows自带的命令sc

首先我们要打开cmd，下面的命令在cmd中运行，最好使用管理员运行cmd

###　注册服务：
~~~ ps
sc create ceshi binpath= D:\test\test.exe type= own start= auto displayname= test
~~~
- binpath：你的应用程序所在的路径。

- displayname：服务显示的名称

**如何判断服务是否注册成功：**

在cmd中输入services.msc打开系统服务，查看是否出现test名称的服务（即displayname=后面的参数，我这里是test）

**or**

**按下面的方式尝试启动服务**

#### 启动服务
~~~ ps
net start test
~~~
#### 停止服务
~~~ ps
net stop test
~~~
#### 删除服务
~~~ ps
sc delete test
~~~
## 使用instsrv+srvany

使用方法一，如果你的exe不符合服务的规范，启动有可能会失败

这种情况下，我们使用instsrv+srvany

### 什么是instsrv+srvany

```instsrv.exe.exe```和```srvany.exe```是Microsoft Windows Resource Kits工具集中 的两个实用工具，这两个工具配合使用可以将任何的exe应用程序作为window服务运行。

```srany.exe```是注册程序的服务外壳，可以通过它让应用程序以system账号启动，可以使应用程序作为windows的服务随机器启动而自动启动，从而隐藏不必要的窗口

下载：

链接：https://pan.baidu.com/s/1gKu_WwVo-TeWXmrGAr9qjw 提取码：s1vm

### window64位系统

#### 安装

1. 将instsrv.exe和srvany.exe拷贝到C:\WINDOWS\SysWOW64目录下

2. 打开cmd

3. 运行命令：instsrv MyService C:\WINDOWS\SysWOW64\srvany.exe

注意：Myservice是自定义的服务的名称，可以根据应用程序名称任意更改

4. 运行成功！
![img](https://pic4.zhimg.com/80/v2-f011a6be0958748c3c9566b3d6c883a3_1440w.webp)
#### 配置

1. 打开注册表：（cmd中输入：regedit）

2. ctrl+F，搜索Myservice（之前自定义的服务名称）

3. 右击Myservice新建项，名称为Parameters

4. 之后在Parameters中新建几个字符串值

5. 名称 Application 值：你要作为服务运行的程序地址。

6. 名称 AppDirectory 值：你要作为服务运行的程序所在文件夹路径。

7. 名称 AppParameters 值：你要作为服务运行的程序启动所需要的参数。
![img](https://pic1.zhimg.com/80/v2-a64b137c58fc1688ce56127469c3fce4_1440w.webp)
之后启动服务Myservice即可后台运行exe！

#### window32位系统

#### 安装

1. 将instsrv.exe和srvany.exe拷贝到C:\WINDOWS\system32目录下

2. 打开cmd

3. 运行命令：instsrv MyService C:\WINDOWS\system32\srvany.exe

注意：Myservice是自定义的服务的名称，可以根据应用程序名称任意更改

4. 运行成功！

我这里是64位。

#### 配置

1. 打开注册表：（cmd中输入：regedit）

2. ctrl+F，搜索Myservice（之前自定义的服务名称）

3. 右击Myservice新建项，名称为Parameters

4. 之后在Parameters中新建几个字符串值

5. 名称 Application 值：你要作为服务运行的程序地址。

6. 名称 AppDirectory 值：你要作为服务运行的程序所在文件夹路径。

7. 名称 AppParameters 值：你要作为服务运行的程序启动所需要的参数。

8. 之后启动服务Myservice即可后台运行exe！

## 使用Winsw

winsw（Windows Service Wrapper）是一个开源项目，它可以让我们快速把一个可执行的程序注册为Windows的系统服务。

### Download & Install

下载地址：https://github.com/kohsuke/winsw/releases  

1. 下载对应平台的.exe（.net2和.net4）。下载地址提供了两份配置文件：

sample-allOptions.xml：包含所有配置项

smaple-minimal.xml：最小配置项

2. 把下载的.exe文件重命名为你自己要用的服务名称，如myapp.exe

3. 在myapp.exe同目录下创建xml配置文件，可以复制上面下载的xml，简单配置如下：
~~~ xml
<service>
  <!-- 该服务的唯一标识 -->
  <id>myapp</id>
  <!-- 注册为系统服务的名称 -->
  <name>myapp</name>
  <!-- 对服务的描述 -->
  <description>Send the data to customer</description>
  <!-- 将java程序添加到系统服务 -->
  <executable>java</executable>
  <!-- 执行的参数 -->
  <arguments>-jar "myapp.jar"</arguments>
  <!-- 日志模式 -->
  <logmode>rotate</logmode>
</service>
~~~
这里配置了一个java的应用程序。

### Use

注册服务
~~~ ps
myapp.exe install
~~~
卸载服务
~~~ ps
myapp.exe uninstall
~~~
启动服务
~~~ ps
myapp.exe start
~~~
关闭服务
~~~ ps
myapp.exe stop
~~~
重启服务
~~~ ps
myapp.exe restart 
~~~
查看状态
~~~ ps
myapp.exe status
~~~
## NSSM

NSSM是一个服务封装程序，它可以将普通exe程序封装成服务，使之像windows服务一样运行。同类型的工具还有微软自己的srvany，不过nssm更加简单易用，并且功能强大。

### Download & Install

下载地址：http://www.nssm.cc/release/nssm-2.24.zip

1. 准备好要使用的exe/bat

2. 找到对应的nssm.exe文件,打开cmd窗口,输入命令:
~~~
nssm.exe install
~~~
出现界面:


### Use

在Path处选择要注册成服务的程序run.exe

Service name 处填写要发布的服务名称

点击Install service安装服务，cmd进入服务就能看到刚发布的服务名称，启动即可

卸载服务命令：```nssm remove 服务名称```