---
title: "Nginx的一些初步配置"
date: 2026-04-17 14:21:56
permalink: "web服务/nginx的一些初步配置/"
updated: 2026-04-17 14:21:56
categories:
  - "web服务"
tags:
  - "nginx"
  - "web中间件"
description: "前情提要：在centos7上通过源码编译安装了nginx并配置了systemctl管理工具 为了让nginx能够正常使用，通常需要创建一个用户如 useradd nginx -s /sbin/nolo"
cover: /img/covers/cover-024.jpg
top_img: false
---

前情提要：在centos7上通过源码编译安装了nginx并配置了systemctl管理工具

为了让nginx能够正常使用，通常需要创建一个用户如

```text
useradd nginx -s /sbin/nologin -M 无登录无家目录
chown -R nginx:nginx /usr/local/nginx 让nginx用户拥有程序文件的权限
```

为了方便使用

```text
ln -s /usr/local/nginx/sbin/nginx /sbin/nginx 创建软链接以便命令行直接使用nginx命令

ngnix -t 检查nginx配置文件语法
nginx -T 查看所有配置内容
nginx -s reload 重载
nginx -s stop 停止
nginx start 启动
```

通过启动nginx就可以访问本地网站了

## conf文件中的server块

```text
    server {
        listen       80;
        server_name  localhost;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
```

server块定义了端口、域名、根目录、网站主页

一个conf文件中可以有多个server块，nginx接收到访问请求后就会对比端口号和域名，提供对应的根目录和网站主页

在一个conf文件中配置过于臃肿，难以管理，可以在conf文件中添加如下配置，也就是添加额外配置目录

```text
include /usr/local/nginx/conf/nginx.conf.d/*.conf;
```
