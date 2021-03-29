---
title: 记一次Spring JDBC内存泄漏引起的线上事故
categories:
  - Java
  - JDBC
tags:
  - Java
abbrlink: 720e88bc
date: 2021-03-28 20:25:00
---

## 问题再现

最近在维护一个基于 Spring Boot 的数据同步系统。项目使用 druid 连接池，配置动态数据源连接了 16 个数据库，主要用于跑任务处理跨库数据同步。其中的某个任务，在线上环境一直稳定运行，前几天任务又一次执行时，突然收到任务报错的邮件告警。

<!-- more -->

日志如下：

```java
com.alibaba.druid.pool.GetConnectionTimeoutException: wait millis 60000, active 20, maxActive 20, creating 0
    at com.alibaba.druid.pool.DruidDataSource.getConnectionInternal(DruidDataSource.java:1512)
    at com.alibaba.druid.pool.DruidDataSource.getConnectionDirect(DruidDataSource.java:1255)
    at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:5007)
    at com.alibaba.druid.filter.stat.StatFilter.dataSource_getConnection(StatFilter.java:680)
    at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:5003)
    at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:1233)
    at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:1225)
    at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:90)
```

从日志上看，`active 20, maxActive 20` 说明配置的最大活跃连接数是 `20`，当前创建的连接数也是 `20`，初步分析报错的直接原因肯定是代码中的某个地方获取连接后一直没有释放，导致连接数达到上限，druid 无法获取更多连接，最终超时报错。



## 问题排查

我们项目中使用的是 `Spring JDBC`，所有的数据库交互都是使用 `jdbcTemplate`。众所周知，Spring 提供的 `jdbcTemplate` 会自动帮我们管理 JDBC 连接的获取和释放，一般我们使用 `jdbcTemplate.query()`、 `jdbcTemplate.update()` 等方法，无需我们自己关闭资源。那为什么又会连接泄露了呢？

排查项目中的代码逻辑，发现一处地方很可疑：

```java
Connection conn = jdbcTemplate.getDataSource().getConnection();
```

查阅资料后发现，该方法显式拿到一个数据库连接，假如后续一直没有使用，Spring 则不会帮助我们释放连接。导致连接泄露。

## 临时处理方案

临时让运营同事重启服务器，把连接数重置，同时修改 druid 配置项，把 maxActive 连接数调大一点，保证受影响的任务下几次跑都有连接可用。

如果任务执行的频率比较高，还可以设置 druid 的以下三个配置强制回收（只在怀疑内存泄露时配置，平时不用配置，以免误杀正常连接）：

```
# 是否打开强制回收连接功能
removeAbandoned=true
# 超时时间（毫秒）
removeAbandonedTimeoutMillis=600000
# 强制回收时记录日志（方便查看是哪个地方泄露）
logAbandoned=true
```

## 代码整改

Spring 提供了 `DataSourceUtils` 工具类。借助此类来手动释放连接。

```java
try {
    conn = DataSourceUtils.getConnection(jdbcTemplate.getDataSource());

    // 业务逻辑
    // ...

} catch (Exception e) {
    log.error();
} finally {
    DataSourceUtils.releaseConnection(conn, jdbcTemplate.getDataSource());
}
```


好在这个系统中只有定时任务，没有前端请求。所幸没有引起严重后果。如果发生在接收前端请求的系统中，那就是另外一个关于“血灾”的故事了。

完。