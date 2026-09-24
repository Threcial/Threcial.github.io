---
title: "linux用户管理"
date: 2026-04-15 10:41:16
permalink: "linux/linux用户管理/"
updated: 2026-04-15 10:41:16
categories:
  - "linux"
tags:
  - "linux从0开始"
description: "在/etc/passwd文件中存放有当前所有用户信息，etc/passwd内容示例如下 [root@centos7 ~]# cat /etc/passwd root:x:0:0:root:/root:"
cover: /img/covers/cover-029.jpg
top_img: false
---

在/etc/passwd文件中存放有当前所有用户信息，etc/passwd内容示例如下

```text
[root@centos7 ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
```

以：为分隔符共有七列，含义分别如下：

1. 用户名
2. 密码占位符
3. uid 0为超级用户，1-999为系统用户，1000+为普通用户
4. gid
5. 描述
6. 用户家目录
7. 用户登录shell，设置为/sbin/nologin即无法登录命令解释器

在/etc/shadow中存放有用户密码信息，示例如下

```text
[root@centos7 ~]# cat /etc/shadow
root:$6$d9.a.Iak$lXdjBLv8533RJ9L1GLbQ22MIY1uMOx4SPADuKthsTRa8AjqpDPui/LJj.MSMLd1e6EqU2eTKSY/v3qNWADn3G/:20558:0:99999:7:::
bin:*:18353:0:99999:7:::
daemon:*:18353:0:99999:7:::
adm:*:18353:0:99999:7:::
lp:*:18353:0:99999:7:::
sync:*:18353:0:99999:7:::
shutdown:*:18353:0:99999:7:::
halt:*:18353:0:99999:7:::
mail:*:18353:0:99999:7:::
operator:*:18353:0:99999:7:::
games:*:18353:0:99999:7:::
ftp:*:18353:0:99999:7:::
nobody:*:18353:0:99999:7:::
systemd-network:!!:20556::::::
dbus:!!:20556::::::
polkitd:!!:20556::::::
sshd:!!:20556::::::
postfix:!!:20556::::::
chrony:!!:20556::::::
```

以：为分隔符共有9列，含义为

```text
用户名:密码:上次修改时间:最小间隔:最大间隔:警告:宽限期:过期时间:保留
```

1. 用户名
2. 密码，非明文，$6$表示加密方法，如$1$表示MD5，$5$表示SHA-256，$6$表示SHA-512
3. 密码的上次修改时间，以1970年1月1日起算的时间
4. 最小修改密码间隔
5. 最大修改密码间隔
6. 提前警告修改密码的天数
7. 密码过期宽限时间
8. 账号过期时间
9. 保留，目前无含义

### useradd

通过useradd可以添加用户

```text
-u 指定uid
-g 指定基本gid
-G 指定附加gid
-m 创建家目录，一般默认创建
-d 指定家目录
```

### usermod

修改用户命令

```text
-G 修改附加组，注意此修改会覆盖原有附加组
-aG a为append附加意思，追加附加组不覆盖
-l 修改用户名，语法为usermod -l newname oldname
-L 锁定用户，无法登录
-U 解锁用户
```

### userdel

删除用户命令，一般带-r，否则无法删除干净

## 用户权限管理

```text
chmod 文件权限修改
可以用数字也可以用字母，如
chmod u+w,o-r file1 file1的所有者添加w权限，其他人删除r权限
u所有者 g组 o其他人 +添加 -删除 =直接设置 a表示all,所有人一起设置
chmod 755 file 以755设置file权限
-R 递归设置
```
```text
chown 所属用户和组修改
语法 
chown user:group file
只改组可以写为 chown :group file
```
