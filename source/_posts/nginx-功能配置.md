---
title: "Nginx 功能配置"
date: 2026-05-30 20:34:05
permalink: "web服务/nginx-功能配置/"
updated: 2026-05-30 20:34:05
categories:
  - "web服务"
tags:
  - "nginx"
  - "配置"
description: "访问控制 allow / deny 和 auth_basic Nginx 可以按 IP 做访问控制： location /a/ { allow 127.0.0.1; allow 117.51.154."
cover: /img/covers/cover-050.jpg
top_img: false
---

## 访问控制 allow / deny 和 auth\_basic

Nginx 可以按 IP 做访问控制：

```text
location /a/ {
    allow 127.0.0.1;
    allow 117.51.154.71;
    allow 192.168.31.0/24;
    deny all;
}
```

顺序是从上往下判断。上面这段意思是：允许本机、指定公网 IP、指定网段访问，其他全部拒绝。

如果不是按 IP，而是要求输入用户名密码，就用 Basic Auth：

```text
location /b/ {
    auth_basic "test";    test只是一个名字，可以任意
    auth_basic_user_file /opt/nginx/passwd/htpasswd;  密码文件路径
}
```

密码文件可以用 `htpasswd` 生成：

```text
yum install httpd-tools -y                     下载 httpasswd
mkdir -p /opt/nginx/passwd                     创建一个放密码文件的目录
htpasswd -c /opt/nginx/passwd/htpasswd user1   -c 创建一个密码文件并添加user1用户
htpasswd -m /opt/nginx/passwd/htpasswd user2   -m 添加用户，-c 会重新创建
```

**`-c` 是创建文件，已有文件后再加用户不要重复用 `-c`，否则会覆盖原文件**

## 防盗链

防盗链原理：根据 HTTP 请求头里的 `Referer` 判断这个资源是不是从允许的页面引用过来的

典型配置：

```text
location ~* \.(png)$ {
    valid_referers none blocked 192.168.31.20;

    if ($invalid_referer) {
        return 403;
    }
}
```

`valid_referers` 会根据 Referer 头设置 `$invalid_referer`

```text
none          允许没有 Referer
blocked       允许 Referer 被代理或防火墙隐藏的情况
server_names  允许 server_name 中的主机名
```

如果 Referer 不符合规则，`$invalid_referer` 就是 1，于是返回 403

这个功能适合减少图片、音视频等资源被别人页面直接引用。它不是强安全手段，因为 Referer 可以伪造，但对普通盗链和爬虫有一定效果

## 虚拟主机

虚拟主机的本质是：一台真实服务器上跑多个逻辑网站

Nginx 的匹配逻辑可以简化为：

```text
先看连接进入哪个 listen IP:port
再看 Host 匹配哪个 server_name
匹配不到就走 default_server 或当前 listen 组第一个 server
```

所以可以基于三种方式区分：

```text
IP
端口
域名
```

### 基于 IP

需要本机有多个 IP

配置：

```text
server {
    listen 192.168.0.129:80;
    location / {
       root /opt/nginx/html/web01;
       index index.html;
    }
}

server {
    listen 192.168.0.130:80;
    location / {
       root /opt/nginx/html/web02;
       index index.html;
    }
}
```

访问不同 IP，进入不同目录，这种方式成本高，因为真实公网环境需要多个 IP，所以企业里不常作为首选

### 基于端口

```text
server {
    listen 80;
    location / {
        root /opt/nginx/html/web01;
        index index.html;
    }
}

server {
    listen 8080;
    location / {
        root /opt/nginx/html/web02;
       index index.html;
    }
}
```

访问：

```text
http://192.168.0.129/
http://192.168.0.129:8080/
```

端口型虚拟主机更适合内部服务，公网用户一般不喜欢记端口

### 基于域名

```text
server {
    listen 80;
    server_name www.abc.com;

    location /web1 {
       root html;
        index index.html;
    }

server {
    listen 80;
    server_name www.aaa.com;

    location /web2 {
        root html;
        index index.html;
    }
}
```

域名型最符合真实业务使用方式，测试时要配 hosts 或 DNS，让域名解析到 Nginx 服务器

## 反向代理

正向代理代理的是客户端

客户端必须知道代理服务器地址，并主动配置。VPN、公司出口代理、某些软件代理都属于这个方向，VPN 还通常带有加密隧道，不只是简单 HTTP 代理

反向代理代理的是服务端

用户不知道后端是谁，只知道访问 Nginx ，Nginx 再把请求转发到内部服务器

```text
server {
    location / {
        proxy_pass http://192.168.31.23:80;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

如果后端要记录真实客户端 IP，需要日志格式配合：

```text
log_format main '$http_x_real_ip|$http_x_forwarded_for - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';
```

否则后端看到的来源只是前端 Nginx

## 限速

Nginx 限速涉及三个概念：

```text
limit_req   限制请求频率
limit_conn  限制并发连接
limit_rate  限制响应传输速度
```

使用 `limit_req` 和 `limit_conn` 需要提前创建区域，而 `limit_rate` 不需要

`limit_req` 基于漏桶算法。请求像水一样进入桶，Nginx 按固定速度处理，超过容量就丢弃

```text
limit_req_zone $binary_remote_addr zone=myzone:10m rate=1r/s;
                 
location / {
    limit_req zone=myzone burst=5 nodelay;
}

创建区域，根据二进制ip分类，区域名 myzone 10m空间 每秒一次请求

使用 myzone 区域
允许突发 5 个
nodelay 表示超过突发能力后直接返回 503
```

下载限制通常用：

```text
limit_conn_zone $binary_remote_addr zone=down:10m;

location /download/ {
   limit_conn down 1;
   limit_rate 10k;
}

limit_conn down 1 表示同一个 IP 同时只能有 1 个连接进入这个限制区域
limit_rate 10k 是每连接限速，不是每用户总限速。如果一个用户开多个连接，总速度会叠加，所以下载限速通常要和 limit_conn 一起配
```

如果把下载限制写在 `location /`，下载和网页都会抢同一个连接额度，所以下载资源要放到单独目录进行限速

## URL 重写 与 rewrite 模块

Nginx 的 rewrite 基于 PCRE 正则，用于 URL 重写、跳转和内部 URI 改写

```text
语法：
rewrite 正则 替换内容 flag;
```

常见 flag：

```text
permanent  301 永久重定向，浏览器地址栏变化
redirect   302 临时重定向，浏览器地址栏变化
last       停止当前 rewrite 指令集，用新 URI 重新匹配 location
break      停止当前 rewrite 指令集，不重新匹配 location
```

如果替换内容以 `http://`、`https://` 或 `$scheme` 开头，也会变成外部跳转，默认临时重定向

如果不写 flag，容易按默认行为继续处理，实际效果接近继续走 rewrite 流程

**目录一定要小心结尾 /**

如果 rewrite 到的是目录，最好写成如：

```text
rewrite passwd$ /dir/ last;
```

而不是：

```text
rewrite passwd$ /dir last;
```

因为 `/dir` 如果对应文件系统里的目录，Nginx 可能自动返回 301，把地址补成 `/dir/`。这时浏览器地址栏会变化，而 `break` 或 `last` 跳转的跳转并不会改变浏览器地址栏，同时因为是目录规范化触发了 301，后续 flag 并不会生效

### **rewrite 里的变量、if、return、break**

Nginx 变量都是字符串。可以用 `set` 定义：

```text
set $test 1111;
rewrite ^(.*)$ http://192.168.111.12/$test/ permanent;
```

变量作用范围和定义位置有关：在 `server` 里定义，该 server 内的 location 可以用；在 `location` 里定义，主要就在当前 location 中使用

`if` 可以根据变量判断：

```text
if ($http_user_agent ~* 'elinks') {
    return 404;
}
```

这表示 User-Agent 包含 elinks 就返回 404

`return` 也是 rewrite 模块的指令，它会直接结束请求

`break` 则不结束请求，只是停止后续 rewrite 模块指令：

```text
if ($http_user_agent ~* 'elinks') {
    break;
    return 404;
}
```

这段里 `break` 执行后，后面的 `return 404` 不会执行。请求并没有被拒绝，而是继续走当前 location 的普通内容处理，所以最后可能仍然返回正常页面

**break 只是不再执行 rewrite 指令，如 return set if 等，其余不影响**

**不要把 Nginx 配置理解成 Shell 脚本**
