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
