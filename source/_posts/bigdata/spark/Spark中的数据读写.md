---
title: Spark中的数据读写
comments: true
categories:
- 大数据
- Spark
tags: 大数据
abbrlink: e884ae58
date: 2018-06-21 14:05:40
---


Spark 支持多种文件格式的读写，包括

- 本地文本文件：Json、SequenceFile 等文件格式
- 文件系统：HDFS、Amazon S3
- 数据库：MySQL、HBase、Hive

<!-- more -->

---

# 本地文件读写

## 文本文件

使用以下语句从文件系统中读写文件

```scala
val text = sc.textFile("file:///home/jerrysheh/word.txt")

// .first() 是一个"action"
text.first()

// 从RDD写回文件系统，saveAsTextFile是一个action
text.saveAsTextFile("file:///home/jerrysheh/wordWriteBack")
```

> spark的惰性机制使得在“转换”操作时，即使遇到错误也不会立即报错，直到”行动（action）“操作时才开始真正的计算，这时候如果有错误才会报错。


wordWriteBack 是一个文件夹，写回后存放在该文件夹里，里面有part-00000 和 \_SUCCESS 两个文件。part-00000 里面的内容就是写会的内容。

当我们想把输出的结果再次加载到RDD中，只要在`textFile()`中定位到 wordWriteBack 这个目录即可。

```scala
val text = sc.textFile("file:///home/jerrysheh/wordWriteBack")
```

## json文件

```scala
// jsonStr的类型是：org.apache.spark.rdd.RDD[String]
val jsonStr = sc.textFile("file:///home/jerrysheh/people.json")

// 使用 foreach 遍历
jsonStr.foreach(println)
```

输出：
```
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

可以用 scala 自带的 JSON 库 —— scala.util.parsing.json.JSON 进行解析。

---

# 从HDFS读写

跟本地文件类似，只不过把 `file://` 换成 `hdfs://`

```scala
val textFile = sc.textFile("hdfs://localhost:9000/user/jerrysheh/word.txt")
```

---
