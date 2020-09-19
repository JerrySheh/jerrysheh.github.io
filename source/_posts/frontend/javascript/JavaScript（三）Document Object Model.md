---
title: JavaScript（三）Document Object Model
abbrlink: f75e92e9
date: 2018-04-16 12:22:19
categories: 前端
tags:
 - Java Web
 - 前端
---

前面提到，完整的 Javascript 由以下三个部分组成：
- 语言基础
- BOM（Browser Object Model 浏览器对象模型）
- DOM（Document Object Model 文档对象模型）

这一篇主要讲讲 DOM ，DOM 其实就是把 html 里面的各种数据当作对象进行操作的一种思路。

<!--more-->

![pic](http://www.w3school.com.cn/i/ct_htmltree.gif)

那么 DOM 有什么作用呢？

通过可编程的对象模型，JavaScript 获得了足够的能力来创建动态的 HTML。

- 能够改变页面中的所有 HTML 元素或属性
- 能够改变页面中的所有 CSS 样式
- 能够对页面中的所有事件做出反应

---

# 改变HTML元素和属性

## 改变元素内容

把 Hello World 替换成了 WTF

```HTML
<html>
<body>

<p id="p1">Hello World!</p>

<script>
document.getElementById("p1").innerHTML="WTF";
</script>

</body>
</html>
```

## 改变元素属性

把 smiley.gif替换成了 landscape.jpg

```html
<!DOCTYPE html>
<html>
<body>

<img id="image" src="smiley.gif">

<script>
document.getElementById("image").src="landscape.jpg";
</script>

</body>
</html>
```

当用户点击按钮时，改变CSS属性

```HTML
<h1 id="id1">My Heading 1</h1>

<button type="button" onclick="document.getElementById('id1').style.color='red'">点击这里</button>
```

效果：

<h1 id="id1">My Heading 1</h1>
<button type="button" onclick="document.getElementById('id1').style.color='red'">点击这里</button>

- 使用`onmouseover=` 来让鼠标移到上面时发生事件，`onmouseout=`来让鼠标移除时发生事件
- 使用`onclick=`来让鼠标点击时发生事件
- 使用`onchange=`来让

> onmousedown, onmouseup 以及 onclick 构成了鼠标点击事件的所有部分。首先当点击鼠标按钮时，会触发 onmousedown 事件，当释放鼠标按钮时，会触发 onmouseup 事件，最后，当完成鼠标点击时，会触发 onclick 事件。

改变元素的 class

```JavaScript
document.getElementById("btn_1").className = 'btn btn-light btn-margin';
```

---

# 删除前提示

```html
<script>
    function deleteRow(link) {
        var c = confirm("确定删除吗？");
        if (!c) return;

        var table = document.getElementById("nameTable");
        var td = link.parentNode;
        var tr = td.parentNode;
        var index = tr.rowIndex;
        table.deleteRow(index);
    }
</script>

<table border="1" id="nameTable">
    <tr>
        <td>名字</td>
        <td>操作</td>
    </tr>
    <tr>
        <td>Jerry</td>
        <td>
            <a href="#" onclick="deleteRow(this)">删除</a>
        </td>
    </tr>
    <tr>
        <td>Calm</td>
        <td>
            <a href="#" onclick="deleteRow(this)">删除</a>
        </td>
    </tr>
    <tr>
        <td>Superman</td>
        <td>
            <a href="#" onclick="deleteRow(this)">删除</a>
        </td>
    </tr>
</table>
```

- 通过`document.getElementById("nameTable")`获取到这张表
- link就是this，通过`link.parentNode`和`td.parentNode`获取到所在的这一行
- 通过`tr.rowIndex`获取到这一行的下标index
- 通过 `table.deleteRow(index)` 删除这一行

---

# 登录前判断用户名或密码是否为空

```html
<body>

<script>
    function login() {
        var name = document.getElementById("name");
        if (name.value.length === 0){
            alert("用户名不能为空");
            return false;
        }

        var psw = document.getElementById("password")
        if (psw.value.length === 0){
            alert("密码不能为空")
            return false;
        }

        return true;
    }
</script>

<form method="get" action="https://www.google.com" onsubmit="return login()">
    账号：<input type="text" name="name" id="name" > <br/>
    密码：<input type="password" name="password"  id="password" > <br/>
    <input type="submit" value="登录">
    <input type="reset" value="重置">
</form>
</body>
```

效果:
<script>
    function login() {
        var name = document.getElementById("name");
        if (name.value.length === 0){
            alert("用户名不能为空");
            return false;
        }

        var psw = document.getElementById("password")
        if (psw.value.length === 0){
            alert("密码不能为空")
            return false;
        }

        return true;
    }
</script>

<form method="get" action="https://www.google.com" onsubmit="return login()">
    账号：<input type="text" name="name" id="name" >
    密码：<input type="password" name="password"  id="password" >
    <input type="submit" value="登录">
    <input type="reset" value="重置">
</form>

- 用`document.getElementById("name")`来获取账号输入框的内容
- 在表单中，用`onsubmit="return login()"`来验证登录

---

# 注册时验证年龄必须是数字 / 整数

```Javascript
var age = document.getElementById("age");
if(age.value.length==0){
  alert("年龄不能为空");
  return false;
}
if(isNaN(age.value)){
  alert("年龄必须是数字");
  return false;
}

if(parseInt(age.value)!=age.value){
  alert("年龄必须是整数");
  return false;
}
return true;
```

---

# 验证 Email 格式

```Javascript
var Regex = /^(?:\w+\.?)*\w+@(?:\w+\.)*\w+$/;      

if (!Regex.test(email.value)){               
     alert("email格式不正确");
     return false;
}           
 return true;
}
```

---

# 隐藏和显示

```Javascript
<div id="d">
  一个普通的div
</div>

<script>
function hide(){
 var d = document.getElementById("d");
 d.style.display="none";
}

function show(){
 var d = document.getElementById("d");
 d.style.display="block";
}

</script>
```

- 用`document.getElementById("d")`来获取对象
- 用`d.style.display`来设置对象的隐藏和显示
