---
title: "linux从0开始"
date: 2026-04-13 14:16:53
permalink: "linux/linux从0开始/"
updated: 2026-04-14 14:16:53
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "以centos7为例 这是一个典型命令提示符，含义为“用户名@主机 当前位置”，“~”表示当前用户默认目录 快捷键 crtl+e 移动光标到行尾 crtl+c 终止当前命令执行 crtl+l 清空屏幕"
cover: /img/covers/cover-035.png
top_img: false
---

以centos7为例

![](/img/wp/1776050590-image.png)

这是一个典型命令提示符，含义为“用户名@主机 当前位置”，“~”表示当前用户默认目录

## 快捷键

- crtl+e 移动光标到行尾
- crtl+c 终止当前命令执行
- crtl+l 清空屏幕显示
- crtl+a 移动光标到行头
- tab 自动补全

## 基础命令

**语法**：命令 [-选项] [参数]

|  |  |  |  |
| --- | --- | --- | --- |
| 命令 | 作用 | 例子 | 说明 |
| pwd | 显示当前路径 | [root@localhost ~]# pwd /root |  |
| echo | 输出 | [root@localhost ~]# a="hello word" [root@localhost ~]# echo $a hello word | 数字可以不加引号，转义符在引号里被解析 |
| history | 历史命令记录 | [root@localhost ~]# history 1 ip adress 2 ip address 3 ip addr 4 cd /etc 5 ls | -w参数可以将history的输出保存到用户目录下的.bash\_history文件中 -c清除历史命令记录 |
| ！ | 历史命令调用符 | [root@localhost ~]# !37 pwd /root [root@localhost ~]# !e echo $a hello word | !! 重复上一条 !xxx 找最近匹配命令 !数字 按编号执行 |
| alias | 别名 | [root@localhost ~]# alias hh='history 2' [root@localhost ~]# hh 43 alias hh='history 2' 44 hh | unalias可以取消别名 |
| date | 时间 | [root@localhost ~]# date 202x年 xx月 xx日 星期x 13:40:52 CST |  |
| timedatectl | 时间状态 | [root@localhost ~]# timedatectl Local time: 一 2026-04-13 13:38:40 CST Universal time: 一 2026-04-13 05:38:40 UTC RTC time: 一 2026-04-13 05:38:40 Time zone: Asia/Shanghai (CST, +0800) NTP enabled: yes NTP synchronized: yes RTC in local TZ: no DST active: n/a | 用于控制时区，时间同步等 |
| shutdown | 关机 |  | -r 重启 -h 关机 now 立刻执行（否则有一分钟等待） |
| ls | 查看目录下文件 | [root@localhost home]# ls -dlah /root dr-xr-x---. 2 root root 135 4月 13 2026 /root | -a 显示所有包括隐藏文件 -l 显示详细信息 -h 以人类可读方式显示，主要是文件大小 -d 显示目录本身而非目录以下文件 -i 显示inode号 |
| tree | 树状显示目录结构 |  | -L 限制显示层级 -d 只显示目录 -C 添加颜色 |

## 根目录结构

- bin 系统命令
- sbin 管理员命令
- etc 配置文件
- home 家目录
- root root用户家目录
- temp 临时文件
- boot 启动文件
- proc 系统运行文件
- dev 设备
- lib/lib64 库文件
- usr 程序存放
- var 常变化的文件储存于此
- mnt 挂载
- sys 内核文件
