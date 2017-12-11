---
title: git服务器搭建
toc: true
comments: true
abbrlink: 64b5e8d2
date: 2017-04-12 17:43:37
categories: 工具
tags: git
---

## 安装git

```shell
yum install -y git
```

## 创建git用户

创建一个`git`用户，用来运行`git`服务

```shell
sudo adduser git
sudo passwd git
```
<!-- more -->

## 创建证书登录

将自己电脑的公钥`～/.ssh/id_rsa.pub`中的内容添加到服务器`/home/git/.ssh/authorized_keys`文件中，添加公钥之后可以防止每次 push 都输入密码。

```
chown -R 700  ~/.ssh
chown -R 600  ~/.ssh/authorized_keys
chown -R git:git /home/git
```
## 修改SSH配置文件，支持使用证书登录（root权限）

```
vi /etc/ssh/sshd_config
```

查找**RSAAuthentication**、**PubkeyAuthentication**、**AuthorizedKeysFile**把所在行修改为：

```
RSAAuthentication yes

PubkeyAuthentication yes

AuthorizedKeysFile .ssh/authorized_keys
```

重启SSH服务

```
systemctl restart sshd.service
```

## 禁止git账户的shell登录权限

出于安全考虑，我们要让 `git` 用户不能通过 shell 登录。可以编辑 `/etc/passwd` 来实现，在 `/etc/passwd`中找到类似下面的一行：

```
git:x:1001:1001:,,,:/home/git:/bin/bash
```

将其改为：

```
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```

这样 `git` 用户可以通过 ssh 正常使用 git，但是无法登录 sehll。

## 测试账户

打开自己电脑上的终端，输入以下命令测试服务是否正常

```shell
ssh -T git@service_ip
```

如果更改过ssh的默认端口则命令如下

```shell
ssh -T -p 8989 git@service_ip
```

## 初始化git仓库

选定一个目录作为Git仓库,我是将其放在 `/var/repo/` 目录下。在`/var/repo/`目录下执行下面命令建立一个裸仓库

```
$ sudo git init --bare sample.git
```

更改仓库sample.git的目录拥有者为git用户

```
$ sudo chown -R git:git sample.git
```

## 克隆git仓库

现在，可以通过git clone命令克隆远程仓库了。

```shell
git clone git@server_ip:/var/repo/sample.git
```

更改过ssh默认端口的使用以下命令

```shell
git clone ssh://git@server_ip:port/var/repo/sample.git
```

## 问题总结

- 使用SSH登录如果提示此异常：fatal: Interactive git shell is not enabled. hint: ~/git-shell-commands should exist and have read and execute access.
  解决方法如下：

  ````shell
  cp /usr/share/doc/git/contrib/git-shell-commands /home/git -R
  chown git:git /home/git/git-shell-commands/ -R
  ````

- 如果使用SSH登录进入，显示结果为：Run 'help' for help, or 'exit' to leave. Available commands:list
  解决方法如下：[参考文档](https://git-scm.com/docs/git-shell)

  ````shell 
  vi /home/git/git-shell-commands/no-interactive-login
  ````

  将以下内容写入到`no-interactive-login`文件中

  ```shell
  #!/bin/sh
  printf '%s\n' "Hi $USER! You've successfully authenticated, but I do not"
  printf '%s\n' "provide interactive shell access."
  exit 128
  EOF
  ```

  执行下面的命令，赋予文件执行权限

  ```shell
  chmod +x /home/git/git-shell-commands/no-interactive-login
  ```