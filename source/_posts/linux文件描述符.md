---
title: "linux文件描述符"
date: 2026-04-14 10:56:22
permalink: "linux/linux文件描述符/"
updated: 2026-04-14 10:56:22
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "文件描述符（File Descriptor, FD）是系统给打开的文件的编号 编号名称作用0stdin标准输入1stdout标准输出2stderr标准错误 常见用法 输出重定向 [root@cento"
cover: /img/covers/cover-033.jpg
top_img: false
---

文件描述符（File Descriptor, FD）是系统给打开的文件的编号

| 编号 | 名称 | 作用 |
| --- | --- | --- |
| 0 | stdin | 标准输入 |
| 1 | stdout | 标准输出 |
| 2 | stderr | 标准错误 |

常见用法

```markdown
输出重定向
[root@centos7 ~]# ls 1>file1
[root@centos7 ~]# cat file1
anaconda-ks.cfg
create-certdb.sh
file1
file2

输入重定向
[root@centos7 ~]# ll file1 2>&1
[root@centos7 ~]# cat file1
ls: 无法访问file5: 没有那个文件或目录
./:
anaconda-ks.cfg
create-certdb.sh
file1
file2

手动创建fd（临时，关闭shell失效）
[root@centos7 ~]# exec 3>>testFile
[root@centos7 ~]# ls -l /proc/$$/fd
总用量 0
lrwx------. 1 root root 64 4月  14 09:24 0 -> /dev/pts/0
lrwx------. 1 root root 64 4月  14 09:24 1 -> /dev/pts/0
lrwx------. 1 root root 64 4月  14 09:24 2 -> /dev/pts/0
lrwx------. 1 root root 64 4月  14 09:38 255 -> /dev/pts/0
l-wx------. 1 root root 64 4月  14 09:24 3 -> /root/testFile
注意，此时是用追加模式创建的，所有重定向到&3的都将以追加模式写入testFile
```
