---
title: "linux文件处理"
date: 2026-04-13 15:47:44
permalink: "linux/linux文件处理/"
updated: 2026-04-13 15:47:44
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "路径 操作文件可用绝对路径和相对路径 切换目录命令 cd .. 上级目录 - 上次所在目录 ~ 或无参数，回到家目录 . 当前目录 [root@localhost lib]# pwd /usr/lib"
cover: /img/covers/cover-034.jpg
top_img: false
---

## 路径

操作文件可用绝对路径和相对路径

```text
切换目录命令 cd
.. 上级目录
- 上次所在目录
~ 或无参数，回到家目录
. 当前目录

[root@localhost lib]# pwd
/usr/lib
[root@localhost lib]# cd 
[root@localhost ~]# pwd
/root
[root@localhost ~]# cd -
/usr/lib
[root@localhost lib]# pwd
/usr/lib
```

## 文件与目录的创建和管理

```text
mkdir 创建目录
-p 创建多级目录，否则默认一个选项创建一个目录

[root@localhost ~]# mkdir dir1/dir2 -p
[root@localhost ~]# ls
anaconda-ks.cfg  dir1
[root@localhost ~]# mkdir dir3/dir4
mkdir: 无法创建目录"dir3/dir4": 没有那个文件或目录
```
```text
touch 创建文件

[root@localhost ~]# touch file1 file2
[root@localhost ~]# ls
anaconda-ks.cfg  dir1  file1  file2
```
```text
cp 复制文件
-a 递归辅助，用于复制目录
用法 cp 源文件 目标文件
mv 移动文件、重命名
用法 mv 源文件 目标文件
查看文件 cat tail more less head
tail -f 可以实时查看，常用查看日志
```

## vim 编辑器

vim是一个文件编辑器，对文件进行编辑可用命令vim filename，文件不存在则会进行创建

vim有三种模式：普通模式、插入模式、命令模式

### 普通模式

普通模式下一般用快捷方式对文件内容进行处理

- G 移动到文件末尾
- gg 移动到文件开头
- yy 复制当前行
- nyy复制当前开始往下n行
- p/P 粘贴到下一行，粘贴到上一行
- h j k l 光标左下上右移动
- dd 删除当前行
- ndd 同nyy
- u 恢复上一个操作
- crtl+r 撤销u操作
- x/X 往后删除字符，往前删除字符
- . 重复执行上一个操作
- i 进入插入模式

### 插入模式

插入模式下可以直接对文本进行编辑

### 命令模式

在普通模式下输入 / ? : 即可进入命令模式

```text
/text 往下寻找text n键下一个
?text 网上寻找text n键下一个

: 常规命令输入
:wq 保存并退出    :q 退出    :q! 强制退出
:set nu 显示行号    :set nonu 不显示行号

:s 替换命令
例 %s#text1#text2#g
%-全部文件，可以用1,4指定1到4行
s替换命令
# 分隔符，也可用其他符号
text1 text2 要被替换的内容
g 全局匹配，否则只替换每行第一个
```
