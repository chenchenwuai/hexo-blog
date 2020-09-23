---
title: mysql root账户远程不能登录解决方法
date: 2019-06-05 22:29:31
tags: mysql
categories: mysql
---
### 原因
mysql 远程不能登录一般是没有设置 *host* 为 *%* 或者对应的ip地址
这种情况下一般是通过修改原来 *root* 账户的 *host* 为 *%* 来进行远程登录，但是这种设置也是有一个缺点，就是某种情况下本地命令行就不能登录了。
<!--more-->

### 解决方法
所以正确的做法是新创建一个 *root* 账户，但是 *host* 为 *%* 。具体操作为

```mysql
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '12306' WITH GRANT OPTION; -- 12306 是密码
    flush privileges;
```
这时候在使用远程连接工具登录 *root* 账户就可以了。
总结一下，就是如果需要用户在本地和远程都可以连接，则需要创建两个账户，一个 *host* 为 *localhost*;另一个 *host* 为 *%*,当然这种方式并不安全，建议两个用户密码不一样

### 语句分析
+ *ALL PRIVILEGES*  
    表示将所有权限授予给用户。也可指定具体的权限，如：SELECT,CREATE,DROP等。
    GRANT SELECT,CREATE,UPDATE,ALTER,DROP ON....
+ *ON*
    表示这些权限对哪些数据库和表生效，格式：数据库名.表名，这里写“*”表示所有数据库，所有表。如果我要指定将权限应用到test库的user表中，可以这么写：test.user
+ *TO*
    将权限授予哪个用户。格式：”用户名”@”登录IP或域名或%”。%表示没有限制，在任何主机都可以登录。比如："web_user"@"192.168.0.%",表示web_user这个用户只能在192.168.0IP段登录
+ *IDENTIFIED BY*
    指定用户的登录密码
+ *WITH GRANT OPTION*
    表示允许用户将自己的权限授权给其它用户
+ *flush privileges*
    更新权限信息
使用GRANT给用户添加权限，权限会自动叠加，不会覆盖之前授予的权限，比如你先给用户添加一个SELECT权限，后来又给用户添加了一个INSERT权限，那么该用户就同时拥有了SELECT和INSERT权限。

[详细的权限列表信息](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html)

