---
title: 忘记 Mysql root账户密码如何修改
date: 2018-03-21 23:06:11
tags: mysql
categories: mysql
---
### 关闭mysql登录验证

打开命令行界面 然后输入打开 *my.cnf* 配置文件的命令
```bash
    vim /etc/my.cnf 
```
<!--more-->

在 [mysqld] 的段中加上一句：skip-grant-tables 

```conf my.cnf
    [mysqld] 
    datadir=/var/lib/mysql 
    socket=/var/lib/mysql/mysql.sock 
    skip-grant-tables 
```

然后保存重启mysql 

```bash
    // mariadb 如果是mysql，则把mariadb改为mysql
    systemctl restart mariadb
```
### 无密码登录
然后登录
```bash
    mysql -uroot -p
    // 然后直接回车，要求输入密码的时候也直接回车
```

### 修改密码

登录之后选择mysql这个数据库然后修改用户密码
```mysql
    use mysql;  --选择要操作的数据库
    UPDATE user SET password = password('vbox_12306') WHERE User = 'root'; -- vbox_12306 为密码
```
回车之后显示下面的表示成功
```mysql
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
```
如果提示下面的错误
```mysql
    The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
```

### 刷新权限
则运行一下刷新权限的命令
```mysql
    flush privileges;
```
然后在执行上面的 update 命令，执行成功之后再次执行 flush 刷新权限命令

### 开启密码登录
此时退出mysql 再次打开 *my.cnf* 配置文件 把已开始加入的 skip-grant-tables 删除 :wq 保存 ，然后重启mysql
```bash
    systemctl restart mariadb
```
然后安正常步骤登录mysql。