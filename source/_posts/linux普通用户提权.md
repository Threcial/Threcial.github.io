---
title: "linux普通用户提权"
date: 2026-04-15 14:07:17
permalink: "linux/linux普通用户提权/"
updated: 2026-04-15 14:07:17
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "sudo可以让普通用户以管理员权限执行命令 /etc/sudoers是相关配置文件，但不推荐直接修改，可使用命令visudo修改，直接在文件末尾可以看到语法 例如 ## Next comes the "
cover: /img/covers/cover-030.png
top_img: false
---

sudo可以让普通用户以管理员权限执行命令

/etc/sudoers是相关配置文件，但不推荐直接修改，可使用命令visudo修改，直接在文件末尾可以看到语法

例如

```text
## Next comes the main part: which users can run what software on 
## which machines (the sudoers file can be shared between multiple
## systems).
## Syntax:
##
## 	user	MACHINE=COMMANDS
##
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere 
root	ALL=(ALL) 	ALL

## Allows members of the 'sys' group to run networking, software, 
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel	ALL=(ALL)	ALL

## Same thing without a password
# %wheel	ALL=(ALL)	NOPASSWD: ALL

## Allows members of the users group to mount and unmount the 
## cdrom as root
# %users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

## Allows members of the users group to shutdown this system
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.d
```
