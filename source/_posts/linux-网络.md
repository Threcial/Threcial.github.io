---
title: "linux-网络"
date: 2026-05-17 19:06:49
permalink: "linux/linux-网络/"
updated: 2026-05-18 19:06:49
categories:
  - "linux"
tags:
  - "linux"
  - "网络"
description: "网络配置 可通过 ip 和 ifconfig 来初步查看本机网络配置 yum install net-tools -y ifconfig 安装 ifconfig属于旧命令 推荐使用ip ip addr"
cover: /img/covers/cover-011.jpg
top_img: false
---

## 网络配置

可通过 ip 和 ifconfig 来初步查看本机网络配置

```text
yum install net-tools -y      ifconfig 安装
ifconfig属于旧命令 推荐使用ip
```
```text
ip addr                                          IP 地址
ip addr show dev ens33                           网卡ens33的 IP
ip addr add 192.168.174.128/24 dev ens33         给网卡添加 IP
ip addr del 192.168.174.128/24 dev ens33         给网卡删除 IP

ip link                                          网卡状态
ip link show dev ens33                           只看ens33
ip link set dev ens33 up                         
ip link set dev ens33 down                       启用和关闭网卡

ip route                                         路由表
ip route add default via 192.168.174.2 dev ens33           添加默认路由
ip route add 10.10.10.0/24 via 192.168.174.1 dev ens33     添加静态路由
ip route del 10.10.10.0/24                                  删除
ip route get 8.8.8.8                                   查看去往8.8.8.8使用的路由

ip neigh                                          ARP 邻居表
```

以上都是临时配置，永久配置需要修改网络配置文件，位于 /etc/sysconfig/network-scripts/

其中 route-ens33 可以配置路由，ifcfg-ens33 可以配置 ip

## 网络状态

```text
ss   用于查看当前端口状况
-l  监听端口
-a  全部端口
-t  tcp
-u  udp
-n  不显示别名
-p  显示进程名
-i  网络接口
```

## 网络测试

```text
ping                    使用icmp包测试网络连通性 
-c 指定包个数 -s指定大小    
nslookup                域名解析查询
traceroute              查看数据包经过的路由器，现已被大部分路由器屏蔽
telnet        简单测试端口是否开放
```

## 下载

```text
常用命令为 curl 和 wget
curl 主要用于测试，wget 主要用于下载

wget
    -O 指定下载文件名 
    -c 断点重传 
    --limit-rate=10k 限速

curl 
    -I 只显示回应数据包头 
    -o 指定下载文件名
    -O 下载，文件名同访问名
    -L 跟随跳转 
    -v 过程
```
