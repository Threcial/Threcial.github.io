---
title: "MySQL 常用函数、触发器、索引"
date: 2026-06-06 20:40:56
permalink: "数据库/mysql-常用函数、触发器、索引/"
updated: 2026-06-06 20:40:56
categories:
  - "数据库"
tags:
  - "mysql"
  - "函数"
  - "索引"
  - "触发器"
description: "触发器：在 insert/update/delete 前后自动执行动作 索引：减少扫描范围，提高查询效率 常用函数 聚合函数 COUNT(col) 统计查询结果的行数 MIN(col) 查询指定列的最"
cover: /img/covers/cover-003.jpg
top_img: false
---

**触发器**：在 `insert/update/delete` 前后自动执行动作  
**索引**：减少扫描范围，提高查询效率

## 常用函数

**聚合函数**

```text
COUNT(col)   统计查询结果的行数
MIN(col)   查询指定列的最小值
MAX(col)   查询指定列的最大值
SUM(col)   求和，返回指定列的总和
AVG(col)   求平均值，返回指定列数据的平均值
```

**数值型函数**

数值型函数主要是对数值型数据进行处理

```text
ABS(x)                 返回x的绝对值
BIN(x)                返回x的二进制
CEILING(x)            返回大于x的最小整数值
EXP(x)                返回值e（自然对数的底）的x次方
FLOOR(x)                返回小于x的最大整数值
GREATEST(x1,x2,...,xn)   返回集合中最大的值
LEAST(x1,x2,...,xn)   返回集合中最小的值
LN(x)                 返回x的自然对数
LOG(x,y)               返回x的以y为底的对数
MOD(x,y)               返回x/y的模（余数）
PI()                  返回pi的值（圆周率）
RAND()                返回０到１内的随机值,可以通过提供一个参数(种子)使RAND()随机数生成器生成一个指定的值
ROUND(x,y)            返回参数x的四舍五入的有y位小数的值
TRUNCATE(x,y)         返回数字x截短为y位小数的结果
```

#### 字符串函数

字符串函数可以对字符串类型数据进行处理

```text
LENGTH(s)              计算字符串长度函数，返回字符串的字节长度
CONCAT(s1,s2...,sn)   合并字符串函数，返回结果为连接参数产生的字符串，参数可以是一个或多个
INSERT(str,x,y,instr)   将字符串str从第x位置开始，y个字符长的子串替换为字符串instr，返回结果
LOWER(str)              将字符串中的字母转换为小写
UPPER(str)             将字符串中的字母转换为大写
LEFT(str,x)             返回字符串str中最左边的x个字符
RIGHT(str,x)             返回字符串str中最右边的x个字符
TRIM(str)               删除字符串左右两侧的空格
REPLACE               字符串替换函数，返回替换后的新字符串
SUBSTRING              截取字符串，返回从指定位置开始的指定长度的字符换
REVERSE(str)           返回颠倒字符串str的结果
```

**日期和时间函数**

```text
CURDATE             返回当前系统的日期值
CURTIME            返回当前系统的时间值
NOW                返回当前系统的日期和时间值
UNIX_TIMESTAMP   获取UNIX时间戳函数，返回一个以 UNIX 时间戳为基础的无符号整数
```

**加密函数**

加密函数主要用于对字符串进行加密

```text
ENCRYPT(str,salt)   使用UNIXcrypt()函数，用关键词salt(一个可以惟一确定口令的字符串，就像钥匙一样)加密字符串str
MD5()              计算字符串str的MD5校验和
```

## 触发器

触发器是和表绑定的对象，在 `INSERT`、`UPDATE`、`DELETE` 前后自动执行一段 SQL

适合用做数据校验、日志记录、审计记录等。MySQL 触发器只支持行级触发，不支持语句级触发。

基本语法：

```text
CREATE TRIGGER trigger_name
BEFORE | AFTER INSERT | UPDATE | DELETE
ON tb_name
FOR EACH ROW
BEGIN
    trigger_statement;
END;
```

操作触发器：

```text
SHOW TRIGGERS;
DROP TRIGGER IF EXISTS trigger_name;
```

触发器里用 `NEW` 和 `OLD` 引用变化前后的数据

```text
INSERT：只有 NEW，表示即将插入或刚插入的新记录
DELETE：只有 OLD，表示即将删除或刚删除的旧记录
UPDATE：既有 OLD，也有 NEW，OLD 是旧值，NEW 是新值
```

### 索引：减少扫描范围

索引的核心作用是减少 MySQL 需要扫描的数据量

没有索引时，查一条数据可能要从头扫到尾；有合适索引时，可以沿着索引结构快速定位范围

但索引并不是越多越好，因为索引也要占用额外磁盘空间；INSERT、UPDATE、DELETE 时索引也要维护；重复值很多的列建索引效果差；小表很多时候全表扫描更快

**索引是为查询服务的，但会拖慢写入和更新**

### 常见索引

```text
ALTER TABLE user ADD INDEX idx_name(name);               普通索引
ALTER TABLE user ADD UNIQUE uk_email(email);            唯一索引
CREATE INDEX idx_name_age ON user(name, age);            联合索引
CREATE INDEX idx_title_prefix ON article(title(20));       短索引
ALTER TABLE user DROP INDEX idx_name;                      删除索引
SHOW INDEX FROM user;                                     查看表索引
```

### 适合建索引的列

经常作为搜索条件；经常用于连接；经常用于范围查询；经常用于排序；经常用于分组；唯一性要求高的字段，适合唯一索引

### 不适合建索引的列

不常出现在查询条件里的列，不适合建索引

重复值太多的列不适合建索引。比如性别字段只有男、女，如果单独建索引，过滤效果通常很差

大文本字段不适合直接建普通索引

写入特别频繁、查询很少的列，也不适合加索引

**查询中很少使用的列、重复值多的列、text/image/bit 这类数据类型、修改性能要求远高于检索性能的列，不应该随便加索引**

### 联合索引之最左前缀

最左前缀原则：联合索引从最左列开始连续匹配，中间断了，后面的列就难以继续按索引顺序使用
