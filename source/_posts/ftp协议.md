---
title: "FTP协议"
date: 2026-04-28 15:48:31
permalink: "net/ftp协议/"
updated: 2026-04-28 15:48:31
categories:
  - "计算机网络"
tags:
  - "ftp"
description: "ftp协议属于应用层协议，使用传输层TCP协议，端口号为20（数据传输）和21（控制），采用C/S架构 ftp针对不同文件有不同的传输模式 bin（二进制模式） 用于传输exe、图片等 ascii模式"
cover: /img/covers/cover-017.png
top_img: false
---

ftp协议属于应用层协议，使用传输层TCP协议，端口号为20（数据传输）和21（控制），采用C/S架构

ftp针对不同文件有不同的传输模式

- **bin（二进制模式）**  
    
  用于传输exe、图片等
- **ascii模式**  
    
  传输文本、日志文件等

ftp协议工作模式分为主动模式和被动模式

- **主动模式**  
    
  客户端连接到ftp服务器的21端口进行身份验证，服务端主动开启20端口并向客户端主动建立连接
- **被动模式**  
    
  被动模式下同样在21端口进行身份验证，但不再主动开启20端口，而是开放一个高于1024的随机端口号，通知客户端连接到此端口进行数据传输

### 网络设备中配置ftp服务

```markdown
ftp server enable

aaa视图中将service-type修改并额外添加一项即可
service-type ftp
ftp-directory flash：

ftp常用命令
get 下载
put 上传
```
