---
title: 给自己的SQL备忘
comments: true
categories: 技术&技巧
tags: SQL
abbrlink: f95479c9
date: 2018-03-21 21:45:47
---

> 填坑系列！！

# SQL简介

SQL （Structured Query Language） 结构化查询语言，是一种大小写不敏感的标准语言。

SQL 包含 `DML （数据操作语言）` 和 `DDL（数据定义语言）`。

* DML有：SELECT、UPDATE、DELETE、INSERT INTO等
* DDL有：CREATE DATABASE、ALTER DATABASE、CREATE TABLE、CREATE INDEX等

在网站中，通常包括：
* RDBMS（关系型数据库管理系统）数据库程序（SQL Server、mySQL等）
* 服务器端脚本语言（PHP、ASP.NET、Java EE等）
* 前端语言（HTML/CSS）

<!--more-->

---

# Ubuntu 18.04 安装 Mysql

## 卸载和安装

干净卸载后安装

```
sudo apt-get --purge remove mysql-server mysql-common mysql-client
sudo apt-get install mysql-server mysql-common mysql-client
```

设置 root 密码

```
mysqladmin -u root password your-new-password
sudo /etc/init.d/mysql restart
```

## 安装完毕后发现没有登录权限

```
ERROR 1045: Access denied for user: 'root@localhost' (Using
password: YES)
```

首先无密码登录

```
sudo mysql -u root
```


然后查看当前用户，删除后重新创建 ROOT 账户，并授权

```sql
mysql> SELECT User,Host FROM mysql.user;

mysql> DROP USER 'root'@'localhost';

mysql> CREATE USER 'root'@'%' IDENTIFIED BY '123456';

mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

mysql> FLUSH PRIVILEGES;
```
---

# 修改ROOT密码

## 方法1： 用SET PASSWORD命令

```sql
MySQL -u root
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
```

## 方法2：用mysqladmin

```sql
mysqladmin -u root password "newpass"
```

如果root已经设置过密码，采用如下方法

```sql
mysqladmin -u root password oldpass "newpass"
```

---

# 如何导入数据库文件

假设我们已经有了一个数据库脚本， projectDB.sql，如何导入到MySQL？

方法一：直接通过 mysql 命令运行

```sql
mysql -u root -p123456 --port 3306 < /home/jerrysheh/projectDB.sql
```

方法二：登录 mysql 后使用 source 命令

```sql
mysql>create database <database-name>;
mysql>use <database-name>;
mysql>source /home/jerrysheh/projectDB.sql;
```

---

# 单表操作

## SELECT

语法：从表里选择列
```SQL
SELECT column FROM table

SELECT LastName,Address FROM Persons
```

---

语法：从表里选择列（不出现重复值）
```SQL
SELECT DISTINCT column FROM table
```

<!-- more -->

---

## WHERE

语法：从表里中选择 city 列等于 Beijing 的所有行
```SQL
SELECT * FROM Persons WHERE city = 'Beijing'
```

> 这里是使用**单引号**，但是大部分数据库也接受双引号。如果是数值类型，不要加引号！

---

## AND、OR

语法：从表里中选择 FirstName 是'Thomas' 并且 LastName 是 'Carter' 的行
```SQL
SELECT * FROM Persons WHERE FirstName = 'Thomas' AND LastName='Carter'
```

---

## ORDER BY 升序排列

* ASC 升序（小→大）
* DESC 降序（大→小）

优先以Company升序排列，然后以OrderNumber升序排列
```SQL
SELECT Company, OrderNumber FROM Orders ORDER BY Company,OrderNumber
```
假设这是一个记录公司订购了多少数量的物品的表，Google公司订购了两次，先以 A - G - I 升序排列，然后在相同值Google的两次订购中以数字升序排列

Company|OrderNumber
---|---
**A** pple|4698
**G** oogle|**2356**
**G** oogle|**6953**
**I** BM|3552

---


## INSERT INTO

### 顺序插入

往 Person 表的1、2、3、4列依次填入值
```SQL
INSERT INTO Persons VALUES('Jerry',"Sheh","Guangdong","China")
```

那么在 Persons 表的最后一行会新增如下

Firstname|LastName|Province|Country
---|---|---|---
Jerry|Sheh|Guangdong|China

### 指定列插入


```SQL
INSERT INTO Persons(LastName, Age) VALUES ('Wilson', 18)
```

---

## UPDATE

修改Address列和City列
```SQL
UPDATE Persons SET Address='Xueyuan Road',City='Zhongshan' WHERE LastName='Wilson'
```

---

## DELETE

删除某行
```SQL
DELETE FROM Persons WHERE LastName='Wilson'
```

删除所有行（表并没有被删除，结构、属性、索引都是完整的）
```SQL
DELETE * FROM table
```

---

## TOP

```SQL
# 显示前2条
SELECT TOP 2 * FROM Persons

# 显示前50%
SELECT TOP 50 PERCENT * FROM Persons
```

* SQL Server可用，其他SQL待验证

---

## LIKE

在 WHERE 子句中搜索列中的指定模式

```SQL
SELECT * FROM Persons
WHERE city LIKE '%N'
```

同理，`NOT LIKE`是不包含

* `%`定义了通配符， `N%`表示以N开头， `%g`表示以g结尾，`%lon%`表示包含lon

### SQL 通配符

* **`%`** ：代替一个或多个字符
* **`_`** ：代替一个字符
* **[charlist]** ：字符列中的任何单一字符
* **[^charlist]** 或 **[!charlist]** ：非字符列中的任何单一字符

选取 c 开头，然后一个任意字符，然后 r，然后任意字符，最后 er
```SQL
SELECT * FROM Persons Where LastName LIKE `c_r_er`
```

选取以 A 或者 L 或者 N 开头
```SQL
SELECT * FROM Persons Where LastName LIKE `[ALN]%`
```
---

## IN

用于在 WHERE 中 规定多个值
```sql
SELECT * FROM Persons Where LastName IN ('Adams','Carter')
```

---

## BETWEEN

选取介于两个值之间的数据范围 （数值、文本、日期）
```sql
SELECT * FROM Persons Where LastName BETWEEN 'Adams' AND 'Carter'
```

* `NOT BETWEEN`，不在某范围内

* 不同数据库 BETWEEN...AND... 包括的范围可能不一样

---

# 多表操作

## AS

假设有两个表， `Persons`表 和 `Product_Orders`表， 给他们指定别名为`p`和`po`，列出`John Adams`的所有订单

```SQL
SELECT Po.OrderID, P.LastName, p.FirstName
FROM Persons AS p, Product_Orders AS po
WHERE p.LastName = 'Adams' AND p.FirstName = 'John'
```

---

## JOIN

将Persons表和Orders表的  Id_p 列关联起来

```sql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
INNER JOIN Orders
ON Person.Id_p = Orders.Id_p
ORDER BY Person.LastName
```

* **INNER JOIN**：内连接
* **JOIN**：如果表中有至少一个匹配，则返回这（些）行
* **LEFT JOIN**：即使右表没有匹配，也从左表返回所有行
* **RIGHT JOIN**：即使左表没有匹配，也从右表返回所有行
* **FULL JOIN** ：必须左右表都有匹配才返回行

---

## UNION

合并两个或多个 SELECT 语句的结果集

合并两个表的员工名字，如果名字一样，只出现一次
```SQL
SELECT E_Name FROM Employees_China
UNION
SELECT E_Name FROM Employees_USA
```

* 不要去重，可以用 `UNION ALL`

---

## SELECT INTO等

从一张表选择数据，然后插入到另一张表里（常用于创建表的备份附件，或者用于对记录进行存档）

```SQL
SELECT * INTO new_table FROM old_table
```

* 可结合 `WHERE` 或者 `JOIN` 使用

---

# SQL 常用数据类型

类型|释义
---|---
integer(size)|整数
int(size)|整数
smallint(size)|整数
tinyint(size)|整数
decimal(size,d)|小数，d=小数点右侧最大位数
numeric(size,d)|小数
char(size)|固定字符串
varchar(size)|可变长字符串
date(yyyymmdd)|日期

---

# 数据库操作

## CREATE DATABASE

创建数据库

```SQL
CREATE DATABASE mydb
```

- 在 MYSQL 中， 也可以用 `CREATE SCHEMA mydb`

摘自MYSQL 5.0官方文档：

> CREATE DATABASE creates a database with the given name.
To use this statement, you need the CREATE privilege for the database.
CREATE SCHEMA is a synonym for CREATE DATABASE as of MySQL 5.0.2.

## CREATE TABLE

创建表
```SQL
CREATE TABLE Persons
(
  Id_p int,
  LastName varchar(255),
  FirstName varchar(255),
  Address Varchar(255),
)
```

## 约束

### 属性约束

```SQL
CREATE TABLE Persons
(
  Id_p int NOT NULL,
  LastName varchar(255),
  ...
)
```

* **NOT NULL**: 不接受NULL值
* **UNIQUE**:为列或者列集合提供唯一性保证
* **PRIMARY KEY**：主键

### CHECK约束

```sql
CREATE TABLE Persons
(
  Id_p int NOT NULL CHECK (Id_p > 0),
  LastName varchar(255) NOT NULL,
  ...
)
```

### CHECK约束

```SQL
CREATE TABLE Persons
(
  ...
  CONSTRAINT CHK_Persons CHECK (Id_p > 0 AND City='Sandnes')
)
```

如果表已经存在，新增 CHECK 约束，用

```sql
ALTER table Persons
    ADD CHECK (ip_p > 0)
```

### DEFAULT约束

设置默认值

---

## CREATE INDEX 创建索引



```sql
CREATE UNIQUE INDEX index_name
ON table_name(column_name)
```

---

## DROP 撤销操作

## ALTER 在已有表 添加/修改/删除 列

```sql
ALTER TABLE table_name
ADD column_name datatype
```
