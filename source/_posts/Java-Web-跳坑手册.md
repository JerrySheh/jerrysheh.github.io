---
title: Java Web 跳坑手册
comments: true
categories: Java Web
tags:
  - Java
  - Web
abbrlink: d772a9a7
date: 2018-04-11 19:41:48
---

这里有个坑，你要跳吗？

<!-- more -->

# 增添字段报错

新增数据库信息时，抛出 SQL Exception

```
Field 'id' doesn't have a default value
```

解决方案：

将mysql中对应表的id字段设置为自增即可 （auto_increment）

---

# 重定向出错

更新数据库信息，抛出 java.lang.IllegalStateException

```
Cannot call sendRedirect() after the response has been committed
```

解决方案：

删除重写方法的 `super.doPost(req, resp);`

---

# maven工程使用 java 8 新特性，IDEA 报错

使用了高版本java的新特性，结果报如下错误
```
Diamond types are not supported at this language level
```

这是因为 maven 工程默认 jdk 支持版本是 1.5

pom.xml 添加如下代码，更改到 1.8 即可

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <encoding>UTF-8</encoding>
    <java.version>1.8</java.version>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

---
