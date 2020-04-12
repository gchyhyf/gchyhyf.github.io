---
title: Git文件管理从入门到精通系列(3)：远程仓库的克隆、管理与使用
description: 在前面的文章中，我们介绍了如何在本地创建git仓库、管理文件，并将它与远程仓库关联，以及推送文件到远程仓库。本文介绍创建git仓库的第二种方法--将远程仓库克隆到本地以及如何管理、使用它。
date: 2020-04-12 01:38:28
categories:
- [版本控制,git]
- [协同工作,git]
tags:
- 文件管理
- 代码管理
- 文件撤消
- 文件恢复
---

# 前言
在前面的文章中，我们介绍了如何在本地创建git仓库、管理文件，并将它与远程仓库关联，以及推送文件到远程仓库。本文介绍创建git仓库的第二种方法--将远程仓库克隆到本地以及如何管理、使用它。
# 1.克隆远程仓库到本地
克隆远程仓库到本地非常简单，你可以选择使用ssh协议、http或https协议。例如把我们之前在github上创建的`testgit`克隆到本地，命令：
```shell
#使用ssh协议
git clone git@github.com:gchyhyf/testgit.git
#使用https协议
git clone https://[你的github帐号]:[你的github密码]@github.com/gchyhyf/testgit.git
```
上述命令会将`testgit`克隆到testgit目录下，你也可以指定一个目标文件夹，例如：
```shell
git clone git@github.com:gchyhyf/testgit.git mytest
```
上述命令会将`testgit`克隆到mytest目录下。

# 2.查看远程仓库
**只列出远程仓库的名字**

```shell
git remote
---
origin
```
上述命令中，输出信息`origin`指的是远程仓库的名字或者说是远程仓库的一个简称，请记住：它仅仅是一个名字而已。

**列出远程仓库的名字与对应URL信息**

```shell
git remote -v
---
origin	https://xx:xxxx@github.com/gchyhyf/testgit.git (fetch)
origin	https://xx:xxxx@github.com/gchyhyf/testgit.git (push)
```
通过上述命令，可以看到：`origin`对应着两个一样的url，一个用来抓取(fetch)，一个用来推送(push)。

**列出远程仓库名对应的不重复URL**

```shell
git remote get-url --all origin
---
https://xx:xxxx@github.com/gchyhyf/testgit.git
```

**查看某个远程仓库的详细信息,例如下述命令显示了`origin`的详细信息:**

```shell
git remote show origin
---
* remote origin
  Fetch URL: https://xx:xxxx@github.com/gchyhyf/testgit.git
  Push  URL: https://xx:xxxx@github.com/gchyhyf/testgit.git
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

# 3.添加远程仓库
**新增一个远程仓库与URL**

假如我们在github上还有一个仓库叫做`testgit1`,可以这样添加：
```shell
#添加https://github.com/gchyhyf/testgit1.git，并命名为test1
git remote add test1 https://github.com/gchyhyf/testgit1.git
#查看
git remote -v
---
origin	https://xx:xxxx@github.com/gchyhyf/testgit.git (fetch)
origin	https://xx:xxxx@github.com/gchyhyf/testgit.git (push)
test1	https://github.com/gchyhyf/testgit1.git (fetch)
test1	https://github.com/gchyhyf/testgit1.git (push)
```

**更改某一个远程仓库的URL**

如果发现上一步通过add添加远程仓库时，不小心把URL写错了（比如没有设定帐号、密码），可用下列命令进行重新设置：
```shell
git remote set-url test1 https://xx:xxxx@github.com/gchyhyf/testgit1.git
git remote -v
---
origin	https://xx:xxxx@github.com/gchyhyf/testgit.git (fetch)
origin	https://xx:xxxx@github.com/gchyhyf/testgit.git (push)
test1	https://xx:xxxx@github.com/gchyhyf/testgit1.git (fetch)
test1	https://xx:xxxx@github.com/gchyhyf/testgit1.git (push)
```

# 4.远程仓库的`抓`取与`拉`取
## 4.1从远程仓库抓取数据
使用命令`git fetch <remote-name>` 可以将数据下载到本地，例如：
```shell
(1) git fetch test1
(2) git fetch test1 develop
```
上述命令解释如下：
+ 命令(1)表示把`test1`所指向的远程仓库（URL)数据下载到本地(本地没有的所有数据);
+ 命令(2)表示把`test1`所指向的远程仓库（URL)的`develop`分支数据下载到本地;
+ 以上均不进行合并操作；如果有必要合并，则需要手动合并（通过`git merge`）。
## 4.2从远程仓库拉取数据
使用命令`git pull <remote-name> <branch-name>`  也可以将数据下载到本地，例如：
```shell
(1) git pull test1 master
(2) git pull test1 develop
(3) git pull
```
以上命令(1)和(2)表示把`test1`所指向的远程仓库（URL)的`master`、`develop`分支数据下载到本地，并与本地分支进行自动合并，这也是它与`git fetch`的区别。
命令(3)是一种简写，如果本地与远程分支已经做了关联的情况下可以使用。例如，在首次克隆远程仓库到本地没有进行其他额外设置时，等效于：
```shell
git pull origin master
```
# 5.推送数据到远程仓库
使用命令`git push <remote-name> <branch-name>`  可以将数据推送到远程仓库，例如：
```shell
(1) git push test1 master
(2) git push test1 develop
(3) git push -u test1 develop
(4) git push
```

上述命令解释如下：
+ 以上命令(1)和(2)表示把本地数据推送到`test1`所指向的远程仓库（URL)的`master`、`develop`分支；
+ 命令(3)除了达到(2)的效果之外，同进将本地当前分支与远程仓库分支进行了关联；
+ 命令(4)在命令执行完之后，以后再执行就可以简写等效于(3)。

> 关于分支的关联问题，我们在后续文章中再详细介绍。

# 6.远程仓库改名与删除
使用命令`git remote rename `可以对远程仓库改名，例如将远程仓库origin改名为一个字母o：
```shell
git remote rename origin o
git remote
---
o
test1
```
如果某个远程仓库不需要了，可以使用可以使用 `git remote remove` 或 `git remote rm` 进行删除，例如删除`test1`:
```shell
git remote rm test1
git remote
---
o
```