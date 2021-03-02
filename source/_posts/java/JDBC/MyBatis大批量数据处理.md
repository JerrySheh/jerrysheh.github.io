---
title: MyBatis大批量数据处理
comments: false
date: '2020-0302 20:44:52'
categories:
  - Java
  - JDBC
tags:
  - Java
abbrlink: 7969a482
---


# 前言

最近项目里需要跨数据库同步大批量数据（百万到千万级别），以前都是用 JDBC 来实现。在 JDBC 里，我们能灵活地使用流查询来批次摄取处理，避免OOM，但 JDBC 这玩意儿写多了，谁都会嫌它既啰嗦又繁琐（但性能真香）。于是这次决定用 Springboot + Mybatis 框架来试试。因为涉及到多个数据源和不同的数据库产品（Oracle、PostgreSQL、MySQL），所以在项目里使用了动态数据源。

关于 Springboot 多数据源方案，我参考过最好的文章为下面的3连载，推荐一看。

- [搞定SpringBoot多数据源(1)：多套源策略](https://mianshenglee.github.io/2020/01/13/multi-datasource-1.html)
- [搞定SpringBoot多数据源(2)：动态数据源](https://mianshenglee.github.io/2020/01/16/multi-datasource-2.html)
- [搞定SpringBoot多数据源(3)：参数化变更源](https://mianshenglee.github.io/2020/01/21/multi-datasource-3.html)

至于使用 Mybatis 做大批量数据读取和插入，先前也是阅读了大量的参考资料和文档。总体思想跟 JDBC 是一致的，即：**流式查询、批量插入**。但在讲 Mybatis 之前，先回顾一下 JDBC 时代是怎么做的吧。

<!-- more -->

---

# JDBC 驱动的那些坑

以前写 JDBC 时，通常是通过设置 `fetchsize` 来控制每次读取的数据量。`fetchsize` 并不是分页查询，而是数据库一次缓存到客户端的数量。所以在查询大量数据时，并不需要在SQL里手动写 `limit n offset m` 这样的分页语法。

JDBC 返回的数据类型是 `ResultSet`，这个玩意特别有意思，它是动态的。意思就是说，在你发起查询时，客户端与数据库的连接会一直保持打开，之后我们通过 `while(rs.next())` 逐条操作 `ResultSet` 里的每一条记录。 等缓存的数据量都遍历完了， 数据库会通过 TCP 连接发送下一批次的数据放到 `ResultSet` 里。全程对我们无感，我们要做的，只是不断地 `rs.next()` 就行了。

但是，不同的数据库产品对 `fetchsize` 的支持不一样。像 Oracle 这种标准的商业数据库，对 `fetchsize` 的支持就比较好，无脑使用即可。而 MySQL 和 PostgreSQL 就没那么简单了。

## MySQL 流查询

查阅 MySQL 的[官方文档](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-implementation-notes.html)，里面提到：

> By default, ResultSets are completely retrieved and stored in memory. In most cases this is the most efficient way to operate and, due to the design of the MySQL network protocol, is easier to implement. If you are working with ResultSets that have a large number of rows or large values and cannot allocate heap space in your JVM for the memory required, you can tell the driver to stream the results back one row at a time.
>
> To enable this functionality, create a Statement instance in the following manner:
```java
stmt = conn.createStatement(java.sql.ResultSet.TYPE_FORWARD_ONLY,
              java.sql.ResultSet.CONCUR_READ_ONLY);
stmt.setFetchSize(Integer.MIN_VALUE);
```
>The combination of a forward-only, read-only result set, with a fetch size of Integer.MIN_VALUE serves as a signal to the driver to stream result sets row-by-row. After this, any result sets created with the statement will be retrieved row-by-row.

也就是说，默认情况下，MySQL会一次性返回所有数据，`fetchsize` 并不起作用。如果想一条一条处理，必须在创建 `prepareStatement` 时指定参数，同时设置 `fetchsize` 的值为 `Integer.MIN_VALUE` 作为流式查询的标志。

这种操作很有意思。之前在阅读《高性能MySQL》时，看到书里面打过一个比喻，说 MySQL 的服务器和客户端，就像是水管的两端。一旦发起查询，水便源源不断地从进水口灌向出水口，此时出水口只能被动地被灌水，做什么也无济于事。既然通信协议都设计成这样了，为什么还可以实现做到流式查询，一条一条地处理呢？查阅了一些资料后得知，其奥妙在于借助了 TCP 的阻塞策略。即服务器向客户端发送 TCP 流，而客户端 TCP buffer 满了，在客户端没消费之前， TCP 连接会一直阻塞。这就像你把水管出水口用塞子塞住了，一次只允许流一滴水出来。

MySQL 默认情况下，创建 `prepareStatement` 时，就已经是 `ResultSet.TYPE_FORWARD_ONLY` 和 `ResultSet.CONCUR_READ_ONLY` ，所以这两个参数可加可不加。

## PostgreSQL 流查询

PostgreSQL 默认情况下， `fetchsize`也是无效的。[官方文档](https://jdbc.postgresql.org/documentation/head/query.html#fetchsize-example)里提到，要让 `fetchsize` 生效，**连接必须是非自动提交** 。即：

```java
conn.setAutoCommit(false);
Statement statement = conn.createStatement();
statement.setFetchSize(500);
```


---

# MyBatis 流查询

Mybatis 也是支持流数据查询的，主要是用了 `ResultHandler` 回调，对结果集的每一条进行处理，处理完即丢弃（释放内存），所以不会内存溢出。

```java
void query(ResultHandler<?> handler);
```

```java
productMapper.query( resultContext -> {
  Product p = (Product) resultContext.getResultObject()
  // 处理单条记录
  // ...
});
```

如果是 MySQL， fetchSize 要设置成 `-2147483648`，即`Integer.MIN_VALUE` 的值。 PostgreSQL 或 Oracle 则设置 `fetchSize=1000` ，fetchSize 大小可以自行调整。

```xml
<select id="query"  fetchSize="-2147483648" resultSetType="FORWARD_ONLY"  resultType="me.jerrysheh.demo.entity.Product">
    SELECT product_id, product_name, product_price FROM fun_product
</select>
```

在 PostgreSQL 测试时，发现不起作用，PostgreSQL 驱动还是一次性把几十万数据拉到了内存里，导致了 OOM。 结合刚刚上面提到的 PostgreSQL 需要把自动提交关闭才会启用 fetchsize，初步判断是自动提交的原因。但我们的 `productMapper` 是自动注入进来的，如果要改，还得从底层的 `datasource` -> `SqlSessionFactory` -> `Sqlsession` -> `mapper` 一层一层地手动注入进来。实际使用中根本不方便。

所以另辟蹊径，这里我给这个查询加了个事务，事务当中的SQL，肯定是手动提交的，只不过Spring帮我们管理了：

```java
transactionTemplate.execute( status -> {

    productMapper.query( resultContext -> {
    Product p = (Product) resultContext.getResultObject()
    // 处理单条记录
    // ...
  });


  return null;
} )

```

这下查询就搞定了！

# MyBatis 批量插入

批量插入没什么好说的，就是在查询过程中，积累了1000条，统一提交插入，用的 mybatis 的 `<foreach>` 标签。

```java
List<Product> list = new ArrayList<>();

// 切换数据源
datasourceContentHolder.setDatasource("my_pg")
transactionTemplate.execute( status -> {

    productMapper.query( resultContext -> {
    Product p = (Product) resultContext.getResultObject()
    // 处理单条记录
    // ...
    list.add(p);
    if (list.size >= 1000){
      DatasourceContentHolder.setDatasource("my_oracle")
      productMapper.batchUpdate(list);
      // 插入完记得把 list 清掉
      list.clear();
    }
  });


  return null;
} )

// 记得处理最后不足1000条的数据
if (list.size > 0){
  DatasourceContentHolder.setDatasource("my_oracle")
  productMapper.batchUpdate(list);
  list.clear();
}


```

但是在实际操作中，发现数据源无论如何都切换不过去。查阅资料发现，在 Spring 的动态数据源（AbstractRoutingDataSource）中，如果使用到了事务，那么当前线程默认拿事务的数据源，除非拿不到，才会去 ThreadLocal 里面拿。所以我们的`      DatasourceContentHolder.setDatasource("my_oracle")
` 不起作用。

解决方法也很简单粗暴，将`productMapper.batchUpdate(list)`封装到另一个 public 方法去，加上注解将插入操作排除在事务之外即可。

```java
@Transactional(propagation=Propagation.NOT_SUPPORTED)
public void doUpdate(List<Product> list){
  DatasourceContentHolder.setDatasource("my_oracle")
  productMapper.batchUpdate(list);
  // 插入完记得把 list 清掉
  list.clear();
}
```
