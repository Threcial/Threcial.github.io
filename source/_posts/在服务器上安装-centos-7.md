---
title: "在服务器上安装 centos-7"
date: 2026-05-24 20:08:10
permalink: "linux/在服务器上安装-centos-7/"
updated: 2026-05-24 20:08:10
categories:
  - "linux"
tags:
  - "centos7"
  - "安装"
description: "CentOS Linux 7 已经在 2024 年 6 月 30 日结束生命周期，不再获得官方更新和安全补丁，实际中应使用其他版本，但安装思路是相近的 u盘启动 进入系统 bios 界面 启动参数里写"
cover: /img/covers/cover-055.png
top_img: false
---

CentOS Linux 7 已经在 2024 年 6 月 30 日结束生命周期，不再获得官方更新和安全补丁，实际中应使用其他版本，但安装思路是相近的

## **u盘启动**

进入系统 bios 界面

启动参数里写的 ISO 卷标，和实际 U 盘的卷标可能不一致，而 CentOS 安装程序会根据 `LABEL=` 去找安装源，如果 U 盘卷标对不上，就会找不到

常见启动参数类似 `inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet`，需要修改成u盘卷标

## 分区

大于 2T 的硬盘要注意 MBR 和 GPT。MBR 最大只能识别到 2T 左右，大容量硬盘应该使用 GPT 分区

如果是服务器，UEFI 启动，需要单独创建 /boot/efi 500M

如果是 Legacy BIOS + GPT，需要 BIOS Boot 分区，也就是 /biosboot

一般 /boot 分区即可

```text
/boot 1G
/boot/efi 500M
swap 实际内存的两倍，不超过16G
/ 200G
/data 剩余所有空间
```

## 系统优化

1. **安装好后首先确保网络正常**
2. **关闭 SELinux**  
     
   编辑配置文件：`vi /etc/selinux/config`  
   修改 `SELINUX=disable`
3. **根据实际情况开关防火墙**
4. **修改 yum 源并安装扩展源**  
     
   `curl -s -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo  
   yum install epel-release -y`
5. **安装常用软件、更新系统**  
     
   `yum groupinstall "Compatibility libraries" "Base" "Development tools" -y  
   yum groupinstall "debugging Tools" "Dial-up Networking Support" -y  
   yum install tree nmap dos2unix lrzsz nc lsof wget tcpdump htop iftop iotop sysstat nethogs -y  
   yum install psmisc net-tools bash-completion vim-enhanced -y  
   yum update -y`
6. **精简开机**  
     
   `systemctl list-unit-files |grep enable|egrep -v "sshd.service|crond.service|sysstat|rsyslog|^NetworkManager.service|irqbalance.service"|awk '{print "systemctl disable",$1}'|bash`
7. **设置时间同步自动任务**  
     
   在计划任务里面添加 `0 */12 * * * /usr/sbin/ntpdate ntp3.aliyun.com >/dev/null 2>&1`
8. **修改文件描述符限制**  
     
   查看当前限制：`ulimit -n`  
   如果小于 65535，修改：`vi /etc/security/limits.conf`  
   添加：`* - nofile 65535`
9. **优化系统内核**  
     
   编辑：`vi /etc/sysctl.conf`  
   添加：  
   `net.ipv4.tcp_keepalive_time = 1800  
   net.ipv4.ip_local_port_range = 4000 65000  
   net.ipv4.tcp_max_syn_backlog = 8192  
   net.core.somaxconn = 16384  
   net.core.netdev_max_backlog = 16384`
10. **清除系统版本信息**  
      
    `>/etc/issue`   
    `>/etc/issue.net`
11. **配置好 ssh 后修改 ssh 配置文件**
12. **锁定关键文件**  
      
    chattr +i /etc/passwd  
    chattr +i /etc/shadow

## vmware 上克隆模板机

在启动虚拟机之前修改 MAC 地址

修改主机名 vim /etc/hostname

修改网卡的 uuid 和 ip 地址 vim /etc/sysconfig/network-scripts/ifcfg-ens33 ，uuid 通过命令 uuidgen 生成
