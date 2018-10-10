---
title: git基本使用
abbrlink: a531cc26
date: 2018-10-10 21:20:10
tags:
---

##  git的操作流程

git的操作流程，简单来说，可以用下图来表示

![流程图](/images/git/bg2015120901.png)

> * Workspace： 工作区
> * Index / Stage: 暂存区
> * Repository：仓库区（本地仓库）
> * Remote：远程仓库

<!-- more -->

## 新建代码库

### 克隆远程代码库

``` shell
# 克隆一个项目  
# project-name: 本地项目名，可以省略，如果省略不写，默认为远程项目名
$ git clone [url] [project-name]

# 克隆一个带有子仓库的项目 
$ git clone --recurse-submodules -j8 [url] [project-name]

```
### 创建代码库

```shell
# 在当前目录下新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 将本地仓库，关联到远程仓库
# origin：远程主机名，git默认将其命名为origin
$ git remote add origin [url]
```
## 配置

```shell
# 显示当前的Git配置
$ git config --list

# 设置提交代码时的用户信息
$ git config --global user.name "[name]"
$ git config --global user.email "[email-address]"

# 让git适当地显示不同的颜色
$ git config --global color.ui true
```
## 增加/删除文件

```shell
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .
$ git add -A

# 删除工作区文件，并且将这次删除放入暂存区（会删除工作区，暂存区或分支上的文件）
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区（只删除暂存区或分支上的文件，本地工作区会保留）
$ git rm --cached [file]

# 改名文件，并且将这个文件放入暂存区
$ git mv [file-original] [file-renamed]
```

添加文件到暂存区示意图 

![添加](/images/git/0.jpg)

## 代码提交

```shell
# 提交暂存区到仓库区
$ git commit -m "[message]"

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m "[message]"

# 提交暂存区所有文件到仓库区
$ git commit -am "[message]"

# 使用一次新的commit，替代上一次提交，主要用来修改上一次commit的提交信息
$ git commit --amend -m "[message]"

# 重做上一次commit,并包括指定文件的新变化
$ git commit --amend [file1] [file2] ... -m "[message]"
```

将暂存区的文件到仓库区示意图

![提交](/images/git/1.jpg)