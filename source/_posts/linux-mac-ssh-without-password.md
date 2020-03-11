---
title: linux-mac系统免密登录和复制文件的设置方法
date: 2020-03-06 19:29:16
categories:
- [运维, 操作系统] 
tags: 
-  linux
-  unix
-  mac
-  ssh
-  scp
---
>命令行方式远程登录LINUX或者UNIX，若每次输入帐号密码等信息，既不方便、也不安全，如果是需要复制文件，就更麻烦，所以一般会进行ssh免密设置。
LINUX类系统操作基本类似，若目标主机是MAC个人电脑，则略有不同，下面进行详细说明。

# 1.LINUX类系统的设置方法
## 1.1生成client host的公钥和私钥
命令：
```shell
ssh-keygen -t rsa

Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
05:c7:16:dc:77:d9:06:fc:df:4b:74:74:6d:28:06:f0 root@iZ94cf7z6g3Z
The key's randomart image is:
+--[ RSA 2048]----+
|        o+++ ..o+|
|         ++ + +.O|
|         .E. o *.|
|         .     .o|
|        S     . +|
|               .o|
|              . .|
|               . |
|                 |
+-----------------+
```
一路回车，一切按默认值就好。

上述命令生成的密钥对, 默认保存在 /root/.ssh(或~/.ssh)目录下, 其中
id_rsa 为私钥；id_rsa.pub为公钥

## 1.2使用ssh-copy-id或scp命令将client host的公钥传输到server host或destination host
命令：
```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub login_username@sever-host-ip
```
本命令能将id_rsa.pub内容追加到server host的login_username/.ssh/authorized_keys文件中，
等效于先在client host执行：
```shell
scp ~/.ssh/id_rsa.pub login_username@sever-host-ip:
```
然后在server host上执行：
```shell
cat id_rsa.pub >> ~/.ssh/authorized_keys
```
## 1.3设置权限
设置authorized_keys权限
```shell
chmod 600 authorized_keys
```
设置.ssh目录权限
```shell
chmod 700   ~/.ssh
```
## 1.4双向免密登录
在另一台机器上进行同样的操作即可，这种情况一般在server之间或本地机器之间操作。
# 2.MAC的注意事项
## 2.1开启远程登录
打开系统偏好设置–>共享，勾选远程登录，允许访问设置可以远程登录的用户即可，如下图所示：
## 2.2重启server host上的相关服务（如果设置不生效）
```shell
sudo launchctl unload -w /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```
