---
title: 聊聊JDBC批量数据处理
categories: Java
abbrlink: 7c78488a
date: 2020-03-04 23:22:38
---

# 问题来源

最近接手了一个生产问题，一个定时任务（JOB）报内存溢出。看了下代码反推逻辑，发现需求很简单，根据mysql库table表中的type字段，去更新level字段。

表 `table` 共有1800万数据，其中符合 `type = P` 的有80万。

定时任务是纯 JDBC 写的，没有用第三方框架，那为什么内存溢出呢？看了下代码，这货竟然是先用 SELECT 把 `type = P` 的所有ID取到应用的 List 里面，再循环遍历 List 根据 ID 一个个去做 update。这80万个ID一下子涌进内存，不OOM才怪呢，查SVN提交记录，查注释，发现这代码竟然是2013年写的，可能是当年数据量还没这么大，没有长远考虑吧，好吧，反正这种祖传代码，想找人理论也死无对证了。

那要怎么改呢？

<!-- more -->

# 直接update

一开始，我直接把原先的逻辑改掉了，直接一个SQL搞定：

```sql
UPDATE table
SET level = '0'
WHERE type = 'P'
```

看似没毛病，但是 table 是个1800万数据的大表，这个update语句在测试环境执行了7分多钟。而且 type 字段也没有索引，这意味着，在执行这个语句期间，MySQL Innodb会锁表，而不是锁行。如果应用或者其他定时任务也写该表，就会造成锁等待，严重的话，等待超时引发应用或其他定时任务报错。

# stream流处理

之前做定时任务的开发，一般都是先 SELECT A表数据，再 INSERT 到 B表。当 SELECT 的数据量过大，为了防止内存溢出导致OOM ，在 Oracle 里面常用 `setFetchSize(5000)` 来做游标查询，在 `while(rs.next())` 循环中，用 `addBatch()`、`executeBatch()` 的方式手动分批提交。在 MySQL 里面推荐stream流处理（当然游标也行）。具体的差异 [MySQL JDBC StreamResult通信原理浅析](https://blog.csdn.net/xieyuooo/article/details/83109971) 这篇文章讲得很好，可以参考。

而本次的需求 SELECT 和 UPDATE 都是在同一张表，stream流处理会在传递的数据的过程中，锁住相应的数据行，导致无法 UPDATE 。所以这种方式不适合。

# 应用层分页

另一种实现方式是在应用层手动分页

```sql
SELECT id FROM table
WHERE type = 'P'
ORDER BY id
limit x,y
```

再根据分页的每次 id 去做 UPDATE ：

```sql
UPDATE table
SET level = '0'
WHERE id in (id1, id2, id3 ...)
```

这种方式可行，至少不会一个SQL把表锁了，只会锁行。符合我们的要求。但显然，这种方式看起来也没那么高效和优雅。

# 使用游标和 ResultSet 的 updateRow

在查了 Google 和一些文档之后，意外地发现 JDBC 的 ResultSet 还有 updateRow 功能。而且该功能是直接 **响应到数据库** 的。实现起来既简单方便，又满足我们的要求，最终决定采用这种方式了。当然使用这种方式还要结合游标，因为 MySQL 默认是把所有数据取到内存后才开始处理，还是会内存溢出，所以需要游标让 MySQL 分批返回。

代码如下：

```java
Connection conn = DButils.getConnection();
PreparedStatement ps = conn.prepareStatement("SELECT id, level FROM table WHERE type = 'P'",
        ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_UPDATABLE);
ps.setFetchSize(1000);
ResultSet rs = ps.executeQuery();
conn.setAutoCommit(false);
int i = 0;
while (rs.next()){
    i++;
    rs.updateString("level", "0");
    rs.updateRow();
    // 每1000条提交一次，提高性能
    if (i%1000==0){
        conn.commit();
    }
}
```

Tips：
1. 默认ResultSet是不允许修改的，prepareStatement第三个参数让JDBC使用可修改模式；
2. 实践表明，关闭自动提交，每1000条手动提交一次，效率更高；
3. 这种方式在 SELECT 中必须包含 primary key，否则会报错。
4. 使用游标，需要在连接加 `useCursorFetch=true` 参数
