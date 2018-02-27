---
title: Java简明笔记（十） 输入与输出
comments: true
date: 2018-02-27 15:06:51
categories: JAVA
tags: Java
---

# 文本输入和输出

## 文本输入

对于较短的文本，我们可以直接把文本存到一个String里

```java
String contents = new String(readAllBytes((Paths.get("alice.txt"))), StandardCharsets.UTF_8);
```

如果想按行读取，可以存到 List 集合里，集合的每一个元素代表每一行的一个String

```java
List<String> lines = Files.readAllLines(path, charset);
```

或者按流处理

```
try (Stream<String> lines = Files.lines(path, charset)) {
  //...
} catch {

}
```

如果想从文件读取数字或单词，可以用 Scanner

```java
Scanner in = new Scanner(path, "UTF-8");
while (in.hasNextDouble()) {
  double value = in.hasNextDouble();
  ...
}
```

如果输入源不是来自文件，可以将InputStream再封装到BufferedReader

```
try (BufferedReader reader = new BufferedReader(new InputStreamReader(url.openStream()))) {
  ...
}
```

## 文本输出

如果我们要把文本输出到一个文件（写文件），构造一个PrintWriter

```java
PrintWriter out = new PrintWriter(Files.newBufferedWriter(path, charset));
```

将文本写到另外一个输出流

```java
PrintWriter out = new PrintWriter(outstream, "UTF-8");
```

将已有的变量写入文件

```java
Files.write(path, lines, charset);
```

追加内容到一个文件

- 追加 String

```java
Files.write(path, content.getBytes(charset), StandardOpenOption.APPEND);
```

- 追加 `Collection<String>`

```java
Files.write(path, lines, charset, StandardOpenOption.APPEND);
```

---

待补充
