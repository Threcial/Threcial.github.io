---
title: "复杂文本处理-grep sed awk"
date: 2026-05-24 18:58:32
permalink: "linux/复杂文本处理-grep-sed-awk/"
updated: 2026-05-24 18:58:32
categories:
  - "linux"
tags:
  - "文本处理"
description: "grep：找出符合条件的行，核心思路是：哪些行符合条件 sed：对行内容进行增删改查，核心思路是：按规则修改某些行 awk：按列处理文本，核心思路是：按字段、按列提取或计算数据，适合处理有规律的字段 "
cover: /img/covers/cover-006.jpg
top_img: false
---

**grep：找出符合条件的行**，核心思路是：哪些行符合条件

**sed：对行内容进行增删改查**，核心思路是：按规则修改某些行

**awk：按列处理文本**，核心思路是：按字段、按列提取或计算数据，适合处理有规律的字段

## grep

```text
grep [选项] 文件
    -v 取反
    -i 不区分大小写
    -n 显示行号
    -w 只匹配单词，单词之间可以是任意分割符
    -o 只显示查找内容，与正则一起使用
    -E 使用扩展正则表达式
```

## sed

sed 的全名为 stream editor，流编辑器，适合编辑文件。sed 默认不会修改原文件，只会把处理结果输出到屏幕

```text
sed [选项] [sed内置命令字符] [文件]
  选项
    -n 取消命令默认输出，默认输出是对每一行处理后都输出	
    -i 修改文件，建议使用 -i.bak 修改时能够备份	    
    -e 多次编辑，或者使用 ; 来执行多个命令
    -E 使用扩展正则表达式
  内置命令字符			         
    s 替换			         
    g 全局			         
    p 打印
    d 删除
    a 增加,在指定行后面添加一行或者多行文本
    i 插入，在指定行前面添加一行或者多行文本
```

## awk

```text
awk '条件 {动作}' 文件     注意：由于动作中有$符号，需要用单引号，否则会被shell解析
  -F 指定分隔符，默认空格
  动作
    $0 表示整行
    $1 表示第一列
    $2 表示第二列
    $NF 表示最后一列
    NR 表示当前行号

awk '{print $1}' linux.txt     打印第一列
awk '{print $NF}' linux.txt    打印最后一列

awk "NR==3" file                打印第三行
awk -F ":" 'NR==2{print $3}' /etc/passwd      打印第二行第3列

awk -F ":" '/shutdown/{print $NF}' /etc/passwd   包含shutdown的行的最后一列
awk -F ":" '$1~/root/{print $NF}' /etc/passwd    第一列有root的行的最后一列
awk -F ":" '$1!~/root/{print $NF}' /etc/passwd   !取反 ~表示正则
```
