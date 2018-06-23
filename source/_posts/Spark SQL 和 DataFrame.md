---
title: Spark SQL 和 DataFrame
comments: true
categories: 大数据
tags: 大数据
abbrlink: 21c6c0f6
date: 2018-06-04 00:00:00
---

# RDD 和 DataFrame 的区别

RDD 是弹性分布式数据集，其本质是 Dataset。Dataset 可以从 JVM 对象中构建 （例如 rating 对象，即 javabean ），然后通过 map、flatMap、filter 等方法转换来操作。

为了更好地读写数据以及使用类似SQL语句一样简单地操作，Spark SQL 提供了 DataFrame (其前身是SchemaRDD)。

![sparksql](../../../../images/hadoop/sparksql.jpg)

DataFrame 能够让你知道数据集中的每一行和列。这个概念跟关系型数据库中的表（table）类似，但是比表更强大。如下图：

![dataframes](../../../../images/hadoop/DataFrame-RDD.jpg)


DataFrame 可以从结构化的数据文件（structured data files）、Hive中的表、外部数据库或者**已存在的RDD**中构建。

在 Java 中，使用 `Dataset<Row>` 来表示 DataFrame。

<!-- more -->

---

# Getting Started

## 初始化 Spark

```java
SparkSession spark = SparkSession
  .builder()
  .appName("Java Spark SQL basic example")
  .config("spark.some.config.option", "some-value")
  .getOrCreate();
```

## 创建 DataFrames

通过 SparkSession，可以从已存在的RDD、Hive表、或者[其他数据源](https://spark.apache.org/docs/latest/sql-programming-guide.html#data-sources) 来创建DataFrames

例如，从一个 json 文件创建 DataFrames：

```java
Dataset<Row> df = spark.read().json("examples/src/main/resources/people.json");

// Displays the content of the DataFrame to stdout
df.show();
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

## 操作 DataFrame

`df`可以像数据库表一样进行操作：

- `df.printSchema()`：打印DataFrames结构
- `df.select("name").show()`: 选择 name 列打印
- `df.filter(col("age").gt(21)).show()`：筛选出年龄列大于21的
- `df.groupBy("age").count().show()`：统计各年龄人数

或者，把它变成一张临时的表

```java
// Register the DataFrame as a SQL temporary view
df.createOrReplaceTempView("people");
```

现在，我们的内存里就存在一张临时 people 表了。然后通过 Spark SQL 来操作：

```java
Dataset<Row> sqlDF = spark.sql("SELECT * FROM people");
    sqlDF.show();
    // +----+-------+
    // | age|   name|
    // +----+-------+
    // |null|Michael|
    // |  30|   Andy|
    // |  19| Justin|
    // +----+-------+
```

`df.createOrReplaceTempView("people")`的生命周期在 Spark Session，Session一关闭临时表就不存在了。如果要用应用程序级别的全局临时表，使用`df.createGlobalTempView("people")`，使用全局表需要在SQL语句添加 `.global_temp`

```java
df.createGlobalTempView("people");

// Global temporary view is tied to a system preserved database `global_temp`
spark.sql("SELECT * FROM global_temp.people").show();
```

---

更多操作见：

- [官方文档](https://spark.apache.org/docs/latest/sql-programming-guide.html#datasets-and-dataframes)
- [官方示例](https://github.com/apache/spark/blob/master/examples/src/main/java/org/apache/spark/examples/sql/JavaSparkSQLExample.java)
