---
title: 使用 Spark Streaming 进行实时流计算(一)
comments: true
categories:
- 大数据
- Spark
tags: 大数据
abbrlink: bcfe91a1
date: 2018-04-06 01:52:20
---

使用Spark Streaming 进行实时流计算，主要运用 DStream 编程模型。这种模型包括 **输入**、**转换**和**输出** 三个部分。这一篇主要介绍三种简单的输入。[下一篇](../post/76774483.html)将会介绍高级数据源和转换，输出部分。

<!-- more-->

# Spark Streaming 简介

Spark Streaming是 Spark 的实时计算框架，为Spark提供了可扩展、高吞吐、容错的流计算能力。支持多种数据输入源，如 Kafka、Flume、HDFS 或 TCP套接字。

![输入输出](http://spark.apache.org/docs/latest/img/streaming-arch.png)

## 基本原理

将实时输入的数据以时间片（秒级）为单位进行拆分，经 Spark 引擎以类似批处理的方式处理每个时间片数据。

Spark Streaming 最主要的抽象是 DStream(Discretized Stream，离散化数据流)，表示连续不断的数据流。Spark Streaming的输入数据按照时间片分成一段一段的 DStream，每一段数据转换为 Spark 中的 RDD，并且对 DStream 的操作最终转变为相应的 RDD 操作。

DStream是 Spark Streaming 的编程模型，DStream 的操作包括 **输入**、**转换**和**输出**。

## Spark Streaming 与 Storm 对比

- Spark Streaming 无法实现毫秒级别的流计算， Storm 可以。
- Spark Streaming 小批量处理的方式可以同时兼容批量和实时数据处理的逻辑和算法，方便在需要历史数据和实时数据联合分析的应用场合。

---

# Spark Streaming编程步骤

> Spark Streaming 应用程序可以用 Scala、Java、Python来编写， 官方提供了一种叫 spark-shell 的命令行环境，使用 Scala 语言来编写，或者使用 python 语言的 pyspark。但是我们一般是在 IDE 里编写独立的应用程序。

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

> pyspark 默认有 sc，那么如何开 local[2] ? 答案：`spark-submit --master local[4] your_file.py`

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

现在，用 vim 在 logdir 目录里新增一个 log3.txt ，并写入 hello， 保存。

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
