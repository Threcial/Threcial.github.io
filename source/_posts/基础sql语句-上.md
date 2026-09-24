---
title: "基础SQL语句 上"
date: 2026-06-05 20:46:01
permalink: "数据库/基础sql语句-上/"
updated: 2026-06-05 20:46:01
categories:
  - "数据库"
tags:
  - "mysql"
  - "语法"
description: "概念 SQL 全称是 Structured Query Language，结构化查询语言，SQL 大致可以分成几类： DDL：Data Definition Language 数据定义语言 DML：D"
cover: /img/covers/cover-004.jpg
top_img: false
---

## 概念

SQL 全称是 Structured Query Language，结构化查询语言，SQL 大致可以分成几类：

```text
DDL：Data Definition Language    数据定义语言
DML：Data Manipulation Language  数据操作语言
DQL：Data Query Language         数据查询语言
DCL：Data Control Language       数据控制语言
```

因为大部分时候在操作表，为了便于记忆，可以不严谨地分为下面三种语句

```text
管理表结构或库结构
管理表数据变化
管理表数据查询          
```

SQL 可以单行写，也可以多行写，但一条语句结束要加分号。MySQL 关键字不区分大小写

注释有三种常见写法：

```text
--           单行注释，注意 -- 后面要有空格
#            MySQL 特有的单行注释
/*    */     多行注释
```

## 数据类型

MySQL 中储存的数据有以下数据类型

- **tinyint**   
  1字节有符号整型，范围是-128-127
- **tinyint unsigned**   
  1字节无符号整型，范围是0-255
- **int**   
  4字节有符号整型，范围是-2147483648-2147483647
- **int unsigned**   
  4字节无符号整型，范围是0-4294967295
- **bigint**  
  8字节有符号整型，范围是-9223372036854775808-9223372036854775807
- **bigint unsigned**   
  8字节无符号整型，范围是0-18446744073709551615
- **float**   
  小数类型，4字节,不精准计算可以使用，不推荐使用
- **double**   
  小数类型，8字节,不精准计算可以使用，不推荐使用
- **decimal**   
  小数类型，精准计算使用，decimal(m,d),m表示数字总个数，d是小数点后的个数，m的最大值是65,d最大值是30，如果小数部分超过了限制的长度，会进行四舍五入
- **char**   
  固定长度字符串类型，最大255个字符
- **varchar**   
  可变长度字符串类型，最大21000个字符
- **text**   
  文本类型，最大65535个字符，使用的时候不需要指定字符串大小
- **datetime**   
  时间类型，保存的格式是年月日时分秒，存储时间与时区无关
- **date**   
  时间类型，保存的格式是年月日，存储时间与时区无关
- **timestamp**   
  时间戳类型，保存的是从1970-01-01 00:00:00到现在的时间戳，当将TIMESTAMP值插入到表中时，MySQL会将其从连接的时区转换为UTC后进行存储。当查询TIMESTAMP值时，MySQL会将UTC值转换回连接的时区进行显示，存储时间与时区有关，最大范围支持到2038-01-19 03:14:07

## 环境查看

使用 MySQL 时，应该先看当前环境

```text
SHOW DATABASES;           查看所有数据库
USE 数据库名;              使用数据库
SELECT DATABASE();        查看当前使用的数据库
SHOW TABLES;              查看当前数据库内所有表
DESC 表名;                 查看表结构

数据库在使用时一般以库为单位，操作库里的表，所以需要使用 use
```

命令行里还有一个显示方式：

```text
SHOW CREATE TABLE mytable\G;
\G 不是 SQL 语法本身，而是 MySQL 客户端的纵向显示方式，一般用于字段多的时候
```

## 管理结构

库

```text
CREATE DATABASE IF NOT EXISTS db_test;    创建库db_test
DROP DATABASE IF EXISTS db_test;          删除库db_test
```

表

```text
CREATE TABLE IF NOT EXISTS 表名 (                             创建表
    字段名 数据类型 约束,
    字段名 数据类型 约束
);
DROP TABLE IF EXISTS student;                    删除表

ALTER TABLE stu ADD address VARCHAR(50);         添加字段
ALTER TABLE stu MODIFY address CHAR(50);         修改字段类型
ALTER TABLE stu CHANGE address addr VARCHAR(50); 修改字段名和字段类型
ALTER TABLE stu DROP addr;                        删除字段
ALTER TABLE student RENAME TO stu;                修改表名
            MODIFY：只改字段定义，不改字段名
            CHANGE：字段名和字段定义一起改
```

### 约束

约束是表结构里很关键的一部分，把规则放到数据库层面可以避免错误数据被写进去

**默认约束：`DEFAULT`**，未指定值时自动使用默认值

```text
CREATE TABLE test (
    id INT PRIMARY KEY AUTO_INCREMENT,
    bonus INT DEFAULT 0
);
```

建表后添加默认值：

```text
ALTER TABLE test MODIFY bonus INT DEFAULT 0;
```

删除默认值本质是重新定义字段：

```text
ALTER TABLE test MODIFY bonus INT;
```

**非空约束：`NOT NULL`**，字段必须有值

```text
CREATE TABLE test (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL
);
```

建表后添加：

```text
ALTER TABLE test MODIFY name VARCHAR(50) NOT NULL;
```

删除非空约束本质也是重新修改字段定义：

```text
ALTER TABLE test MODIFY name VARCHAR(50);
```

**唯一约束：`UNIQUE`**，字段不能重复

```text
CREATE TABLE test (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE
);
```

给约束命名：

```text
CREATE TABLE test (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    CONSTRAINT uk_name UNIQUE(name)
);
```

建表后添加：

```text
ALTER TABLE test ADD CONSTRAINT uk_name UNIQUE(name);
```

删除唯一约束时用的是 `DROP INDEX`：

```text
ALTER TABLE test DROP INDEX uk_name;
```

**MySQL 里唯一约束底层依赖唯一索引，所以删除是 `DROP INDEX`**

**主键约束：`PRIMARY KEY`**，主键是一行数据的唯一标识，特点是：非空、唯一、一张表只能有一个主键

```text
CREATE TABLE test (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50)
);
```

或者：

```text
CREATE TABLE test (
    id INT,
    name VARCHAR(50),
    CONSTRAINT uk_name PRIMARY KEY(id)
);
```

建表后添加：

```text
ALTER TABLE test ADD PRIMARY KEY(id);
```

删除主键：

```text
ALTER TABLE test DROP PRIMARY KEY;
```

主键通常使用自增 ID

**外键约束：`FOREIGN KEY`**，外键用于建立表和表之间的数据库层面的关系

比如员工属于部门，员工表里的部门 `id` 指向部门表的 `id`

```text
CREATE TABLE dept(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20),
    addr VARCHAR(20)
);

CREATE TABLE emp(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20),
    age INT,
    dep_id INT,
    CONSTRAINT fk_emp_dept FOREIGN KEY(dep_id) REFERENCES dept(id)
);
```

建表后添加外键：

```text
ALTER TABLE test ADD CONSTRAINT uk_name FOREIGN KEY(id) REFERENCES test2(id);
```

删除外键：

```text
ALTER TABLE test DROP FOREIGN KEY uk_name;
```

外键的意义是：如果部门表里 `id=1` 的部门还有员工引用，那么直接删除这个部门会失败。它保证的是引用完整性。

**外键字段和被引用字段的数据类型要一致  
被引用字段通常必须是主键或唯一键  
两张表要使用支持外键的存储引擎，比如 InnoDB  
删除外键用 DROP FOREIGN KEY，不是 DROP INDEX**
