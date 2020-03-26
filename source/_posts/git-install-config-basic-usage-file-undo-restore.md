---
title: Git文件管理从入门到精通系统系列：安装配置、基本使用与撤消恢复
date: 2020-03-26 21:26:18
categories:
- [版本控制,git]
- [协同工作,git]
tags:
- 文件管理
- 代码管理
- 文件撤消
- 文件恢复
---
# 1.Git起步
## 1.1GIT使用背景知识（必读）
**git是什么？**  
Git是目前世界上最先进的、开源的、分布式的版本控制系统，注意这里面最重要的一个词：**分布式**。  
这意味着，每个人都可以通过git独立工作并且电脑上有自己的独立仓库，当需要协同工作时，以某种方式进行协作即可。  
git不仅可以用来管理软件源代码，也可以用来管理各类文档，尽管它被大多数程序员用来管理程序代码。

**git协同工作的方式**  
通过本地-->中央（远程）仓库，中央（远程）仓库-->本地的方法进行同步。具体来说，就是一部分团队成员先在本地工作，然后推送（同步）至中央（远程）仓库，而另一部分团队成员再从中央仓库拉取（同步）至本地（远程）仓库。

**git使用的相关概念**

现在请注意，如果你希望后面的学习更顺利，请记住下面这些关于 Git 的概念。

+ **中央（远程）仓库:** 方便团队协作的中介和枢纽，也作为一个组织进行资料（如代码）统一管理和备份的必要方式、措施。
+ **本地仓库：** 团队成员用于本机工作的目录,本地仓库分为三个区域：  
   - Working Tree：工作目录(也叫Working Area，工作区)，即可以用`ls/dir`命令可以看到的文件等信息。工作区是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。
   - Stage Area：暂存区域。暂存区是一个文件(即文件Index)，保存了下次将要提交的文件列表信息，一般在 Git 仓库目录中。 按照 Git 的术语叫做“索引”，不过一般说法还是叫“暂存区”。
   - .git Directory: Git 仓库目录(repository)是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，复制的就是这里的数据。

 ![三种状态](/images/areas.png)

+ **三种状态：**  Git 有三种状态，你的文件可能处于其中之一： 已提交（committed）、已修改（modified） 和 已暂存（staged）。   
   - 已修改表示修改了文件，但还没保存到数据库中。   
   - 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。   
   - 已提交表示数据已经安全地保存在本地数据库中。

   这会让我们的 Git 项目拥有三个阶段：工作区、暂存区以及 Git 目录，即在上文提到的本地仓库的三个区域。

 ![三种状态](/images/lifecycle.png)

**基本的 Git 工作流程如下：**
+ 在工作区中修改文件。
+ 将你想要下次提交的更改选择性地暂存，这样只会将更改的部分添加到暂存区。
+ 提交更新，找到暂存区的文件，将快照永久性存储到 Git 目录。
>如果 Git 目录中保存着特定版本的文件，就属于 已提交 状态。 如果文件已修改并放入暂存区，就属于 已暂存 状态。 如果自上次检出后，作了修改但还没有放到暂存区域，就是 已修改 状态。

## 1.2Git安装
**Ubuntu LINUX**
```shell
$ sudo apt install git(git-all)
```
**Centos LINUX**
```shell
$ sudo yum install git(git-all)
```
**Mac OS**
```shell
$ brew install git(git-all)
```
**在 Windows 上安装**
在 Windows 上安装 Git 也有几种安装方法。 官方版本可以在 Git 官方网站下载。 打开 [https://git-scm.com/download/win](https://git-scm.com/download/win)，下载会自动开始。 要注意这是一个名为 Git for Windows 的项目（也叫做 msysGit），和 Git 是分别独立的项目；更多信息请访问 [http://msysgit.github.io/](http://msysgit.github.io/)。

要进行自动安装，你可以使用 [Git Chocolatey](https://chocolatey.org/packages/git) 包。 注意 Chocolatey 包是由社区维护的。

另一个简单的方法是安装 GitHub Desktop。 该安装程序包含图形化和命令行版本的 Git。 它也能支持 Powershell，提供了稳定的凭证缓存和健全的换行设置。 稍后我们会对这方面有更多了解，现在只要一句话就够了，这些都是你所需要的。 你可以在 GitHub for Windows 网站下载，网址为 [GitHub Desktop](https://desktop.github.com/) 网站。

## 1.3Git基本配置设置
**查看所有的配置以及它们所在的文件：**
```shell
$ git config --list --show-origin
```
**修改配置文件及路径：**
```shell
$ git config -f <file>
```
**用户信息配置**  
安装完 Git 之后，要做的第一件事就是设置你的用户名和邮件地址。 这一点很重要，因为每一个 Git 提交都会使用这些信息，它们会写入到你的每一次提交中，不可更改：
```shell
$ git config --global user.name "Guo chunhui"
$ git config --global user.email gch@qq.com
```

再次强调，如果使用了 --global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。 当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行没有 --global 选项的命令来配置。

很多 GUI 工具都会在第一次运行时帮助你配置这些信息。

**文本编辑器**
既然用户信息已经设置完毕，你可以配置默认文本编辑器了，当 Git 需要你输入信息时会调用它。 如果未配置，Git 会使用操作系统默认的文本编辑器，例如vi（windows版本会默认一个类似vi的编辑器) 。

如果你想使用不同的文本编辑器，例如 Emacs，可以这样做：
```shell
$ git config --global core.editor emacs
```
在 Windows 系统上，如果你想要使用别的文本编辑器，那么必须指定可执行文件的完整路径。 它可能随你的编辑器的打包方式而不同。

设置windows下默认的记事本作为编辑器代码如下：

```shell
$ git config core.editor notepad
```

# 2.创建新仓库

**创建测试目录**
```shell
$ mkdir localtest
$ cd localtest
```
**对目录初始化**
```shell
$ git init
---
#此时应该出现下列信息：
Initialized empty Git repository in xxx/localtest/.git
```
**查看状态**
```shell
On branch master
No commits yet
nothing to commit (create/copy files and use "git add" to track)
```

# 3.文件新增、提交、删除以及撤消
## 3.1创建新文件
**创建一个文件,并查看状态**
```shell
$ echo 'hello'>readme
$ git status
---
On branch master  
No commits yet
Untracked files:  
  (use "git add <file>..." to include in what will be committed)  
        readme  
nothing added to commit but untracked files present (use "git add" to track)
```
>注意到 Untracked files下方出现了readme文件，这说明readme文件尚未纳入版本管理系统。  
Untracked files的意思是“未跟踪的文件”。

## 3.2从暂存区中撤消
**加入到暂存区,并查看状态**
```shell
$ git add readme
$ git status
---
On branch master  
No commits yet
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   readme
```
>注意`new file:   readme`这一行，说明readme已经被加入了暂存区。

**由于某种原因，现在不想将readme加入暂存区，需要移除**
```shell
$ git rm --cached readme
---
rm 'readme'

$ git status
---
On branch master  
No commits yet
Untracked files:  
  (use "git add <file>..." to include in what will be committed)  
        readme  
nothing added to commit but untracked files present (use "git add" to track)
```
>执行完`git rm --cached readme`命令后，发现回到了3.1的初始状态。

## 3.3从本地仓库删除
>由于readme文件只提交了一次，所以无法进行撤消。  
如果需要从本地仓库中移走这个文件，你需要执行的是删除。

**从本地仓库中删除**
```shell
$ git rm -- readme
---
rm 'readme'
 ```
**查看仓库状态**
```shell
$ git status
---
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        deleted:    readme
```
**提交操作，完成删除操作**
```shell
$ git commit -m "test rm file" -- readme
---
[master 79cf27c] test rm file
 1 file changed, 1 deletion(-)
 delete mode 100644 readme

$ git status
---
On branch master
nothing to commit, working tree clean
```
## 3.4从本地仓库撤消删除
**查看仓库操作历史记录**
```shell
$ git log --pretty=oneline
---
79cf27c63142370db73b2d7812417618c4634bb2 (HEAD -> master) test rm file
d429425f21866a39f6291c9ab822ce9c0214fb9f test commit and undo
 ```
**撤消删除，恢复到我们删除之前的版本**
```shell
$ git reset --hard HEAD^  # WINDODOWS 下用 git reset --hard "HEAD^"
$ git log --pretty=oneline
---
d429425f21866a39f6291c9ab822ce9c0214fb9f (HEAD -> master) test commit and undo
```
# 4.文件修改、比较、撤消
## 4.1 新增一个文件learngit.txt文件并修改readme
```shell
#新增文件
$ echo "How to learn git?">learngit.txt
#修改文件
$ echo Hello World!>readme
#查看状态
$ git status
---
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
        modified:   readme
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        learngit.txt
no changes added to commit (use "git add" and/or "git commit -a")
```
>注意到 `modified:   readme`这一行，有这个信息说明文件readme确实被修改过了。

## 4.2 确认修改的内容是否正确
```shell
$ git diff
---
diff --git a/readme b/readme
index 30f1577..980a0d5 100644
--- a/readme
+++ b/readme
@@ -1 +1 @@
-'hello'
+Hello World!
```
>`-'hello'`表示我们移除了，`'hello'`这几个字符；  
`+Hello World!`表示我们增加了`Hello World!`;  
换句话说，将文件readme中的`'hello'`替换成了`Hello World!`。

## 4.3撤消对文件readme的修改
```shell
$ git checkout -- readme
#查看状态
$ git status
---
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        learngit.txt
```
## 4.4重新修改文件并加入到暂存区
```shell
#重新修改文件
$ echo Hello World!>readme
#将所有变更加入暂存区
$ git add .
#查看状态
$ git status
---
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        new file:   learngit.txt
        modified:   readme
```
## 4.5将文件从暂存区撤消
在3.2节，我们说到文件从暂存区撤消可以使用`git rm --cached -- <filename>`，如果要撤消所有暂存区的变更，可以执行`git rm --cached -- *`。  
>但是！但是！但是！重要的事说三遍：如果暂存区文件全是新增的，这是没有问题的；如果有些文件属于修改操作，当执行`git rm --cached`操作后，这些文件将变为未控制文件，并在进行commit操作之后，从仓库中移除。

**使用`git reset`命令进行撤消**

```shell
#从暂存区撤消单个文件
$ git reset head readme
$ git reset head learngit.txt
#将所有变更从暂存区撤消
$ git reset head
#查看状态
$ git status
---
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
        modified:   readme
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        learngit.txt
no changes added to commit (use "git add" and/or "git commit -a")
```
>本节介绍的方法，对文件新增、修改等均有效，且内容不会丢失。

# 5.文件在工作目录中被删除之后的恢复（撤消）

>在工作目录中删除，是指通过操作系统进行删除操作，例如通过del,rm命令。

## 5.1未添加到暂存区之前删除恢复

分两种情况：
+ 对于新文件（从未提交到仓库过的文件），删除后的撤消只能依赖于操作系统或第三方软件来处理。
+ 对于以前已经提交仓库、进行修改过的文件，删除后只能恢复到最后一次提交的内容。

```shell
#利用操作系统命令删除
$ rm/del readme  
#查看状态
$ git status
---      
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
        deleted:    readme
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        learngit.txt
#恢复文件readme
$ git checkout -- readme        
```
>最后一步恢复文件readme,但在工作区对文件的所有修改丢失。

## 5.2添加到暂存区之后删除恢复

```shell
#将所有变更加入暂存区
$ git add .  
#利用操作系统命令删除
$ rm/del readme  
$ rm/del learngit.txt
#查看状态
$ git status
---      
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        new file:   learngit.txt
        modified:   readme
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
        deleted:    learngit.txt
        deleted:    readme
#恢复文件
$ git checkout -- readme
$ git checkout -- learngit.txt    
#查看状态
$ git status
---      
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        new file:   learngit.txt
        modified:   readme  
```

>最后一步不仅在工作区恢复了文件（内容没有丢失），并且文件状态也回到正常状态。

## 5.3文件提交到仓库后删除恢复
```shell
#将所有变更提交到仓库
$ git commit -m "test add and modify commmit"
#查看状态
$ git status
---
On branch master
nothing to commit, working tree clean
#利用操作系统命令删除
$ rm/del learngit.txt
#查看状态
$ git status
---  
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
        deleted:    learngit.txt

#恢复文件
$ git checkout -- learngit.txt         
#查看状态
$ git status
---  
On branch master
nothing to commit, working tree clean
```
# 6.合并提交
有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 --amend 选项的提交命令来重新提交：
```shell
#新建文件forgotten
$ echo this is forgotten test>forgotten
#添加到暂存区之后删除恢复
$ git add .  
#查看当前提交历史记录
$ git log --pretty=oneline
--
af6d5646b66d08cb182f626ffaa62e53ee7898c8 (HEAD -> master) test add and modify commmit  
d429425f21866a39f6291c9ab822ce9c0214fb9f test commit and undo
#将当前暂存区内容合并到最后一次提交
$ git commit --amend --no-edit
[master 69e1e2e] test add and modify commmit
 Date: Thu Mar 26 15:02:06 2020 +0800
 3 files changed, 3 insertions(+), 1 deletion(-)
 create mode 100644 forgotten
 create mode 100644 learngit.txt
 #再次查看提交历史
 $ git log --pretty=oneline
 --
 0e3c4ff5255197d9dfd93549afe1befe2139deb7 (HEAD -> master) test add and modify commmit  
 d429425f21866a39f6291c9ab822ce9c0214fb9f test commit and undo
```
>观察上述命令输出的结果，发现提交历史记录仍然是2条，如果上面的`git commit --amend`命令换成正常提交，则此处会有3条提交历史记录。  
另外，若没有添加` --no-edit`参数，则会打开一个编辑窗口，可以修改两次提交的信息。
