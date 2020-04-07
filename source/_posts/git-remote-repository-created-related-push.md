---
title: Git文件管理从入门到精通系统系列：远程仓库创建、关联与推送
date: 2020-03-30 22:31:19
categories:
- [版本控制,git]
- [协同工作,git]
tags:
- 文件管理
- 代码管理
- 文件撤消
- 文件恢复
---

# 1.创建远程仓库（以github为例）

**注册帐号**

打开github网址：[https://github.com/](https://github.com/) ,注册一个帐号，并登录。

**创建一个新的仓库**

登录后，点击链接 `Repositories`,然后选择`new`按钮；如果你已经登录，直接通过网址[https://github.com/new](https://github.com/new)进入到新建仓库地址，输入名字：`testgit`,然后其他选择默认即可,如下图所示：

 ![新建仓库](/images/git/create_new_repository.png)

>关于仓库类型`public`、`private`,主要是用来区分是否允许其他人员可看。   
public:任何人可以查看，但不能提交；创建者可以授权其他人提交。   
private:创建者自己可以提交，查看。

# 2.基于https关联远程仓库并推送本地仓库信息

```shell
#关联
$ git remote add origin https://github.com/gchyhyf/testgit.git
#推送
$ git push -u origin master
---
Username for 'https://github.com':
```

提示要输入帐号信息，这时输入你在github上注册的帐号和密码就可以推送成功，不过每次都这样是不太麻烦？我们换个方法：

```shell
#重新关联
$ git remote set-url origin https://[你的github帐号]:[你的github密码]@github.com/gchyhyf/testgit.git
#推送
$ git push -u origin master
---
Counting objects: 11, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (11/11), 812 bytes | 406.00 KiB/s, done.
Total 11 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To https://github.com/gchyhyf/testgit.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

这时查看远程仓库（gitub)，如下图所示：

 ![firstpush](/images/git/firstpush.png)

这表明推送成功，以后再推送就不用输入帐号和密码啦！这种方法的好处是简单，无需额外配置。

# 3.基于ssh关联远程仓库并推送本地仓库信息

这种方法稍微复杂一些，windows用户请使用`git bash`工具（可以在程序菜单中找到）。

## 3.1设置免密登录
**创建ssh-key,生成公钥和私钥**
```shell
#命令:生成公钥和私钥
$ ssh-keygen -t rsa -C "xxx@xx.com"
---
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa):
Created directory '~/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ~/.ssh/id_rsa.
Your public key has been saved in ~/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ZDpOyZxfjzKJlDqfbV6QA3ciuI544cJeBoZov6Vfzto xxx@xx.com
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|     .           |
|    . o = .      |
|o    + @ +       |
|o+. . @ S .      |
|+oo+ = + = o     |
|o.+++.o.= o .    |
|.oo +o=o.+       |
| . o.o+Eo        |
+----[SHA256]-----+
```    
>上述除了email,其他选项直接回车即可

**查看是否正确生成公钥和私钥**
```shell
$ ls ~/.ssh
id_rsa  id_rsa.pub
```
>id_rsa是私钥，id_rsa.pub是公钥。

**使用ssh-agent代理**
```shell
# 启动让它在后台运行
# windows
$ eval "ssh-agent -s"
---
SSH_AUTH_SOCK=/tmp/ssh-4HutyuJFUJHr/agent.13920; export SSH_AUTH_SOCK;
SSH_AGENT_PID=12108; export SSH_AGENT_PID;
echo Agent pid 12108;
# linux
$ eval "$(ssh-agent -s)"
---
> Agent pid 59324

# windows用户先使用本条命令
$ ssh-agent bash
# 将id_rsa私钥交给ssh-agent保管
$ ssh-add ~/.ssh/id_rsa
```
>注：本步不是必须的，是可选的。用ssh-agent的目的，是为了方便在多个主机上漫游。

**复制公钥内容到github**
```shell
$ cat ~/.ssh/id_rsa.pub
```

打开GitHub“Account settings”,如下图所示：

 ![Account settings](/images/git/github_settings.png)

再点击“SSH and GPG keys”页面，点“New SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容,如下图所示：

 ![sskey](/images/git/github_settings_ssh.png)

 ![sskey](/images/git/github_settings_mac.png)

>假如你需要在多台电脑上切换工作，你可以添加多个主机key,并可以windows-host,mac-host等方式进行区分。

**测试ssh设置是否成功**
```shell
$ ssh -T git@gitub.com
---
The authenticity of host 'github.com (13.229.188.59)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,13.229.188.59' (RSA) to the list of known hosts.
Hi gchyhyf! You've successfully authenticated, but GitHub does not provide shell access.
```
>出现该信息表示设置成功。

## 3.2更改本地仓库与远程仓库关联方式为ssh

**更改本地仓库的关联**
```shell
# 更改
$ git remote set-url origin git@github.com:gchyhyf/testgit.git
# 查看
$ git remote get-url --all origin
---
git@github.com:gchyhyf/testgit.git
```

**推送文件**
```shell
# 新建一个文件
$ echo test ssh push>testssh.txt
$ git add .
# 提交到本地仓库
$ git commit -m "merge ssh/http"
[master 5d433d5] merge ssh/http
# 推送到远程仓库
$ git push -u origin master / git push
---
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 558 bytes | 558.00 KiB/s, done.
Total 6 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
To github.com:gchyhyf/testgit.git
   2eb54e5..5d433d5  master -> master
```
成果如下图所示：

![github-ssh-finished](/images/git/github-ssh-finished.png)

>在只一个分支的情况下，可以直接使用命令`git push`;在第一次Push成功之后，`-u` 参数可以省略。
