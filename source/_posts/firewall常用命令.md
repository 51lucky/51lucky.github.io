---
title: firewall常用命令
categories: linux
tags: centos
abbrlink: a5e18cd0
date: 2017-04-9 12:35:27
---

1. 安装firewalld

   ```shell
   yum install firewalld
   ```
2. 启动firewalld

   ```shell
   systemctl start firewalld.service
   ```
3. 将firewalld设为开机自启动

   ```shell
   systemctl enable firewalld.service
   ```
   <!-- more -->

4. 查看firewalld状态

   ```shell
   systemctl status firewalld.service
   ```
5. 停止firewalld服务

   ```shell
   systemctl stop firewalld.service
   ```
6. 禁用firewalld服务

   ```shell
   systemctl disable firewalld.service
   ```
7. 永久开放端口

   ```shell
   firewall-cmd --zone=public --add-port=80/tcp --permanent
   ```
8. 重载firewalld配置文件

   ```shell
   firewall-cmd --reload
   ```
9. 查询某个端口状态

   ```shell
   firewall-cmd --query-port=9200/tcp
   ```
   返回yes表示打开
10. 关闭某个端口

   ```shell
   firewall-cmd --zone=public --remove-port=80/tcp --permanent
   ```
11. 启动服务

   ```shell
   firewall-cmd --permanent --add-service=http
   ```
12. 停止服务
   ```shell
   firewall-cmd --permanent --remove-service=http    
   ```
