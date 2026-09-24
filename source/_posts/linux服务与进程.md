---
title: "linux服务与进程"
date: 2026-05-17 15:41:38
permalink: "linux/linux服务与进程/"
updated: 2026-05-17 15:41:38
categories:
  - "linux"
tags:
  - "服务"
  - "进程"
description: "服务 在 Linux 中，服务通常指一种长期运行在后台的程序，它启动后不会马上退出，而是持续等待用户、系统或其他程序调用 早期 Linux 使用 SysV init 来管理服务。CentOS 7 开始"
cover: /img/covers/cover-010.jpg
top_img: false
---

## 服务

在 Linux 中，服务通常指一种长期运行在后台的程序，它启动后不会马上退出，而是持续等待用户、系统或其他程序调用

早期 Linux 使用 SysV init 来管理服务。CentOS 7 开始，systemd 取代了传统 init，成为系统初始化和服务管理的核心，并且 systemctl 兼容了部分传统 service 用法

systemd 管理的对象称为 unit

常见 unit 类型：

```text
.service   服务
.target    启动目标
.timer     定时器
.socket    socket 激活
.mount     挂载点
.path      路径监听
```

服务最常见的是 `.service` 文件。

systemd 相关目录：

```text
/usr/lib/systemd/system/   软件包安装的 unit 文件
/run/systemd/system/       运行时生成的 unit 文件
/etc/systemd/system/       管理员自定义或覆盖的 unit 文件
/etc/sysconfig/            某些服务的默认参数配置
```

`/etc/systemd/system/` 的优先级通常高于 `/usr/lib/systemd/system/`，适合管理员自定义覆盖服务配置

```text
sytemctl 命令用于管理这些服务

systemctl start nginx           启动nginx
systemctl stop nginx            停止nginx
systemctl restart nginx         重启nginx
systemctl reload nginx          重载nginx
systemctl status nginx          查看nginx状态
systemctl enable nginx          设置开机自启
systemctl disable nginx         关闭开机自启
systemctl is-active nginx       检查服务是否已经启动
systemctl is-enabled nginx	查看服务是否开机自启动
```

## 进程

程序是存放在磁盘上的静态文件，进程是程序运行后的实例。

每个进程都有一个唯一编号，称为 PID，同时进程之间有父子关系，一个进程可以创建子进程，子进程的 PPID 就是父进程的 PID

### 0号进程、1号进程、2号进程

**PID 0：swapper / idle 进程**

PID 0 通常被称为：swapper / idle process ，它是 Linux 内核启动早期创建的特殊进程，不是普通用户进程

它的主要作用是：**当 CPU 没有其他可运行任务时，执行 idle 空闲循环**

也就是说，当系统中没有进程需要占用 CPU 时，CPU 并不是“停止工作”，而是进入空闲状态，这个空闲状态就和 PID 0 有关

PID 0 不会像普通进程一样出现在 ps 输出中，它属于内核调度体系的一部分，不能被 kill 也不是 systemd 管理的服务

**PID 1：init / systemd**

PID 1 是第一个用户空间进程，在 CentOS 7 及之后的系统中，PID 1 为 systemd

PID 1 的地位非常特殊，它负责系统初始化、启动系统服务、管理服务依赖、回收孤儿进程、维护系统运行状态等

如果一个父进程退出了，但它的子进程还在运行，这个子进程就会变成孤儿进程，孤儿进程会由 PID 1 收养

**PID 2：kthreadd**

kthreadd 主要负责创建和管理其他内核线程

### 静态查看进程

ps 静态进程管理命令，常用参数 -ef 和 aux

```text
-e 全部进程
-f 完整显示
-l 长格式显示
-p 指定pid
-u 指定用户
-H 树状显示
```

| 字段 | 含义 |
| --- | --- |
| `USER` | 进程所属用户 |
| `PID` | 进程号 |
| `%CPU` | CPU 占用百分比 |
| `%MEM` | 内存占用百分比 |
| `VSZ` | 虚拟内存大小 |
| `RSS` | 实际物理内存占用 |
| `TTY` | 启动该进程的终端 |
| `STAT` | 进程状态 |
| `START` | 进程启动时间 |
| `TIME` | 累计 CPU 时间 |
| `COMMAND` | 启动命令 |

常见 stat

- **R running** 运行中或可运行
- **S sleeping** 可中断睡眠
- **D uninterruptible sleep** 不可中断睡眠，常见于 I/O 等待
- **T stopped** 暂停
- **Z zombie** 僵尸进程
- **s session leader** 会话首进程
- **+** 前台进程组
- **<** 高优先级
- **N** 低优先级

```text
pstree 树状显示进程

安装 yum install psmisc -y
使用 pstree -p 与pid一起显示
```

### 动态查看进程

top 实时显示进程情况，用法 [Linux 系统信息查看与性能排查基础](https://threcial.cn/linux/linux-%e7%b3%bb%e7%bb%9f%e4%bf%a1%e6%81%af%e6%9f%a5%e7%9c%8b%e4%b8%8e%e6%80%a7%e8%83%bd%e6%8e%92%e6%9f%a5%e5%9f%ba%e7%a1%80/)

### 进程管理

**pidof 和 pgrep 用于根据进程名查找 pid**

```text
pidof sshd  查看sshd的pid

pgrep比pidof更加灵活，类似grep不需要完全匹配而是部分匹配

pgrep ssh 也能查看到sshd的pid
pgrep -P ppid 根据父进程pid查找子进程pid
pgrep -u username 根据用户查找
```

**kill用于给进程发送信号**

```text
kill -l 显示可用信号

常见
-15 终止
-9 强制终止
-19 暂停
-18 继续运行

kill -19 1234 暂停1234进程的运行
killall ping   终止所有ping进程
```

**后台任务**

```text
通过 & 可以让一个命令在后台执行，如：
ping www.qq.com >> ping.log 2>&1 &
```

jobs 查看当前 shell 的后台任务，注意只能查看当前 shell 的后台任务，除此之外的进程都看不了

```text
fg %1        切换到前台运行
bg %1        切换到后台运行
             其中 %1 是 jobs 中显示的任务编号
```

& 只是让程序在后台运行，不一定能保证断开终端后继续运行，如果需要断开终端后继续运行，需要用 nohup

```text
nohup ping www.qq.com >> ping.log 2>&1 &

nohup 的作用是让进程忽略 SIGHUP 信号，nohup 本身不负责后台运行，后台运行是 & 的作用

注:通过kill -l 可以看到1号信号就是hup信号
```

**lsof : list open files**

显示进程所打开的文件，也就是占用情况

```text
lsof -p 1234                1234进程打开的文件
lsof -u root                root用户使用的文件
lsof /var/log/messages      messages文件被哪些进程使用
lsof -i:22                  22端口的占用
```
