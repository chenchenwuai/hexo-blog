---
title: Nginx出现403 forbidden
date: 2021-03-24 18:16:05
tags: 
  - nginx
categories: nginx
---

Nginx出现403 forbidden的四种解决方法。
<!--more-->

### 一、server配置缺少index
server配置中的缺少 `index index.html index.htm index.php`
```nginx website.conf
server{
  listen       80;
  server_name  _;
  index index.html index.htm index.php; #缺少这一行内容
  root  /var/www/html;
}
```
添加后要重启nginx

### 二、nginx启动用户和工作用户不一致
查看nginx的启动用户和nginx.conf里面的user 用户是否一致
查看启动用户
```shell
ps aux | grep "nginx: worker process" | awk '{print $1}'
```
查看nginx.conf里面的user配置
```nginx website.conf
user root; #一般是root
```
如果两个用户不一致，可以将user的配置改为启动用户

### 三、nginx没有访问web目录的权限
修改web目录的读写权限，或者是把nginx的启动用户改成目录的所属用户，重启nginx即可解决
```shell
chmod -R 777 /var/www/html
# or
chown -R root:root /var/www/html #然后将nginx的user配置为root
```

### 四、SELinux为开启状态
```shell
vim /etc/selinux/config # 查看配置文件
#如果 SELINUX的值不值disabled,需要改为disable
# 改为  SELINUX=disabled
# 然后保存，重启系统  reboot
reboot
```