---
title: "MySQL 备份恢复与主从复制"
date: 2026-06-07 10:22:41
permalink: "数据库/mysql-备份恢复与主从复制/"
updated: 2026-06-07 10:22:41
categories:
  - "数据库"
tags:
  - "mysql"
  - "主从复制"
  - "备份恢复"
description: "MySQL 备份方式可以分类 按备份方式分： 逻辑备份：导出 SQL 语句，比如 mysqldump 物理备份：直接备份数据文件，比如 xtrabackup 按备份范围分： 全量备份：某个时间点完整导"
cover: /img/covers/cover-002.jpg
top_img: false
---

MySQL 备份方式可以分类

按备份方式分：

```text
逻辑备份：导出 SQL 语句，比如 mysqldump
物理备份：直接备份数据文件，比如 xtrabackup
```

按备份范围分：

```text
全量备份：某个时间点完整导出一份数据
增量备份：只记录全量备份之后发生的变化
```

本文主要讲逻辑备份，主从复制也是基于逻辑备份

## mysqldump：全量备份

```text
mysqldump -u root -p --single-transaction --master-data=2 -B rocky >rocky.sql
```

`--single-transaction` 会在备份开始时开启一个事务，这样可以保证备份一致性。`-x` 会使用锁死表的方式来保证一致性，但这样会导致前端业务卡住，所以不推荐使用 `-x`

`--master-data=2` 会在备份文件里以注释的方式记录 binlog 的位置，便于恢复时使用

`-B` 会将后面所有参数视为库，否则只有第一参数为库，后面的参数都为表

## binlog：增量备份

`mysqldump` 只能恢复到备份那个时间点，如果凌晨 2 点做了全量备份，下午 5 点误删数据，那么只靠凌晨 2 点的备份，最多恢复到 2 点，2 点到 5 点之间的写入就丢了

这时候就需要 binlog

binlog 是 MySQL 的二进制日志，记录数据库中除了查询以外的变更操作

开启 binlog，需要在 `my.cnf` 里配置：

```text
[mysqld]

server-id=1                               标识，用于主从复制使用
log-bin=/data/mysql-bin/binlog             备份文件前缀，结尾的binlog不是目录
binlog_format=ROW                          记录方式
expire_logs_days=7                         过期时间
max_binlog_size=500M                        文件大小
sync_binlog=0                               同步方式

log-bin 指定 binlog 文件目录和前缀
binlog 目录必须存在
目录用户和用户组要改成 mysql
server-id 必须配置，否则 binlog复制相关配置可能不正常
```
```text
sync_binlog=0：由操作系统决定什么时候刷盘，性能好，安全性弱一些
sync_binlog=1：每次事务提交都刷 binlog，安全性高，性能下降
sync_binlog=N：每 N 次事务提交刷一次
```

## mysqlbinlog：把 binlog 转成可恢复的 SQL

binlog 是二进制文件，不能直接 `cat` 看。要用：

```text
mysqlbinlog binlog.000001
```

如果遇到字符集配置报错，可以加：

```text
mysqlbinlog --no-defaults binlog.000001
```

常用参数：

```text
-d                  指定数据库
-r                  输出到文件
--start-position    指定开始位置
--stop-position     指定结束位置
--start-datetime    指定开始时间
--stop-datetime     指定结束时间
--no-defaults       不读取默认配置文件，避免客户端配置影响 mysqlbinlog
--skip-gtids        跳过 GTID 信息，某些场景配合 -d 使用
```

把 binlog 转成 SQL 文件：

```text
mysqlbinlog --no-defaults binlog.000016 -r incr.sql
```

多个 binlog 文件一起处理：

```text
mysqlbinlog --no-defaults 
  binlog.000016 binlog.000017 
  -d rocky 
  --skip-gtids 
  --start-position=2725 
  -r rocky_incr.sql
```

按时间：

```text
mysqlbinlog --no-defaults 
  --start-datetime='2026-06-04 13:02:00' 
  --stop-datetime='2026-06-04 13:30:00' 
  binlog.000016 
  -r incr.sql
```

## 备份文件校验：md5sum

备份文件生成之后还要确认传输过程没有损坏

生成校验文件：

```text
md5sum rocky.sql >rocky.sql.flag
```

传到远端后，在同目录验证：

```text
md5sum -c rocky.sql.flag
```

## 备份恢复流程

假设有：

```text
全量备份：rocky_2026-06-04_02-00-00.sql
binlog：mysql8.000008、mysql8.000009
全量备份里记录的起点：mysql8.000008:478
```

**恢复时先隔离业务**：关闭应用连接；禁止继续写入；只允许本机操作数据库；确认恢复目标库和当前库状态

生成增量 SQL：

```text
mysqlbinlog --no-defaults
  mysql8.000008 mysql8.000009 
  -d rocky 
  --skip-gtids 
  --start-position=478 
  -r rocky_incr.sql
```

恢复全量：

```markdown
mysql -u root -p < rocky_2026-06-04_02-00-00.sql
```

再恢复增量：

```markdown
mysql -u root -p < rocky_incr.sql
```

恢复完成后验证数据

恢复过程本身会产生大量 `INSERT/UPDATE/CREATE`，如果 binlog 开着，这些恢复动作也会被重新写入 binlog，需要根据实际情况考虑是否临时关闭 binlog

## 主从复制

主从复制其实就是把“binlog 增量恢复”自动化

主从复制原理是：从库自动连接主库自动拉取主库 binlog，写成自己的 relay log，再由 SQL 线程自动重放

### 主从复制作用

读写分离：主库写，从库读  
故障切换：主库故障时，从库可以提升为新主库  
报表查询：复杂查询放到从库，避免影响主库

**主从复制不是备份**，如果主库执行 update 从库也会执行

### 配置

假设：

```text
主库：192.168.174.128
从库：192.168.174.132
同步库：rocky
主库用于复制的用户：repli
```

主库`my.cnf`：

```text
[mysqld]
server-id=1
log-bin=/data/mysql-bin/mysql8
binlog_format=ROW
```

主库用户 repli：

```text
CREATE USER 'repli'@'192.168.174.%' IDENTIFIED BY 'repli';
GRANT REPLICATION SLAVE ON *.* TO 'repli'@'192.168.174.%';
FLUSH PRIVILEGES;
```

主库导出：

```text
mysqldump -u root -p --single-transaction --master-data=2 -B rocky >rocky.sql
```

导出后找坐标：

```text
grep -i "CHANGE MASTER" rocky.sql

-- CHANGE MASTER TO MASTER_LOG_FILE='mysql8.000008', MASTER_LOG_POS=478; 
得到从库起点
```

从库 `my.cnf`：

```text
[mysqld]
server-id=2
relay_log=mysql-relay-bin
read_only=ON
super_read_only=ON
```

如果这个从库不会再作为其他从库的主库，可以不开 binlog

如果只同步某个库，可以配置：

```text
replicate-do-db=rocky
```

从库导入主库全量备份：

```markdown
mysql -u root -p < rocky.sql
```

从库登录 MySQL ：

```text
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='192.168.174.128',
SOURCE_PORT=3306,
SOURCE_USER='repli',
SOURCE_PASSWORD='repli',
SOURCE_LOG_FILE='mysql8.000008',
SOURCE_LOG_POS=478,
GET_SOURCE_PUBLIC_KEY=1;

START REPLICA;
SHOW REPLICA STATUS;
```

旧版本写法：

```text
CHANGE MASTER TO
MASTER_HOST='192.168.174.128',
MASTER_PORT=3306,
MASTER_USER='repli',
MASTER_PASSWORD='repli',
MASTER_LOG_FILE='mysql8.000008',
MASTER_LOG_POS=478,
GET_MASTER_PUBLIC_KEY=1;

START SLAVE;
SHOW SLAVE STATUS;
```
```text
STOP REPLICA;                停止复制
STOP SLAVE;

RESET REPLICA ALL;        删除复制配置
RESET SLAVE ALL;
```

**主从复制默认是异步的。主库提交成功，不代表从库已经执行完成**

延迟常见原因：从库太多，主库网络和发送压力变大；从库硬件弱于主库；主库写入压力过高等
