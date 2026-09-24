---
title: "linux源码编译安装"
date: 2026-04-17 13:55:33
permalink: "linux/linux源码编译安装/"
updated: 2026-04-17 13:55:33
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "以centos7安装nginx为例， yum install -y gcc gcc-c++ make pcre pcre-devel zlib zlib-devel openssl openssl-d"
cover: /img/covers/cover-023.jpg
top_img: false
---

以centos7安装nginx为例，

```markdown
yum install -y gcc gcc-c++ make pcre pcre-devel zlib zlib-devel openssl openssl-devel 本地编译安装首先要安装相关的编译环境和依赖

wget http://nginx.org/download/nginx-1.30.0.tar.gz
从nginx官方网站下载源码包

tar -zxf nginx-1.30.0.tar.gz 
解压

./configure --prefix=/usr/local/nginx --with-http_ssl_module
进入解压的目录下配置编译相关，所有源码包都是通用的./configure 具体参数--help

make && make install
编译 && 安装

在配置的安装目录下即可使用相关服务，但无法使用systemctl管理

如果要使用systemctl对安装的服务进行管理，可以通过创建/etc/systemd/system/youservicename.service来完成
内容如下：
[root@centos7 system]# cat nginx.service
[Unit]
Description=nginx #服务名称
After=network.target #在network服务之后启动

[Service]
Type=forking #后台
PIDFile=/usr/local/nginx/logs/nginx.pid #pid
ExecStart=/usr/local/nginx/sbin/nginx #start命令执行
ExecReload=/usr/local/nginx/sbin/nginx -s reload #reload命令
ExecStop=/usr/local/nginx/sbin/nginx -s quit #stop命令

[Install]
WantedBy=multi-user.target

systemctl daemon-reload 创建配置文件后需要重载才能生效
```
