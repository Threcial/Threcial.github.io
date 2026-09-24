---
title: "nginx 基本配置"
date: 2026-05-30 17:11:24
permalink: "web服务/nginx-基本配置/"
updated: 2026-05-30 17:11:24
categories:
  - "web服务"
tags:
  - "nginx"
  - "配置"
description: "Nginx 可以做 Web 服务器、反向代理服务器，也能承担邮件代理相关功能。它比较典型的特点是轻量、资源消耗低、并发能力强。 和 Apache 相比，Nginx 的优势主要来自事件驱动和异步非阻塞模"
cover: /img/covers/cover-051.jpg
top_img: false
---

Nginx 可以做 Web 服务器、反向代理服务器，也能承担邮件代理相关功能。它比较典型的特点是轻量、资源消耗低、并发能力强。

和 Apache 相比，Nginx 的优势主要来自事件驱动和异步非阻塞模型。Apache 更偏阻塞式处理，请求多了以后进程/线程资源压力会变大；Nginx 借助 epoll 模型，用较少的 worker 来处理大量连接。

Nginx 是典型的 master + worker 模型。master 负责读取配置、管理 worker ，worker 负责处理请求。

```text
通过yum安装的nginx有以下四个重要目录
/etc/nginx 					nginx所有的文件和配置都在这个目录
/usr/share/nginx/html				网站根目录
/var/log/nginx					nginx日志目录
/var/cache/nginx                                缓存目录
```

本文重点就在于 nginx 配置文件写法

## 结构

Nginx 配置不是平铺的，它按上下文划分，一个常见的配置文件 nginx.conf 如

```text
user  nginx;                            指定启动Nginx工作进程的用户
worker_processes  auto;    工作进程的数量，进程数，建议设置为auto，由nginx自己配置
           
error_log  logs/error.log  error;      错误日志及存放目录、日志级别
#pid        logs/nginx.pid;               nginx启动之后pid存放位置

events {
    worker_connections  1024;            每个进程的连接数
}

http {                                   http模块设置
   nginx支持类型
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;      访问日志 目录与格式

    sendfile        on;               零拷贝
    #tcp_nopush     on;               连接超时，时间 单位s
    #keepalive_timeout  0;
    keepalive_timeout  65;
    #gzip  on;                      	开启gzip压缩
    server {                             一个虚拟主机
        listen       80;                	监听端口
        server_name  localhost;                    域名
	#charset koi8-r;                            字符集
        #access_log  logs/host.access.log  main;    定义访问日志
        location / {                              默认请求
            root   html;                     定义虚拟主机默认网站的目录
            index  index.html index.htm;     定义首页索引文件的名称
            }
        #error_page  404              /404.html;    
        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }  
```

`events` 里配置连接能力

`http` 里是 HTTP 层配置，比如日志格式、gzip、sendfile、server、include 等

`server` 是虚拟主机

`location` 是 URI 匹配规则

## 日志

Nginx 有访问日志和错误日志

错误日志级别有：

```text
debug
info
notice
warn
error
crit
alert
emerg
```

级别越高，记录越少。生产里常用 `warn`、`error`、`crit`，不要轻易开 `info` 或 `debug`，否则磁盘 IO 很容易被日志拖住

访问日志通常通过 `log_format` 定义格式：

```text
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" "$http_x_forwarded_for"';

$remote_addr              客户端 IP
$http_x_forwarded_for     代理链路里的真实客户端信息
$remote_user              认证用户
$time_local               访问时间
$request                  请求行
$status                   状态码
$body_bytes_sent          响应体大小
$http_referer             来源页面
$http_user_agent          浏览器信息
```

## include

Nginx 主配置文件不应该无限膨胀，常见写法是在 `http` 块里加：

```text
include /etc/nginx/conf.d/*.conf;
```

这样 `/etc/nginx/conf.d/` 下所有 `.conf` 文件都会被包含进来

## location 块

nginx有两层指令来匹配请求 URI 。第一个层次是 server 指令，它通过域名、ip 和端口来做第一层级匹配，当找到匹配的 server 后就进入此 server 的 location 匹配。

location 的匹配并不完全按照其在配置文件中出现的顺序来匹配，请求URI 会按如下规则进行匹配：

1. 先精准匹配 **`=`** ，精准匹配成功则会立即停止其他类型匹配
2. 没有精准匹配成功时，进行带有 `^~` 的前缀匹配
3. **`=`** 和 **`^~`** 均未匹配成功前提下，查找正则匹配 **`~`** 和 **`~*`** 。当同时有多个正则匹配时，按其在配置文件中出现的先后顺序优先匹配，命中则立即停止其他类型匹配
4. 以上都未成功时进行普通前缀匹配，多个匹配成功时以最长前缀为准
5. 默认匹配规则进行兜底，即使配置文件里未有默认匹配 location 块，nginx 依旧会用默认规则处理

以上规则简单总结就是优先级从高到低依次为：

```text
1. location =    精准匹配
2. location ^~   带参前缀匹配
3. location ~  或 ~*  正则匹配
4. location /a   普通前缀匹配，优先级低于带参数前缀匹配
5. location /    默认规则，任何没有匹配成功的，都会匹配这里处理
```

**`root` 和 `alias` 的区别**

`root` 是把 URI 拼到 root 后面：

```text
location /img/ {
    root /data/www;
}
访问      /img/a.png
实际路径  /data/www/img/a.png
```

`alias` 是用指定路径替换 location 前缀：

```text
location /img/ {
    alias /data/images/;
}
访问      /img/a.png
实际路径  /data/images/a.png
```

所以 `alias` 更像“映射到另一个目录”，`root` 更像“以这个目录为根再拼 URI”
