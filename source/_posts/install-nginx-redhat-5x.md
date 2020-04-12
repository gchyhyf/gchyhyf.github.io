---
title: 在RedHat 5.6 x64下安装Nginx
description: 有一台客户服务器所安装的系统比较老，据说无法安装Nginx，今天有点时间，特意看了一下顺手解决之。老版本linux的问:版本适配问题；yum源失效问题，无法使用yum安装；无法安装GCC等等
date: 2019-05-23 19:04:21
categories:
- [运维,web-server]
tags:
- linux
- redhat
- nginx
---
　　有一台客户服务器所安装的系统比较老，据说无法安装Nginx，今天有点时间，特意看了一下顺手解决之。

## 1.老版本linux的问题
+ 版本适配问题；
+ yum源失效问题，无法使用yum安装；
+ 无法安装GCC等等

## 2. yum安装方式

**删除原有的yum软件包**

~~~shell
rpm -aq|grep yum|xargs rpm -e --nodeps 
~~~

**下载软件包**

~~~shell
wget http://vault.centos.org/5.6/os/x86_64/CentOS/yum-3.2.22-33.el5.centos.noarch.rpm
wget http://vault.centos.org/5.6/os/x86_64/CentOS/yum-metadata-parser-1.1.2-3.el5.centos.x86_64.rpm
wget http://vault.centos.org/5.6/os/x86_64/CentOS/yum-fastestmirror-1.1.16-14.el5.centos.1.noarch.rpm
wget http://vault.centos.org/5.6/os/x86_64/CentOS/python-iniparse-0.2.3-4.el5.noarch.rpm
wget http://vault.centos.org/5.6/os/x86_64/CentOS/centos-release-5-6.el5.centos.1.x86_64.rpm
wget http://vault.centos.org/5.6/os/x86_64/CentOS/centos-release-notes-5.6-0.x86_64.rpm
~~~
**貌似除了http://vault.centos.org 其他地方都找不到相应的包了。**

**备份原配置文件**
~~~shell
mv /etc/yum.repos.d/CentOS-Base.repo  /etc/yum.repos.d/CentOS-Base.repo.bak
~~~

**创建新配置文件**
~~~shell
vi /etc/yum.repos.d/CentOS-Base.repo 
~~~
输入以下内容：
```
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
baseurl=http://vault.centos.org/5.6/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#released updates
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
baseurl=http://vault.centos.org/5.6/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
baseurl=http://vault.centos.org/5.6/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
baseurl=http://vault.centos.org/5.6/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
#baseurl=http://mirror.centos.org/centos/$releasever/contrib/$basearch/
baseurl=http://vault.centos.org/5.6/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
```
**安装相关软件**
~~~shell
rpm -ivh --force *.rpm
~~~

**清理，生成缓存**
~~~shell
yum clean all 
yum makecache
~~~
此时，yum可用了，你可以做你想做的事啦.......,例如

~~~shell
yum install nginx
~~~

## 3.rpm安装方式
如果觉得上述方式实在太啰嗦，则用rpm包安装吧，三个字：快好省。

**下载：**
~~~shell
wget http://nginx.org/packages/rhel/5/x86_64/RPMS/nginx-1.10.3-1.el5.ngx.x86_64.rpm 
~~~
>注：nginx-1.10.3版本是RedHat 5.6下所能支持的最大Nginx版本。

**安装：**
~~~shell
rpm -ivh nginx-1.10.3-1.el5.ngx.x86_64.rpm
~~~
**运行后会提示：**

~~~text
warning: nginx-1.10.3-1.el5.ngx.x86_64.rpm: Header V3 RSA/SHA1 signature: NOKEY, key ID 7bd9bf62
Preparing...                ########################################### [100%]
   1:nginx                  ########################################### [100%]
----------------------------------------------------------------------

Thanks for using nginx!

Please find the official documentation for nginx here:
* http://nginx.org/en/docs/

Commercial subscriptions for nginx are available on:
* http://nginx.com/products/

----------------------------------------------------------------------
~~~

表示安装成功。

**Nginx目录**

用此种方式安装的Nginx程序所在目录：
~~~text
/usr/sbin/nginx 
~~~
配置文件目录
~~~text
/etc/nginx 
~~~