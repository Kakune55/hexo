---
layout: _posts
title: Nginx的配置与使用-下
date: 2023-03-17 16:03:27
tags: ["教程"]
excerpt: "nginx.conf 配置文件的详解"
---
**Nginx配置入门**  
nginx的配置文件 通常在安装目录中的``conf``文件夹中 或 ``/etc/nginx/``中

默认为``nginx.conf``

默认配置文件为
~~~
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
#
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
~~~
## 配置文件详解
### 全局块
全局块是默认配置文件从开始到events块之间的一部分内容，主要设置一些影响Nginx服务器整体运行的配置指令，因此，这些指令的作用域是Nginx服务器全局。

通常包括配置运行Nginx服务器的用户（组）、允许生成的worker process数、Nginx进程PID存放路径、日志的存放路径和类型以及配置文件引入等。
~~~ 
# 指定可以运行nginx服务的用户和用户组，只能在全局块配置
# user [user] [group]
# 将user指令注释掉，或者配置成nobody的话所有用户都可以运行
# user nobody nobody;
# user指令在Windows上不生效，如果你制定具体用户和用户组会报小面警告
# nginx: [warn] "user" is not supported, ignored in D:\software\nginx-1.18.0/conf/nginx.conf:2

# 指定工作线程数，可以制定具体的进程数，也可使用自动模式，这个指令只能在全局块配置
# worker_processes number | auto；
# 例子：指定4个工作线程，这种情况下会生成一个master进程和4个worker进程
# worker_processes 4;

# 指定pid文件存放的路径，这个指令只能在全局块配置
# pid logs/nginx.pid;

# 指定错误日志的路径和日志级别，此指令可以在全局块、http块、server块以及location块中配置。(在不同的块配置有啥区别？？)
# 其中debug级别的日志需要编译时使用--with-debug开启debug开关
# error_log [path] [debug | info | notice | warn | error | crit | alert | emerg] 
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;
~~~
### events块
events块涉及的指令主要影响Nginx服务器与用户的网络连接。常用到的设置包括是否开启对多worker process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型处理连接请求，每个worker process可以同时支持的最大连接数等。

这一部分的指令对Nginx服务器的性能影响较大，在实际配置中应该根据实际情况灵活调整。
~~~
# 当某一时刻只有一个网络连接到来时，多个睡眠进程会被同时叫醒，但只有一个进程可获得连接。如果每次唤醒的进程数目太多，会影响一部分系统性能。在Nginx服务器的多进程下，就有可能出现这样的问题。
# 开启的时候，将会对多个Nginx进程接收连接进行序列化，防止多个进程对连接的争抢
# 默认是开启状态，只能在events块中进行配置
# accept_mutex on | off;

# 如果multi_accept被禁止了，nginx一个工作进程只能同时接受一个新的连接。否则，一个工作进程可以同时接受所有的新连接。 
# 如果nginx使用kqueue连接方法，那么这条指令会被忽略，因为这个方法会报告在等待被接受的新连接的数量。
# 默认是off状态，只能在event块配置
# multi_accept on | off;

# 指定使用哪种网络IO模型，method可选择的内容有：select、poll、kqueue、epoll、rtsig、/dev/poll以及eventport，一般操作系统不是支持上面所有模型的。
# 只能在events块中进行配置
# use method
# use epoll

# 设置允许每一个worker process同时开启的最大连接数，当每个工作进程接受的连接数超过这个值时将不再接收连接
# 当所有的工作进程都接收满时，连接进入logback，logback满后连接被拒绝
# 只能在events块中进行配置
# 注意：这个值不能超过超过系统支持打开的最大文件数，也不能超过单个进程支持打开的最大文件数，具体可以参考这篇文章：https://cloud.tencent.com/developer/article/1114773
# worker_connections  1024;
~~~
### http块
http块是Nginx服务器配置中的重要部分，代理、缓存和日志定义等绝大多数的功能和第三方模块的配置都可以放在这个模块中。

前面已经提到，http块中可以包含自己的全局块，也可以包含server块，server块中又可以进一步包含location块，我们使用“http全局块”来表示http中自己的全局块，即http块中不包含在server块中的部分。

可以在http全局块中配置的指令包括文件引入、MIME-Type定义、日志自定义、是否使用sendfile传输文件、连接超时时间、单连接请求数上限等。
~~~
# 常用的浏览器中，可以显示的内容有HTML、XML、GIF及Flash等种类繁多的文本、媒体等资源，浏览器为区分这些资源，需要使用MIME Type。换言之，MIME Type是网络资源的媒体类型。Nginx服务器作为Web服务器，必须能够识别前端请求的资源类型。

# include指令，用于包含其他的配置文件，可以放在配置文件的任何地方，但是要注意你包含进来的配置文件一定符合配置规范，比如说你include进来的配置是worker_processes指令的配置，而你将这个指令包含到了http块中，着肯定是不行的，上面已经介绍过worker_processes指令只能在全局块中。
# 下面的指令将mime.types包含进来，mime.types和ngin.cfg同级目录，不同级的话需要指定具体路径
# include  mime.types;

# 配置默认类型，如果不加此指令，默认值为text/plain。
# 此指令还可以在http块、server块或者location块中进行配置。
# default_type  application/octet-stream;

# access_log配置，此指令可以在http块、server块或者location块中进行设置
# 在全局块中，我们介绍过errer_log指令，其用于配置Nginx进程运行时的日志存放和级别，此处所指的日志与常规的不同，它是指记录Nginx服务器提供服务过程应答前端请求的日志
# access_log path [format [buffer=size]]
# 如果你要关闭access_log,你可以使用下面的命令
# access_log off;

# log_format指令，用于定义日志格式，此指令只能在http块中进行配置
# log_format  main '$remote_addr - $remote_user [$time_local] "$request" '
#                  '$status $body_bytes_sent "$http_referer" '
#                  '"$http_user_agent" "$http_x_forwarded_for"';
# 定义了上面的日志格式后，可以以下面的形式使用日志
# access_log  logs/access.log  main;

# 开启关闭sendfile方式传输文件，可以在http块、server块或者location块中进行配置
# sendfile  on | off;

# 设置sendfile最大数据量,此指令可以在http块、server块或location块中配置
# sendfile_max_chunk size;
# 其中，size值如果大于0，Nginx进程的每个worker process每次调用sendfile()传输的数据量最大不能超过这个值(这里是128k，所以每次不能超过128k)；如果设置为0，则无限制。默认值为0。
# sendfile_max_chunk 128k;

# 配置连接超时时间,此指令可以在http块、server块或location块中配置。
# 与用户建立会话连接后，Nginx服务器可以保持这些连接打开一段时间
# timeout，服务器端对连接的保持时间。默认值为75s;header_timeout，可选项，在应答报文头部的Keep-Alive域设置超时时间：“Keep-Alive:timeout= header_timeout”。报文中的这个指令可以被Mozilla或者Konqueror识别。
# keepalive_timeout timeout [header_timeout]
# 下面配置的含义是，在服务器端保持连接的时间设置为120 s，发给用户端的应答报文头部中Keep-Alive域的超时时间设置为100 s。
# keepalive_timeout 120s 100s

# 配置单连接请求数上限，此指令可以在http块、server块或location块中配置。
# Nginx服务器端和用户端建立会话连接后，用户端通过此连接发送请求。指令keepalive_requests用于限制用户通过某一连接向Nginx服务器发送请求的次数。默认是100
# keepalive_requests number;
~~~
### server块
server块和“虚拟主机”的概念有密切联系。

虚拟主机，又称虚拟服务器、主机空间或是网页空间，它是一种技术。该技术是为了节省互联网服务器硬件成本而出现的。这里的“主机”或“空间”是由实体的服务器延伸而来，硬件系统可以基于服务器群，或者单个服务器等。虚拟主机技术主要应用于HTTP、FTP及EMAIL等多项服务，将一台服务器的某项或者全部服务内容逻辑划分为多个服务单位，对外表现为多个服务器，从而充分利用服务器硬件资源。从用户角度来看，一台虚拟主机和一台独立的硬件主机是完全一样的。

在使用Nginx服务器提供Web服务时，利用虚拟主机的技术就可以避免为每一个要运行的网站提供单独的Nginx服务器，也无需为每个网站对应运行一组Nginx进程。虚拟主机技术使得Nginx服务器可以在同一台服务器上只运行一组Nginx进程，就可以运行多个网站。

在前面提到过，每一个http块都可以包含多个server块，而每个server块就相当于一台虚拟主机，它内部可有多台主机联合提供服务，一起对外提供在逻辑上关系密切的一组服务（或网站）。

和http块相同，server块也可以包含自己的全局块，同时可以包含多个location块。在server全局块中，最常见的两个配置项是本虚拟主机的监听配置和本虚拟主机的名称或IP配置。

### listen指令
server块中最重要的指令就是listen指令，这个指令有三种配置语法。这个指令默认的配置值是：listen :80 | :8000；只能在server块种配置这个指令。
~~~
//第一种
listen address[:port] [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];

//第二种
listen port [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];

//第三种（可以不用重点关注）
listen unix:path [default_server] [ssl] [http2 | spdy] [proxy_protocol] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
~~~
listen指令的配置非常灵活，可以单独制定ip，单独指定端口或者同时指定ip和端口。
~~~
listen 127.0.0.1:8000;  #只监听来自127.0.0.1这个IP，请求8000端口的请求
listen 127.0.0.1; #只监听来自127.0.0.1这个IP，请求80端口的请求（不指定端口，默认80）
listen 8000; #监听来自所有IP，请求8000端口的请求
listen *:8000; #和上面效果一样
listen localhost:8000; #和第一种效果一致
~~~
关于上面的一些重要参数做如下说明：
- address：监听的IP地址（请求来源的IP地址），如果是IPv6的地址，需要使用中括号“[]”括起来，比如[fe80::1]等。
- port：端口号，如果只定义了IP地址没有定义端口号，就使用80端口。这边需要做个说明：要是你压根没配置listen指令，那么那么如果nginx以超级用户权限运行，则使用*:80，否则使用*:8000。多个虚拟主机可以同时监听同一个端口,但是server_name需要设置成不一样；
- default_server：假如通过Host没匹配到对应的虚拟主机，则通过这台虚拟主机处理。具体的可以参考这篇文章，写的不错。
- backlog=number：设置监听函数listen()最多允许多少网络连接同时处于挂起状态，在FreeBSD中默认为-1，其他平台默认为511。
- accept_filter=filter，设置监听端口对请求的过滤，被过滤的内容不能被接收和处理。本指令只在FreeBSD和NetBSD 5.0+平台下有效。filter可以设置为dataready或httpready，感兴趣的读者可以参阅Nginx的官方文档。
- bind：标识符，使用独立的bind()处理此address:port；一般情况下，对于端口相同而IP地址不同的多个连接，Nginx服务器将只使用一个监听命令，并使用bind()处理端口相同的所有连接。
- ssl：标识符，设置会话连接使用SSL模式进行，此标识符和Nginx服务器提供的HTTPS服务有关。

listen指令的使用看起来比较复杂，但其实在一般的使用过程中，相对来说比较简单，并不会进行太复杂的配置。

### server_name指令
用于配置虚拟主机的名称，语法如下
~~~
Syntax:	server_name name ...;
Default:	
server_name "";
Context:	server
~~~
对于name 来说，可以只有一个名称，也可以由多个名称并列，之间用空格隔开。每个名字就是一个域名，由两段或者三段组成，之间由点号“.”隔开。比如
~~~
server_name myserver.com www.myserver.com
~~~
在该例中，此虚拟主机的名称设置为myserver.com或www. myserver.com。Nginx服务器规定，第一个名称作为此虚拟主机的主要名称。

在name 中可以使用通配符“*”，但通配符只能用在由三段字符串组成的名称的首段或尾段，或者由两段字符串组成的名称的尾段，如：
~~~
server_name myserver.* *.myserver.com
~~~
另外name还支持正则表达式的形式。这边就不详细展开了。

由于server_name指令支持使用通配符和正则表达式两种配置名称的方式，因此在包含有多个虚拟主机的配置文件中，可能会出现一个名称被多个虚拟主机的server_name匹配成功。那么，来自这个名称的请求到底要交给哪个虚拟主机处理呢？Nginx服务器做出如下规定：

1. 对于匹配方式不同的，按照以下的优先级选择虚拟主机，排在前面的优先处理请求。
   1. 准确匹配server_name
   2. 通配符在开始时匹配server_name成功
   3. 通配符在结尾时匹配server_name成功
   4. 正则表达式匹配server_name成功

2. 在以上四种匹配方式中，如果server_name被处于同一优先级的匹配方式多次匹配成功，则首次匹配成功的虚拟主机处理请求。

有时候我们希望使用基于IP地址的虚拟主机配置，比如访问192.168.1.31有虚拟主机1处理，访问192.168.1.32由虚拟主机2处理。

这时我们要先网卡绑定别名，比如说网卡之前绑定的IP是192.168.1.30，现在将192.168.1.31和192.168.1.32这两个IP都绑定到这个网卡上，那么请求这个两个IP的请求才会到达这台机器。

绑定别名后进行以下配置即可：
~~~
http
{
	{
	   listen:  80;
	   server_name:  192.168.1.31;
     ...
	}
	{
	   listen:  80;
	   server_name:  192.168.1.32;
     ...
	}
}
~~~
### location块
每个server块中可以包含多个location块。在整个Nginx配置文档中起着重要的作用，而且Nginx服务器在许多功能上的灵活性往往在location指令的配置中体现出来。

location块的主要作用是，基于Nginx服务器接收到的请求字符串（例如， server_name/uri-string），对除虚拟主机名称（也可以是IP别名，后文有详细阐述）之外的字符串（前例中“/uri-string”部分）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能都是在这部分实现。许多第三方模块的配置也是在location块中提供功能。

在Nginx的官方文档中定义的location的语法结构为：
~~~
location [ = | ~ | ~* | ^~ ] uri { ... }
~~~
其中，uri变量是待匹配的请求字符串，可以是不含正则表达的字符串，如/myserver.php等；也可以是包含有正则表达的字符串，如 .php$（表示以.php结尾的URL）等。为了下文叙述方便，我们约定，不含正则表达的uri称为“标准uri”，使用正则表达式的uri称为“正则uri”。

其中方括号里的部分，是可选项，用来改变请求字符串与 uri 的匹配方式。在介绍四种标识的含义之前，我们需要先了解不添加此选项时，Nginx服务器是如何在server块中搜索并使用location块的uri和请求字符串匹配的。

在不添加此选项时，Nginx服务器首先在server块的多个location块中搜索是否有标准uri和请求字符串匹配，如果有多个可以匹配，就记录匹配度最高的一个。然后，服务器再用location块中的正则uri和请求字符串匹配，当第一个正则uri匹配成功，结束搜索，并使用这个location块处理此请求；如果正则匹配全部失败，就使用刚才记录的匹配度最高的location块处理此请求。


**了解了上面的内容，就可以解释可选项中各个标识的含义了：**

- “=”，用于标准uri前，要求请求字符串与uri严格匹配。如果已经匹配成功，就停止继续向下搜索并立即处理此请求。

- “^～”，用于标准uri前，要求Nginx服务器找到标识uri和请求字符串匹配度最高的location后，立即使用此location处理请求，而不再使用location块中的正则uri和请求字符串做匹配。

- “～”，用于表示uri包含正则表达式，并且区分大小写。

- “～*”，用于表示uri包含正则表达式，并且不区分大小写。注意如果uri包含正则表达式，就必须要使用“～”或者“～*”标识。

我们知道，在浏览器传送URI时对一部分字符进行URL编码，比如空格被编码为“%20”，问号被编码为“%3f”等。“～”有一个特点是，它对uri中的这些符号将会进行编码处理。比如，如果location块收到的URI为“/html/%20/data”，则当Nginx服务器搜索到配置为“～ /html/ /data”的location时，可以匹配成功。

### root指令
这个指令用于设置请求寻找资源的跟目录，此指令可以在http块、server块或者location块中配置。由于使用Nginx服务器多数情况下要配置多个location块对不同的请求分别做出处理，因此该指令通常在location块中进行设置。
~~~
root path
~~~
path变量中可以包含Nginx服务器预设的大多数变量，只有documentroot和������������和realpath_root不可以使用。

### alisa指令
### index指令
### error_page指令
## 一点说明
上面列举了Nignx中全局块、event块和http块的一些配置指令，但是Nginx的指令远远不止这些。其实这边最主要的还是讲解整个配置文件的结构，如果大家要看比较全的指令介绍、模块介绍的话，建议去Nginx的官网。
