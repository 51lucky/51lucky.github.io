---
title: centos7系统初始化设置
abbrlink: d1339487
---

安装好Centos系统后，我们还需要在服务器上做一些简单的操作，以进一步增强服务器的安全性。

## 修改root用户的密码（可选）

如果root用户的密码是安装时自动生成无法确认安全，建议修改root密码。

````shell
$ passwd root
````

系统会提示输入新的密码，需要输入两次相同的新密码。

服务器上所有用户的密码都要采用毫无关联的强密码，密码为不少于16位的大小写字母数字特殊符号的组合。推荐采用专门的密码管理软件，比如KeePass。

<!-- more -->

## 新增普通用户

在日常的服务器管理过程中，并不需要使用root来登录。创建一个拥有一定权限的普通用户进行日常的操作与维护。

```shell
$ useradd lucky
```

lucky 就是普通用户的名称，根据实际情况取名，随后给该账号设定密码

```shell
$ passwd lucky
```

设置完密码后，需要对lucky用户赋权。

```shell
$ visudo
```

在sudoers的最后一行加入：

```shell
lucky            ALL=(ALL)             ALL
```

## 修改SSH配置文件（改默认端口，禁止Root用户，指定允许登录的用户）

编辑修改SSH配置文件

```shell
$ vi /etc/ssh/sshd_config
```

打开配置文件后，查找

```shell
#Port 22
```

修改为

```shell
Port 2345
```

2345是更改后的端口号，可以是1024～65535之间的任意数字，不要和系统其它服务端口冲突即可。

查找

```shell
#PermitRootLogin yes
```

修改为

```shell
PermitRootLogin no
AllowUsers lucky
```

第一行是禁止root用户通过SSH登录

第二行是指允许通过SSH登录的用户为lucky，可以输入多个用户用空格隔开

然后检查配置文件中，以下字段是否设置为no，字段最前面有#的删除#

```shell
PermitEmptyPasswords no
X11Forwarding no
UseDNS no
```

第一行是禁止空密码登录

第二行是禁止X11转发,如果允许那么将可能有额外的信息被泄漏。

第三行是当客户端试图登录SSH服务器时，服务器端先根据客户端的IP地址进行DNS PTR反向查询出客户端的主机名，然后根据查询出的客户端主机名进行DNS正向A记录查询，验证与其原始IP地址是否一致，这是防止客户端欺骗的一种措施，但一般我们的是动态IP不会有PTR记录，打开这个选项不过是在白白浪费时间而已，不如将其关闭。

再检查配置文件中，以下字段设置，字段最前面有#的删除#

```shell
Protocol 2
MaxAuthTries 3
MaxSessions 2
```

第一行是ssh协议的版本

第二行是ssh连接服务器时，一次连接最多能尝试的次数，超过将断开连接

第三行是最大并发允许链接数，超过将拒绝

修改完成，保存配置文件，退出编辑器。重启SSH服务，使得配置文件生效

```shell
systemctl restart sshd.service
```

更高安全需求请查看[ssh证书登录](/posts/cecc8cb2/)

## 启用firewalld防火墙

安装firewalld防火墙。

```shell
$ yum -y install firewalld
```

因为更改过SSH默认端口，所以先修改firewall中SSH服务配置文件

```shell
$ cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/
$ vi /etc/firewalld/services/ssh.xml
```

查找

```shell
< port protocol="tcp" port="22" />
```

修改为

```shell
< port protocol="tcp" port="2345" />
```

启动firewalld并设为开机自启

```shell
$ systemctl start firewalld.service
$ systemctl enable firewalld.service
```

在firewalld中开放ssh服务端口

```shell
$ firewall-cmd --permanent --add-service=ssh
```

重启防火墙

```shell
$ firewall-cmd --reload
```

为防止失误，保持当前窗口连接，新建SSH连接2345端口测试lucky能否登录系统。更多常用命令查看[firewall常用命令](posts/a5e18cd0/)

## 系统自动更新

安装EPEL源

```shell
$ yum -y install epel-release.noarch
```

手动进行系统更新，安装所需要软件cron和yum-cron

```shell
$ yum -y update
$ yum -y install cronie
$ yum -y install yum-cron
```

修改yum-cron配置文件

```shell
$ vi /etc/yum/yum-cron.conf
```

查找

```shell
apply_updates = no
```

修改为

```shell
apply_updates = yes
```

再检查配置文件中，以下字段是否设置为yes

```shell
update_messages = yes
download_updates = yes
apply_updates = yes
```

修改完成，保存配置文件退出编辑器。启动cron和yum-cron并设为开机自启

```shell
$ systemctl start crond.service
$ systemctl enable crond.service
$ systemctl start yum-cron.service
$ systemctl enable yum-cron.service
```

## 修改时区

各个时区设置文件放在/usr/share/zoneinfo/中，将/etc/localtime通过文件链接指向欲设置的时区配置文件即可。

```shell
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
