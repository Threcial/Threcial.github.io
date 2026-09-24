---
title: "简单文本处理-sort uniq wc"
date: 2026-05-20 09:28:55
permalink: "linux/简单文本处理-sort-uniq-wc/"
updated: 2026-05-20 09:28:55
categories:
  - "linux"
tags:
  - "工具"
  - "文本处理"
description: "这三个工具作用分别是：sort 排序；uniq 去重和统计重复行；wc 统计行数、单词数、字节数 sort sort能够对文本进行简单排序，默认是根据 ascii 码排序 -t 指定分隔符 -k 指定"
cover: /img/covers/cover-059.png
top_img: false
---

这三个工具作用分别是：sort 排序；uniq 去重和统计重复行；wc 统计行数、单词数、字节数

## sort

sort能够对文本进行简单排序，默认是根据 ascii 码排序

```text
-t    指定分隔符
-k    指定排序使用的列，默认使用第一列
-r    倒序
-n    根据数值大小而非 ascii 
-o    将输出写入文件
-u    去重，等效于 sort file | uniq
```

## uniq

uniq 用来处理重复行，它有一个非常重要的特点：只能处理相邻的重复行，因此 uniq 要和 sort 一起使用，否则无法达到去重的效果

```text
-c    在行首加上重复的次数
-d    仅显示重复的行
-u    仅显示不重复的行
```

## wc

wc 是 word count 的缩写，用来统计文本数量

```text
常规输出为
行数  单词数  字节数
-l 只输出行数
-w 只输出单词数
-m 只统计字符数，换行符不计入统计，EOF会计入统计
```
