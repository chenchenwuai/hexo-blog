---
title: nginx开启gzip和缓存
date: 2020-07-05 19:15:55
tags: 
  - nginx
  - gzip
categories: nginx
---

### 极简配置
在站点的配置文件中(例如`website.conf`)的server中添加`gzip on`.
<!--more-->
```nginx website.conf
server{
  gzip on;
}
```

### 常规配置
按照下面代码进行配置
```nginx website.conf
server{
  gzip on; # 是否开启gzip
  gzip_buffers 32 4K; # 缓冲(压缩在内存中缓冲几块? 每块多大?)
  gzip_comp_level 6; # 推荐6 压缩级别(级别越高,压的越小,越浪费CPU计算资源)
  gzip_min_length 1k; # 开始压缩的最小长度(再小就不要压缩了,意义不在)
  gzip_types application/javascript text/css text/xml; # 对哪些类型的文件用压缩 如txt,xml,html ,css
  gzip_disable "MSIE [1-6]\."; #正则匹配UA，配置禁用gzip条件。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
  gzip_http_version 1.1; # 开始压缩的http协议版本(可以不设置,目前几乎全是1.1协议)
  gzip_vary on; # 是否传输gzip压缩标志
}
```

保存并重启nginx，刷新页面（为了避免缓存，请强制刷新）就能看到效果了。以谷歌浏览器为例，通过F12看请求的响应头部。

### gzip参数详解

#### gzip

```nginx website.conf
#打开或关闭gzip
gzip on; # on | off
```

解释：打开或关闭gzip

#### gzip_buffers

```nginx website.conf
# 设置用于处理请求压缩的缓冲区数量和大小
# 比如32 4K表示按照内存页（one memory page）大小以4K为单位（即一个系统中内存页为4K），申请32倍的内存空间。建议此项不设置，使用默认值
gzip_buffers 32 4k;
```

#### gzip_comp_level

```nginx website.conf
# 设置gzip压缩级别，取值 1-9
# 值越高越消耗cpu的性能，高并发情况下cpu可能达到100%
# 级别越底压缩速度越快文件压缩比越小，反之速度越慢文件压缩比越大，一般6之后压缩比很难提升
# 一方面，gzip_comp_level 1的压缩能力已经够用。另一方面，压缩一定要和静态资源缓存相结合，缓存压缩后的版本，否则每次都压缩高负载下服务器肯定吃不住。
gzip_comp_level 2;
```

#### gzip_disable

```nginx website.conf
# 表明哪些UA(usergent)头不使用gzip压缩，可以正则
gzip_disable "MSIE [1-6]\.";
```

#### gzip_min_length

```nginx website.conf
# 当返回内容大于此值时才会使用gzip进行压缩,以k为单位,当值为0时，所有页面都进行压缩。
gzip_min_length 1k; # 100 | 500 | 1k
```

#### gzip_http_version

```nginx website.conf
# 用于识别http协议的版本，早期的浏览器不支持gzip压缩，用户会看到乱码，所以为了支持前期版本加了此选项。默认在http/1.0的协议下不开启gzip压缩。
gzip_http_version 1.1; # 1.0 | 1.1
```

#### gzip_proxied

```nginx website.conf
# Nginx做为反向代理的时候启用:
#   off – 关闭所有的代理结果数据压缩，默认值
#   expired – 如果header中包含”Expires”头信息，启用压缩
#   no-cache – 如果header中包含”Cache-Control:no-cache”头信息，启用压缩
#   no-store – 如果header中包含”Cache-Control:no-store”头信息，启用压缩
#   private – 如果header中包含”Cache-Control:private”头信息，启用压缩
#   no_last_modified – 启用压缩，如果header中包含”Last_Modified”头信息，启用压缩
#   no_etag – 启用压缩，如果header中包含“ETag”头信息，启用压缩
#   auth – 启用压缩，如果header中包含“Authorization”头信息，启用压缩
#   any – 无条件压缩所有结果数据
gzip_proxied  off;
```

#### gzip_vary

```nginx website.conf
# 浏览器请求增加响应头"Vary: Accept-Encoding"
gzip_vary on;
```

#### gzip_types

```nginx website.conf
# 设置需要压缩的MIME类型,如果不在设置类型范围内的请求不进行压缩，其中的值可以在 mime.types 文件中找到
# 压缩字体类型 font/ttf font/otf image/svg+xml
gzip_types  text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png font/ttf font/otf image/svg+xml;
```
这里需要说明一些特殊的类型，使用”字体类型”的资源，而这些资源类型往往会被忽略，且这些资源又比较大，没有被压缩很不合算。（可以参考：http://www.darrenfang.com/2015/01/setting-up-http-cache-and-gzip-with-nginx/）

### 开启缓存

```nginx website.conf
# 缓存图片
location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ { 
  access_log   off; 
  expires      30d; #根据自己需要修改时间
}

#缓存js、css、视频文件
location ~* ^.+\.(css|js|txt|xml|swf|wav)$ {
  access_log   off;
  expires      24h;
}

# 缓存html类型文件
location ~* ^.+\.(html|htm)$ {
  expires      1h;
}

# 缓存字体文件，配合gzip更好
location ~* ^.+\.(eot|ttf|otf|woff|svg)$ {
  access_log   off;
  expires 30d;
}
```

