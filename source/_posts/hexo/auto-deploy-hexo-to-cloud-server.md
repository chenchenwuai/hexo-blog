---
title: 一键部署hexo程序到云服务器[简易版]
date: 2021-07-03 11:32:45
tags: 
  - hexo
  - deploy
categories: hexo
---

虽然平常写文章不多，但是有了这个功能还是能节省很多时间的。功能简单了点，原理是利用hexo的部署脚本[`hexo deploy`](https://hexo.io/zh-cn/docs/deployment#Git)，推送`commit`到云服务器，然后调用`Git`的`hook`功能，将编译好的html及其他文件移动nginx配置的文件夹下面。
<!--more-->

### 一、云服务器配置
请先确保已经安装git，并配置好了用户和email。确保已经配置好了nginx。
#### 1. 创建blog文件夹
首先创建`blog`文件夹，这就是nginx配置的博客的根目录，如果已经创建过，无需再创建
```shell
mkdir /var/www/blog
```
#### 2. 初始化git仓库
然后创建git程序的文件夹和初始化hexo程序的git仓库`hexo.git`(仓库名称可自定义)
```shell
mkdir /var/www/git
cd /var/www/git
git init --bare hexo.git
```
#### 3. 创建git hook
首先创建git hook文件
```shell
vim /var/www/git/hexo.git/hooks/post-receive
```
然后写入以下内容
```shell
#!/bin/bash
git --work-tree=/var/www/blog --git-dir=/var/www/git/hexo.git checkout -f
```
给文件增加可执行权限
```shell
chmod +x /var/www/git/hexo.git/hooks/post-receive
```

### 二、本地配置
本地配置很简单，需要安装`hexo-deploy-git`库,并配置根目录下面的`_config.yml`文件中的`deploy`项
#### 1. 安装 hexo-deploy-git
可能hexo程序已经自带此依赖，可以在package.json里面查看是否已存在
```node
npm install hexo-deploy-git -D
```
#### 2. 配置 _config.yml
首先找到deploy配置项，type和branch按照下面配置不变
里面repo的组成方式是 `(你的服务器用户)@(你的服务器ip):git仓库的绝对路径`
```yml
deploy:
  type: 'git'
  repo: root@12.34.56.78:/var/www/git/hexo.git
  branch: master
```
> 如果服务器使用的密钥登录，未防止出错，请先使用其他ssh工具登录一下服务器，
> 如果使用的是密码登录，接下来的操作中可能会需要输入你设置的服务器用户的密码
### 三、开始部署
首先使用`hexo clean`清空一下缓存
然后使用`hexo g`编译文件
最后使用`hexo deploy`命令部署到服务器(此步骤会在根目录下面创建`.deploy_git`文件夹)
等到终端提示 `Deploy done: git` 后就表示已经成功,可以访问浏览器查看页面或者去服务器`/var/www/blog`下面查看部署好的文件
**部署成功**