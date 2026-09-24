---
title: "Linux 系统信息查看与性能排查基础"
date: 2026-05-16 22:13:37
permalink: "linux/linux-系统信息查看与性能排查基础/"
updated: 2026-05-16 22:13:37
categories:
  - "linux"
tags:
  - "linux"
  - "排查"
description: "uptime 用来查看系统运行时间和负载 [root@centos7 ~]# uptime 21:48:14 up 8:26, 1 user, load average: 0.00, 0.04, 0."
cover: /img/covers/cover-012.jpg
top_img: false
---

## uptime

用来查看系统运行时间和负载

```text
[root@centos7 ~]# uptime
 21:48:14 up  8:26,  1 user,  load average: 0.00, 0.04, 0.05

21:48:14 up                       当前时间
8:26                              已运行8分26秒
1 user                            当前一个用户在线
load average: 0.00, 0.04, 0.05    1 分钟、5 分钟、15 分钟内平均负载
```

## lscpu

查看 CPU 架构、核心数、线程数、型号等信息

```text
[root@centos7 ~]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    1
Core(s) per socket:    2
座：                 1
NUMA 节点：         1
厂商 ID：           GenuineIntel
CPU 系列：          6
型号：              183
型号名称：        Intel(R) Core(TM) i7-14650HX
步进：              1
CPU MHz：             2419.200
BogoMIPS：            4838.40
超管理器厂商：  VMware
虚拟化类型：     完全
L1d 缓存：          48K
L1i 缓存：          32K
L2 缓存：           2048K
L3 缓存：           30720K
NUMA 节点0 CPU：    0,1
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology tsc_reliable nonstop_tsc eagerfpu pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single ssbd ibrs ibpb stibp ibrs_enhanced fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 arat umip pku ospke gfni vaes vpclmulqdq movdiri movdir64b md_clear spec_ctrl intel_stibp flush_l1d arch_capabilities
```

## free

查看内存情况

```text
[root@centos7 ~]# free -hs 2
              total        used        free      shared  buff/cache   available
Mem:           972M        176M        696M        7.7M         99M        672M
Swap:          2.0G        407M        1.6G

              total        used        free      shared  buff/cache   available
Mem:           972M        176M        696M        7.7M         99M        672M
Swap:          2.0G        407M        1.6G

              total        used        free      shared  buff/cache   available
Mem:           972M        176M        696M        7.7M         99M        672M
Swap:          2.0G        407M        1.6G

              total        used        free      shared  buff/cache   available
Mem:           972M        176M        696M        7.7M         99M        672M
Swap:          2.0G        407M        1.6G

-h 人类可读方式显示 -s 每2秒显示一次，
判断内存是否紧张，应关注 available 而非 free ，因为 Linux 会主动使用空闲内存做缓存，free 低并不一定表示内存不够。
```

## last

登录历史

```text
[root@centos7 ~]# last
root     pts/0        192.168.174.1    Sat May 16 20:34   still logged in  
root     pts/1        192.168.174.1    Fri May 15 15:12 - 15:35  (00:23)  
reboot   system boot  3.10.0-1160.el7. Mon Apr 13 18:47 - 15:49 (23+21:01)  

wtmp begins Mon Apr 13 18:47:34 2026

root                登录用户
pts/0               登录终端
192.168.174.1       登录ip
Fri May 15 15:12    登录时间
15:35  (00:23)      离开时间以及在线时长，如果在线则still logged in 

lastb 查看登录失败历史
last reboot 重启历史
```

## iftop

查看网络状况

```text
安装
yum install epel-release -y
yum install iftop -y
```
```text
[root@centos7 ~]# iftop -Bnt -i ens33

-i  指定网卡
-n  不解析域名，只显示 IP
-t  文本模式显示
-B  以 Byte 为单位显示
```

## iostat

```text
yum install sysstat -y
```

## top

实时查看系统进程和运行状况

```text
快捷键
P  按 CPU 排序
M  按内存排序
T  按累计 CPU 时间排序
R  倒序
c  显示完整命令
q  退出
```
