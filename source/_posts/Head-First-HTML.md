---
title: Head First HTML
comments: true
date: 2018-04-02 10:05:16
categories: 前端
tags:
 - HTML
---

# HTML

HTML是`HyperText Markup Language 超文本标记语言`的缩写。

HTML是由一套标记标签 （markup tag）组成，通常就叫标签

标签由开始标签和结束标签组成
<p> 这是一个开始标签
</p> 这是一个结束标签
<p> Hello World </p> 标签之间的文本叫做内容

## 最简单的HTML例子

```HTML
<html>
  <body>
    <p>Hello World</p>
  </body>
</html>
```

## 中文编码问题

```HTML
<head>
<meta http-equiv="Content-Type" content="text/html; charset=GB2312">
</head>
```

- 如果 GB2312不行，就用UTF-8
