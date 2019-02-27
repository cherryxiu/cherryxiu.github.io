---
layout: post
title: "Git操作之cherrypick"
date: 2019-02-25
description: "Git操作，cherry-pick，切换分支提交"
tag: Git

---

* 目录
{:toc}


已知：在分支1上有 代码a,代码b 两部分内容，代码a 已经 commit 但还没有 push，代码b 没有 add <br/> 
需求：将 代码a 内容提交到分支2上，并消除在分支1上的记录


### 前言

  在分支1上努力搬砖时，老大突然要我修改个其它的小需求（也就是  代码a），我本打算先将 代码a 提交到分支1，等我写完 代码b 再提交一次，就可以完成我的任务啦。正当我已经add ，commit 代码a 之后准备 push 到远程分支1时，老大说要把代码a 提交到分支2上。。。emmmm，一脸懵逼，老大过来一顿操作，问题妥妥解决。我就老老实实记个笔记~~~

### 思路历程

1. 在分支1上找到提交的 commit 编号1
2. 切换到分支2，利用`git cherry-pick 编号1`使代码a 复制到分支2，并提交到分支2
3. 切换到分支1，利用`git reset HEAD` 回退版本

### 具体代码

在 分支1 `feature_encrypt_phone`上的操作

```
$ git status
On branch feature_encrypt_phone
Your branch is ahead of 'origin/feature_encrypt_phone' by 1 commit.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   cashloan-core/src/main/resources/config/mappers/UserMapper.xml
no changes added to commit (use "git add" and/or "git commit -a")


$ git log
commit 8068057224480cbd1d8527980c358491c56b36cd
Date:   Thu Feb 21 17:07:40 2019 +0800

    客服时间

commit 640eca897160d762b186c741b13d116bb8176a65
Date:   Thu Feb 21 17:00:57 2019 +0800

    修改   除用户管理以外的所有导出接口


$ git branch -a
* feature_encrypt_phone
  remotes/origin/Release1.1.6R190218
```

在 分支2 Release1.1.6R190218上的操作

```
$ git checkout Release1.1.6R190218
Switched to branch 'Release1.1.6R190218'
M       cashloan-core/src/main/resources/config/mappers/UserMapper.xml
Your branch is up-to-date with 'origin/Release1.1.6R190218'.

$ git status
On branch Release1.1.6R190218
Your branch is up-to-date with 'origin/Release1.1.6R190218'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   cashloan-core/src/main/resources/config/mappers/UserMapper.xml

no changes added to commit (use "git add" and/or "git commit -a")

$ git cherry-pick 8068057224
[Release1.1.6R190218 8fb3985] 客服时间
 Date: Thu Feb 21 17:07:40 2019 +0800
 2 files changed, 2 insertions(+), 2 deletions(-)

$ git status
On branch Release1.1.6R190218
Your branch is ahead of 'origin/Release1.1.6R190218' by 1 commit.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   cashloan-core/src/main/resources/config/mappers/UserMapper.xml

no changes added to commit (use "git add" and/or "git commit -a")

$ git push origin Release1.1.6R190218
Counting objects: 9, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (8/8), done.
Writing objects: 100% (9/9), 754 bytes | 0 bytes/s, done.
Total 9 (delta 6), reused 0 (delta 0)
remote:
remote: ========================================================================
remote:
remote:        Done is better than perfect ，先有后优！发射曳光弹，快速出原型；自动化集成，高频率迭代。
remote:                           没有commit和merge何以服人？
remote:
remote: ========================================================================
remote:
remote:
    Release1.1.6R190218 -> Release1.1.6R190218

$ git status
On branch Release1.1.6R190218
Your branch is up-to-date with 'origin/Release1.1.6R190218'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   cashloan-core/src/main/resources/config/mappers/UserMapper.xml

no changes added to commit (use "git add" and/or "git commit -a")
```

切回到 分支 1 操作

```
$ git checkout feature_encrypt_phone
Switched to branch 'feature_encrypt_phone'
M       cashloan-core/src/main/resources/config/mappers/UserMapper.xml
Your branch is ahead of 'origin/feature_encrypt_phone' by 1 commit.
  (use "git push" to publish your local commits)

$ git status
On branch feature_encrypt_phone
Your branch is ahead of 'origin/feature_encrypt_phone' by 1 commit.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   cashloan-core/src/main/resources/config/mappers/UserMapper.xml

no changes added to commit (use "git add" and/or "git commit -a")


$ git log
commit 8068057224480cbd1d8527980c358491c56b36cd
Date:   Thu Feb 21 17:07:40 2019 +0800

    客服时间

commit 640eca897160d762b186c741b13d116bb8176a65
Date:   Thu Feb 21 17:00:57 2019 +0800

    修改   除用户管理以外的所有导出接口


$ git reset --hard HEAD
HEAD is now at 8068057 客服时间

$ git reset --hard 640eca897160
HEAD is now at 640eca8 修改   除用户管理以外的所有导出接口

$ git status
On branch feature_encrypt_phone
Your branch is up-to-date with 'origin/feature_encrypt_phone'.
nothing to commit, working tree clean

```

###  附加知识点

[[Git] Git整理(五) git cherry-pick的使用](https://blog.csdn.net/fightfightfight/article/details/81039050)  
