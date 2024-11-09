---
layout: _posts
title: 为Nginx网站启用Https
date: 2023-6-24 17:37:59
tags: ["教程"]
---
# Nginx 配置 HTTPS 服务指南

在配置Nginx以支持HTTPS服务之前，请确保已完成以下准备工作：

- **启用SSL模块**：确认Nginx已安装并启用了SSL模块。
- **可用的HTTP网站**：已配置好一个可以通过域名访问的HTTP网站。
- **有效的证书**：已为您的域名申请了有效的SSL/TLS证书。

如果您对Nginx的安装及配置不太熟悉，建议先阅读[Nginx的使用与配置(上)](/2023/03/08/Nginx的配置与使用-上/)这篇文章。

## 正式配置

### 1. 上传SSL证书

- 在服务器上创建一个专门用于存储证书的文件夹，并确保Nginx具有读取该文件夹的权限。
- 将您要配置的域名的SSL证书上传到此文件夹中。

例如，如果您使用的是`/root/cert/www.kakuweb.top/`目录，其中包含以下文件：

```bash
root@server:/etc/nginx# ls /root/cert/www.kakuweb.top/
9769805_www.kakuweb.top.key  9769805_www.kakuweb.top.pem
```

### 2. 编写Nginx配置文件

- 使用任意文本编辑器打开Nginx的主配置文件`nginx.conf`。

```bash
cd /etc/nginx/
nano nginx.conf
```

#### 配置HTTPS服务

- 在`nginx.conf`文件的`http`区块内，根据以下示例修改`server`属性配置：

```nginx
server {
    listen 443 ssl;  # 配置HTTPS的默认访问端口为443

    server_name <yourdomain>;  # 填写证书绑定的域名
    root html;
    index index.html index.htm;

    ssl_certificate cert/<cert-file-name>.pem;  # 证书文件名称
    ssl_certificate_key cert/<cert-file-name>.key;  # 证书私钥文件名称

    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    location / {
        root html;
        index index.html index.htm;
    }
}
```

- 如果您使用的是反向代理，`location`段应修改为：

```nginx
location / {
    proxy_pass http://ip:端口;
}
```

#### 配置HTTP自动跳转至HTTPS (可选)

为了让用户在访问HTTP站点时自动跳转到HTTPS站点，可以采用以下两种方法之一：

- **使用rewrite指令**

```nginx
server {
    listen 80;
    server_name www.xxx.com;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```

- **使用301重定向**

```nginx
server {
    listen 80;
    server_name www.xxx.com;
    return 301 https://$host$request_uri;
}
```

### 3. 重启Nginx

- 完成上述配置后，通过以下命令重启Nginx以使新配置生效：

```bash
nginx -s reload
```

### 4. 检验

- 至此，所有的配置工作已完成。尝试通过HTTPS访问您的网站，检查配置是否成功。
