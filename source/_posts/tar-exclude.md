---
title: "tar –exclude"
date: 2026-05-11 15:25:55
permalink: "linux/tar-exclude/"
updated: 2026-05-11 15:25:55
categories:
  - "linux"
tags:
  - "bash"
  - "tar"
  - "打包压缩"
description: "--exclude 可以排除不想被打包的文件，但是匹配方法有些特别 假设在alice家目录下有 1.txt 和 logs/1.log tar -zcvf 1.tar.gz --exclude=logs"
cover: /img/covers/cover-013.jpg
top_img: false
---

--exclude 可以排除不想被打包的文件，但是匹配方法有些特别

```text
假设在alice家目录下有 1.txt 和 logs/1.log

tar -zcvf 1.tar.gz --exclude=logs/1.log --exclude=1.txt *
tar -zcvf 1.tar.gz --exclude=logs/1.log --exclude=./1.txt *
tar -zcvf 1.tar.gz --exclude=logs/1.log --exclude=./1.txt ./*
tar -zcvf 1.tar.gz --exclude=logs/1.log --exclude=./1.txt /home/alice
```

以上四种命令会得到不同的结果，究其原因是 exclude 的匹配机制和 tar、shell 的展开机制导致的。通过 `-v` 可以发现，不同的打包目录写法会输出不同的打包过程，而 exclude 的匹配类似于 grep 的匹配，将会在展开的行中去除所有匹配的行

第一条命令可以正常匹配两项，都排除

第二条命令无法匹配到 `./1.txt`

第三条命令因为展开带了 `./` 前缀，`./1.txt` 自然匹配成功

第四条命令依旧无法匹配 `./1.txt`

可以发现无论何种形式展开，`log/1.log` 都能匹配，所以都能排除

### 其他

对于多个文件打包压缩建议使用 -T 指定读取文件，-exec {} + 可能因为参数数量限制导致压缩包覆盖。对于排除 --exclude--from 也可以使用指定文件内容来排除
