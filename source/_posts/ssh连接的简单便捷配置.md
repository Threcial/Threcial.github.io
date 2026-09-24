---
title: "ssh连接的简单便捷配置"
date: 2026-04-04 11:06:28
permalink: "linux/ssh连接的简单便捷配置/"
updated: 2026-04-04 11:06:28
categories:
  - "linux"
tags:
  - "ssh"
  - "配置"
description: "主要记录ssh密钥连接的一些配置 ssh支持密钥连接而无需密码。客户端上通过ssh-keygen命令即可生成一个密钥对，私钥保留本地公钥上传服务端。以本地windows服务端linux为例，私钥在保证"
cover: /img/covers/cover-037.png
top_img: false
---

主要记录ssh密钥连接的一些配置

ssh支持密钥连接而无需密码。客户端上通过`ssh-keygen`命令即可生成一个密钥对，私钥保留本地公钥上传服务端。以本地windows服务端linux为例，私钥在保证不泄露的情况下可以存放在任一位置，公钥需要放在linux端的用户目录的.ssh目录下，命名为authorized\_keys。

在服务端中可以通过修改/etc/ssh/sshd\_config文件来修改配置，包括但不限于允许root用户通过ssh登录、禁止密码登录、ssh的登录端口。

通过在客户端的.ssh目录下添加config文件可以更方便的使用ssh功能，以下为例

```text
Host 任意名字
    HostName 服务端ip
    User 用户名
    Port 服务端的sshd端口
    IdentityFile 存放私钥的路径
```

配置这样一个config文件可以实现仅用`ssh 任意名字`就能登录服务端
