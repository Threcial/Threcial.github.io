---
title: "linux文本处理"
date: 2026-04-14 14:41:35
permalink: "linux/linux文本处理/"
updated: 2026-04-15 14:41:35
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "sort 对文件内容进行排序 -n 按数字排序而非ascii码 -r 倒序 -t 指定分隔符 -k 指定列排序 uniq 去重，uniq只对相邻行去重，所以必须先排序 常用 sort | uniq -"
cover: /img/covers/cover-032.jpg
top_img: false
---

sort 对文件内容进行排序

```text
-n 按数字排序而非ascii码
-r 倒序
-t 指定分隔符
-k 指定列排序
```

uniq 去重，uniq只对相邻行去重，所以必须先排序

```text
常用 sort | uniq
-d 只显示重复行
-c 显示重复行和次数
-u 只显示不重复的行
```

grep 过滤符合条件的行

```text
语法：grep "关键词" 文件
-v 反选，不包含关键词的行
-i 忽略大小写
-n 显示行号
-c 只统计行数
-w 只过滤单词
-B n 输出包括符合条件行的前面n行
-A n 输出包括符合条件行的后面n行
-E 正则表达式
```

wc 统计命令

```text
-l 统计行数
-w 统计单词数
-c 统计字节数
```

sed 流式文本编辑器，在不打开文件的情况下处理内容

```text
sed 选项 '命令' 文件
这个'命令'通常为'地址'+'命令'的写法
-i 通过这个选项sed会修改文件，否则不修改文件内容
   特别用法 sed -i.bak "命令" file 会生成一份.bak结尾的未修改文件，常用于修改时备份
-n 取消默认输出
-r 扩展正则

1，2 地址-1到2行
1，2！地址-非1到2行
p 命令-输出指定行
d 命令-删除指定行

常见用法
sed -n '1,2p' file     输出前两行
sed 's#text1#text2#'     替换
sed '2d' file     删除第二行
sed -n '/text/p'     输出匹配text的行
```

awk 按列处理文本

```text
语法 awk '条件' 文件名
注意：awk后的命令最好用单引号，否则变量会被shell解析，用双引号时需转义$符号
NR 行数
NF 列数
-F 指定分隔符，参数加引号
常见用法
awk '$1=="Tom"' file     第一列是Tom的行
awk '$2>=10 && $2<=15' file     第二列大于等于10小于等于15的行
awk '{print $1}' file     输出第一列
awk '{print $NR}' file     输出最后一列
awk '{print sum+=$3}' file     输出第三列总和
awk -F ":" '$7=="/bin/bash"' file     用:分割，第七列为/bin/bash的行
awk '{print NR,$0}' file     添加行号输出全部
```
