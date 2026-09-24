---
title: "数据库概念以及MySQL配置"
date: 2026-06-04 13:30:03
permalink: "数据库/数据库概念以及mysql配置/"
updated: 2026-06-04 13:30:03
categories:
  - "数据库"
tags:
  - "mysql"
  - "数据库"
description: "数据库概念 数据库，英文是 Database，简称 DB，可以把他简单理解为有组织地存储和管理数据的仓库，普通文件也能存储数据，但数据库会按照库、表、字段、记录这些结构来管理数据 数据库管理系统，英文"
cover: /img/covers/cover-005.jpg
top_img: false
---

## 数据库概念

数据库，英文是 Database，简称 DB，可以把他简单理解为**有组织地存储和管理数据的仓库**，普通文件也能存储数据，但数据库会按照库、表、字段、记录这些结构来管理数据

数据库管理系统，英文是 DBMS，全称 Database Management System，是实际安装在系统里的大型软件，MySQL、MariaDB、Oracle、SQL Server、PostgreSQL、DB2、SQLite 这些都属于数据库管理系统

具体概念如下：

```text
DB：数据库，保存数据的仓库
DBMS：数据库管理系统，管理数据库的软件
SQL：操作关系型数据库的语言
MySQL：一种具体的数据库管理系统
```

## MySQL数据库配置

MySQL 常见主配置文件是 `/etc/my.cnf`

```text
[mysqld]
max_connections=200                      设置最大连接数
max_allowed_packet=32M
character-set-server=utf8mb4             设置字符集
collation-server=utf8mb4_unicode_ci      排序规则，决定字符串比较和排序方式
skip_name_resolve                        跳过反向域名解析，可以加快连接速度
skip-character-set-client-handshake      

datadir=/data/mysql                       数据库目录
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log             日志
pid-file=/var/run/mysqld/mysqld.pid

log_bin=/data/mysql_bin/mysql8            binlog的basename配置
server_id=1                                 
expire_logs_days=7                       保留七天
max_binlog_size=100k                     大小设置
sync_binlog=0                            写入硬盘频率，0为系统自动安排

[client]
default-character-set=utf8mb4           
```

**给 MySQL 使用的目录需要注意权限问题，最好直接把目录的所有者和组都改成 mysql**

配置完成启动 mysqld 服务，在如下日志文件中会生成临时密码

```text
/var/log/mysqld.log
```

找到临时密码以后登录

```text
mysql -u root -p
```

远程登录则使用 -h 指定主机，-P 指定端口

登录后第一件事通常是修改 root 密码

```text
ALTER USER 'root'@'localhost' IDENTIFIED BY '你的密码';
```

如果密码策略比较严格，简单密码会失败，可能要求包含大写、小写、数字、特殊字符，修改后刷新权限

```text
FLUSH PRIVILEGES;
```

检查配置文件是否生效

```text
SHOW VARIABLES LIKE '%chara%';
SHOW MASTER STATUS;
SHOW VARIABLES LIKE '%log_bin%';
SHOW VARIABLES LIKE '%datadir%';
SHOW VARIABLES LIKE '%coll%';
```

一些其他查看类语法

```text
show processlist;            查看数据库正在执行的sql语句，不完整 
show session status;         查看当前会话的数据库状态信息
show global status;          查看整个数据库运行的状态信息
show engine innodb status;   查看innodb引擎的状态信息
set 参数 = 值                 临时修改mysql参数的值，重启后失效
kill ID;                     杀掉某一个sql线程的命令，ID表示线程号
```

## 权限管理

在 MySQL 中创建一个用户默认是最小权限，需要额外给予权限

```text
CREATE USER 'root'@'%' IDENTIFIED BY '某个密码';
GRANT ALL ON *.* TO 'root'@'%' [ WITH GRANT OPTION ]; 
FLUSH PRIVILEGES;

MySQL 中用户是由名字和ip共同构成，'%'在 MySQL 中表示任意数量字符，在这里代表任意地址
授予权限时，all 表示所有权限，也可以根据具体权限划分。* 作用类似 shell 中的通配符，*.* 表示所有库中的所有表，实际使用时一定要遵循最小权限原则

revoke select on data1.* from 'alice'@'%';
收回权限：收回用户 'alice'@'%' 在 data1 库上任意表的 select 权限
```

## 忘记密码

如果忘记 root 密码，可以在配置文件中添加

```text
skip-grant-tables
```

来跳过密码验证，登录后修改 root 密码，同时记得再将配置文件改回去
