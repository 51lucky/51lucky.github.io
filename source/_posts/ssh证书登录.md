---
title: ssh证书登录
categories: linux
tags: centos
toc: true
abbrlink: cecc8cb2
date: 2017-04-08 12:38:29
---

## 生成用于SSH的公钥和私钥（本例用户为lucky）

```shell
ssh-keygen -t rsa
```

会提示输入：密钥存放位置（直接回车，默认在/home/lucky/.ssh/目录）、密码短语、重复密码短语。

完成后在/home/lucky/.ssh/目录下生成了2个文件：id_rsa为私钥，id_rsa.pub为公钥。

<!-- more -->

## 导入公钥

```shell
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

## 设置正确的文件和文件夹权限（root权限）

```shell
chown -R 700  ~/.ssh
chown -R 600  ~/.ssh/authorized_keys
chown -R lucky:lucky /home/lucky
```

开启SELinux时，还需要执行（root用户把/home改成/root）

```shell
restorecon -R -v /home
```

## 修改SSH配置文件，支持使用证书登录（root权限）

```shell
vi /etc/ssh/sshd_config
```

查找**RSAAuthentication**、**PubkeyAuthentication**、**AuthorizedKeysFile**把所在行修改为：

```shell
RSAAuthentication yes

PubkeyAuthentication yes

AuthorizedKeysFile .ssh/authorized_keys
```

重启SSH服务

```shell
systemctl restart sshd.service
```

##  CentOS 7客户端使用证书登录

将服务端生成的id_rsa私钥保存到本地.ssh文件夹中

ssh命令

```shell
ssh -i ~/.ssh/id_rsa remote_username@remote_ip
```

scp命令

```shell
scp -i ~/.ssh/id_rsa local_file remote_username@remote_ip:remote_file
```

## 修改SSH配置文件，禁止使用密码登录（root权限）

```shell
vi /etc/ssh/sshd_config
```

查找

```shell
PasswordAuthentication yes
```

修改为：

```shell
PasswordAuthentication no
```

重启SSH服务

```shell
systemctl restart sshd.service
```
