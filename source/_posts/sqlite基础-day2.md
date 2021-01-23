---
title: SQLite基础—Day2
tags:
  - Database
  - SQL
  - SQLite
id: '142'
categories:
  - - Sqlite
date: 2020-07-26 11:20:00
---

> SQLite 是一个软件库，实现了自给自足的、无服务器的、零配置的、事务性的 SQL 数据库引擎。SQLite 是在世界上最广泛部署的 SQL 数据库引擎。
> 
> 本篇是该系列教程的第二篇，新来的朋友可以在我的博客搜索第一篇，从头开始看

上篇介绍了Sqlite的基本知识，今天要写的部分是：表的操作、运算符以及表达式

## 数据表

### 创建数据表

创建语句格式：

```
Create Table table_name(
    column1 datatype primary key,
    column2 datatype not null,
    column3 datatype not null
);
```

我们来创建如下的简单数据表：

`CREATE TABLE` 是告诉数据库系统创建一个新表的关键字。`CREATE TABLE` 语句后跟着表的唯一的名称或标识。您也可以选择指定带有 `table_name` 的 `database_name`。

ID🔑

Name

Age

int

text

int

```
Create Table info(
     id int primary key,
     name text not null,
     age int not null
 );
```

可以用`.tables`查看当前数据库的所有表

![](http://img.varsion.cn/blog-img/2020/07/image-20191223160439789-1.png)

### Schema信息

`.schema`可以查看某个表的完整信息:

![](http://img.varsion.cn/blog-img/2020/07/image-20191223160611909-1.png)

因为所有的**点命令**只在 SQLite 提示符中可用，所以当您进行带有 SQLite 的编程时，您要使用下面的带有 **`sqlite_master`** 表的 `SELECT` 语句来列出所有在数据库中创建的表：

```
select tbl_name from sqlite_master where type = 'table';
```

假设在 testDB.db 中已经存在唯一的 info 表，则将产生以下结果：

![](http://img.varsion.cn/blog-img/2020/07/image-20191223170607817-1.png)

您可以列出关于表的完整信息，如下所示：

```
select sql from sqlite_master where type = 'table' and tbl_name = 'info';
```

![image-20191223171249482](http://img.varsion.cn/blog-img/2020/07/image-20191223171249482-1.png)

### 删除表

```
Drop table table_name;
```

也可以

```
Drop table databaseName.tableName;
```

删除指定数据库的表

![image-20191223162306749](http://img.varsion.cn/blog-img/2020/07/image-20191223162306749-1.png)

**同时，无法删除关联数据库的表**

* * *

## 运算符

运算符是一个保留字或字符，主要用于 SQLite 语句的 WHERE 子句中执行操作，如比较和算术运算。

### 算数运算符

sqlite一共有这些算数运算符 `＋,－,*,/,％`

![](http://img.varsion.cn/blog-img/2020/07/image-20191223181829939-1.png)

### 比较运算符

运算符

描述

实例

==

检查两个操作数的值是否相等，如果相等则条件为真。

(a == b) 不为真。

=

检查两个操作数的值是否相等，如果相等则条件为真。

(a = b) 不为真。

!=

检查两个操作数的值是否相等，如果不相等则条件为真。

(a != b) 为真。

<>

检查两个操作数的值是否相等，如果不相等则条件为真。

(a <> b) 为真。

>

检查左操作数的值是否大于右操作数的值，如果是则条件为真。

(a > b) 不为真。

<

检查左操作数的值是否小于右操作数的值，如果是则条件为真。

(a < b) 为真。

>=

检查左操作数的值是否大于等于右操作数的值，如果是则条件为真。

(a >= b) 不为真。

<=

检查左操作数的值是否小于等于右操作数的值，如果是则条件为真。

(a <= b) 为真。

!<

检查左操作数的值是否不小于右操作数的值，如果是则条件为真。

(a !< b) 为假。

!>

检查左操作数的值是否不大于右操作数的值，如果是则条件为真。

(a !> b) 为真。

假设当前表中有如下记录

![](http://img.varsion.cn/blog-img/2020/07/13MFLZH75GSQLTIEJ.png)

```
select * from info where name = '充鸭';
//从info表中检索name 为 充鸭 的记录
 select * from info where name = '易哒';
//从info表中检索name 为  易哒 的记录 
```

![](http://img.varsion.cn/blog-img/2020/07/image-1.png)

```
select * from info where id > 0;
//查询id大于0的
```

![](http://img.varsion.cn/blog-img/2020/07/image-2.png)

* * *

## 表达式

表达式是一个或多个值、运算符和计算值的SQL函数的组合。

### 布尔表达式

SQLite 的布尔表达式在匹配单个值的基础上获取数据。语法如下：

```
select column1,column2  
from tableName  
where [condition];
```

布尔表达式就是 `where`语句的判断条件，布尔表达式的值就是`ture , false`

### 数值表达式

```
select numExpression 
as 
name 
[from tableName where condition]
```

numExpression:是数学表达式 或 公式

name：作为name输出

![](http://img.varsion.cn/blog-img/2020/07/image-3.png)

常用内置函数：

avg()、sum()、count()，常用于数据计算

![](http://img.varsion.cn/blog-img/2020/07/image-4.png)

* * *