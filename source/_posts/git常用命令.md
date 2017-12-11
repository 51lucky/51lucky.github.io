---
title: git常用命令
categories: 工具
abbrlink: 423abe9e
date: 2017-04-22 14:29:15
tags: git
---

1. git clone 项目地址 本地项目别名

2. git add 文件名

3. git add -A / git add --all 添加所有文件到仓库

4. git commit -m "提交文件的附加信息，log信息"

5. 将该目录下的文件推送到远端（origin）上的feature_x分支

   git push -u origin feature_x
   <!-- more -->
6. git init 初始化一个空的项目

7. 将本地仓库，关联到远端，然后才能进行推送

   git remote add origin  https://git.coding.net/codingTutorial/gitDemo.git

8. 提交本地feature_x分支作为远程feature_x分支

   git push origin feature_x:feature_x

9. 创建本地dev分支，并将本地dev分支和远端dev分支进行关联

   git checkout -t origin/dev 

10. 在功能分支上合并其他分支内容
  git merge f2 or git merge origin/f2

11. 删除远端分支（已删除的分支）在本地的映射

   git remote prune origin

12. 建立本地分支和远程分支的关联

   git branch --set-upstream branch-name origin/branch-name

13. 创建"feature_x"分支，并切换分支

   git checkout -b feature_x

14. 删除本地分支

   git branch -d feature_x

15. 删除远端分支

   git push origin :feature_x

16. 将分支推送到远端仓库

   git push origin branch

17. 更新  git pull 

18. 查看修改过的文件  git status

19. 查看文件修改内容  

   git diff readme.txt

20. 合并分支，从分支feature_y合并到feature_x分支

   git checkout feature_x 先切换到feature_x分支  在进行合并git merge feature_y

21. 查看你当前所在的分支

   git branch -a

22. 删除你不想要的文件，需先提交到暂存区

   git add -A

   git reset --hard

23. 回退到相应的版本

   git reset --hard commit-id

24. 当改动的文件中没有新添加的文件时

   git commit -am "提交文件的附加信息，log信息" 该命令不会将新添加的文件加入到工作区

25. 将commit_id之前的commit全部取消。但文件改动还存在，可以重新add,commit

   git reset --soft commit_id

26. git reflog 查看命令历史

   用来记录你的每一次命令：

27. git stash

   将改动的代码放到存储区

28. git stash list 

   查看存储区

29. git stash apply 

   将存储区的的文件恢复

30. git stash drop 

   删除存储区

31. git stash pop    

   将存储区的文件恢复并删除

32. git fetch 

   拉取远端服务器上的内容

33. 解决完冲突后，先用git add -A命令将冲突文件提交到暂存区，然后在用git commit -am ""命令提交

34. git tag tag_name

   创建标签

35. git tag 

   查看所有标签

36. git tag tag_name commit_id 

    在commit_id这次提交上添加标签

37. git tag -a tag_name -m "标签说明文字"

   commit_id 创建带有说明的标签

38. git show tag_name 

   查看标签tag_name的说明文字

39. git tag -d tag_name

   删除本地标签

40. git push origin tag_name

   推送标签到远程服务器

41. git push origin --tags 

   推送本地所有未推送到远程的本地标签

42. 删除远程标签：先在本地删除标签，然后删除远端标签 

   git push origin :refs/tags/tag_name