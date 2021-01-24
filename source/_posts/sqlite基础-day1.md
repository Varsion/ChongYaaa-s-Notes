---
title: SQLite基础—Day1
tags:
  - Database
  - SQL
  - SQLite
id: '133'
categories:
  - - Sqlite
abbrlink: cd182e89
date: 2020-07-25 11:28:00
---

> SQLite 是一个软件库，实现了自给自足的、无服务器的、零配置的、事务性的 SQL 数据库引擎。SQLite 是在世界上最广泛部署的 SQL 数据库引擎。

这里将记录我的SQLite的学习过程

SQLite的大多数操作都类似于MySQL或者说基本是一样的

熟悉使用MySQL的朋友应该很容易就会上手

## 数据类型

#### 数据类型

SQLite 数据类型是一个用来指定任何对象的数据类型的属性。SQLite 中的每一列，每个变量和表达式都有相关的数据类型。

您可以在创建表的同时使用这些数据类型。SQLite 使用一个更普遍的动态类型系统。在 SQLite 中，值的数据类型与值本身是相关的，而不是与它的容器相关。

###### 存储类

每个存储在 SQLite 数据库中的值都具有以下存储类之一：

存储类

描述

NULL

值是一个 NULL 值。

INTEGER

值是一个带符号的整数，根据值的大小存储在 1、2、3、4、6 或 8 字节中。

REAL

值是一个浮点值，存储为 8 字节的 IEEE 浮点数字。

TEXT

值是一个文本字符串，使用数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储。

BLOB

值是一个 blob 数据，完全根据它的输入存储。

SQLite 的存储类稍微比数据类型更普遍。INTEGER 存储类，例如，包含 6 种不同的不同长度的整数数据类型。

##### SQLite 亲和(Affinity)类型

SQLite支持列的亲和类型概念。任何列仍然可以存储任何类型的数据，当数据插入时，该字段的数据将会优先采用亲缘类型作为该值的存储方式。SQLite目前的版本支持以下五种亲缘类型：

亲和类型

描述

TEXT

数值型数据在被插入之前，需要先被转换为文本格式，之后再插入到目标字段中。

NUMERIC

当文本数据被插入到亲缘性为NUMERIC的字段中时，如果转换操作不会导致数据信息丢失以及完全可逆，那么SQLite就会将该文本数据转换为INTEGER或REAL类型的数据，如果转换失败，SQLite仍会以TEXT方式存储该数据。对于NULL或BLOB类型的新数据，SQLite将不做任何转换，直接以NULL或BLOB的方式存储该数据。需要额外说明的是，对于浮点格式的常量文本，如"30000.0"，如果该值可以转换为INTEGER同时又不会丢失数值信息，那么SQLite就会将其转换为INTEGER的存储方式。

INTEGER

对于亲缘类型为INTEGER的字段，其规则等同于NUMERIC，唯一差别是在执行CAST表达式时。

REAL

其规则基本等同于NUMERIC，唯一的差别是不会将"30000.0"这样的文本数据转换为INTEGER存储方式。

NONE

不做任何的转换，直接以该数据所属的数据类型进行存储。

##### SQLite 亲和类型(Affinity)及类型名称

下表列出了当创建 SQLite3 表时可使用的各种数据类型名称，同时也显示了相应的亲和类型：

![](http://img.varsion.cn/blog-img/2020/07/image-20191223104340340-535x1024.png)

##### Boolean 数据类型

SQLite 没有单独的 Boolean 存储类。相反，布尔值被存储为整数 0（false）和 1（true）。

##### Date 与 Time 数据类型

SQLite 没有一个单独的用于存储日期和/或时间的存储类，但 SQLite 能够把日期和时间存储为 TEXT、REAL 或 INTEGER 值。

存储类

日期格式

TEXT

格式为 "YYYY-MM-DD HH:MM:SS.SSS" 的日期。

REAL

从公元前 4714 年 11 月 24 日格林尼治时间的正午开始算起的天数。

INTEGER

从 1970-01-01 00:00:00 UTC 算起的秒数。

您可以以任何上述格式来存储日期和时间，并且可以使用内置的日期和时间函数来自由转换不同格式。

* * *

## 创建数据库

```
sqlite3 DatabaseName.db
```

![](http://img.varsion.cn/blog-img/2020/07/image-20191223104932956.png)

### .dump命令

转换整个 `testDB.db` 数据库的内容到 SQLite 的语句中，并将其转储到 ASCII 文本文件 `testDB.sql` 中。数据库名称和`.sql`文件名称可以不同,但是为了方便,应该保持相同

```
DatabaseName.db .dump > DatabaseName.sql
//将该db文件输出为.sql文件 (在cmd命令中)
```

![](http://img.varsion.cn/blog-img/2020/07/image-20191223105612674.png)

也可以通过该方式,从生成的`.sql`文件中恢复

```
$sqlite3 testDB.db < testDB.sql
```

* * *

## 附加/分离数据库

假设这样一种情况，当在同一时间有多个数据库可用，您想使用其中的任何一个。SQLite 的 `ATTACH DATABASE` 语句是用来选择一个特定的数据库，使用该命令后，所有的 SQLite 语句将在附加的数据库下执行。

SQLite 允许你用`attach`命令将多个数据库“附加”到当前连接上。当你附加了一个数据库时，它的所有内容在当前数据库文件的全局范围内都是可存取的。attach 的语法如下：

将一个数据库附加到当前数据库,并命名

将'DatabaseName'附加到当前数据库并用给他一个逻辑名'name'

```
# 规范
Attach Database 'fileName' as 'Name';
# 例子
Attach Database 'DatabaseName.db' as 'Name';
```

![](http://img.varsion.cn/blog-img/2020/07/image-20191223113111570.png)

现在,`monitroDB`数据库里有表`control`有部分数据,可以以该方式在`testDB`里查询`monitorDB`中的数据:

```
 select * from monitor.control;
```

![](http://img.varsion.cn/blog-img/2020/07/image-20191223131602049.png)

如果要查询主数据库中的表，需要指定逻辑名 main，因为当有数据库附加的时候，主数据库会自动赋名为 main

如果同一个数据库文件已经被附加上多个别名，`DETACH` 命令将只断开给定名称的连接，而其余的仍然有效。您无法分离 **main**或者**temp**数据库:

```
Detach Database 'Alias-Name';
```

如果数据库是在内存中或者是临时数据库，则该数据库将被摧毁，且内容将会丢失。

![](http://img.varsion.cn/blog-img/2020/07/image-20191223134149719.png)

* * *