---
title: Spring（四）使用 RESTful 风格
comments: true
categories: Java Web
tags: Java Web
abbrlink: 3479b7db
date: 2018-07-05 16:16:42
---

![RESTful](http://www.runoob.com/wp-content/uploads/2015/07/restful.gif)

# 什么是RESTful风格？

REST是 REpresentational State Transfer 的缩写（一般中文翻译为表述性状态转移），REST 是一种体系结构，而 HTTP 是一种包含了 REST 架构属性的协议，为了便于理解，我们把它的首字母拆分成不同的几个部分：

- 表述性（REpresentational）： REST 资源实际上可以用各种形式来进行表述，包括 XML、JSON 甚至 HTML——最适合资源使用者的任意形式；
- 状态（State）： 当使用 REST 的时候，我们更关注资源的状态而不是对资源采取的行为；
- 转义（Transfer）： REST 涉及到转移资源数据，它以某种表述性形式从一个应用转移到另一个应用。

简单地说，REST 就是**将资源的状态以适合客户端或服务端的形式从服务端转移到客户端（或者反过来）** 。

在 REST 中，资源通过 URL 进行识别和定位，然后通过行为(即 HTTP 方法)来定义 REST 来完成怎样的功能。

<!--more-->

## 实例说明

在HTTP协议中，有GET、POST、HEAD、PUT、DELETE、OPTIONS、TRACE、CONNECT 众多方法，我们可以通过不同的HTTP方法来对应CRUD的操作。

例如：

HTTP方法|CRUD操作
---|---
GET| 查询(Retrieve)
POST| 更新(Update)
PUT| 增加(Create)
DELETE | 删除(Delete)

尽管通常来讲，HTTP 方法会映射为 CRUD 动作，但这并不是严格的限制，有时候 PUT 也可以用来创建新的资源，POST 也可以用来更新资源。实际上，**POST 请求非幂等的特性(即同一个 URL 可以得到不同的结果)** 使其成一个非常灵活地方法，对于无法适应其他 HTTP 方法语义的操作，它都能够胜任。

在使用 RESTful 风格之前，我们如果想要增加一条商品数据通常是这样的:

```
/addCategory?name=xxx
```

但是使用了 RESTful 风格之后就会变成:

```
/category
```

这就变成了使用同一个 URL ，通过约定不同的 HTTP 方法来实施不同的业务，这就是 RESTful 风格所做的事情了。

![RESTful](../../../../images/Webapp/RESTful.png)

> 图片引自 how2j.cn

---

# Springboot 实战

RESTful API设计

请求类型|URL|说明
---|---|---
GET|/students|查询所有学生
POST|/students|创建一个学生
GET|/students/{id}|根据id查询一个学生
PUT|/students/{id}|根据id更新一个学生
DELETE|/students/{id}|根据id删除一个学生

实体类 Student.java

```java
public class Student {
    Integer id;
    String name;
    String number;
    String major;
    Integer grade;
    // 省略 setter getter
}
```

控制器

```java
@RestController
@RequestMapping(value = "/students")
public class StudentAPIController {

    @Autowired
    StudentMapper studentMapper;

    // 查询所有学生
    @GetMapping("/")
    public List<Student> getStudentList(){
        return studentMapper.findAll();
    }

    // 增加学生
    @PostMapping("/")
    public Student addStudent(@ModelAttribute Student student){
        studentMapper.addStudent(student);
        return student;
    }

    // 查询学生
    @GetMapping("/{id}")
    public Student getStudent(@PathVariable Integer id){
        return studentMapper.findById(id);
    }

    // 删除学生
    @DeleteMapping("/{id}")
    public String deleteStudent(@PathVariable Integer id){
        studentMapper.deleteById(id);
        return "redirect:/";
    }

    // 更新学生
    @PutMapping("/{id}")
    public Student updateStudent(@PathVariable Integer id, @ModelAttribute Student student){
        studentMapper.updateStudent(student);
        return student;
    }

}
```


---

参考
- http://www.cnblogs.com/wmyskxz/p/9104368.html
- http://blog.didispace.com/springbootrestfulapi/
