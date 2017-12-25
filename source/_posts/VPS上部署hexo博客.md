---
title: VPS上部署hexo博客
toc: true
comments: true
categories: web
tags: hexo
abbrlink: 2ebeffde
date: 2017-04-13 14:26:14
updated:
---

## 安装Nginx

### 添加nginx官方库

在[http://nginx.org/packages/centos/7/noarch/RPMS/](http://nginx.org/packages/centos/7/noarch/RPMS/)查看最新库信息

```shell
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```
<!-- more -->
### 安装nginx

```shell
yum -y install nginx
```

### 编辑nginx主配置文件

```shell
vi /etc/nginx/nginx.conf
```
```shell
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes 1;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
    accept_mutex on;
    multi_accept on;
    use epoll;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    server_tokens       off;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # gzip压缩功能设置
    gzip on;
    gzip_min_length 1k;
    gzip_buffers    4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 6;
    gzip_types text/html text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
    gzip_vary on;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    #屏蔽空机头
    server {
        listen        80 default_server;
        listen        [::]:80 default_server;
        server_name   _;
        return        500;
    }

#    server {
#        listen       80 default_server;
#        listen       [::]:80 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }
#
# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers HIGH:!aNULL:!MD5;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```

### 配置防火墙开启Http服务端口

```shell
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

### 启动nginx并设为开机自启

```shell
systemctl start nginx.service
systemctl enable nginx.service
```

### Nginx站点配置

建立网站的目录及子目录

```shell
mkdir -p /data/www/web
mkdir -p /data/www/log
```

添加站点的nginx配置文件

```shell
vi /etc/nginx/conf.d/yourdomain.conf
```

输入以下内容

```shell
server {
  listen 80;
  server_name www.yourdomain.com;
  access_log /data/www/log/access.log;
  error_log /data/www/log/error.log;
  root /data/www/web;
  index index.html index.htm;
}
server {
   server_name yourdomain.com;
   rewrite ^/(.*)$ http://www.$host/$1 permanent;
}
```

测试语法是否通过

```she
nginx -t
```

### 重启nginx服务

```she
systemctl restart nginx.service
```

在/data/www/web目录下可以新建一个简单的index网页，进行查看；

```html
<html>
  Hello,World!
</html>
```

nginx主配置文件：/etc/nginx/nginx.conf
nginx默认配置文件目录：/etc/nginx/conf.d/
nginx默认站点主目录：/usr/share/nginx/html/
nginx默认日志目录：/var/log/nginx/

## Git Hook自动部署

### 搭建git服务器

具体方法请查看[git服务器搭建](/posts/64b5e8d2/)

### 初始化 Git 仓库

初始化 Git 仓库，我是将其放在 `/var/repo/blog.git` 目录下的：

```
$ sudo mkdir /var/repo
$ cd /var/repo
$ sudo git init --bare blog.git
```

使用 `--bare` 参数，Git 就会创建一个裸仓库，裸仓库没有工作区，我们不会在裸仓库上进行操作，它只为共享而存在。

### 配置git hooks

配置 git hooks，关于 hooks 的详情内容可以[参考这里](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)。

我们这里要使用的是 `post-receive` 的 hook，这个 hook 会在整个 git 操作过程完结以后被运行。

在 `blog.git/hooks` 目录下新建一个 `post-receive` 文件：

```
$ cd /var/repo/blog.git/hooks
$ vim post-receive
```

在 `post-receive` 文件中写入如下内容：

```
#!/bin/sh
git --work-tree=/data/www/web --git-dir=/var/repo/blog.git checkout -f
```

注意，`/var/www/hexo` 要换成你自己的部署目录，一般可能都是 `/var/www/html`。上面那句 git 命令可以在我们每次 push 完之后，把部署目录更新到博客的最新生成状态。这样便可以完成达到自动部署的目的了。

不要忘记设置这个文件的可执行权限：

```
chmod +x post-receive
```

### 修改目录权限

改变 `blog.git`和`/data/www/web`的目录拥有者为 `git` 用户：

```
$ sudo chown -R git:git blog.git
```

至此，服务器端的配置就完成了。

## 本地配置

配置你的 hexo 博客可以自动 deploy 到服务器上，再也不用 ftp 上传了。

修改 hexo 目录下的 `_config.yml` 文件，找到 [deploy] 条目，并修改为：

```
deploy:
  type: git
  repo: git@yourdomain.com:/var/repo/blog.git
  branch: master
```

要注意切换成你自己的服务器地址，以及服务器端 git 仓库的目录。如果你的ssh端口不是默认的22端口，则**repo**修改为ssh://git@yourdomain.com:端口号/var/repo/blog.git，至此，我们的 hexo 自动部署已经全部配置好了。
