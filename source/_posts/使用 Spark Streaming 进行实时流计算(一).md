---
title: 使用 Spark Streaming 进行实时流计算(一)
comments: true
categories: 大数据
tags: 大数据
abbrlink: bcfe91a1
date: 2018-04-05 16:09:20
---

在实际应用中，大数据处理主要包括：
- 复杂的批量数据处理（数十分钟 - 几小时）
- 基于历史数据的交互式查询（数十秒 - 几分钟）
- 基于实时流的数据处理 （数百毫秒 - 几秒）

Spark 的设计遵循“一个软件栈满足不同的应用场景”，有一套完整的生态系统。包括内存计算框架、SQL即时查询、实时流式计算、机器学习和图计算等。Spark可以部署在 YARN 资源管理器上，提供一站式的大数据解决方案。

<!-- more-->

# Spark 基本概念

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

# Spark 三种部署方式

* **standalone**
* **Spark on Mesos**（官方推荐）
* **Spark on YARN**

---

# Spark Streaming 简介

Spark Streaming是 Spark 的实时计算框架，为Spark提供了可扩展、高吞吐、容错的流计算能力。支持多种数据输入源，如 Kafka、Flume、HDFS 或 TCP套接字。

![输入输出](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE10-19-Spark-Streaming%E6%94%AF%E6%8C%81%E7%9A%84%E8%BE%93%E5%85%A5%E3%80%81%E8%BE%93%E5%87%BA%E6%95%B0%E6%8D%AE%E6%BA%90.jpg)

## 基本原理

将实时输入的数据以时间片（秒级）为单位进行拆分，经 Spark 引擎以类似批处理的方式处理每个时间片数据。

Spark Streaming 最主要的抽象是 DStream(Discretized Stream，离散化数据流)，表示连续不断的数据流。Spark Streaming的输入数据按照时间片分成一段一段的 DStream，每一段数据转换为 Spark 中的 RDD，并且对 DStream 的操作最终转变为相应的 RDD 操作。

DStream是 Spark Streaming 的编程模型，DStream 的操作包括 **输入**、**转换**和**输出**。

## Spark Streaming 与 Storm 对比

- Spark Streaming 无法实现毫秒级别的流计算， Storm 可以。
- Spark Streaming 小批量处理的方式可以同时兼容批量和实时数据处理的逻辑和算法，方便在需要历史数据和实时数据联合分析的应用场合。

---

# Spark Streaming编程步骤

编写 Spark Streaming 程序的基本步骤是：

1. 通过创建输入DStream来定义**输入源**;
2. 通过对 DStream 应用的 **转换操作** 和 **输出操作** 来定义流计算;
3. 用`streamingContext.start()`来开始接收数据和处理流程;
4. 通过`streamingContext.awaitTermination()`方法来等待处理结束（手动结束或因为错误而结束）;
5. 可以通过`streamingContext.stop()`来手动结束流计算进程。

## 创建对象

如果用 pyspark，默认已经获得了一个SparkConext（sc），否则，需要手动创建，如下

```python
from pyspark import SparkContext, SparkConf
from pyspark.streaming import StreamingContext
conf = SparkConf()
conf.setMaster('local[2]')  # 表示运行在本地模式下，并且启动2个工作线程。
conf.setAppName('TestDStream')
sc = SparkContext(conf = conf)
```

要运行一个Spark Streaming程序，首先要生成一个StreamingContext对象

```python
ssc = StreamingContext(sc, 15)  # 15表示每隔15秒钟自动执行一次流计算
```

## 从文件流读取数据

Spark支持从兼容HDFS API的文件系统中读取数据，创建数据流。

首先创建一个 logdir 目录，里面有 log1.txt 和 log2.txt 两个日志文件

然后在 python 中继续写

```python
lines = ssc.textFileStream('file:///home/jerrysheh/logdir')
words = lines.flatMap(lambda line: line.split(' '))
wordCounts = words.map(lambda x : (x,1)).reduceByKey(add)
wordCounts.pprint()
ssc.start()  # 如果用 pyspark，到这里会循环监听，下面的语句无法输入
ssc.awaitTermination()
```

输出：
```
-------------------------------------------
Time: 2018-04-05 23:45:00
-------------------------------------------

-------------------------------------------
Time: 2018-04-05 23:45:15
-------------------------------------------

-------------------------------------------
Time: 2018-04-05 23:45:30
-------------------------------------------
```

可以发现，程序每隔10秒监听一次。但是没有把 logdir 目录下的 log1.txt 和 log2.txt 这两个文件中的内容读取出来。原因是，监听程序只监听目录下在程序**启动后新增的文件**，不会去处理历史上已经存在的文件。

现在，用 vim 在 logdir 目录里新增一个 log3.txt ，并写入 heelo， 保存。

过一会儿，就能发现屏幕中输出了 log3.txt 里面的信息。


```
-------------------------------------------
Time: 2018-04-05 23:45:45
-------------------------------------------
('hello', 1)

```

## 从TCP套接字流读取数据

pyWordCountServer.py

```python
from __future__ import print_function
import sys
from pyspark import SparkContext
from pyspark.streaming import StreamingContext

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: network_wordcount.py <hostname> <port>", file=sys.stderr)
        exit(-1)
    sc = SparkContext(appName="PythonStreamingNetworkWordCount")
    ssc = StreamingContext(sc, 5)

    lines = ssc.socketTextStream(sys.argv[1], int(sys.argv[2]))
    counts = lines.flatMap(lambda line: line.split(" "))\
                  .map(lambda word: (word, 1))\
                  .reduceByKey(lambda a, b: a+b)
    counts.pprint()

    ssc.start()
    ssc.awaitTermination()
```

在9999端口设置网络监听

* -k 参数	Keep inbound sockets open for multiple connects
* -l 参数	Listen mode, for inbound connects

shell 1
```
nc -lk 9999
```

shell 2
```
python3 pyWordCountServer.py localhost 9999
```

在 shell 1 里输入一些东西， shell 2可以接收到

但是这里报了 WARN

```
Block input-0-1522945797400 replicated to only 0 peer(s) instead of 1 peers
```

而且也没有任何数据

[stackoverflow](https://stackoverflow.com/questions/32583273/spark-streaming-get-warn-replicated-to-only-0-peers-instead-of-1-peers) 给出了解释

> The warning in your case means that incoming data from stream are not replicated at all. The reason for that may be that you run the app with just one instance of Spark worker or running in local mode. Try to start more Spark workers and see if the warning is gone.

所以应该跟我是单机环境有关， [stackoverflow](https://stackoverflow.com/questions/28050262/spark-streaming-network-wordcount-py-does-not-print-result)，说

> I think you should specify more executors while submitting the application. For example:

> `spark-submit --master local[4] your_file.py`

> Do not run Spark Streaming programs locally with master configured as  local or local[1]. This allocates only one CPU for tasks and if a receiver is running on it, there is no resource left to process the received data. Use at least local[2] to have more cores.

所以开多几个线程就好了。

修改程序 创建SparkContext加 master 参数

```python
sc = SparkContext(appName="PythonStreamingNetworkWordCount", master="local[4]")
```

重新运行，还是报了 WARN， 但是 shell 1 的数据已经能够接收了

```
2018-04-06 00:46:45 WARN  RandomBlockReplicationPolicy:66 - Expecting 1 replicas with only 0 peer/s.
2018-04-06 00:46:45 WARN  BlockManager:66 - Block input-0-1522946805600 replicated to only 0 peer(s) instead of 1 peers
-------------------------------------------                                     
Time: 2018-04-06 00:46:48
-------------------------------------------
('are', 5)
('now', 1)
('you', 2)
('what', 1)
('doing', 1)

```

## 从 RDD 队列流读取数据

在调试Spark Streaming应用程序的时候，我们可以使用streamingContext.queueStream(queueOfRDD)创建基于RDD队列的DStream。

```python
import time

from pyspark import SparkContext
from pyspark.streaming import StreamingContext

if __name__ == "__main__":

    sc = SparkContext(appName="PythonStreamingQueueStream")
    ssc = StreamingContext(sc, 1)

    # Create the queue through which RDDs can be pushed to
    # a QueueInputDStream
    rddQueue = []
    for i in range(5):
        rddQueue += [ssc.sparkContext.parallelize([j for j in range(1, 1001)], 10)]

    # Create the QueueInputDStream and use it do some processing
    inputStream = ssc.queueStream(rddQueue)
    mappedStream = inputStream.map(lambda x: (x % 10, 1))
    reducedStream = mappedStream.reduceByKey(lambda a, b: a + b)
    reducedStream.pprint()

    ssc.start()
    time.sleep(6)
    ssc.stop(stopSparkContext=True, stopGraceFully=True)
```

---

以上主要介绍了从三个基本数据源（文件、TCP套接字、RDD）读取数据操作。下一篇继续介绍高级数据源（Kafka、Flume）以及数据的转换操作 和 输出操作。



参考

- 《大数据技术原理与应用》 林子雨
