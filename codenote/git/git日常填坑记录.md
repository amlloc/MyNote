---
title: git日常填坑记录
date: 2018-09-16 22:35:07
categories: codenote
tags: [git, 填坑记录]
---

使用git的零零碎碎
<!--more-->

### 一、每次将代码push到github时都需要输入用户和密码

#### 环境

- 操作系统：Ubuntu 17.10
- 备注：已配置好gitlab和github双账号 
- 原因：`clone/remote add`项目时使用了`http/https`
- 解决：`clone/remote add`项目时使用了`ssh`方式，尽量不使用`http/https`方式

### 二、git log (git lg)

- 通过commit号的一部分模糊查找完整的commit号

  `git log | grep <part of commitid>  `

- 查看某文件在哪些commit之中被修改了

  `git log filename`

- 彩色log(git lg)

  ` git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"`

### 三、git merge

- 强行关闭fast-forward方式，保留分支提交历史（建议每次合并的时候都使用这个参数,因为默认会使用fast-forward模式，此时的分支提交历史不会保留）

  `git merge --no-ff <branch name>`

### 四、git checkout(git co)

- 切换分支

  `git checkout <branch name>`

- 新建本地分支并将远程对应分支提出
  `git checkout -t origin/2.0.0`


### 五、git add <filename>

- 将特定文件/文件夹放置到暂存区

  `git add <filename/document>`

### 六、 git commit(git ci)

- 提交暂存区内的所有改动，并附加说明

  `git commit -m <msg>`

- 在vim中添加附加说明，回车两次则第一行为msg的title

  `git commit`

- 对暂存区的文件commit后显示文件差异

  `git commit -v`

### 七、git status( git st)

- 查看暂存区，工作区，以及未跟踪文件的情况

  `git status`

### 八、git reset

- 取消暂存区某个文件

  `git reset HEAD <filename>` 或 `git unstage <filename>`

- 取消整个暂存区

  `git reset`

### 九、git tag

- 列出tag

  `git tag`

- 给某个特定的commit打tag

  `git tag -a "tag" <commitid> -m "msg"`

- 提交tags

  ` git push origin --tags`

- 移除tag

  本地：`git tag -d V1.2`

  线上：`git push origin :refs/tags/v1.4.5` (这个需要一定的权限才能做到)

### 十、git config

- 查看用户名和邮箱

  `git config user.name`
  `git config user.email`

- 修改用户名和邮箱地址

  `git config --global user.name "username"`

  `git config --global user.email "email"`

- 取消用户名和邮箱地址

  ` git config --global --unset user.name`

  ` git config --global --unset user.email`

### 十一、~/.gitconfig

- 可以直接在这里修改global的name和email
- 可修改git使用时用到的editor

### 十二、git branch

- 查看本地+远程分支

  `git branch -va`

### 十三、.gitignore

- 过滤文件夹

  `/builds/`

- 过滤某种类型文件

  `*.java`、`*.c`

- 指定过滤某个文件

  `/local.properties`

- 保守模式设置哪些文件不被过滤，也就是设置跟踪哪些文件夹

  `!/local.properties`

  `!*.java`、`!*.c`

  `!/builds/`
### 十四、git log(git lg)



### 十五、git branch(git br)

- 查看分支

  `git branch`
