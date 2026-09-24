---
title: "Nginx 性能优化配置"
date: 2026-05-31 13:15:04
permalink: "web服务/nginx-性能优化配置/"
updated: 2026-05-31 13:15:04
categories:
  - "web服务"
tags:
  - "nginx"
  - "性能优化"
  - "配置"
description: "Nginx 默认参数偏保守，适合最小硬件环境。真实业务里通常会调整 高并发优化：worker 和连接数 worker_processes auto; events { worker_connectio"
cover: /img/covers/cover-052.jpg
top_img: false
---

Nginx 默认参数偏保守，适合最小硬件环境。真实业务里通常会调整

## 高并发优化：worker 和连接数

```text
worker_processes auto;

events {
    worker_connections 4096;
}
```

`worker_processes` 通常按 CPU 核心数设置，直接 `auto` 就可以

`worker_connections` 是每个 worker 的最大连接数，可以从 4096 起步，并且习惯上取 2 的指数。如果 Nginx 是反向代理，一个客户端请求可能同时占用客户端连接和后端连接，所以理论连接数不能简单等于客户端数量

## 长连接优化：keepalive

HTTP 基于 TCP，每次短连接都要三次握手、四次挥手。keepalive 的作用就是复用连接

```text
keepalive_timeout 15;
keepalive_requests 8192;
```

默认是 65 秒，建议改到 15 秒左右，太长会让空闲连接长期占用 fd、worker\_connections 和内存

`keepalive_requests` 表示一个长连接最多处理多少个请求，达到后关闭

## 数据传输优化：sendfile 和 gzip

静态文件传输建议开启：

```text
sendfile on;
这是零拷贝方向的优化，可以减少用户态和内核态之间的数据拷贝
```

gzip 用于文本资源压缩：

```text
gzip on;
gzip_proxied any;
gzip_min_length 10k;
gzip_comp_level 6;
gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/javascript application/json;
```

压缩能节省带宽，让用户更快拿到文本内容。但不要压缩图片、视频、音频、压缩包，这些本身已经压缩过，再压缩收益很小，还消耗 CPU

隐藏版本号：

```text
server_tokens off;
```

## 客户端缓存

音视频、图片这类资源无法通过 gzip 得到明显收益，更适合让浏览器缓存：

```text
location ~* \.(png|gif|jpg|mp3|mp4|wav|avi)$ {
    expires 1h;        可以写30m 1h 1d 表示30分钟、一小时、一天
    root html;
}
```

缓存时间要按业务来，频繁变化的资源不要长缓存，带 hash 或版本号的静态资源可以缓存更久

## 负载均衡：Nginx 作为七层分发器

单台服务器总会遇到瓶颈，也会有单点故障，所以需要在前面放分发器，也就是反向代理的实际应用

常见方式有

### 轮询

```text
upstream web {
    server 192.168.0.120;
    server 192.168.0.121;
    server 192.168.0.122;
}

server {
    listen 80;

    location / {
        proxy_pass http://web;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

默认按请求顺序轮询转发

### 权重

```text
upstream web {
    server 192.168.0.120 weight=10;
    server 192.168.0.121 weight=3;
    server 192.168.0.122;
}
```

没有写 `weight` 时默认是 1，权重越高，分到的请求越多

### ip\_hash

```text
upstream web {
    ip_hash;
    server 192.168.0.120;
    server 192.168.0.121;
    server 192.168.0.122;
}
```

`ip_hash` 会基于客户端 IP 做哈希，让同一个用户尽量落到同一台后端。这个常用于保持 session 一致性

```text
后端状态有：
down    不参与处理请求
backup  其他机器忙或不可用时才参与
weight  权重
```

## Keepalived：解决分发器单点故障

后端业务服务器挂了，Nginx upstream 可以临时绕开，但如果前端分发器本身挂了，就需要高可用

这时就用到 Keepalived ， Keepalived 基于 VRRP 协议，实现 VIP 漂移

比如一个 keepalived 结构：

```text
Nginx-1  MASTER
Nginx-2  BACKUP
 VIP      192.168.31.33
```

用户访问 VIP，正常情况下 VIP 在 MASTER 上，MASTER 挂了，BACKUP 接管 VIP

Keepalived 默认通过 VRRP 多播发送通告。MASTER 定时告诉 BACKUP：我还活着。如果 BACKUP 一段时间收不到，就会参与选举并接管 VIP

云主机环境通常不支持这种二层 VIP 漂移，云厂商一般提供 SLB/负载均衡产品，所以云上直接用 SLB 更合适

MASTER 示例：

```text
global_defs {
    router_id wb01
}                      当前机器的唯一标识

vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_check.sh"
    interval 1
}                       定义一个脚本，每秒执行一次

vrrp_instance VI_1 {
    state MASTER                master
    interface ens33             本机网卡
    virtual_router_id 33        vip标识
    priority 150                优先级，数值越大越高
    advert_int 1           

    authentication {         
        auth_type PASS
        auth_pass 1111
    }                          验证方式

    track_script {
        chk_nginx              使用脚本
    }

    virtual_ipaddress {
        192.168.31.33/24 dev ens33 label ens33:3         
    }                            vip在本机的配置
}
```

BACKUP 示例：

```text
global_defs {
    router_id wb02
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 33        与 master 一致
    priority 100                 比 master 低
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1111
    }

    virtual_ipaddress {
        192.168.31.33/24 dev ens33 label ens33:3      虚拟ip配置
    }
}
```

以下四个配置必须一致

```text
virtual_router_id
authentication
virtual_ipaddress
advert_int
```

而以下三个一般不同

```text
router_id          唯一
priority        MASTER 通常高于 BACKUP
interface        真实网卡
```

因为 Keepalived 只能保证主机存活而不能与 Nginx 产生联动，所以需要在 Master 上添加一个脚本来时刻检测 Nginx 的状态，当 Nginx 出现问题时就移交 Vip

```text
#!/bin/bash
pidof nginx >/dev/null 2>&1
if [ $? -ne 0 ]; then
    sleep 1
fi
exit 0
```

### 裂脑问题

裂脑就是两台机器都认为自己是 MASTER，于是同时持有同一个 VIP，结果就是同一个 IP 在网络里对应两个 MAC，访问混乱

常见原因：

```text
心跳链路问题
网卡故障
交换机故障
VRRP 报文被拦截
配置不一致
```

缓解方式：

```text
增加冗余心跳链路
监控 VIP 是否同时存在
监控 BACKUP 是否在 MASTER 正常时错误持有 VIP
```

比如：在备节点上检查，如果能连通主节点 80 端口，并且备节点自己又持有 VIP，就报警，因为主节点 Web 服务正常时，备节点不应该接管 VIP。
