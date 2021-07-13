---
title: Oracle 和 MySQL 语法的一些区别
categories: 
- 数据库
- SQL语法
tags: SQL
abbrlink: 65a27c74
date: 2019-03-03 20:41:48
---

团队最近正在计划将项目从 Oracle 数据库迁移到 MySQL，特此整理一份 Oracle 字段、函数、建表语句的差异。

<!-- more -->

# 字段差异

## 整数

在 Oracle 中，数字统一使用 NUMBER(m) ，NUMBER(6) 最大值为 999999 ，NUMBER(6,2) 最大值为 9999.99，如果不指定m值，直接写 NUMBER，则默认为浮点数，精度根据实际情况。

在 MySQL 中，整数分为 tinyint、smallint、mediumint、int 和 bigint 五种，区别如下：

| 整数类型 | 字节数  | 比特数   | 取值范围 |
| :------------- | :------------- |:------------- |:------------- |
| tinytint       | 1字节      | 8 bit (2^8) | -128 到 127
| smallint       | 2字节      | 16 bit (2^16)  | -32768 到 32767
| mediumint       | 3字节     | 24 bit (2^24)   | -8388608 到 8388607
| int       | 4字节           | 32 bit (2^32)   | -2147483648 到	2147483647
| bigint       | 8字节           |  64 bit (2^64)        | -9223372036854775808 到	9223372036854775807

> 在 MySQL 中，int(4) 表示 zerofill 为 4，实际上是可以插入大于 4 位的数值比如 12345，只不过，当不足 4 位的时候，在左边会填充 0 ，例如对于 int(4)， 插入 123，显示的是 0123， 对于 int(5)，插入 123，显示的是 00123。

## 小数

MySQL的小数不建议用 FLOAT 或 DOUBLE，会损失精度，推荐用 DECIMAL 。DECIMAL(6,2) 最大值为 9999.99

Oracle NUMBER(6,2) 最大值为 9999.99

## 时间

在 Oracle 中，DATE 精确到秒，获取当前时间用 sysdate

在 MySQL 中，DATETIME 精确到秒，而 DATE 只精确到天，获取当前时间用 CURRENT_TIMESTAMP 或 NOW() 或 sysdate()

虽然差异很小，但是 CURRENT_TIMESTAMP 和 NOW() 取的是本次查询开始的时间，而 sysdate() 取的是执行到 sysdate() 语句时动态的实时时间。

## 结果字符集

在 Oracle 中，用 ROWNUM 筛选结果集。但 ROWNUM 是一个伪列，即先有结果集，然后再筛选。在一个20行记录的表中，当我们想查询表的第11-20行记录，输入`select rownum,c1 from t1 where rownum > 10`，我们会得不到任何结果，因为 Oracle 会先筛选出第一条，发现 不满足 > 10，于是排除，将第二条记录放到 1 的位置，发现还是 不满足 > 10，以此类推，20条记录都被推到第一的位置，都不满足 > 10 的条件。简而言之，rownum 条件要包含到 1，否则永远查不到结果。

在 MySQL 中，用 LIMIT 筛选结果集， LIMIT(10)筛选前 10 条，LIMIT(30,10)，从第 31 条开始，取 10 条（即31-40行记录）。

## 字符串

在 Oracle 中，字符串有 VARCHAR2，且只可以用单引号包起字符串。

在 MySQL 中，只有 VARCHAR， MySQL里可以用双引号包起字符串。

## NULL和空字符

在 Oracle 中，只有 NULL 的概念，没有空字符。

在 MySQL 中，有 NULL，也有空字符。

## default

创建表结构时，Oracle 允许 default+函数

MySQL 不允许 default+函数（时间函数除外），因此 `default user()` 是会报错的，解决这一问题只能用触发器，但在开发规范里面触发器是禁止使用的，只能在应用层添加。

---

# 函数差异

## 连接字符串

在 Oracle 中，连接字符串用 `||`。 如  `'%'||'es'||'%'`，表示任意字符串连接 es 再连接任意字符串。

在 MySQL 中，连接字符串用 concat 函数。如 `concat('%', 'es','%')`。

## 子串函数

在 Oracle 中，子串用 `SUBSTR('abcd', 2, 2)`

在 MySQL 中，子串用 `substring('abcd',2, 2)`

注意：mysql的 start 从 1 开始，如果 start 为 0 ，那么返回空

Oracle
```sql
substr('shit', 0, 1) -- 返回 s
substr('shit', 1, 1) -- 返回 s
```

MySQL
```sql
substring('shit',0, 1) -- 返回空串
substring('shit',1, 1) -- 返回 s
```

## 时间转char

在 Oracle 中，用 `to_char(time, format)`：

```sql
SELECT to_char(sysdate, 'yyyy-mm-dd') from dual;
SELECT to_char(sysdate, 'hh24-mi-ss') from dual;
```

在 MySQL 中，用 `date_format(time, format)`（可表示年月日时分秒多种格式） 或 `time_format(time, format)`（只可表示时分秒）

```sql
SELECT date_format(now(), '%Y-%m-%d %H:%i:%S');
SELECT time_format(now(), '%H:%i:%S');
```

## char 转时间

在 Oracle 中，用 `to_date(str,format)`：

```sql
SELECT to_date(`2019-3-6`, 'yyyy-mm-dd')
```

在 MySQL 中， 用 `str_to_date(str,format)`：

```sql
SELECT str_to_date('2019-3-6 16:53:58', '%Y-%m-%d %H:%i:%S')
```

#### YYYYMM 注意点

Oracle
```sql
to_date('201907', 'YYYYMM') -- 返回 2019-7-1
```

MySQL`
```sql
str_to_date('201907', '%Y%m') -- 返回空串
str_to_date('20190701', '%Y%m%d') -- 返回 2019-07-01
```

所以，遇到 `201907` 这样的 char 要转成日期，只能用 `concat('201907', '01')`

## 时间截取

在 Oracle 中，sysdate在当天的零时零分零秒等于 `trunc(sysdate)` ， trunc 函数用来截取时间。

```sql
# 当年的第一天
trunc(sysdate, 'yy')

# 当月的第一天
trunc(sysdate, 'mm')

# 本周的第一天（周日）
trunc(sysdate, 'd')

# 当天（零时零分零秒）
trunc(sysdate)
```

在 MySQL 中，只能用 DATE_FORMAT 函数来格式化时间。

```sql
# 自定义日期格式
SELECT date_format(now(), '%Y-%m-%d');

# 当月的第一天
SELECT date_format(now(), '%Y-%m-01');

# 下个月的第一天
SELECT date_add(date_format(now(), '%Y-%m-01'), INTERVAL 1 MONTH)
```

不过，如果要取当天，Oracle 里的 trunc(sysdate) 在 MySQL 可以直接写成 CURDATE()

## 日期相加

在 Oracle 中，当前时间的三个月后，写法为：

```sql
add_months(sysdate, 3)
```

在 MySQL 中，写法为：

```sql
date_add(now(), INTERVAL 3 montn)
```

## 相差月数

在 Oracle 中，用 `months_between(日期1，日期2)` 计算日期相差的月数

在 MySQL 中，用 `timestampdiff(month, 日期1，日期2)`

## nvl

nvl(a, b) 如果 a 为 null，则函数结果为b，否则函数结果为a

在 MySQL 中，用 ifnull(a, b) ，注意 ifnull 不能判空串

## nvl2

在 Oracle 中，`nvl2(a,b,c)` 的逻辑是： 如果 a 为 null ，则c，否则 b

在 MySQL 中写法为：

```
IF ( ISNULL(a), c, b );
```

注意 b 和 c 的位置

## decode()

在 Oracle 中，decode(条件，值1，返回值1，值2，返回值2, ......) 用来选择数据

在 MySQL 中，用 case when 代替

```sql
case 条件
  when '值1' then '返回值1'
  when '值2' then '返回值2'
  when ...   then ...      END
```

## 保留两位小数

在 Oracle 中，用 `to_char(123.345, '0.99')` 保留两位小数并转换成字符串

在 MySQL 中，用 `convert(123.345, decimal(10,2))` 来保留两位小数

## 位数不够补空格

Oracle，用 `to_char(num, '999')` ， 会补齐到四位

如：
```
[空格]100
[空格][空格]83
```

MySQL，用 `LPAD(char, 4, ' ')`，第一个参数是传入的字符串，第二个是要补齐到多少位，第三个是补什么。

Oracle 也有 LPAD 函数。

---

# 其他

## 获取当前用户

在 Oracle 中，用 USER 获取当前数据库连接用户

在 MySQL 中，用 user() 获取当前数据库连接用户

## 常量

在 Oracle 中，有 CONSTANT 关键字，声明一个常量

在 MySQL 中，没有常量的概念

## 约束

在 Oracle 中，可以使用 check 约束某些值的范围，如 Y N

在 MySQL 中，可以使用 check 约束，但没有任何效果，因为 MySQL 底层没有实现。一般这种检查在应用层做。

## 计列

在 Oracle 中，有时候用

```sql
SELECT rownum num FROM table
```

这样的语法来增加一行辅助列，并在后面 WHERE 用于限制结果集和分页。但在 MySQL 中，直接用 `LIMIT` 即可。

当 MySQL 确实需要辅助计数列时，可以用以下语法

```sql
SELECT @rownum := @rownum +1 num
FROM table t, (SELECT @rownum:=0) tt
```

## 左右连接

在 Oracle 中，有时候会遇到

```sql
FROM table_a a,
     table_b b,
WHERE a.series = b.code(+)
```

实际上这是左右连接的特殊写法

(+)符号放在非主表，上面的例子是左连接，在 MySQL 中只能用 JOIN

```sql
FROM table_a a
LEFT JOIN table_b b
ON a.series = b.code
```

## 更新或插入

某些列值存在时则更新，不存在时则插入一条新记录。在 Oracle 中，用 `MERGE INTO` 语法。

```sql
MERGE INTO table
USING dual
ON ( .. = ..)
WHEN MATCHED THEN
...
WHEN NOT MATCHED THEN
...
```

在 MySQL 中，可以用 `REPLACE INTO`，但主键冲突或唯一索引冲突时，会删除原有数据，再插入。 或者用 `INSERT INTO ON DUPLICATE KEY UPDATE` ，在主键冲突或唯一索引冲突时更新原数据，否则插入。

```sql
INSERT INTO table
(
  ...,
  ...,
)
VALUES
(
  ...,
  ...,
)
ON DUPLICATE KEY UPDATE
column1 = v1,
column2 = v2
```

## 别名

mysql delete语句不支持别名，Oracle可以
