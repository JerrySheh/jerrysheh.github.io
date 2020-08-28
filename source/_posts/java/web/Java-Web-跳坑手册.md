---
title: Java Web 跳坑手册
categories:
- Java Web
- Web
tags:
  - Java
  - Web
abbrlink: d772a9a7
date: 2018-04-11 19:41:48
---

这里有个坑，你要跳吗？

<!-- more -->

# 从 Eclipse 导入工程到IDEA

0. IDEA 选择 import  ，选择项目下的 .project 文件
1. 随便打开一个 Java 类，右上角出现 Setup SDK，选择你的JDK版本
2. Project Structure -> Modules -> Dependencies，把红色的删掉，然后点"+" -> Jars ，添加 WEB-INF 下面的jar包。再点"+" -> Libraries -> Application Libraries， 选择 Tomcat
3. Project Structure -> Facets -> "+"号 -> Web -> OK -> 上面的 Path 改为 web.xml 所在路径，下面的 Web Resource Directory 改为 WebContent 文件夹所在路径
4. 点击右下角 Create Artifacts，点击 Apply，OK
5. Edit Configurations，添加Tomcat服务器
6. Deployment选项卡，点 + ，选择 Artifacts，Apply，OK

- [IntelliJ使用指南—— 导入Eclipse的Web项目](https://blog.csdn.net/qq_15096707/article/details/51464073)


---

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

# IDEA导入SSM工程，404

2018版本的IDEA导入SSM工程可能导致部署后出现404错误。

解决办法：

File -> Settings -> Build,Execution,Deployment -> Build Tools -> Maven -> Importing

取消`Store generated project files externally`选项即可

---


# BeanFactory not initialized

从 github 获取其他人拷贝项目过来，Tomcat 无法运行，报错如下：

```
BeanFactory not initialized or already closed - call ‘refresh’ before accessing beans via the ApplicationContext
```

原因： 没有设置 resource 目录

解决办法：

1. 右键 resource 目录， 选择 mark Directory as ... 选择 test resource derectory
2. rebuild

---

# SpringBoot集成JPA报错：No identifier specified for entity

在 id 字段加 `@Id` 注解 和 `@GeneratedValue(strategy=GenerationType.IDENTITY)` 注解。

```java
@Entity
@Data
public class Message {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    Integer id;

    String message;
    Date gmtCreate;
    Date gmtModified;

}
```

几种注解的区别：

1. `@GeneratedValue(strategy = GenerationType.AUTO)`主键增长方式由数据库自动选择。
2. `@GeneratedValue(strategy = GenerationType.IDENTITY)` 要求数据库选择自增方式，oracle不支持此种方式。
3. `@GeneratedValue(strategy = GenerationType.SEQUENCE)` 采用数据库提供的sequence机制生成主键。mysql不支持此种方式。

---

```
'findById(java.lang.Long)' in 'org.springframework.data.repository.CrudRepository' cannot be applied
```
