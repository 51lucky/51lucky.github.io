---
title: 开启Https
toc: true
comments: true
categories: web
abbrlink: 3add964c
date: 2017-04-11 17:25:38
tags:
---

推荐Let's Encrypt免费证书。这里使用acme.sh来自动生成Let's Encrypt证书。更多介绍及安装说明请查看[acme.sh的wiki](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)

## 安装acme.sh

```shell
curl  https://get.acme.sh | sh
```

## 重新载入.bashrc

```shell
source ~/.bashrc
```
<!-- more -->

## 生成证书

```shell
acme.sh  --issue  -d 51lucky.me -d www.51lucky.me  --webroot  /data/www/web/
```

## copy/安装证书

前面证书生成以后, 接下来需要把证书 copy 到真正需要用它的地方.

注意, 默认生成的证书都放在安装目录下: `~/.acme.sh/`, 请不要直接使用此目录下的文件, 例如: 不要直接让 nginx 的配置文件使用这下面的文件. 这里面的文件都是内部使用, 而且目录结构可能会变化.

正确的使用方法是使用 `--installcert` 命令,并指定目标位置, 然后证书文件会被copy到相应的位置, 例如:

```shell
acme.sh  --installcert  -d  51lucky.me   \
        --key-file   /etc/nginx/ssl/51lucky.key \
        --fullchain-file /etc/nginx/ssl/51lucky.cer \
        --reloadcmd  "service nginx force-reload"
```

(一个小提醒, 这里用的是 `service nginx force-reload`, 不是 `service nginx reload`, 据测试, `reload` 并不会重新加载证书, 所以用的 `force-reload`)

`--installcert`命令可以携带很多参数, 来指定目标文件. 并且可以指定 reloadcmd, 当证书更新以后, reloadcmd会被自动调用,让服务器生效.

## 更新Nginx配置

贴上我的配置，供大家参考

```shell
server {
   listen 443 ssl;
   server_name www.51lucky.me;
   ssl_certificate /etc/nginx/ssl/51lucky.cer;
   ssl_certificate_key /etc/nginx/ssl/51lucky.key;
 
   access_log /data/wwwroot/log/access.log;
   error_log /data/wwwroot/log/error.log;
   root /data/wwwroot/web;
   index index.html index.htm;
}
server {
   listen 80;
   server_name www.51lucky.me;
   rewrite ^(.*)$ https://$host$1 permanent;
}
server {
   server_name 51lucky.me;
   rewrite ^/(.*)$ https://www.$host/$1 permanent;
}
```

然后测试语法,重启Nginx服务器。开启https服务

```shell
nginx -t
systemctl restart nginx.service
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

打开浏览器，访问网站，是否看到了小绿锁呢！

## 更新证书

目前证书在 60 天以后会自动更新, 你无需任何操作. 今后有可能会缩短这个时间, 不过都是自动的, 你不用关心.

## 更新acme.sh

目前由于 acme 协议和 letsencrypt CA 都在频繁的更新, 因此 acme.sh 也经常更新以保持同步.

升级 acme.sh 到最新版 :

```shell
acme.sh --upgrade
```

如果你不想手动升级, 可以开启自动升级:

```shell
acme.sh  --upgrade  --auto-upgrade
```

之后, acme.sh 就会自动保持更新了.

你也可以随时关闭自动更新:

```shell
acme.sh --upgrade  --auto-upgrade  0
```

