---
title: "基础SQL语句 下"
date: 2026-06-06 19:35:25
permalink: "数据库/基础sql语句-下/"
updated: 2026-06-06 19:35:25
categories:
  - "数据库"
tags:
  - "sql"
  - "查询语句"
description: "管理数据变化 数据变化主要是三件事：插入、修改、删除 插入数据：INSERT 指定字段插入： INSERT INTO stu(id, name) VALUES(1, '张三'); 批量插入： INSE"
cover: /img/covers/cover-056.jpg
top_img: false
---

## 管理数据变化

数据变化主要是三件事：插入、修改、删除

**插入数据：`INSERT`**

指定字段插入：

```text
INSERT INTO stu(id, name) VALUES(1, '张三');
```

批量插入：

```text
INSERT INTO stu(id, name, gender) VALUES
(2, '李四1', '男'),
(3, '李四2', '男'),
(4, '李四3', '男');
```

给所有字段插入时，可以省略字段列表：

```text
INSERT INTO stu VALUES
(2, '李四', '男', '1999-11-11', 88.88, 'lisi@example.com', '13888888888', 1);
```

但实际使用时，推荐保留字段列表，因为表结构有可能发生改变，如果省略字段列表，一旦字段顺序或字段数量变化，旧 SQL 语句将失效

**修改数据：`UPDATE`**

```text
UPDATE 表名
SET 字段1 = 值1,
    字段2 = 值2
WHERE 条件;
```

比如：

```text
UPDATE stu
SET sex = '女'
WHERE name = '张三';
```

一次改多个字段：

```text
UPDATE stu
SET birthday = '1999-12-12',
    score = 99.99
WHERE name = '张三';
```

**如果没有 `WHERE`，整张表都会被改，所以真正操作数据时，要先用 `SELECT` 确认影响范围**

**删除数据：`DELETE` 和 `TRUNCATE`**

删除指定记录：

```text
DELETE FROM stu WHERE name = '张三';
```

删除全表数据：

```text
DELETE FROM stu;
```

清空表也可以用：

```text
TRUNCATE TABLE stu;
```

清空大表时，`TRUNCATE` 更快，它的效果类似重置整张表，不能像普通 `DELETE WHERE` 那样精确控制范围

**删之前先查询，确定范围**

## 管理数据查询

一个完整的查询骨架可以记成：

```text
SELECT
    字段列表
FROM
   表名列表
WHERE
    条件列表
GROUP BY
    分组字段
HAVING
    分组后条件
ORDER BY
    排序字段
LIMIT
    分页限定;
```

### 基础查询

查询全部字段：

```text
SELECT * FROM stu;
```

查询指定字段：

```text
SELECT name, age FROM stu;
```

去重：

```text
SELECT DISTINCT address FROM stu;
```

别名：

```text
SELECT name, math AS 数学成绩, english AS 英语成绩 FROM stu;
SELECT name, math 数学成绩, english 英语成绩 FROM stu;        as可以省略
```

### 条件查询：`WHERE`

```text
SELECT * FROM stu WHERE age > 20;                      大于
SELECT * FROM stu WHERE age >= 20;                     大于等于
SELECT * FROM stu WHERE age > 20 AND age  18;                   不等于的两种写法
SELECT * FROM stu WHERE age IN (18, 20, 22);         多个取值
SELECT * FROM stu WHERE age NOT IN (55, 45, 57);     排除多个取值
```

**`NULL` 不能用 `=` 或 `!=` 判断：**

```text
SELECT * FROM stu WHERE english IS NULL;
SELECT * FROM stu WHERE english IS NOT NULL;
```

### 模糊查询：`LIKE`

通配符：

```text
_ ：单个任意字符
% ：任意多个字符
```

用法：

```text
SELECT * FROM stu WHERE name LIKE '马%';             马字开头
SELECT * FROM stu WHERE name LIKE '_花%';            二个字是花
SELECT * FROM stu WHERE name LIKE '%德%';            名字中包含德
```

### 排序：`ORDER BY`

```text
SELECT * FROM stu ORDER BY age;         按年龄升序，默认就是升序
SELECT * FROM stu ORDER BY age ASC;      完整写法
SELECT * FROM stu ORDER BY age DESC;      降序
```

多个排序条件：

```text
SELECT * FROM stu ORDER BY math DESC, english ASC;
```

多个排序条件的规则是：先按第一个字段排序，第一个字段相同，再按第二个字段排序

### 聚合函数：纵向计算

聚合函数是对一列数据做整体计算

```text
COUNT：统计数量
MAX：最大值
MIN：最小值
SUM：求和
AVG：平均值
```

示例：

```text
SELECT COUNT(*) FROM stu;
SELECT MAX(math) FROM stu;
SELECT MIN(math) FROM stu;
SELECT SUM(math) FROM stu;
SELECT AVG(math) FROM stu;
```

`NULL` 不会参与聚合运算

统计行数：

```text
SELECT COUNT(*) FROM stu;
SELECT COUNT(id) FROM stu;
```

### 分组查询：`GROUP BY` 和 `HAVING`

按性别分组，统计数学平均分：

```text
SELECT sex, AVG(math) FROM stu GROUP BY sex;
```

先过滤，再分组：

```text
SELECT sex, AVG(math), COUNT(*) FROM stu WHERE math > 70 GROUP BY sex;
```

分组后再过滤：

```text
SELECT sex, AVG(math), COUNT(*) FROM stu WHERE math > 70 GROUP BY sex HAVING COUNT(*) > 2;
```

**WHERE：分组前过滤，不能直接判断聚合函数  
HAVING：分组后过滤，可以判断聚合函数**

### 分页查询：`LIMIT`

基本格式：

```text
SELECT 字段列表 FROM 表名 LIMIT 起始索引, 查询条目数;
```

起始索引从 `0` 开始

查询前 3 条：

```text
SELECT * FROM stu LIMIT 0, 3;
```

从第 4 条开始，查 10 条：

```text
SELECT * FROM stu LIMIT 3, 10;
```

### 多表查询

两张表直接联查：

```text
SELECT * FROM emp, dept;
```

会得到笛卡尔积，比如 6 个员工、4 个部门，结果就是 24 行，其中大量是无意义组合

所以使用多表查询必须添加关联条件：

```text
SELECT * FROM emp, dept WHERE emp.dep_id = dept.id;
```

写多表查询的关键是：**表和表靠哪个字段关联？要保留匹配数据，还是保留某一边的全部数据？**

内连接：

```text
SELECT * FROM emp INNER JOIN dept ON emp.dep_id = dept.did;
```

只保留两边都匹配的数据

外连接：

```text
SELECT * FROM emp LEFT JOIN dept ON emp.dep_id = dept.did;
```

保留左表 emp 的全部数据，右表 dept 能匹配就显示，不能匹配就显示 NULL

```text
SELECT * FROM emp RIGHT JOIN dept ON emp.dep_id = dept.did;
```

保留右表 dept 的全部数据，左表 emp 能匹配就显示，不能匹配就显示 NULL

### 子查询：把查询结果当条件或临时表

子查询按结果形态分几种

单行单列：当普通值用

```text
SELECT * FROM emp WHERE salary = 
( SELECT salary FROM emp WHERE name = '猪八戒');
```

多行单列：配合 `IN` 或 `NOT IN`

```text
SELECT * FROM emp WHERE dep_id IN 
( SELECT id FROM dept WHERE dname = '财务部' OR dname = '市场部');
```

多行多列：当临时表用

```text
SELECT * FROM ( SELECT * FROM emp  WHERE join_date > '2011-11-11') t1, dept WHERE t1.dep_id = dept.did;
```

这里的 `t1` 是派生表别名，不能省略
