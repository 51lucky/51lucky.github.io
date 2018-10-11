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

## 代码同步

```shell
# 拉取远程仓库的所有分支的更新
$ git fetch origin

# 拉取远程仓库对应分支上的更新
$ git fetch origin master

# 在当前分支上合并指定分支(远程，本地)
$ git merge [origin/branch-name]
$ git merge [branch-name]

# 取回远程仓库某个分支的更新，再与本地的指定分支合并
$ git pull origin [service-branch-name]:[local-branch-name]

# 提交本地分支feature-x作为远程feature-x分支
$ git push origin feature-x:feature-x

# 推送所有分支到远程仓库
$ git push [remote] --all
```
##  分支管理

```shell
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch-name]

# 新建一个分支，指向指定的commit
$ git branch [branch-name] [commit-id]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch-name] [remote/branch-name]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream-to [branch-name] [remote/branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 删除本地分支
$ git branch -d [local-branch-name]

# 删除远程分支
$ git push origin --delete [remote-branch-name]
$ git push origin :[remote-branch-name]

# 删除远程分支在本地的映射
$ git remote prune origin
```
## 工作流程

标准工作流程（Git Flow）

![分支](/images/git/IMG_0391.png)

两个长期分支:

> * 主分支master
> * 开发分支develop

三个短期分支:

> * 功能分支（feature branch）
> * bug分支（bugfix branch / hotFix branch）
> * 预发布分支（release branch）

## 标签

```shell
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit-id]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tag]

# 查看tag及tag上的标签说明信息
$ git show [tag]

# 提交指定tag
$ git push origin [tag]

# 提交所有tag
$ git push origin --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch-name] [tag]
```

## 查看信息

```shell
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 根据关键词，搜素提交历史
$ git log -S [keyword]

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [commit-id]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD
```


## 撤销

```shell
# 用暂存区的指定文件覆盖工作区的该文件
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit-id] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]
$ git reset HEAD [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 将commit-id之前的commit全部取消，但文件改动还存在，可以重新add，commit
$ git reset --soft commit-id
```
## 其它

```shell
# 查看git历史命令记录
$ git reflog

# 将工作区改动放到存储区/缓存区，此时工作区是干净的（未改动之前的）
$ git stash

# 查看存储区/缓存区的列表
$ git stash list

# 恢复缓存区的文件到工作区
$ git stash apply

# 删除缓存区
$ git stash drop

# 恢复缓存区的文件并删除缓存区
$ git stash pop
```
## <font color=red>注意</font> 

当项目含有子仓库，并子仓库发生变更时，要先处理子仓库变更。