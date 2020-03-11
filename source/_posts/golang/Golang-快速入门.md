---
title: Golang 快速入门
comments: true
categories: Golang
tags: Golang
abbrlink: dcaae91f
date: 2019-04-28 19:28:02
---

![](../../../../images/golang/GO.png)

这几天闲来无事，想试着玩下 Golang。对于 Go 的认知，最开始是在知乎上看到一些相关的讨论，大概知道这是一门2009年由谷歌工程师捣鼓出来的新语言，其最大的特点是在语言层面支持并发（goroutine），因此天生适合用来做服务端开发。

> **学 Golang 只是为了好玩！**

参考教程

- [Go 语言指引](https://tour.go-zh.org/)
- [官方文档](https://go-zh.org/doc/)
- [标准库手册](https://go-zh.org/pkg/)

---

<!-- more-->

# 基本语法

## Hello World

1. 不需要分号
2. 已导出的方法调用以大写字母开头（Println）

hello.go

```go
package main

import (
    "fmt"
)

func main()  {
    fmt.Println("hello world!")
}
```

编译执行

```
C:\Users\Gopher\go\src\hello> go build
C:\Users\Gopher\go\src\hello> hello
```

参考：https://golang.google.cn/doc/install

## 声明变量、常量

跟C++这类语言不同，Go的类型放在变量名后面。之所以这么写，Go官方在 [一篇博客](https://blog.golang.org/gos-declaration-syntax) 中说是为了更加简洁清晰，特别是加入指针后不那么容易混淆。

```go
package main

import "fmt"

// 声明全局变量
var status bool
var j float64
var i = 4

func main()  {
    var a = 2 // 声明函数变量
    b := 1    // 简洁方式
}
```

## 函数

1. 函数支持多值返回。
2. 返回值可以命名，相当于在函数首行声明了这个变量。

```go
// 函数
func add(x, y int) int{
    return x + y
}

// 支持多值返回
func swap(x, y int) (int, int){
    return y, x
}

// 返回值可以命名
func splitNum(num int) (x, y int){
    x = num + 5
    y = num - 5
    return
}

func main()  {
    a := 1
    b := 2
    a, b = swap(a, b)
    a, b = splitNum(50)
}
```

## 类型转换

简单粗暴

```go
// 类型转换
func cast(){
    var x int = 1
    y := float64(x)
    fmt.Println("the type of y is: ", reflect.TypeOf(y) )
}
```

## for循环

C 语言中的 while 在 Go 里面叫做 for

```go
func forLoop(){
    sum := 0
    for i := 1; i < 100; i++ { // 普通for循环
        sum = sum + i
    }

    num := 1
    for num < 2000 {    // 省略前置后置
        num++
    }

    v := 0
    for{     // 无限循环
        v++
    }

}
```

## if 语句

if语句可以简短声明表达式

```go
func ifStatement(a, b int){
    if v:=a * b; v < 20 {
        fmt.Println(v)
    } else {
        fmt.Println("wrong")
    }
}
```

## defer

1. defer语法，推迟到外层函数结束后执行
2. defer会立即求值，但推迟调用，多个 defer 以压栈的方式后进先出

以下程序输出： `three two one`

```go
func deferStament(){
    defer fmt.Println("one")
    defer fmt.Println("two")
    fmt.Println("three")
}
```

## 指针

跟C++一样，`&`是取地址符号

```go
func pointer() {
    var i = 66  // 一个 int
    var p *int  // 一个指针
    p = &i      // 指针指向 int
    fmt.Println(p)
}
```

输出：`0xc000054080`，即变量 i 所在内存中的地址

在指针变量前加 `*` 可以获得原变量

```go
func pointer2() {
    var i int
    p := &i
    *p = 6 // 通过指针设置 i 的值
    fmt.Println(i)
}
```

与 C++ 不同的是，Go 的指针不能用于运算。

## struct

跟 C 好像没啥区别

```go
type rectangle struct {
    long int
    width int
}

func main() {
    rec := rectangle{3,4}
    rec.long = 5
    fmt.Println(rec.long, rec.width)
}
```

## struct pointer

在 C++ 中，p是指向结构体的指针，可以用 `(*p).X` 来访问结构体中的变量。在 Go 中也可以这么干，但 Go 也允许我们直接用 `p.X` 来访问。

```go
p := &rec
p.width = 4 // 相当于 rec.width = 4
```

如果不指定结构体变量的值，会给默认值

```go
var (
    v1 = Vertex{1, 2}  // 创建一个 Vertex 类型的结构体
    v2 = Vertex{X: 1}  // Y:0 被隐式地赋予
    v3 = Vertex{}      // X:0 Y:0
    p  = &Vertex{1, 2} // 创建一个 *Vertex 类型的结构体（指针）
)
```

---

# 稍微高级一点的语法

## 数组(array)和切片(slice)

在 Go 中，数组是不可变的。声明数组时必须指定长度，`[3]int{}`和`[2]int{}`是两种不同的类型。不声明长度则叫做切片，可以把切片理解为数组的一段（有时是全部）。通常用切片操作数组。

```go
var names = [4]string{
    "John",
    "Paul",
    "George",
    "Ringo",
} // 声明数组的一种方法

func main() {
    primes := [...]int{2, 3, 5, 7, 11, 13} // 也可以用 ... 让编译器帮你计算长度
    fmt.Println(primes)

    primes[2] = 1 // 修改数组第 3 个元素值

    sub := primes[1:4] // 切片，获取数组的第 1-3 个元素
}
```

来自官方文档：

1. 切片是数组的片段，它并不存储任何数据，更改切片的元素会修改其底层数组中对应的元素。共享同一数组的切片都会看到这些修改。
2. 切片下界的默认值为 0，上界则是该切片的长度。
3. 切片的长度就是它所包含的元素个数。切片的容量是从它的第一个元素开始数，到其底层数组元素末尾的个数。

对于数组`var a [10]int`来说，以下切片是等价的：

```
a[0:10]
a[:10]
a[0:]
a[:]
```

### 使用 make 创建切片

make 传入两个或三个参数，底层数组类型，长度，容量（可选）
```go
func cut() {
    b := make([]string, 2)
	c := make([]int, 3, 5)
}
```

### 使用 append 为切片添加元素

```go
a := []int{1,2,3}
fmt.Println(a) // [1 2 3]

a= append(a, 6,7,8)
fmt.Println(a) // [1 2 3 6 7 8]
```

当 s 的底层数组太小，不足以容纳所有给定的值时，它就会分配一个更大的数组。返回的切片会指向这个新分配的数组。

### 使用 for range 遍历切片

for range 输出两个值，一个当前元素的下标，一个当前元素的值。

```go
func cut() {
	s := []int{1,2,4,8,16}
	for i, v:=range s{
		fmt.Printf("切片的第 %d 个元素为 %d \n", i, v)
	}
}
```

输出：

```
切片的第 0 个元素为 1
切片的第 1 个元素为 2
切片的第 2 个元素为 4
切片的第 3 个元素为 8
切片的第 4 个元素为 16
```

如果不需要下标，或不需要值，用 _ 代替 i 或 v 即可忽略。

```go
for _, v:=range s{

}

for i, _:=range s{

}
```

## 映射

使用 `map[string]int` 这样的语法来创建一个 map 键值对，[]括号里面是 key 的类型，后面是 value 的类型。

```go
func test() {
	m := make(map[string]int)

	m["a"] = 42
	m["b"] = 43
	fmt.Println(m["b"])  // 43

	elem := m["a"]
	fmt.Println(elem) // 42
}
```

用双赋值检测某个键是否存在。如果存在，第一个值为键值对的value，第二个值为 true ，如果不存在，第一个值为 value 的零值，第二个值为 false。

```go
e, ok := m["a"]
fmt.Println(e, ok) // 42 true

e, ok := m["d"]
fmt.Println(e, ok) // 0 false
```

用 delete 删除元素

```go
delete(m, "a")
```

一个 wordcount 小程序

```go
func WordCount(s string) map[string]int {
	m := make(map[string]int)
	for _, key:=range strings.Fields(s){
		m[key]++
	}
	return m
}
```

## 闭包

Go 支持闭包，即函数可以作为值，函数也可以返回一个函数。

用闭包实现斐波那契数列（0,1,1,2,3,5,8,13,21,34...）

```go
// 返回一个“返回int的函数”
func fibonacci() func() int {
	first := 0
	second := 1
	return func() int {
		result := first
		first, second = second, first+second
		return result
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

f 是一个“返回int的函数”，f() 是 f 的调用，即一个 int 值。闭包的作用在于“内部封闭外部”，即变量在外部作用域结束之后，还保留一份在内部。

---

# 方法和接口

## 方法

在 Golang 中，没有类，但结构体是一样的。Java的类里面可以定义方法，Go可以为结构体定义方法。例如：

一个长方形结构体

```go
type Rectangle struct {
	long float64
	width float64
}
```

为其定义求面积的方法，

```go
func (r Rectangle) Area() float64  {
	return r.long * r.width
}

func main() {
	r := Rectangle{3.5, 1.8}
	area := r.Area()
}
```

事实上，方法跟函数是一样的。上面的方法可以改写成函数。

```go
func getArea(r Rectangle) float64  {
	return r.long * r.width
}

func main() {
    r := Rectangle{3.5, 1.8}
	area := getArea(r)
}
```

当然，也可以直接操作指针(通常的做法)

```go
// 将长方形的长和宽扩大两倍
func (r *Rectangle) scala()  {
    r.width = 2 * r.width
    r.long = 2 * r.long
}

func main()  {
	r := Rectangle{3.4,2.1}
	r.scala()
}
```

## 接口

如果一个类型，实现了一个接口所定义的所有方法，那就说这个类型实现了这个接口，并不用显式去声明。

```go
// 一个图形接口
type shape interface {
	Area() float64
    Perimeter() float64
}

// 长方形
type Rectangle struct {
	long float64
	width float64
}

// 长方形求面积
func (r Rectangle) Area() float64  {
	return r.long * r.width
}

// 长方形求周长
func (r Rectangle) Perimeter() float64  {
	return r.long * 2 + r.width * 2
}
```

在 Go 中，接口也是可以作为参数或者返回值的。

---

# 错误

Go 使用 `error` 表示错误。error 本质上是个接口：

```go
type error interface {
    Error() string
}
```

通常一个方法会返回一个错误，如果这个错误不为空，则说明它发生了错误。

```go
// 一个 http 请求
http.HandleFunc("/", sayHello)

// 错误
err := http.ListenAndServe(":9090", nil)

// 如果检测到错误，执行对应的动作
if err != nil {
    log.Fatal("ListenAndServe: ", err)
}
```

---

# 并发

Go 语言最大的特点就是 goroutine 了

```go
func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}
```

`go say("world")` 会发起一个新的协程执行，而`say("hello")` 在 main 主程里执行，所以是并发执行的。

## 信道（channal）

信道非常适合在各个 Go 协程间进行通信。箭头就是数据流动的方向。

```go
// 创建一个接收或发送 int 型的信道
ch := make(chan int)

ch <- v    // 将 v 发送至信道 ch。
v := <-ch  // 从 ch 接收值并赋予 v。
```

官方示例

```go
// 求和
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // 将和送入 channal
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}
	c := make(chan int)

	go sum(s[:len(s)/2], c) // 计算前半部分和
	go sum(s[len(s)/2:], c) // 计算后半部分和

	x, y := <-c, <-c // 从 c 中接收
    sum := x + y // 合并
}
```

用 for range 来不断地从信道获取值，用双值检测信道是否被关闭。

```go
// 若没有值可以接收且信道已经被关闭，ok会被置为false
v, ok := <-ch

for i := range c {
    fmt.Println(i)
}
```


---

# IDE 使用

![](http://resources.jetbrains.com/storage/products/goland/img/meta/goland_logo_300x300.png)

我使用的是 JetBrain 全家桶的 Goland。有几个注意点：

1. main 函数一定要在 main package 里面
2. 如果同包不同文件互相调用，编译的时候记得在 Configuration 的 Run Kind 选择 Package
