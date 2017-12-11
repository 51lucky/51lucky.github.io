---
title: Shadowsocks服务端安装
categories: 科学上网
abbrlink: 5b74033d
date: 2017-02-25 22:42:24
toc: true
tags: 
---

推荐使用centos系统，其它系统请自测。

## 安装脚本

推荐使用*91yun的一键安装脚本*[https://www.91yun.org/archives/2079](https://www.91yun.org/archives/2079)

```shell
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/shadowsocks_install/master/shadowsocksR.sh && bash shadowsocksR.sh
```
<!-- more -->
## 卸载方法

```shell
bash ./shadowsocksR.sh uninstall
```

## 升级方法

```shell
cd /usr/local/shadowsocks/shadowsocks
git pull
```

## 常用命令

* 启动：`/etc/init.d/shadowsocks start`
* 停止：`/etc/init.d/shadowsocks stop`
* 重启：`/etc/init.d/shadowsocks restart`
* 查看状态：`/etc/init.d/shadowsocks status`

## 文件路径

- 配置文件路径：`/ect/shadowsocks.json`
- 日志文件路径：`/var/log/shadowsocks.log`
- 安装路径：`/usr/local/shadowsocks/shadowsocks`

## 多用户配置

修改配置文件`vi /etc/shadowsocks.json`.

> "port_password":{
>
> ​	"80":"password1",
>
> ​	"443":"password2"
>
> },

查看更多配置信息[https://github.com/breakwa11/shadowsocks-rss/wiki/config.json](https://github.com/breakwa11/shadowsocks-rss/wiki/config.json)

## 数据库多用户安装

* 安装依赖库

  ```shell
  yum install python-setuptools && easy_install pip
  pip install cymysql
  ```

* 服务端配置

  初始化配置`bash initcfg.sh`
  修改userapiconfig.py,usermysql.json,user-config.json相应信息。

查看更多详情:[https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup(manyuser-with-mysql)](https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup(manyuser-with-mysql))

