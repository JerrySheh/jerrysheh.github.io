---
title: 使用 ajax 和 Vue 进行前后端交互
abbrlink: a24bf899
date: 2018-07-28 11:33:30
categories: 前端
tags: 前端
---

![Separation](../../../../images/Webapp/Separation.jpg)

在传统的 Web 应用开发中，浏览器的请求首先会发送到 Controller， 由 Controller 交由相应的 Model 进行业务逻辑处理，处理完毕后将结果交由 View 进行渲染，动态输出完整的 HTML 返回给浏览器。这么做有两个缺点，一是需要在 HTML 中嵌入后端模板引擎，如 JSP、Thymeleaf 等。JSP饱受诟病，在HTML中写大量的java代码，显然让程序显得很混乱，无法很好地解耦。而 Thylemeaf 虽然是纯 HTML 的后端模板引擎，但这样前后端的关联性还是比较强。二是使用后端模板引擎的坏处是，HTML页面需要完全渲染完毕后，才返回给浏览器展示。这样一来，如果页面元素非常多，页面加载时间就很长了，影响用户体验不说，还会给服务器造成很大的处理压力。

<!--more-->

如果后端仅提供数据，而把渲染页面这种事交给前端来做，因为前端是在浏览器执行的，这样就能够减缓服务器的压力了。从另一个角度，随着移动互联网的兴起，客户端不再仅仅是浏览器，还可能是手机，例如Android客户端、iOS客户端、微信小程序。如果一套互联网服务需要同时有Web端、手机端，那么传统的后端包办一切的开发模式显然是行不通的。

在这种大环境下，于是催生了前后端分离。简单地说，就是把传统网络应用的 Controller 层，交给了客户端来掌控。后端只负责业务逻辑处理。当然，这里的 Controller 不是指 Spring MVC 中的 Controller，而是前端对一个页面中什么时间点应该展示什么内容，有自己的控制权。

具体到 Spring Boot 中的前后端分离，就是利用 `@RestController` 注解，让后端仅仅提供 json 格式的纯数据。前端（包括Web端和移动端）拿到数据之后，该怎么渲染，怎么展示，那是前端的事情。例如，Android客户端拿到数据，将数据显示到 RecyclerView 上， Web前端拿到数据，将数据显示在网页表格里。

如何提供一套好的规范，可以参考之前写过的 [Spring（四）使用 RESTful 风格](../post/3479b7db.html)。 RESTful 风格就是一种很好前后端数据交互的的规范。

那么我们的问题是，后端通过 RESTful 风格的设计， 向前端提供纯 json 数据，对于Web前端来说，我们该如何“拿取”这些数据，并进行展示呢？这就需要用到 Ajax 和 Vue 技术了。

---

![ajax](../../../../images/Webapp/ajax.png)

# Ajax

Ajax 的全称是 Asynchronous JavaScript and XML （异步的 JavaScript 和 XML）

Ajax 不是新的编程语言，而是一种使用现有标准的新方法。其最大的优点是提供了异步请求的能力，即在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。想象一下，如果有一个验证注册用户名是否可用的服务，按照传统的方法，前端向后端请求数据后需要刷新页面才能看到。而使用 Ajax，可以直接在当前页面更新数据内容。

## 使用原生 Ajax

原生 Ajax 的语法大概如下：

```javascript
var xmlhttp;
function check(){

  // 创建一个 http请求
  xmlhttp =new XMLHttpRequest();

  //响应函数
  xmlhttp.onreadystatechange=checkResult;

  //设置访问的页面
  xmlhttp.open("GET", url, true);   

  //执行访问
  xmlhttp.send(null);  
}

function checkResult(){
  if (xmlhttp.readyState==4 && xmlhttp.status==200){
      // 更新页面数据
      document.getElementById('checkResult').innerHTML=xmlhttp.responseText;
  }

}
```

## 使用 jQuery

现在几乎没有人用原生 Ajax 去发起 HTTP 请求，因为语法太啰嗦了，都是用一些js框架简化请求过程。使用 jQuery 发起 Ajax 大概如下：

### 发起 GET 请求

```javascript
$.get(
    url,                //参数1，url
    success: function (data) {   //参数，返回处理函数
        // data 就是服务器返回的数据
    },
    error : function() {
        alert("服务器发生错误！请稍后再试");
    }
);

```

### 发起 POST 请求

```javascript
// 提供一个 json
var loginjson = {
    "username": username,
    "password": password
};

// 将 json 变成字符串
var jsonData = JSON.stringify(loginjson);

$.ajax({
    type: "POST",  //请求方法
    contentType: "application/json;charset=UTF-8", // 要发送的数据类型
    dataType: "text", //预期服务器返回的数据类型 json text html
    url: "/api/user/login" , //url
    data: jsonData,
    success: function (data) {
        console.log(data);   //打印服务端返回的数据(调试用)
    },
    error : function() {
        alert("服务器发生错误！请稍后再试");
    }
});
```

#### javascript 中 json对象 和 字符串 的转换

- `JSON.parse(data)`：字符串 → json对象
- `JSON.stringify(data)`：json对象 → 字符串

例子：

```javascript

//定义一个字符串
var data='{"name":"goatling"}'

//变成对象​
​JSON.parse(data)  // ​name:"goatling"
```

```javascript
// 定义一个 json对象
var data={name:'goatling'}

// 变成字符串
JSON.stringify(data)  //'{"name":"goatling"}'
```

## 使用 axios

axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中。关于 promise 的内容，可以参考 [廖雪峰的javascript教程](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/0014345008539155e93fc16046d4bb7854943814c4f9dc2000)

axios 的特点是：

- 从浏览器中创建 XMLHttpRequests
- 从 node.js 创建 http 请求
- 支持 Promise API
- 拦截请求和响应
- 转换请求数据和响应数据
- 取消请求
- 自动转换 JSON 数据
- 客户端支持防御 XSRF

要使用 axios ，首先需要在 html 中引入axios.min.js

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

### 发起 GET 请求

```javascript
// 为给定 ID 的 user 创建请求
axios.get('/user?name=jerry&age=18')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });

// 可选地，上面的请求可以这样做
// param 就是 url 问号后面的参数
axios.get('/user', {
    params: {
      name: "jerry",
      age: 18
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

### 发起 POST 请求

```javascript
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

### 执行多个并发请求

```javascript
function getUserAccount() {
  return axios.get('/user/12345');
}

function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}

axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // 两个请求现在都执行完成
  }));
```

- 更多关于 axios 的内容，请参考 [Axios 中文说明
](https://www.kancloud.cn/yunye/axios/234845)


---

![Vue](https://cn.vuejs.org/images/logo.png)

# Vue.js

Vue.js是一款优秀的渐进式 JavaScript 框架。

Vue.js要系统学习起来需要花费比较大的学习成本，作为一个定位并非专业前端工程师的开发者来说，先暂且达到会用的水平就足够了。因此下面只是简要说一下我在项目中是如何使用 Vue 的。

想象一个场景，在基本页面加载完毕后，我需要异步去获取一份购买订单数据，并显示在 HTML 上。如果使用 JavaScript 或者 jQuery，我们一般是用操作 HTML DOM 的方式，把数据显示上去。例如`document.getElementById("some_id").innerHTML = data`。但如果使用Vue, 只需提供数据，以及数据要绑定到的元素的id就行了,不再需要显式地操作HTML DOM。这正是 Vue 的优势所在。

要使用 Vue.js，首先需要在 HTML 中引入：

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<!-- 开发的时候用这个 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

<!-- 生产环境版本，优化了尺寸和速度 -->
<!-- 上线的时候换成这个 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```

order.js

```javascript
// 首先 new 一个 Vue 对象
var buy_v = new Vue({

    //  绑定 HTML 的某个div
    el:'#IBuy_pageinfo_div',

    // 即将从服务器获取的数据，这里暂时为空
    data:{
        pageinfo: []
    },

    // 这个 VUE 对象拥有的方法，可以在 HTML 中调用
    methods:{
        getDate:function (strDate) {
            var d = new Date(strDate);
            return d.getFullYear() + '-' + (d.getMonth() + 1) + '-' + d.getDate();
        },
        getData(self, u){
            axios.get(u).then(function (response) {
                // 将服务器返回的结果赋值给VUE数据
                self.pageinfo = response.data;
                // ... 其他逻辑处理
            })
        }
    }
});

// 获取我的订单（购买）
function getMyBuy() {
    const url = "/api/exchange"

    // 调用 VUE 对象的方法
    buy_v.getData(buy_v._self, url);
}
```


order.html

```html
<!--购买订单 -->
<div class="tab-pane " id="panel-3">
    <div class="myOrder">
        <div class="container" style="margin-top: 20px">
            <div class="card" style="border-radius: 20px">
                <div class="card-header">我的订单</div>
                <div class="card-body">
                    <!-- 这里的 id 跟上面 VUE 的el 绑定起来  -->
                    <div class="container" id="IBuy_pageinfo_div">
                        <table class="">
                            <!-- pageinfo是服务器返回的json数据对象，list是pageinfo下的某个对象 -->
                            <tr v-for="exchange in pageinfo.list">
                                <td>
                                    <div style="width: 260px;height: 240px;background-color: white" class="container">
                                        <a :href="/product/ + exchange.product.id">
                                            <img :src="exchange.product.images[0]" style="width: 240px;height: 220px"/>
                                        </a>
                                    </div>
                                </td>
                                <td>
                                    <div class="container">
                                        <p> 商品名字：{{exchange.product.name}}</p>
                                        <p> 商品单价：{{exchange.product.price}}元</p>
                                        <p> 交易金额：{{exchange.trade_number}}件，共{{exchange.total}}元</p>
                                        <p> 成交时间：{{getDate(exchange.trade_time)}}</p>
                                        <p> 商品状态： <span style="color: green">{{exchange.status}}</span></p>
                                    </div>
                                </td>
                            </tr>
                             <!-- 循环结束 -->
                        </table>

                        <!-- 加载更多 -->
                        <div align="center">
                            <!-- 调用方法也是没问题的 -->
                            <button id="IBuy_loadmore_btn" style="color: #787878;" class="btn btn-default btn-lg" role="button"
                                    onclick="getMyBuy()">加载更多
                            </button>
                        </div>
                    </div>
                 </div>
            </div>
        </div>
     </div>
</div>
```

关于 Vue 的更多内容，参考：

- [官方文档](https://cn.vuejs.org/)
- [how2j - VUE.JS 系列教程](http://how2j.cn/k/vuejs/vuejs-start/1744.html)
