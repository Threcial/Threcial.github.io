---
title: "linux进程与服务"
date: 2026-04-16 14:28:06
permalink: "linux/linux进程与服务/"
updated: 2026-04-16 14:28:06
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "服务管理命令 systeamctl systemctl start/restart/status/stop/disable/enable/is-enabled/reload 服务名 表示 开始/重启/"
cover: /img/covers/cover-025.jpg
top_img: false
---

## 服务管理命令 systeamctl

```markdown
systemctl start/restart/status/stop/disable/enable/is-enabled/reload 服务名
表示 开始/重启/状态/停止/关闭开机自启/开启开机自启/查看是否自启/重载配置
```

## 进程管理 ps 和 top

ps和top都可以查看进程，区别在于ps为静态查看，top为动态查看

```text
常用
ps aux
ps -ef

top可用快捷键给显示内容排序
P 按cpu使用率排序
M 内存使用率排序
```

### 杀死进程 kill

```text
用法
kill PID
kill -15 PID 向进程发送结束信号
kill -9 PID 强制杀死进程，可能有副作用
kill -1 PID 重载进程
```

## 端口检查

```text
ss 查看当前端口连接
ss -lntp
-a 所有连接
-l 只看监听
-n 数字显示
-t tcp连接
-p 显示进程
```
```text
lsof 查看端口占用
lsof -i 查看所有端口
lsof -i:80  80端口
lsof -i tcp tcp连接
lsof -i @1.1.1.1 ip根据连接ip查看
```
