---
title: "MySQL 事务、视图、存储过程"
date: 2026-06-06 20:14:32
permalink: "数据库/mysql-事务、视图、存储过程/"
updated: 2026-06-06 20:14:32
categories:
  - "数据库"
tags:
  - "mysql"
  - "事务"
  - "存储过程"
  - "视图"
description: "事务：保证一组数据修改要么都成功，要么都失败 视图：把复杂查询封装成一张虚拟表 存储过程/函数：把 SQL 逻辑封装在数据库里 事务：把一组修改当成一个整体 事务最容易理解的场景就是转账，比如李四给张"
cover: /img/covers/cover-049.jpg
top_img: false
---

**事务：**保证一组数据修改要么都成功，要么都失败 **视图**：把复杂查询封装成一张虚拟表  
**存储过程/函数**：把 SQL 逻辑封装在数据库里

## 事务：把一组修改当成一个整体

事务最容易理解的场景就是转账，比如李四给张三转 500 元，表面上是两条 `UPDATE`：

```text
UPDATE account SET money = money - 500 WHERE name = '李四';
UPDATE account SET money = money + 500 WHERE name = '张三';
```

如果第一条执行成功，第二条还没执行时程序崩了，就会出现李四少了 500，张三没收到 500 的情况

事务要解决的就是这种“中间状态不能留下来”的问题

不加事务时，第二步出错会留下错误数据，加事务后，异常时可以回滚

事务写法：

```text
BEGIN;
UPDATE account SET money = money - 500 WHERE name = '李四';
UPDATE account SET money = money + 500WHERE name = '张三';
COMMIT;
```

如果发现了异常就 `ROLLBACK;`

常见语法：

```text
START TRANSACTION;   或者  BEGIN;              开启一个事务
COMMIT;                                         提交事务
ROLLBACK;                                       回滚
```

**`COMMIT` 之后就不能再 `ROLLBACK`，回滚只能撤销还没有提交的事务**

MySQL 默认自动提交每句 SQL 语句

查看自动提交状态：

```text
SELECT @@autocommit;

1：自动提交
0：手动提交
```

事务的四个特征 **ACID**

```text
Atomicity      原子性：事务不可再分，要么全部成功，要么全部失败
Consistency    一致性：事务执行前后，数据要保持合理状态
Isolation      隔离性：多个事务之间互相影响的程度
Durability     持久性：事务提交后，数据修改要能持久保存
```

## 视图：保存查询逻辑

视图可以理解为一张虚拟表，它本身不保存数据，保存的是一条查询语句。查询视图时，MySQL 会根据视图定义去底层表里动态取数据

比如有一张学生表，创建一个只看部分字段的视图：

```text
CREATE VIEW student_simple AS SELECT id, name FROM student;
```
```text
SELECT * FROM student_simple;
看起来像查表，实际是查这个视图背后的 SELECT id, name FROM student
```

### 视图的作用

第一，简化复杂查询。多表连接、过滤、计算字段可以封装成视图，以后直接查视图

第二，控制访问权限。比如底层表有工资、身份证、手机号，但只想让某个用户看姓名和部门，就可以只授权访问视图

第三，屏蔽部分表结构变化。如果底层表增加了视图没用到的字段，访问视图的人一般不受影响

### 操作视图

语法：

```text
CREATE OR REPLACE VIEW 视图名 AS SELECT语句 WITH CASCADED CHECK OPTION;
```

删除视图：

```text
DROP VIEW IF EXISTS 视图名;
```

## 存储过程：把 SQL 逻辑封装在数据库里

存储过程就是保存在数据库中的一段 SQL 语句集合。它的意义是把一组重复使用的 SQL 逻辑封装起来，需要时 `CALL` 一下

一个简单的存储过程：

```text
DELIMITER $$

CREATE PROCEDURE p1()
BEGIN
    SELECT COUNT(*) FROM student;
END$$

DELIMITER ;
```

调用：

```text
CALL p1();
```

**因为存储过程内部有很多分号，如果仍然用 `;` 作为结束符，MySQL 命令行会提前认为语句结束，所以要先临时把结束符改成 `$$`，过程写完后再改回来**

删除：

```text
DROP PROCEDURE IF EXISTS p1;
```

### 变量：系统变量、用户变量、局部变量

系统变量是 MySQL 服务器层面的变量，分全局和会话

查看：

```text
SHOW GLOBAL VARIABLES;
SHOW SESSION VARIABLES;
SHOW VARIABLES LIKE 'innodb%';
ELECT @@global.max_connections;
SELECT @@session.autocommit;
```

设置：

```text
SET GLOBAL max_connections = 500;
SET SESSION autocommit = 0;
```

用户变量是当前连接里的变量，不用提前声明，前面加 `@`：

```text
SET @num = 100;
SELECT @num;
```

也可以把查询结果放进去：

```text
SELECT COUNT(*) INTO @num FROM student;
SELECT @num;
```

局部变量只能在 `BEGIN ... END` 块里用，需要 `DECLARE` 声明：

```text
DECLARE total INT DEFAULT 0;
SET total = 100;
SELECT total;
```

### 参数：`IN`、`OUT`、`INOUT`

存储过程参数有三种：

```text
IN：输入参数，调用时传入，默认类型
OUT：输出参数，调用后作为返回值
INOUT：既能传入，也能被过程修改后带出
```

比如判断成绩：

```text
DELIMITER $$
CREATE PROCEDURE p_score(IN score INT, OUT result VARCHAR(10))
BEGIN
    IF score >= 85 THEN
        SET result = '优秀';
    ELSEIF score >= 60 THEN
        SET result = '及格';
    ELSE
        SET result = '不及格';
    END IF;
END$$
DELIMITER ;
```

调用：

```text
CALL p_score(58, @result);
SELECT @result;
```

### 控制语句：`IF`、循环

`IF`：

```text
IF 条件 THEN
    -- SQL
ELSEIF 条件 THEN
    -- SQL
ELSE
    -- SQL
END IF;
```

`WHILE` 是先判断再执行：

```text
DELIMITER $$

CREATE PROCEDURE p_sum(IN n INT)
BEGIN
    DECLARE total INT DEFAULT 0;

    WHILE n > 0 DO
        SET total = total + n;
        SET n = n - 1;
    END WHILE;

    SELECT total;
END$$

DELIMITER ;
```

`REPEAT` 是先执行一次，再判断退出条件：

```text
DELIMITER $$
CREATE PROCEDURE p_repeat(IN n INT)
BEGIN
    DECLARE total INT DEFAULT 0;
    REPEAT
        SET total = total + n;
        SET n = n - 1;
    UNTIL n <= 0
    END REPEAT;
    SELECT total;
END$$
DELIMITER ;
```

`LOOP` 本身没有条件，需要配合 `LEAVE` 退出，或者用 `ITERATE` 跳过当前循环剩余内容进入下一轮：

```text
DELIMITER $$
CREATE PROCEDURE p_loop(IN n INT)
BEGIN
    DECLARE total INT DEFAULT 0;
    sum_loop: LOOP
        IF n <= 0 THEN
            LEAVE sum_loop;
        END IF;
        IF n % 2 = 1 THEN
            SET n = n - 1;
            ITERATE sum_loop;
        END IF;
        SET total = total + n;
        SET n = n - 1;
    END LOOP sum_loop;
    SELECT total;
END$$
DELIMITER ;
```

### 游标：逐行处理查询结果

游标用来保存查询结果集，然后一行一行取出来处理

**使用游标时，要先定义变量，再定义游标**，游标和 handler 的声明顺序不对会报错

基本语法：

```text
DECLARE 游标名 CURSOR FOR 查询语句;
OPEN 游标名;
FETCH 游标名 INTO 变量1, 变量2;
CLOSE 游标名;
```

**注意：如果 `FETCH` 到最后没有数据时，会触发 `NOT FOUND`**

所以应该配合条件处理程序：

```text
DELIMITER $$
CREATE PROCEDURE p_cursor(IN uage INT)
BEGIN
    DECLARE uname VARCHAR(100);
    DECLARE uid INT;
    DECLARE done INT DEFAULT 0;
    DECLARE u_cursor CURSOR FOR
        SELECT ename, id
        FROM emp
        WHERE id <= uage;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
    DROP TABLE IF EXISTS tb_user_pro;
    CREATE TABLE IF NOT EXISTS tb_user_pro(
        id INT PRIMARY KEY AUTO_INCREMENT,
        name VARCHAR(100),
        uid INT
    );
    OPEN u_cursor;
    read_loop: LOOP
        FETCH u_cursor INTO uname, uid;
        IF done = 1 THEN
            LEAVE read_loop;
        END IF;
        INSERT INTO tb_user_pro(name, uid)
        VALUES (uname, uid);
    END LOOP read_loop;
    CLOSE u_cursor;
END$$
DELIMITER ;
```

### 条件处理程序 Handler

条件处理程序用于定义遇到错误或特定状态时怎么处理，比如使用游标的时候：

```text
DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
```

### 存储函数

存储函数可以理解成有返回值的存储过程。它必须通过 `RETURNS` 指定返回类型，用 `RETURN` 返回值

```text
DELIMITER $$

CREATE FUNCTION fun1(n INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE total INT DEFAULT 0;

    WHILE n > 0 DO
        SET total = total + n;
        SET n = n - 1;
    END WHILE;

    RETURN total;
END$$

DELIMITER ;
```

调用：

```text
SELECT fun1(100);
```

存储函数的参数只能是输入类型，本质上通过返回值输出结果
