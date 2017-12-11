---
title: Shadowsocks优化
toc: true
comments: true
abbrlink: fe739bd3
date: 2017-04-10 17:16:07
categories: 科学上网
tags: 
---

以下所有优化都是基于KVM架构的VPS。

## TCP优化

修改文件句柄数限制

如果是ubuntu/centos均可修改`vi /etc/sysctl.conf`

找到`fs.file-max`这一行，修改其值为`1024000`，并保存退出。然后执行`sysctl -p`使其生效

修改`vi /etc/security/limits.conf`文件，加入

```c
*      soft    nofile           512000
*      hard    nofile          1024000
```

针对centos,还需要于`cat /etc/pam.d/login`检查有没有session required pam_limits.so，没有就加上 保存后，重启操作系统生效
<!-- more -->

## 修改sysctl.conf配置文件，优化TCP参数（针对大量TCP连接场合）

`vi /etc/sysctl.conf`在最后添加以下内容

```shell
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1
```

## 编译并启用hybla模块（系统自带的跳过本节）

查看系统可用算法，如果有显示hybla就表示系统自带

```shell
sysctl net.ipv4.tcp_available_congestion_control
```

### 安装开发工具：

```shell
yum -y groupinstall "Development Tools"
yum -y install ncurses-devel ncurses
```

### 下载同版本内核并解压

查看当前系统的内核版本

```shell
uname -r
```

去 [https://www.kernel.org/pub/linux/kernel/v3.0/](https://www.kernel.org/pub/linux/kernel/v3.0/) 下载相同内核版本（本例为linux-3.19.1）的源码到任意目录（比如/root/mykernel），解压文件。

```
mkdir /root/mykernel
cd /root/mykernel
wget https://www.kernel.org/pub/linux/kernel/v3.0/linux-3.19.1.tar.gz
tar xzvf linux-3.19.1.tar.gz
```

### 编译hybla模块

复制系统当前配置文件：

```shell
cd /root/mykernel/linux-3.19.1
zcat /proc/config.gz > .config
```

进行内核配置：

```shell
make menuconfig
```

进入配置界面后，箭头键移动光标用回车逐级选择：

```shell
Networking suport - Networking options - TCP advanced congestion control - TCP-Hybla congestion control algorithm
```

选中TCP-Hybla congestion control algorithm后，按下M设置为模块；设置完成后移动光标选择Save再Exit退出。

接下去进行模块编译：

```shell
make modules
```

会在/root/mykernel/linux-3.19.1/net/ipv4/目录生成tcp_hybla.ko模块。

注意：千万不能运行命令make modules_install，否则将带来严重的后果，它会删除你系统中的所有模块，只安装刚刚编译的模块。

设置开机自动加载hybla模块

复制tcp_hybla.ko模块到系统特定位置，就可以实现开机自动加载：

```shell
mkdir -p /lib/modules/3.19.1-x86_64-linode53/kernel/net/ipv4
cd /lib/modules/3.19.1-x86_64-linode53/kernel/net/ipv4
cp /root/mykernel/linux-3.19.1/net/ipv4/tcp_hybla.ko .
```

### 开启DigitalOcean等KVM自带的hybla模块

添加开机自动运行脚本

`vi /etc/sysconfig/modules/hybla.modules`输入以下内容

```shell
#!/bin/sh
/sbin/modprobe tcp_hybla
```

保存退出，添加该文件可执行属性

```shell
chmod +x /etc/sysconfig/modules/hybla.modules
```

## 修改/etc/sysctl.conf 让开机自动设置hybal为优先（针对高延迟网络环境）

`vi /etc/sysctl.conf`在最后加入以下内容

```shell
# for high-latency network
net.ipv4.tcp_congestion_control=hybla
# forward ipv4
net.ipv4.ip_forward = 1
```
保存生效
```shell
sysctl -p
```
对于低延迟的网络（如日本，香港等），可以使用`htcp`，可以非常显著的提高速度，首先使用`modprobe tcp_htcp`开启，再将`net.ipv4.tcp_congestion_control = hybla`改为`net.ipv4.tcp_congestion_control = htcp`，建议EC2日本用户使用这个算法。