---
title: "linux网络初探"
date: 2026-04-16 11:31:41
permalink: "linux/linux网络初探/"
updated: 2026-04-16 11:31:41
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "wget和curl wget主要用于下载文件，默认下载到当前目录 wget https://xxx.com/xxx.tar.gz wget -O xxx.tar.gz https://xxx.com/"
cover: /img/covers/cover-027.jpg
top_img: false
---

## wget和curl

wget主要用于下载文件，默认下载到当前目录

```text
wget https://xxx.com/xxx.tar.gz
wget -O xxx.tar.gz https://xxx.com/xxx.tar.gz 指定下载文件名
```

curl可用于测试网络

```text
curl https://xxx.com 可快速测试网站不下载
curl -o https://xxx.com 下载
-O 自定义文件名
-v 查看连接过程
```

## 网络配置文件

通过ifconfig可以查看当前ip和网卡之类信息，如

```text
[root@centos7 script]# ifconfig
ens33: flags=4163  mtu 1500
        inet 192.168.174.128  netmask 255.255.255.0  broadcast 192.168.174.255
        inet6 fe80::379d:8222:b905:dbee  prefixlen 64  scopeid 0x20
        ether 00:0c:29:3f:8d:f7  txqueuelen 1000  (Ethernet)
        RX packets 106225  bytes 90538396 (86.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 33186  bytes 16519144 (15.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...............
```

网络配置文件则在于（当前环境centos7）文件/etc/sysconfig/network-scripts/ifcfg-ens33中，不同网卡对应不同文件，如网卡eth0则配置则在同目录下的ifcfg-eth0文件中

```text
[root@centos7 script]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE=Ethernet    网络类型
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp   静态ip动态ip
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33  网卡描述
UUID=642c1613-893d-4c53-b6bf-75e7715faa4d  网卡id
DEVICE=ens33  网卡名称
ONBOOT=yes  开机自启
```

修改配置文件后需重启网络服务或相关网卡

```text
systemctl restart network
或
ifdown ens33
ifup ens33
```

### dns解析

/etc/hosts为dns解析配置文件，优先于dns服务器

```text
语法
IP    域名1 域名2 域名3

如下即可屏蔽www.google.com
127.0.0.1   www.google.com
```
