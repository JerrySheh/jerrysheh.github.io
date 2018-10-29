---
title: 数据库（三）MySQL数据类型、索引、查询的优化
comments: true
categories: 数据库
tags: SQL
abbrlink: 2bb1b1ab
date: 2018-9-18 20:37:38
---

在 MySQL 中，使用恰当的数据类型，以及合理使用索引和查询，能够提升不少性能。这一篇介绍一下这三种情况的优化。

# Schema与数据类型优化

## 基本原则

- **更小**：如果只要存0-200，`tinyint unsigned` 比 `int` 好
- **简单**：用内建类型表示时间而不是varchar
- **避免NULL**：有 NULL 的列使得索引、索引统计和值比较更加复杂。虽然调优时把NULL改NOT NULL性能提升较小，但是如果要在列上建索引，就应该避免 NULL

<!-- more  -->

## 整数类型

整数类型包括 TINYINT（8位）、SMALLINT（16位）、MEDIUINT（24位）、INT（32位）、BIGINT（64位）。无符号数和有符号数使用相同的存储空间，具有相同的性能。整数计算一般用 64位的 BIGINT 整数，但一些聚合函数用 DECIMAL或 DOUBLE。

MySQL 可以为整数指定宽度，如 `INT(11)`，但这不会限制值的合法范围。

## 实数（小数）类型

实数不仅仅存储小数，也可以用 DECIMAL 存储比 BIGINT 大的整数。DECIMAL 一般用来做精确计算，但是需要的额外空间和开销也大。因此，如果不需要精确计算，4字节的FLOAT或8字节的DOUBLE已经足够。

## CHAR 和 VARCHAR

- **CHAR**：定长字符串。会截断末尾的空格。适合存储较短的字符串或所有值长度接近。
- **VARCHAR**：可变长字符串。需要用1或2个额外字节记录字符串的长度。VARCHAR虽然节省空间性能较好，但 UPDATE 时由于长度的改变，需要额外的工作。适用场景：字符串的最大长度比平均长度大很多，列很少更新。

需要注意的是，`VARCHAR(5)`和`VARCHAR(200)`存储`hello`的空间开销是一样的，但是更长的列会消耗更多的内存，所以最好根据需要来分配。

同理，有 BINARY 和 VARBINARY，存储的是二进制值，二进制的比较比字符比较要快。

## BLOB 和 TEXT

BLOB 和 TEXT 都是设计用来存储很大的字符串数据的，但 BLOB 采用二进制存储，TEXT采用字符方式存储。

跟其他类型不一样的是，当 BLOB 或 TEXT 值太大时，InnoDB会用专门的“外部”存储区来存储。每个值只需要在行内用1-4个字节存储指针，然后指向外部真正存储的区域。

- **BLOB**：二进制数据，没有排序规则和字符集
- **TEXT**：字符数据，有排序规则和字符集

MEMORY 存储引擎不支持 BLOB 和 TEXT，如果使用到了，将不得不转换成 MyISAM 磁盘临时表，这将带来很大的开销。MEMORY中最好避免使用 BLOB 和 TEXT。

## 枚举类代替字符串

有时候可以用枚举类代替不重复的字符串。其内部是用整数实际存储的，而不是字符串。因此最好不要往里面插入常量（如'1','2'）以避免混乱。但是也有缺点，添加或删除字符串需要用 `ALTER TABLE`，因此对于一些未来可能会改变的字符串，使用枚举是不明智的。

```sql
CREATE TABLE enum_test(
  e ENUM('fish','apple','dog') NOT NULL
);
INSERT INTO enum_test(e) VALUES ('fish', 'dog', 'apple');
```

## DATETIME 和 TIMESTAMP

- **DATETIME**：能保存1001年-9999年，精度为秒。将日期和时间封装到 YYYYMMDDHHMMSS 格式的整数中，与时区无关。使用8个字节的存储空间。
- **TIMESTAMP**：能保存1970-2038年，只使用4个字节，存储的是1970年1月1日到现在的秒数，时区相关。

## 其他

- MySQL 把 bit 当作字符串，而不是数字
- MySQL 内部使用整数存储 ENUM 和 SET 类型，比较时再转换成字符串
- 应该用无符号整数（unsigned int）存储IP地址，MySQL提供 `INET_ATON()`（字符串转整数） 和 `INET_NTOA()`（整数转字符串）

---

# MySQL 高效索引

## 1.独立的列

对于独立的列来说，要创建高效索引，必须满足：**索引列不能是表达式的一部分，也不能是函数的参数**。例如：

```sql
// 使用了表达式，索引失效
SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;

SELECT ... WHERE TO_DAYS(CURRENT_DATE) - TO_DAYS(DATE_COL) <= 10;
```

应该养成简化 WHERE 条件的习惯，**始终将索引列放在比较符号的一侧**。

## 2.前缀索引

有时候要索引很长的字符列，这会让索引变得很大且慢。一种解决办法是在索引上再建哈希索引。但还可以用 **前缀索引** 来解决。

前缀索引，顾名思义，只索引字符串的前面一部分，例如，对于数据`University`，我们可以建立索引`Uni`。但这样会降低索引的选择性，<font color="red">索引选择性是指不重复的索引值 和 表记录数的比值</font>。选择性越高，说明索引越多。唯一索引的选择性是1，因此性能最高。

在 MySQL 里面，BLOB、TEXT 和 很长的 VARCHAR 必须使用前缀索引。

查看前缀为3的情况

```sql
SELECT COUNT(*) AS cnt, LEFT(city, 3) AS pref
FROM city_demo
GROUP BY pref
ORDER BY cnt DESC LIMIT 10;
```

那索引前缀多长比较合适呢？诀窍是，**前缀应该足够长，使得选择性接近于整个列，但不能太长（以便节约空间）**。

计算完整列的选择性方法：
```sql
SELECT COUNT(DISTINCT city) / COUNT(*) FROM city_demo;
```

假如计算出来结果是 0.0312，那么选择性接近 0.0312 的前缀就差不多了。

测试各个前缀的选择性：

```sql
SELECT COUNT(DISTINCT LEFT(city, 3)) AS sel3,
       COUNT(DISTINCT LEFT(city, 4)) AS sel4,
       COUNT(DISTINCT LEFT(city, 5)) AS sel5
FROM city_demo;
```

当我们找到一个合适的前缀，比如是5，用下面的方式来创建前缀为5的前缀索引：

```sql
ALTER TABLE city_demo ADD KEY (city(5));
```

### 前缀索引的缺点

无法使用前缀索引做 GROUP BY 和 ORDER BY 和 覆盖扫描。

## 3.多列索引

常见多列索引的错误有：为每一列创建独立的索引，或者按照错误的顺序创建索引。那什么是正确的顺序呢？一个经验法则是：**当不需要考虑排序和分组时，将选择性最高的列放在最前面**。

一个简单的例子

```sql
SELECT * FROM payment WHERE staff_id = 2 AND customer_id = 584;
```

创建索引时，是应该创建 (staff_id, customer_id) 还是 (customer_id,staff_id) ？这取决于哪一列的选择性更高。但这也不是绝对的，还要考虑WHERE 子句中的排序、分组、范围条件等其他因素。

## 4.聚簇索引

聚簇的意思是：数据行和相邻的键值紧凑地存储在一起。当表有聚簇索引时，数据行本身存放在索引的叶子页。InnoDB的实现是，通过主键聚集数据，被索引的列就是主键列。如果没有主键，InnoDB会选择一个非空索引代替，如果没有这样的索引，就隐式创建一个。

InnoDB支持聚簇索引，而MyISAM不支持，使用了聚簇索引和非聚簇索引的存储方式区别可参考 [数据库（二）MySQL必知必会概念](../post/4c81d70.html#%E7%B4%A2%E5%BC%95%E7%9A%84%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0)

聚簇索引优点：

- 把相关数据保存在一起
- 数据访问更快
- 使用覆盖索引扫描的查询可以直接使用页节点中的主键值

聚簇索引缺点：

- 聚簇索引提高了I/O密集型应用的性能，但如果数据全部在内存中，那就没有优势
- 插入速度严重依赖于插入顺序
- 更新列代价高
- 页分裂问题，占用更多磁盘空间
- 全表扫描更慢
- 二级索引较大，访问要2次

### 5.覆盖索引

正如聚簇索引中你看到的，索引本身是可以包含数据本身的，这样我们就不必回表查询，直接在索引拿到数据就行了。想象一下，如果一本书需要知道第 11 章是什么标题，你会翻开第 11 章对应的那一页吗？目录浏览一下就好，这个目录就是起到覆盖索引的作用。

**如果一个索引包含（覆盖）所有需要查询的字段的值，我们就称之为覆盖索引**。覆盖索引也不一定是聚簇索引，在MySQL中，只有 BTree 索引能做覆盖索引。

---

# MySQL 查询优化

## 查询慢的原因

1. **查询了不需要的记录**。一个典型的错误是先 SELECT 查出所有结果集，然后获取前面的 N 行后关闭结果。这样 N 行后面的数据就是不需要的数据，MySQL会把时间浪费在这上面。最好的解决办法是用 limit N，这样MySQL只会去找 N 行而不是所有。
2. **多表关联时返回全部列**。比如 `SELECT * FROM xxx join yyy ON ...`，其实可以用 `SELECT sakila.actor.* FROM sakila join yyy ON ... `，只取关键的列。
3. **总是取出全部列**。`SELECT *`的做法在数据库的角度是不考虑周全的，但是有时候从开发的角度看却能简化开发，因为能提高相同代码片段的复用性。
4. **重复查询相同的数据**。需要多次重复查询的数据，最好第一次查询后缓存起来，可以使用 redis 等。

## 重构查询的两种方法

### 1.切分查询

一次大查询（例如删除旧的数据）可能需要一次锁住很多数据，占满整个事务日志、耗尽系统资源、阻塞很多其他重要的查询。可以把大查询切分成很多个小查询。

```sql
// 原始 大查询
DELETE FROM messages WHERE created < DATE_SUB(NOW(), INTERVEL 3 MONTH);

// 切分 小查询
rows_affected = 0;
do {
    rows_affected = do_query(
      "DELETE FROM messages WHERE created < DATE_SUB(NOW(), INTERVEL 3 LIMIT 10000"
    )
} while rows_affected > 0;
```

### 2.分解关联查询

高性能应用都会对关联查询进行分解，先对每一个表进行单表查询，再将结果在应用程序进行关联。

```sql
// 分解前
SELECT * FROM tag
  JOIN tag_post ON tag_post.tag_id = tag.id
  JOIN post ON tag_post.post_id = post.id
WHERE tag.tag = 'mysql';

//分解后
SELECT * FROM tag WHERE tag = 'mysql';
SELECT * FROM tag_post WHERE tag_id = 1234;
SELECT * FROM post WHERE post.id in (123,456,7897,9090)
```

## 优化特定类型的查询

### 优化COUNT()

- 如果要统计所有行，用 `COUNT(*)` 而不是 `COUNT(col)` 。
- `COUNT(col)`统计的是不为NULL的行
- `COUNT(distinct col)`统计不为NULL且不重复的行
- `COUNT(distinct col 1, col 2)` 如果其中一列全为 NULL ，那么即使另一列有不同的值，也返回为 0

```sql
// 统计行数，假如该表有100行，返回100
count(*);

// 统计 last_name 这一列不为NULL的数量
count(last_name);
```

- MyISAM中，不带 WHERE 的 `COUNT(*)` 速度非常快，因为可以直接利用存储引擎的特征获取这个值。但是带 WHERE 的跟其他存储引擎没区别
- 如果某列col不可能为NULL，那 `COUNT(col)` 将被自动优化成 `COUNT(*)`

借助 MyISAM `COUNT(*)` 非常快的特性，我们可以优化如下：

```sql
// 原语句，求大于5
SELECT COUNT(*) FROM city WHERE id > 5;

// 优化后，总数 - 小于等于5
SELECT (SELECT COUNT(*) FROM city) - COUNT(*)
FROM city WHERE id <= 5;
```

### 优化关联查询

- 确保 ON 或 USING 子句的列上有索引。也就是说，表A和表B用列c关联时，如果优化器的关联顺序是B、A，那只需要在 **第二张表**（A表） 的相应列上创建索引。
- 确保 GROUP BY 和 ORDER BY 中的表达式 **只涉及到一个表中的列**，这样MySQL才有可能使用索引来优化这个过程。

### 优化 GROUP BY

MySQL在无法使用索引时，GROUP BY会用临时表或文件排序来做分组。在 GROUP BY 的时候，如果标识列（如用户id）和查找列（如用户名）是对应的，那用标识列做分组，效率会比查找列高，GROUP BY右表标识列比GROUP BY左表标识列高。

如果不关心结果集的顺序，但GROUP BY默认会按分组的字段排序从而使用了文件排序功能，不需要的时候可以`ORDER BY NULL`。

### 优化 LIMIT 分页

MySQL limit接收一个或两个参数，如

```sql
// 取出前18条记录
SELECT ... limit 18;

// 取出第51-53条记录
SELECT ... limit 50,3
```

但有两个参数的时候，且第一个参数（偏移量）非常大，如 `limit 10000,30`，那MySQL需要查询 10030 条记录，然后抛弃前面 10000 条，返回最后30条。这样的代价是非常高的。

一个优化思路是：**尽可能使用索引覆盖扫描，而不是查询所有的列，然后根据需要做一次关联操作再返回所需的列**。考虑下面的例子：

```sql
// 改写前
SELECT film_id, description FROM sakila.film
ORDER BY title
LIMIT 50,5;

// 改写后
SELECT film.film_id, film.description FROM sakila.film
  INNER JOIN (
    SELECT film_id FROM sakila.film
    ORDER BY title
    LIMIT 50,5;
  ) AS lim USING(film_id);
```

先快速定位需要获取的 id 段，然后再关联。

```sql
SELECT a.* FROM 表 1 a, (select id from 表 1 where 条件 LIMIT 100000,20 ) b
where a.id=b.id
```



### 优化 UNION

MySQL 总是通过创建并填充临时表的方式来执行 UNION。除非确实需要消除重复的行，否则一定要使用 UNION ALL，没有 ALL 时 MySQL 会给临时表加 IDSTINCT 对数据做唯一性检查，这样做的代价非常高。

---

参考：
- 《高性能MySQL》
- 《阿里巴巴Java开发手册》
