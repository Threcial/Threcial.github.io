---
title: "SSH连接网络设备"
date: 2026-04-23 15:48:20
permalink: "net/ssh连接网络设备/"
updated: 2026-04-23 15:48:20
categories:
  - "计算机网络"
tags:
  - "ssh"
  - "数通"
  - "网络设备"
description: "telnet连接方式缺少安全认证，数据传输过程也采用明文传输，存在极大安全隐患，且单纯telnet服务容易招致Dos（Denial of Service）、主机ip地址欺骗、路由欺骗等恶意攻击。 因此"
cover: /img/covers/cover-020.jpg
top_img: false
---

telnet连接方式缺少安全认证，数据传输过程也采用明文传输，存在极大安全隐患，且单纯telnet服务容易招致Dos（Denial of Service）、主机ip地址欺骗、路由欺骗等恶意攻击。

因此ssh协议目前使用更加广泛，ssh是一个网络安全协议，通过对网络传输数据加密、安全登录等方式来构建一个安全的通道，ssh默认使用22端口，支持修改端口号

## SSH登录配置

### 被登录方配置

```markdown
stelnet server enable     开启ssh服务
rsa local-key-pair create     创建密钥，使用密钥加密

在aaa视图中的额外配置
local-user username service-type ssh 服务类型设为ssh

在vty视图中的额外配置
protocol inbound ssh     绑定ssh协议

ssh user username authentication-type all 支持ssh用户本地认证方式为全部
```

### 登录方配置

```markdown
ssh client first-time enable 首次登录开启客户登录功能
stelnet ip 在系统视图下远程连接
```
