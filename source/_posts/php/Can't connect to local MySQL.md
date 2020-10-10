---
title: 解决 Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2）的方法
date: 2020-10-10 10:17:00
tags: 
  - php
  - mysql
  - mysql.sock
categories: php
---

PHP连接MySQL报错：SQLSTATE[HY000] [2002] Can't connect to local MySQL server through socket 'MySQL' (2)

<!--more-->

## 一.问题发现
今天在服务器上同时安装了mysql和mariadb，经过一番操作之后，发现原来的php程序不能连接mysql了，每次连接就会提示
```mysql
SQLSTATE[HY000] [2002] Can't connect to local MySQL server through socket 'MySQL' (2)
```
在网上找了很多解决方法（csdn越来越垃圾了，都是千篇一律的复制粘贴，没有真正解决问题的），么有找到方法，在google找了几个解决方法，经过测试可以使用，并明白了原理。

## 二.问题解析
如果提示这个错误，如果将连接参数中的localhost换成 `127.0.0.1`就能正常使用，这是为什么呢？
这是因为使用localhost是通过套接字(socket)通讯，而`127.0.0.1`是通过tcp/ip协议通讯，而mysql的报错就是没有找到这个套接字文件(mysql.sock)，
一般出现这种情况是因为（个人总结，可能还有其他原因，欢迎补充）
+ 有些安装mysql的方法没有将此文件放在了其他位置,php连接mysql时没有找到这个文件
+ php没有访问这个文件的权限
+ 这个文件被删除了

## 三.问题解决

找到问题了，现在就要解决问题
首先既然php找不到这个文件，那我们先全局找一下这个文件在哪儿
```bash
    sudo find / -name mysql.sock
```
如果没找到这个文件，尝试搜索一下 mysqld.sock
```bash
    sudo find / -name mysqld.sock
```
找到以上两个文件的任一文件，首先记一下文件的绝对路径， 我这边找到的是 `/usr/local/mysql/data/mysqld.sock`
>如果没有找到，需要自己从其他地方复制一份，这个需要自己去搜索解决方法

我找到了两种解决方法
### 1.修改php.ini

打开php的配置文件, 一般是`/etc/php.ini`，
然后找到`[MYSQL]`段落，下面有一个`mysql.default_socket`，将上面记录的socket文件的绝对路径填写到 等号后面。
```ini
    mysql.default_socket= /usr/local/mysql/data/mysqld.sock
```
然后再找到`[Pdo_mysql]`段落，下面有一个`pdo_mysql.default_socket`，将上面记录的socket文件的绝对路径填写到 等号后面。
```ini
    pdo_mysql.default_socket= /usr/local/mysql/data/mysqld.sock
```
修改好之后，保存退出。重启apache或者nginx，必须要重启，不然不生效。
再次测试连接mysql，看是否已经正常。

### 2.设置软连接（推荐）

执行命令即可
```bash
    sudo ln -s /usr/local/mysql/data/mysqld.sock /var/lib/mysql/mysql.sock
```
前一个地址是mysql.sock或mysqld.sock的绝对路径，后一个路径为php没有找到socket文件的路径，我这边是`/var/lib/mysql/mysql.sock`,执行过命令之后，再次重新连接一下mysql，看是否已经正常。

