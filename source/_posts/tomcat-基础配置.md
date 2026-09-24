---
title: "tomcat 基础配置"
date: 2026-05-31 17:13:06
permalink: "web服务/tomcat-基础配置/"
updated: 2026-05-31 17:13:06
categories:
  - "web服务"
tags:
  - "tomcat"
description: "Tomcat 是 Java 写的，运行离不开 JDK/JRE，需要有 JAVA 环境 在 catalina.sh 和 setclasspath.sh 中添加 JAVA_HOME=java环境目录 To"
cover: /img/covers/cover-054.jpg
top_img: false
---

Tomcat 是 Java 写的，运行离不开 JDK/JRE，需要有 JAVA 环境

在 `catalina.sh` 和 `setclasspath.sh` 中添加

```text
JAVA_HOME=java环境目录
```

Tomcat 解压后常见目录：

```text
bin       启动、关闭、核心脚本
conf      配置文件
lib       Tomcat 和 Web 应用依赖的公共库
logs      日志
webapps   应用发布目录
temp      临时文件
work      JSP 编译后的工作目录
```

`bin` 里常见：

```text
startup.sh    启动脚本
shutdown.sh   关闭脚本
catalina.sh   核心控制脚本
```

`conf` 里重点是：

```text
server.xml          端口、Connector、Host 等核心配置
web.xml             默认 Web 应用配置
tomcat-users.xml    管理用户和角色
```
