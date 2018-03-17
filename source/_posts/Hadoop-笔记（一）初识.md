---
title: Hadoop 笔记（一）初识
comments: true
categories: 大数据
tags: Hadoop
abbrlink: e2142b57
date: 2018-03-14 18:47:46
---

当数据量变大的时候，一台机器完成一个问题要计算好久好久。这时候就需要多台机器并行运算。然而，每台机器不能用单台机器运行的算法，自己算自己的。而是要有不同的分工，联合起来共同算完这个问题。

Hadoop就是这样的一个大数据处理框架。其中包括很多开源的处理框架，比如：

* **文件存储**：Hadoop HDFS、Tachyon、KFS
* **离线计算**：Hadoop MapReduce、Spark
* **流式、实时计算**：Storm、Spark Streaming、S4、Heron
* **K-V、NOSQL数据库**：HBase、Redis、MongoDB
* **资源管理**：YARN、Mesos
* **日志收集**：Flume、Scribe、Logstash、Kibana
* **消息系统**：Kafka、StormMQ、ZeroMQ、RabbitMQ
* **查询分析**：Hive、Impala、Pig、Presto、Phoenix、SparkSQL、Drill、Flink、Kylin、Druid
* **分布式协调服务**：Zookeeper
* **集群管理与监控**：Ambari、Ganglia、Nagios、Cloudera Manager
* **数据挖掘、机器学习**：Mahout、Spark MLLib
* **数据同步**：Sqoop
* **任务调度**：Oozie

那这么多，要怎么学呢？吴军博士在《数学之美》中提到：

> 分治算法是计算机科学中最漂亮的工具之一，我称为“各个击破”法。

我们就来各个击破。当然，先挑重点的学习。

---

# MapReduce

假设我们要统计一本10000页的书里面，"apple"、"banana"、"orange"这三个单词出现的次数。由于规模很大，用一台机器来算，要算很久。我们能不能把规模缩小，交给多台机器去算呢？

我们想到，可以拿4台服务器，假设为1，2，3，4，每台服务器计算2500页，各自算各自的。

好了，现在每台服务器把各自负责的2500页统计完了。 但我们关心的是10000页这个总量里面单词出现的次数，而不是4个独立的2500页。

这4个2500页的结果分别保存在1，2，3，4四台服务器上。我们现在要想办法合并结果。

于是我们找来另外三台服务器，假设为A，B，C。

让 A 计算在机器1，2，3，4上面 "apple" 单词出现的总次数。

让 B 计算在机器1，2，3，4上面 "banana" 单词出现的总次数。

让 C 计算在机器1，2，3，4上面 "orange" 单词出现的总次数。

这样，我们就知道每个单词出现的总次数了。

<!-- more -->

以上就是 Hadoop 简单的基本原理。我们称为`MapReduce` 模型。这个模型分为三个阶段。

**Map阶段**：每台机器先处理本机上的数据。（对于机器1来说，就是计算前2500页"apple"出现的次数）

**Shuffle阶段**：各个机器处理完自己的数据后，用另一批机器（或者还是这些机器）去收集某个数据的总和。（对于机器 A 来说，就是把 4 个 2500页 的"apple" 汇总。）

**Reduce阶段**：把多个数据的总和规约、合并，出最终结果。（把汇总的“apple”、"banana"、"orange" 归并）

---

# Hadoop Distribute Filesystem (HDFS)

当数据集大小超过一台计算机的存储能力时，就有必要对它进行分区（partition）并存储到多台计算机上。管理网络中跨多台计算机存储的文件系统称为`分布式文件系统（distribute filesystem）`。

Hadoop 自带一个分布式文件系统，称为 HDFS。

## HDFS 的设计

包括可存储超大文件、流式数据访问、用于商用硬件（指的是普通硬件而不是精密昂贵的硬件）、低时间延迟的数据访问、大量的小文件、单用户写入和只添加 等设计特点。

## HDFS 的一些术语

* **数据块**： HDFS上的文件被划分为多个分块（chunk），作为独立的存储单元
* **namenode**：管理节点，管理文件系统的命名空间，也就是用于维护系统树和树内所有的文件和目录的东西。namenode十分重要，因此需要实现容错。一般在另一台单独计算机配置辅助namenode（secondarynamenode）
* **datanode**：工作节点，存储并检索数据块。定期向 namenode 发心跳包。

![](../../../../images/hadoop/node.png)

## HDFS 的基本操作

将本地文件上传到 hdfs 上（原路径只能是一个文件）
```
hdfs dfs -copyFromLocal /local/data /hdfs/data
```

和 copyFromLocal 区别是，put 原路径可以是文件夹等
```
hdfs dfs -put /tmp/ /hdfs/
```

查看根目录文件
```
hadoop fs -ls /
```

查看/tmp/data目录
```
hadoop fs -ls /tmp/data
```

查看 a.txt，与 -text 一样
```
hadoop fs -cat /tmp/a.txt
```

创建目录dir
```
hadoop fs -mkdir dir
```

删除目录dir
```
hadoop fs -rm -r dir
```

> 在 HDFS 中，就不要 cd 了， 用 ls + 目录

### Hadoop fs 和 hdfs dfs的区别

Hadoop fs：使用面最广，可以操作任何文件系统。

hadoop dfs与hdfs dfs：只能操作HDFS文件系统相关（包括与Local FS间的操作），前者已经Deprecated，一般使用后者。

---

# Yet Another Resource Negotiator (YARN)

YARN 是 Hadoop 的集群资源管理系统。 YARN 提供请求和使用集群资源的API。但很少用于用户代码，因为在 YARN 之上的如MapReduce、Spark等分布式计算框架向用户隐藏了资源管理的细节。

一般来说，HDFS在Storage底层、YARN在Compute中间层，往上的Application上层才是MapReduce、Spark等计算框架。（甚至Application之上还能再封装一层，如Pig、Hive等）

Hadoop有两类长期运行的守护进程：

* **资源管理器**：管理集群上资源的使用
* **节点管理器**：运行在集群中所有节点上切能够启动和监控容器的东西

 YARN 则是管理这两个守护进程的。

 ---

# Hive

hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。

 ---

# 小结

HDFS是Hadoop提供的分布式存储框架，它可以用来存储海量数据，MapReduce是Hadoop提供的分布式计算框架，它可以用来统计和分析HDFS上的海量数据，而Hive则是SQL On Hadoop，Hive提供了SQL接口，开发人员只需要编写简单易上手的SQL语句，Hive负责把SQL翻译成MapReduce，提交运行。

---
