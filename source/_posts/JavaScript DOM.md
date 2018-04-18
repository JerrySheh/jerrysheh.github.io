---
title: JavaScript DOM
comments: true
abbrlink: f75e92e9
date: 2018-04-16 12:22:19
categories: Java Web
tags:
 - Java Web
 - 前端
---

前面提到，完整的 Javascript 由以下三个部分组成
- 语言基础
- BOM(Browser Object Model 浏览器对象模型)
- DOM（Document Object Model 文档对象模型）

这一篇主要讲讲 DOM ，DOM 其实就是把 html 里面的各种数据当作对象进行操作的一种思路。

<!--more-->

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
