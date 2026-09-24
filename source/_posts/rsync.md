---
title: "rsync"
date: 2026-05-18 17:38:56
permalink: "linux/rsync/"
updated: 2026-05-18 17:38:56
categories:
  - "linux"
tags:
  - "rsync"
  - "远程同步"
description: "rsync ，Linux 文件同步与增量备份工具，一般用于两台服务器远程同步文件 rsync具有以下特点： 只同步发生变化的文件 支持本地同步 支持远程同步 支持保留权限、属主、时间戳、软链接 支持排"
cover: /img/covers/cover-008.jpg
top_img: false
---

rsync ，Linux 文件同步与增量备份工具，一般用于两台服务器远程同步文件

rsync具有以下特点：

- 只同步发生变化的文件
- 支持本地同步
- 支持远程同步
- 支持保留权限、属主、时间戳、软链接
- 支持排除文件
- 支持删除目标端多余文件
- 支持断点续传和传输压缩

```text
安装
yum install rsync -y
远程同步时，本机和远程主机都需要安装 rsync

基本用法
rsync [选项] 源路径 目标路径

rsync -av /data/ /backup/data/                         本地同步
rsync -av /data/ root@192.168.174.127:/backup/data/    远程同步
rsync -av root@192.168.174.127:/backup/data/ /data/    远程拉取
注意：/data/ 等同 /data/* ，不包含 /data 本身这个目录，/data 则包含 /data 本身
```

常见参数

| 参数 | 含义 |
| --- | --- |
| `-a` | 归档模式，保留权限、属主、时间戳、软链接等 |
| `-v` | 显示详细过程 |
| `-z` | 传输时压缩，适合远程同步 |
| `-P` | 显示进度，并支持断点续传 |
| `--delete` | 删除目标端多余文件，使目标和源完全一致 |
| `--exclude` | 排除指定文件或目录 |
| `--exclude-from` | 从文件读取排除规则 |
| `-n` / `--dry-run` | 预演，不真正执行 |
| `--bwlimit` | 限制传输速度 |
| `-e` | 指定远程 shell，常用于指定 SSH 端口或私钥 |

由于 rsync 在传输多个小型文件时容易出现进程僵死的问题，推荐用 tar 打包后传输

在使用 --delete 参数时，先使用 -n 参数预演，否则容易出现误删文件的情况

rsync 是基于 ssh 的，同时默认使用 22 端口，如果需要使用其他端口、指定私钥等操作，需要用到 -e 参数，.ssh/config 文件也是生效的

```text
rsync -av /data/ -e "ssh -p 22222" root@192.168.174.127:/backup/data/
                          修改了ssh端口的情况

假设在家目录下的 .ssh/config 中配置
Host Clone
	Hostname 192.168.174.127
	Port 22022
	User root
	IdentityFile /root/.ssh/id_rsa
那么可以使用如下命令便捷推送
rsync -av /data/ Clone:/backup/data/
```
