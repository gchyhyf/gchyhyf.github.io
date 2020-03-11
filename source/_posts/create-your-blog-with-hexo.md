---
title: 用hexo创建你的个人博客--快速入门
date: 2019-04-24 11:24:19
categories:
- 建站
tags:
- 博客
- 网站
- git
---
## Hexo是什么？
　　Hexo是一个静态建站工具，它的网页用Markdown语法编写，它有各种主题（模板）。您可以自行选择并生成漂亮的博客。
## 安装
### 1.安装node.js
　　进入NodeJs官方网站进行下载：[https://nodejs.org](https://nodejs.org/en/)

　　可选择最新的 10.15.3 LTS 版，windows用户下载msi安装包直接安装即可。

### 2.安装Git
　　进入Git官方网站进行下载：[https://git-scm.com](https://git-scm.com)

>　　*Windows 用户*
>　　对于windows用户来说，建议使用安装程序进行安装。安装时，请勾选Add to PATH选项。
>　　另外，您也可以使用Git Bash，这是git for windows自带的一组程序，提供了Linux风格的shell，在该环境下，您可以直接用上面提到的命令来安装Node.js。打开它的方法很简单，在任意位置单击右键，选择“Git Bash Here”即可。由于Hexo的很多操作都涉及到命令行，您可以考虑始终使用Git Bash来进行操作。

### 3.安装Hexo

~~~shell
$ npm install -g hexo-cli
~~~

## 创建一个站点

　　执行下列命令，创建一个站点，folder为站点目录
~~~shell
$ hexo init <folder>
$ cd <folder>
$ npm install
~~~

　　此时，folder下的目录如下：

~~~
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
~~~

## 基本使用

### 1.创建一篇文章

~~~shell
$ hexo new first
~~~

　　在source/_posts下你会发现多了一个first.md文件，打开编辑它：
~~~
---
title: first
date: 2019-04-26 19:24:19
tags:
---
this is my first blog
~~~
### 2.生成静态站

~~~shell
$ hexo g
~~~

### 3.启动server进行测试

~~~shell
$ hexo s
~~~

　　会出现：

~~~shell
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
~~~

　　访问http://localhost:4000 即可查看你的站点

## 安装主题

### 安装
　　将第三方主题下载下来，并放在 themes/下面。
### 配置
　　修改站点根目录下的_config.yml,例如（anodyne是主题文件夹）：
~~~shell
theme: anodyne
~~~
　　另外，每个主题有自己的_config.yml，不可混淆。
## 发布到github,coding
　　安装 hexo-deployer-git
~~~shell
$ npm install hexo-deployer-git --save
~~~
　　修改站点根目录下的_config.yml,增加或修改deploy的内容，如下：
~~~shell
deploy:
  type: git
  repo: git地址
  branch: master
~~~
　　官方设置
~~~shell
deploy:
  type: git
  message: [message]
  repo:
    github: <repository url>,[branch]
    coding: <repository url>,[branch] 
~~~

　　发布：
~~~shell
hexo d 或 hexo deploy
~~~

## 发布到私服

　　要达到类似效果，需要使用git hook.




