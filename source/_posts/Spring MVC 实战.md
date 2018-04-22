---
title: Spring MVC 实战
comments: true
abbrlink: 1459f5bc
date: 2018-04-18 17:09:56
categories: Java Web
tags:
 - Java
 - Web
---

在 [从MVC到Spring Boot](../post/6200df85.html) 这一篇中，搭建了 Spring Boot 的 Hello World 程序。这一篇继续学习基于 Spring MVC 框架的应用程序。应用程序由Spring Boot搭建。

<!--more-->

---

# 配置注解

- `@SpringBootApplication`：表示这是一个Spring Boot应用程序，它其实包含了`@ComponentScan`、`@Configuration`和`@EnableAutoConfiguration`等多个注解。
- `@Configuration`： 表示这是一个Java配置（相当于xml配置）
- `@EnableAutoConfiguration`：顾名思义，开启自动配置
- `@ComponentScan`：扫描包。默认扫描跟`@SpringBootApplication`所在类同级目录以及子目录。

---

# URL路由

- `@Controller`： 表示是一个处理HTTP请求的控制器(即MVC中的C)，该类中所有被@RequestMapping标注的方法都会用来处理对应URL的请求。

## 返回字符串

- `@ResponseBody`：直接将函数的返回值传回到浏览器端显示。

```java
@Controller
public class IndexController {

    @GetMapping("/")
    public String index() {
        return "index";
    }

    @RequestMapping("/hello")
    @ResponseBody
    public String hello() {
        return "hello";
    }
}
```

- `@RequestMapping`： 浏览器请求映射路由，可以标记在类上
- `@GetMapping`： 浏览器 GET 方法请求映射路由
- `@PutMapping`
- `@PostMapping`
- `@DeleteMapping`

```java
@Controller
@RequestMapping("/blogs")
public class AppController {

    @RequestMapping("/create")
    @ResponseBody
    public String create() {
        return "mapping url is /blogs/create";
    }
}
```

此时，`create()`方法绑定的路由是 /blogs/create

## 返回HTML文件

在上面的例子中，`index()`方法没有被`@ResponseBody`标记，可以在resource/templates 目录下放置一个 index.html， 然后在 pom.xml 增加 thymeleaf 依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency> <!-- 为了让Thymeleaf能够正确的识别HTML5 -->
  <groupId>net.sourceforge.nekohtml</groupId>
  <artifactId>nekohtml</artifactId>
  <version>1.9.22</version>
</dependency>
```

这样，访问 127.0.0.1:8080/ 的时候自动跳转到 index.html

> Thymeleaf 遇到没有闭合的HTML标签会报错，可以在 application.properties 文件增加一行 `spring.thymeleaf.mode=LEGACYHTML5` 以支持HTML5


在 HTML中引入 CSS和 JavaScript

```html
<link rel="stylesheet" href="/css/style.css"/>
<script src="/js/main.js"></script>
```

---

# PathVariable 和 RequestParam

## 使用PathVariable获取路径变量

```java
@GetMapping("/users/{username}")
public String userProfile(@PathVariable String username) {
    return String.format("user %s", username);
}
```

可以使用下列形式来匹配正则表达式，语法`{变量名:正则表达式}`，不匹配的URL将不会被处理，直接返回404

```java
@GetMapping("/users/{username:[a-z0-9_]+}")
```

> [a-z0-9_]+是一个正则表达式，表示只能包含小写字母、数字和下划线。

## 使用 RequestParam 获取参数

有时候我们需要处理 URL 中的参数 ，比如`127.0.0.1:8080/userinfo?key1=value1&key2=value2`

```java
@RestController
public class EditPetForm {

    @GetMapping("/blogs")
    public String setupForm(@RequestParam("id") int blogId) {
        return String.format("blog id = %d", blogId);
    }

}
```

访问 127.0.0.1:8080/blogs?id=66， 可以看到浏览器显示了 blog id = 66

- 使用`@RequestParam(name = "id", required = false, defaultValue = "1")`，当参数不存在的时候，默认为1

## 如何选择

两种方式都能获取用户输入

- 通过@PathVariable，例如/blogs/1
- 通过@RequestParam，例如blogs?blogId=1

建议：

- 当URL指向的是某一具体业务资源（或者资源列表），例如博客、用户时，使用`@PathVariable`
- 当URL需要对资源或者资源列表进行过滤、筛选时，用`@RequestParam`

例如用/blogs?state=publish而不是/blogs/state/publish来表示处于发布状态的博客文章

---

# 模板渲染

## 使用Thymeleaf

使用`Thymeleaf`作为模板引擎，在需要动态获取的地方用Thymeleaf标签替代。如

```html
<h2 th:text="${title}">博客标题</h2>
<span th:text="${createdTime}">
```


在HTML中增加命令空间，避免IDE错误提示

```html
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">
    ...
</html>
```

在Thymeleaf添加 `${param}` 变量后，需要在 Controller 做一些工作

```java
@GetMapping(value = "/bookList")
public String getBookList(ModelMap map){
    map.addAttribute("bookList", bookService.findAll());
    return "bookList";
}
```

这样就会把 `bookService.findAll()` 的结果（ `List<book>` 类型）传给 bookList.html


在 bookList.html 用 `th:each`遍历这个 List
```html
<thead>
<tr>
    <th>书籍编号</th>
    <th>书名</th>
    <th>描述</th>
</tr>
</thead>
<tbody>
<tr th:each="book: ${bookList}">
    <th scope="row" th:text="${book.id}"></th>
    <td th:text="${book.bookName}"> 书名</td>
    <td th:text="${book.bookDescription}">简介</td>
</tr>
</tbody>
```

---

# 全局配置

在项目 resource 目录下，修改`application.properties`文件。

```
## 服务器配置
server.port = 8089
server.context-path=/test

## 数据源配置
spring.datasource.url=jdbc:mysql://localhost:3306/springbootdb?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

## Mybatis 配置（仅xml方式）
mybatis.typeAliasesPackage=org.spring.springboot.domain
mybatis.mapperLocations=classpath:mapper/*.xml

## thymeleaf 配置
spring.thymeleaf.suffix=.html
spring.thymeleaf.prefix=classpath:templates
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.cache=false
spring.thymeleaf.mode=LEGACYHTML5
```
