---
title: Spark编程入门（一） QuickStart
comments: true
categories:
- 大数据
- Spark
tags: 大数据
abbrlink: a08d78bf
date: 2018-04-05 15:38:00
---

# Spark 基本概念

在实际应用中，大数据处理主要包括：
- 复杂的批量数据处理（数十分钟 - 几小时）
- 基于历史数据的交互式查询（数十秒 - 几分钟）
- 基于实时流的数据处理 （数百毫秒 - 几秒）

Spark 的设计遵循“一个软件栈满足不同的应用场景”，有一套完整的生态系统。包括内存计算框架、SQL即时查询、实时流式计算、机器学习和图计算等。Spark可以部署在 YARN 资源管理器上，提供一站式的大数据解决方案。

<!-- more -->

* **RDD**：弹性分布式数据集（Resilient Distributed Dataset），分布式内存的一个抽象概念，提供了一种高度受限的内存模型。
* **DAG**：有向无环图（Directed Acyclic Graph, DAG），反映RDD之间的依赖关系。
* **Executor**：运行在工作节点（Worker Node）上的一个进程，负责运行任务，为应用程序存储数据。
* **应用**：用户编写的 Spark 程序
* **任务**：运行在 Executor 上的工作单元
* **作业**：一个作业包含多个RDD以及作用域相应RDD的各种操作
* **阶段**：是作业的基本调度单位，一个作业分为多组任务，每组任务被称为“阶段”，或者“任务集”。

在 Spark 中，一个应用（Application）由一个任务控制节点（Driver）和若干个作业（Job）构成，一个作业由多个阶段（Stage）构成，一个阶段由多个任务（Task）组成。

---

# Spark 生态系统组件

- **Spark Core**：内存计算、任务调度、部署模式、故障恢复、存储管理等，主要面向批数据处理
- **Spark SQL**：允许开发者直接处理RDD，也可查询 Hive、Hbase等外部数据源。统一处理数据关系表和RDD，开发者无需自己编写Spark程序，即可用 SQL 命令查询。
- **Spark Streaming**： 实时流处理。支持多种数据输入源，如 Kafka、Flume、HDFS 或 TCP套接字。
- **MLlib（机器学习）**：提供机器学习算法实现，包括聚类、分类、回归、协同过滤。
- **GraphX（图计算）**

无论哪个组件，都可以用 Core 的API处理问题。

---

# Spark 的安装

在 Spark 官网下载 tgz 压缩包

```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-2.3.1/spark-2.3.1-bin-hadoop2.7.tgz
```

* 注：以上是 2.3.1 版本的清华大学镜像，最新版本在 [spark 官网](https://spark.apache.org/downloads.html) 找

将 spark 解压到当前目录

```
tar -zxvf spark-2.3.1-bin-hadoop2.7.tgz -C .
```

---

# Spark 三种部署方式

* **standalone**
* **Spark on Mesos**（官方推荐）
* **Spark on YARN**

---

# 使用 Spark Shell 进行交互式编程

Spark 官方提供了命令行交互式编程，也就是一边输入一边输出。

安装完 spark 后， 运行 `./bin/spark-shell` 即可启动 scala 语言的 spark shell， 运行 `./bin/pyspark`即可启动 Python 语言的 spark shell (pyspark)。

---

# 独立 spark 应用程序

但是我们一般都是在 IDE 里编写独立的应用程序（Self-Contained Applications），再部署打包运行。

我这里根据[官方示例](http://spark.apache.org/docs/latest/quick-start.html)，使用 IDEA，编写一个统计文本中包含字母 'a' 的行数和包含字母 'b'的行数的 spark 应用程序。


## 新建 Maven 工程

首先新建一个 Maven 工程，然后在 pom.xml 中添加依赖

```xml
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>

<dependencies>
    <dependency> <!-- Spark dependency -->
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-sql_2.11</artifactId>
        <version>2.3.0</version>
    </dependency>
```

- `properties`标签里的内容目的是让我们的工程基于 jdk 1.8
- `dependencies`标签里的内容表示依赖，需要哪些依赖一般 [Spark 官方文档](http://spark.apache.org/docs/latest/quick-start.html) 在相应的地方都会有标注，然后可以到[MVN仓库](http://mvnrepository.com/)去搜，上面会告诉你对应版本的`dependency`该怎么写。

## 编写java代码

SimpleApp.java

```java
/* SimpleApp.java */
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.Dataset;

public class SimpleApp {
  public static void main(String[] args) {
    // 定义文件路径 （可以说 file:// 也可以是 hdfs://）
    String logFile = "file:///home/jerrysheh/spark-2.3.0/README.md";

    //定义 spark 会话
    SparkSession spark = SparkSession.builder().appName("Simple Application").getOrCreate();

    // 定义数据集（ spark读 logfile 文本文件，缓存）
    Dataset<String> logData = spark.read().textFile(logFile).cache();

    // 数据集过滤，包含 a / b 的就进行统计
    long numAs = logData.filter((FilterFunction<String>)  s -> s.contains("a")).count();
    long numBs = logData.filter((FilterFunction<String>)  s -> s.contains("b")).count();

    //输出
    System.out.println("Lines with a: " + numAs + ", lines with b: " + numBs);

    spark.stop();
  }
}
```

## 配置运行环境

点击edit configuration，在左侧点击该项目。在右侧 VM options中 输入`-Dspark.master=local`，指示本程序本地单线程运行。

spark 支持的 master URL 有：
- local 本地单线程
- local[K] 本地多线程（指定K个内核）
- local[\*] 本地多线程（指定所有可用内核）
- spark://HOST:PORT 连接到指定的 Spark standalone cluster master，需要指定端口。
- mesos://HOST:PORT 连接到指定的 Mesos 集群，需要指定端口。
- yarn-client 客户端模式 连接到 YARN 集群。需要配置 HADOOP_CONF_DIR。
- yarn-cluster 集群模式 连接到 YARN 集群。需要配置 HADOOP_CONF_DIR。

配置完以后，就可以直接在 IDEA 运行，开发阶段不必打包后在命令行运行。

## 打包

### 使用 mvn 命令打包

安装 maven
```
sudo apt install maven
```

打包 jar
```
mvn package
```

这样 out 目录就生成了 一个 jar 文件，这就是我们的 spark 应用程序了。

### 使用 IDEA 打包

用 maven 打包的 jar 非常大，因为把很多依赖都打包进去了，我们可以用 IDEA 删除我们不需要的组件，这样打包出来的 jar 就很小了。

使用 IDEA 打包的方法可以参考：[利用开发工具IntelliJ IDEA编写Spark应用程序](http://dblab.xmu.edu.cn/blog/1327/)

如果需要 sql 驱动，可以这样写：

```java
// 从数据库读 DataFrame
def readFromMySQL(spark: SparkSession, tableName:String): DataFrame = {
    val prop=new java.util.Properties
    prop.setProperty("driver", "com.mysql.cj.jdbc.Driver")
    prop.setProperty("user","root")
    prop.setProperty("password","YOURPASSWORD")

    val df = spark.read.jdbc(jdbcURL, tableName, prop)
    df
}
```

这样当提交到 spark-submit 的时候会读取你的驱动，否则报 Driver not found

## 运行

在 spark 安装目录下

```
./bin/spark-submit --class "SimpleApp" --master local[4] simple-project.jar
```

输出：

```
...
...
Lines with a: 46, Lines with b: 23
```

这样，一个简单的 spark 应用程序就运行成功了。

## 只输出 WARN 和 ERROR 不输出 INFO

把 SPARK_HOME/conf 下的 log4j.properties.template 复制到 工程 Resource Root 下面（我这里是 Scala 包），重命名为 log4j.properties 。 打开，修改其中的

```
log4j.rootCategory=INFO, console
```

将 INFO 改成 WARN 或者 ERROR 即可
