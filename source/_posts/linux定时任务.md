---
title: "linux定时任务"
date: 2026-04-15 16:02:59
permalink: "linux/linux定时任务/"
updated: 2026-04-15 16:02:59
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "时间同步 对集群服务器编写定时任务首先要将各个服务器时间同步 例如通过阿里云时间同步服务器来调整时间 [root@centos7 ~]# ntpdate time1.aliyun.com 15 Apr"
cover: /img/covers/cover-048.jpg
top_img: false
---

## 时间同步

对集群服务器编写定时任务首先要将各个服务器时间同步

```bash
例如通过阿里云时间同步服务器来调整时间
[root@centos7 ~]# ntpdate time1.aliyun.com
15 Apr 14:14:34 ntpdate[54709]: adjust time server 203.107.6.88 offset 0.006337 sec
```

## 定时任务

相关命令 crontab

```markdown
-e 创建当前用户定时任务
相关语法在/etc/crontab中

-l 查看定时任务
-r 删除所有定时任务

注意，创建定时任务本质是在一个类似任务列表得文件中添加任务，可以添加多个
```

### 日志处理

logrotate

在/etc/logrotate.conf文件中可以配置日志管理，同时有/etc/logrotate.d目录，在这里放配置文件更容易管理

配置文件写法为

```text
/var/log/mylog{
    daily 每天检查，weekly 每周，monthly 每月
    rotate 5 日志保留数
    size 10M 切割时的大小
    create 644 root root 创建新日志 权限：用户：组
    compress 压缩日志
    ...其他配置等
}
```
```text
logrotate -f 强制执行轮转，可以用于测试
```
