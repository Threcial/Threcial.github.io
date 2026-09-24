---
title: "Linux-firewalld"
date: 2026-05-16 21:41:13
permalink: "linux/linux-firewalld/"
updated: 2026-05-16 21:41:13
categories:
  - "linux"
tags:
  - "firewalld"
  - "linux"
description: "firewalld 是 centos7 及以后版本所使用的防火墙管理服务 zone firewalld 与传统防火墙 iptables 最大的区别就是 firewalld 引入了 zone 的概念。一"
cover: /img/covers/cover-047.png
top_img: false
---

firewalld 是 centos7 及以后版本所使用的防火墙管理服务

## zone

firewalld 与传统防火墙 iptables 最大的区别就是 firewalld 引入了 zone 的概念。一个 zone 可以理解成一组防火墙策略，多个 zone 就有多组不同的策略应对不同环境，默认有以下九种 zone

| zone | 作用 |
| --- | --- |
| `drop` | 默认丢弃所有进入本机的流量，不回复 |
| `block` | 默认拒绝进入本机的流量，会回复拒绝信息 |
| `public` | 公共网络区域，默认较严格，常作为默认区域 |
| `external` | 外部网络区域，常用于 NAT 场景 |
| `dmz` | 隔离区，适合暴露少量服务 |
| `work` | 工作网络区域 |
| `home` | 家庭网络区域 |
| `internal` | 内部网络区域 |
| `trusted` | 完全信任区域，默认接受所有连接 |

一般默认使用 public zone ，public zone 开放 ssh 22 端口，一个 dhcp-client 端口，以及接受 ping

## 基本命令

```markdown
常见命令示例

firewall-cmd --list-all                 查看当前区域配置
             --list-all-zones           查看所有区域配置

firewall-cmd --get-default-zone         查看默认区域            
             --get-zones                查看可用区域
             --get-active-zones         查看当前正在使用的区域
             --set-default-zone=drop    设置drop区域为默认区域

firewall-cmd --get-zone-of-interface    查看网卡所在的区域
             --change-interface=ens33 --zone=drop
             --add-interface=ens33 --zone=public 
                                        网卡默认在public，用change即可修改网卡区域

firewall-cmd --add-port=80/tcp          
             --remove-port              默认区域开放或关闭端口
          
firewall-cmd --add-protocol=icmp        
             --remove-protocol          允许或禁止某个ip协议

firewall-cmd --get-services             显示可用的服务
             --add-service=http
             --remove-service=http      允许和禁止服务

firewall-cmd --add-source=192.168.10.0/24 --zone=trusted
             --remove-source=           将指定来源ip的流量交给指定区域处理

firewall-cmd --add-rich-rule=                 
             --remove-rich-rule=        富规则，下文详解
        
             --reload                   重载配置
             --runtime-to-permanent     runtime配置转为permanent配置
             --permanent                当前命令配置写入永久配置

firewall-cmd --panic-on                 应急状况模式，断开所有连接，只能本地登录系统
             --panic-off                现在云主机非常常见，不推荐随便使用此命令
```

## runtime 与 permanent

firewalld 有两个配置层

- **runtime** 当前运行配置，立即生效，但重启后失效
- **permanent** 永久配置，重启后仍然存在，但当前不生效，reload后生效

因此可以在不加 permanent 参数的情况下测试 firewalld 配置，无误后使用 --runtime-to-permanent 来写入永久配置

## service、port、protocol

- **service** 按服务名开放  
    
  service 本质上是 firewalld 预定义好的服务模板。比如 http 对应 80/tcp，https 对应 443/tcp
- **port** 按端口开放
- **protocol** 按 IP 协议开放

## 流量处理逻辑

firewalld 处理一个进入本机的数据包时，大致可以理解为：

- 先看来源 IP 是否匹配 source，如果匹配 source，则进入对应 zone
- 如果没有匹配 source，则看通信的网卡归属的 zone，由该 zone 规则处理

但是在 CentOS 7 的 firewalld/iptables 后端中，还要注意一个细节：source zone 先处理，不一定最终处理，流量可能经过多个区域规则处理

举个例子：ens33是默认网卡绑定 drop 区域，drop 区域开放 http，192.168.10.1 作为 source 绑定 public 区域，public 区域没有 http 以及其他额外配置，请问此时 192.168.10.1 能够访问 http 服务吗

答案是能。因为 public zone 的 target 是 default，并且 public 区域中没有命中明确规则，这时来自 192.168.10.1 的 HTTP 流量先进入 public，但 public 没有最终处理它，所以它又进入 ens33 所在的 drop 区域，而 drop 开放了 http，最终访问 HTTP

其中关键是 public zone 的 target ，这个参数表示流量未命中区域中的规则时该如何处理，default 意味着并不会拒绝流量，而是交于网卡所在的 zone 处理

如果将 public zone 的 target 修改成 DROP，那么这个流量就能被 pubic zone 最终处理（直接丢弃），而不会经过 drop zone。

## rich rules

富规则，用来写更精确的规则。比如

```text
firewall-cmd --zone=drop --add-rich-rule='rule family="ipv4" source address="192.168.0.81" port port="22" protocol="tcp" accept'
    drop 区域，放行192.168.0.81对22端口的tcp流量 

firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.0.81" port port="22" protocol="tcp" reject'
    public 区域，拒绝192.168.0.81对22端口的tcp流量

firewall-cmd --zone=drop --add-rich-rule="rule family="ipv4" source address="192.168.31.77" protocol value="icmp" accept"
    drop 区域，放行 192.168.31.77 的icmp协议流量

remove 时需要原样复制添加时候的命令

accept drop reject 含义；drop不回应直接丢弃，reject明确拒接，accept接受
```
