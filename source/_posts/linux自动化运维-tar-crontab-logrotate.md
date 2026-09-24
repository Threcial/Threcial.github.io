---
title: "linux自动化运维-tar crontab logrotate"
date: 2026-05-17 11:58:57
permalink: "linux/linux自动化运维-tar-crontab-logrotate/"
updated: 2026-05-17 11:58:57
categories:
  - "linux"
tags:
  - "定时任务"
  - "打包"
  - "日志"
  - "自动化运维"
description: "在 Linux 运维中，tar、crontab、logrotate 是三个非常基础但非常重要的工具。它们分别解决三个问题：tar 负责把文件打包、压缩、备份，crontab 负责定时执行任务，logr"
cover: /img/covers/cover-009.jpg
top_img: false
---

在 Linux 运维中，tar、crontab、logrotate 是三个非常基础但非常重要的工具。它们分别解决三个问题：tar 负责把文件打包、压缩、备份，crontab 负责定时执行任务，logrotate 负责自动切割日志，防止日志无限增长，这三者经常组合使用。例如：每天凌晨自动打包网站目录、定期备份配置文件、定期清理或切割日志

## tar

不同于 windows 下的打包压缩工具，在 linux 中，需要把打包和压缩分开看，打包就是把多个文件或目录合并成一个文件，压缩则是通过算法减少文件体积

tar 本身主要负责打包，配合其他压缩算法才能做到打包压缩，比如常见的 .tar.gz 文件，就是经过 tar 打包后再用 gzip 进行压缩

```text
tar -cvf data.tar ./data

-c  create    创建归档包
-v  verbose   显示打包过程
-f  file      指定归档文件名
./data        要打包的目录

使用 -z 参数即可使用 gzip 进行压缩
tar -zcvf data.tar.gz ./data

tar -tf data.tar.gz  查看压缩包内容
tar -zxf data.tar.gz -C /root/   指定解压目录
```

### gzip

gzip 主要用于压缩单个文件，无法压缩目录

```text
gzip file                 压缩文件，默认删除源文件
gzip -r /root/data         递归压缩
gzip -v file              压缩并显示过程
gzip -c file > file.gz    压缩并保留源文件

gzip -d file.gz           解压，默认删除 .gz 文件
gzip -dc file.gz > file   解压，保留 .gz 文件

gzip -t file.gz
gzip -tv file.gz           测试压缩包

gzip -1c file > file-1.gz
gzip -9c file > file-9.gz   指定压缩级别，默认为6

zless file.gz            查看压缩文件

zgrep 'keyword' file.gz
zgrep -n 'keyword' file.gz    搜索压缩文件内容，-n显示行号
```

### 排除文件、指定文件

备份时如果有不希望被打包压缩的文件，可以使用 --exclude

```text
tar -zcf site.tar.gz --exclude=*.log ./site     排除.log结尾的文件
tar -zcf backup.tar.gz --exclude-from=exclude.list ./site   根据文本文件内容排除
```

**注意：打包目标用相对路径，exclude 也用相对路径，打包目标用绝对路径，exclude 也用绝对路径**

一个好的习惯可以解决很多意想不到的问题，详情可见 [tar –exclude](https://threcial.cn/linux/tar-exclude/)

有排除也有指定

```text
-T 从指定文件中获取文件名来解压或创建文件
tar -zcf backup.tar.gz -T include.list  打包压缩 include.list 中的文件
```

### 与 find 联动

```text
打包当前目录下大于 10KB 的文件
find . -type f -size +10k | xargs tar -zcf bigfiles.tar.gz 
```

## crontab

定时任务三个概念：

- **cron** 定时任务体系
- **crond** 定时任务服务进程
- **crontab** 管理定时任务的命令

作为一个服务，可以使用 systemctl 来查看，crontab 用于配置

```text
crontab -e                       编辑定时任务
crontab -l                       查看定时任务
crontab -u username -l           查看指定用户的定时任务

在 /var/spool/cron 中存放着每个用户的定时任务文件，命名为用户名
```

### cron 格式

```text
* * * * * command

前五个*号分别表示：
分    0-59
时    0-23
日    1-31
月    1-12
周    0-7，0 和 7 通常都表示周日

特殊表示方式
*      任意时间
-      时间范围
,      多个指定时间
*/n    每隔 n 个单位

如每天 8 点到 15 点整点执行command：
0 8-15 * * * command
```

cron 由于环境变量问题，不推荐直接在定时任务文件里执行命令，而是写好脚本，定时执行脚本

同时这些输出应该被重定向到其他文件中

## logrotate

无论是备份还是日志，都会有一个问题，服务器运行时间越长，日志文件就越大，针对这个问题，就需要用到 logrotate 日志切割

/etc/logrotate.conf 为 logrotate 主配置文件，/etc/logrotate.d/ 是子配置目录，推荐将配置放入子配置目录中，便于管理

```text
配置内容格式为：
日志文件路径 {
    配置项
}

例如
/var/log/myapp/*.log {
    daily                        每日检查，weekly每周、monthly每月
    rotate 7                      保留七份日志 
    size  5k                      切割大小为5k
    compress                      压缩
    dateext                       以日期时间命名日志切割文件
    create 0644 root root          创建的新日志权限与所属
}
```

系统通过系统级每日任务来调用 logrotate，*cat /etc/cron.daily/logrotate* 可以查看

如果想要测试新写入的配置使用如下命令

```text
logrotate -d /etc/logrotate.d/刚写入的配置文件   测试
logrotate -f /etc/logrotate.d/刚写入的配置文件   强制切割，也就是立马执行一次
logrotate -vf /etc/logrotate.d/刚写入的配置文件   -v 显示过程
```
