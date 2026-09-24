---
title: "linux下的存储设备"
date: 2026-04-16 16:34:23
permalink: "linux/linux下的存储设备/"
updated: 2026-04-16 16:34:23
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "磁盘 在 /dev（设备目录）下可以找到所有设备，包括磁盘。常见sda、sdb、hda，hdb等 命名方式： h表示IDE，s表示SATA 第二个字母：disk 第三个字母：序号，由a开始 分区类型 "
cover: /img/covers/cover-028.png
top_img: false
---

## 磁盘

在 /dev（设备目录）下可以找到所有设备，包括磁盘。常见sda、sdb、hda，hdb等

命名方式：

1. h表示IDE，s表示SATA
2. 第二个字母：disk
3. 第三个字母：序号，由a开始

### 分区类型

**Master Boot Record（主引导记录）（MBR）**

**GUID Partition Table（全局唯一标识分区表）（GPT）**

区别：

- MBR最多4个主分区，GPT最多128个分区
- MBR设计简单，GPT功能更全面
- MBR是传统标准，GPT是现代标准
- MBR适用于小型磁盘（<2TB），GPT适用于大型磁盘

## 管理命令

```text
lsblk查看当前分区情况
-f 显示文件系统
-a 查看全部信息
```

## 新硬盘的处理

```text
[root@centos7 ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0    2G  0 disk 
sr0              11:0    1  4.4G  0 rom  
```

以sdb和centos7环境为例说明，sdb为新加入的硬盘，无法直接使用，需要经过分区、格式化、挂载三步后才能使用

### 分区

```text
fdisk /dev/sdb 进入交互界面对新硬盘进行处理

[root@centos7 ~]# fdisk /dev/sdb
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。

Device does not contain a recognized partition table
使用磁盘标识符 0xb54254b0 创建新的 DOS 磁盘标签。

命令(输入 m 获取帮助)：m
命令操作
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)
```

根据指示创建即可，mbr分区只能有4个主分区，因此在创建完三个主分区和一个扩展分区后，再次用n将创建逻辑分区，最后w保存并写入硬盘

### 格式化

```text
mkfs.xfs /dev/sdb1 xfs并非唯一文件系统，有其他选择
lsblk -f 即可查看到对应分区已经被格式化
```

### 挂载

```text
mount 源 目标
mount /dev/sdb1 /root/disk1
umont /dev/sdb1 卸载分区
```

df -h 可以查看挂载情况

## LVM扩容

首先有三个概念

- PV（Physical Volume）物理卷（磁盘/分区）
- VG（Volume Group）卷组（空间池）
- LV（Logical Volume）逻辑卷（给系统用的“分区”）

扩容原理是将pv加入vg，再从vg中分出lv。相当于把几桶水倒入一个大水池，再用其他桶装出来

未格式化的分区可以用来创建pv

```text
pvcreate /dev/sdb2 用/dev/sdb2创建pv
pvs 查看当前有的pv
vgcreate vgname /dev/sdb2 创建名为vgname的vg同时将/dev/sdb2加入该vg
vgs 查看当前vg
lvcreate -l 100M -n lv1 vgname 从名为vgname的vg中创建一个大小100M名为lv1的lv
lvs 查看当前lv
```

lv再格式化和挂载就可以正常使用了，注意此时路径为/dev/vgname/lvname

### 容量管理

```text
pvremove pvname 移除一个pv

vgextend vgname pvname 添加一个pv到vg里
vgreduce vgname pvname 移除一个vg里的pv

lvextend -L +100M lvpath 给一个存在的lv扩容
注意扩容后新加的容量尚未格式化，需要用到
xfs_grows mntpath 对挂载路径进行处理
lvreduce -L -50M lvpath 给lv缩容50M
lvextend -l +100%FREE lvpath 将剩余空间全部分配给lv
```

### 数据迁移

要求处于同一vg

```text
在不指定迁移目标
pvmove /dev/vgname/pvname 会将pvname上的所有数据迁移到同vg下的其他pv
指定源和目标
pvmove /dev/sdb1 /dev/sdc1
```

注意-L 和 -l 大小写区分，-L 后跟容量大下，如+10G，-20M，-l 后跟比例，如+10%FREE
