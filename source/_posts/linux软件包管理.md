---
title: "linux软件包管理"
date: 2026-04-16 10:47:01
permalink: "linux/linux软件包管理/"
updated: 2026-04-16 10:47:01
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "rpm rpm是管理rpm包的命令 -i install，安装 -v 显示过程 -h 显示进度 -e 卸载 -q 查看是否安装 -qa 查看全部已安装的包 -qi 查看包信息 -ql 查看安装文件 -"
cover: /img/covers/cover-026.jpg
top_img: false
---

## rpm

rpm是管理rpm包的命令

```text
-i install，安装
-v 显示过程
-h 显示进度
-e 卸载
-q 查看是否安装
-qa 查看全部已安装的包
-qi 查看包信息
-ql 查看安装文件
-qc 查看配置文件
-qf 查看文件所属安装包
```

与dnf和yum的区别：rpm不会自动处理依赖

## yum

/etc/yum.repos.d/下存放有 yum 软件仓库配置文件的目录，示例：

```text
[root@centos7 script]# ls /etc/yum.repos.d/
CentOS-Base.repo      CentOS-Debuginfo.repo  CentOS-Sources.repo
CentOS-Base.repo.bak  CentOS-fasttrack.repo  CentOS-Vault.repo
CentOS-CR.repo        CentOS-Media.repo      CentOS-x86_64-kernel.repo
```

每个repo文件都是一个软件源配置，部分内容如下：

```text
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#released updates 
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

- [base] 仓库id，也是唯一标识
- name 描述
- baseurl 下载地址
- gpgcheck 校验软件签名，1为开启，0为关闭
- enabled 启用，参数同gpgcheck
- gpgkey 指定校验公钥

```text
yum命令用法
yum install xxx 安装
yum remove xxx 卸载
yum search xxx 寻找
yum list installed xxx 查看是否安装
yum info xxx 信息
yum repolist 仓库列表
yum repolist all 全部仓库列表
yum update 更新
yum update xxx 更新xxx
yum clean 清除缓存
yum makecache 生成新缓存，更新软件源常用
yum histroy 历史yun命令记录
```
