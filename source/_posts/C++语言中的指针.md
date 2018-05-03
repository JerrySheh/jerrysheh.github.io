---
title: C/C++语言中的指针
comments: true
categories: C/C++
tags: C/C++
abbrlink: 82d9a37c
date: 2018-03-25 18:57:20
---

这几天在接触一些C语言的项目，发现自己对C语言，包括C++的知识理解不透彻，尤其是指针。导致项目完全看不懂。因此这一篇就来补补C/C++中指针的知识。

参考书籍《C++ Primer》

---

# 指针

## 简单比喻

什么是指针，假如你住5楼503号房间。那么有一张纸条，纸条上写着5楼503。那么，我们就说这张纸条就是指向你房间的一个指针。

## 定义

指针（Pointer）是指向（Point to）另外一种类型的复合类型。

指针有两个特点：
- 本身是一个对象，允许对指针进行赋值和拷贝，而且在指针的生命周期内可以先后指向几个不同的对象。
- 无须在定义时赋初始值

## 一个简单例子

```
#include <stdio.h>

int main() {
    int ival = 42;
    int *p = &ival;

    printf("p是一个指针，P为%p\n",p);
    printf("*p是指针指向的变量，*p为%d\n",*p);
    return 0;
}

```

输出

```
p是一个指针，P为0x7ffe09031b1c
*p是指针指向的变量，*p为42
```

在这个例子中，我们用 `int *p`来定义指针，这时候`p`是一个指针。`&ival`的意思是取`int`型变量`ival`的地址。

而在输出的时候，`*p`是指针指向的变量，也就是说，`*`号在这里成了解引用符号。仅在指针确实指向了某个对象，即指针确实有效的情况下，`*p`才适用。

> 理解的关键：在定义阶段， 用`int *p`用来定义指针。在操作阶段，`*p`是解引用。

<!-- more -->

## 空指针

```
int *p1 = nullptr; // only C++11
int *p2 = 0;       // ok
int *p3 = NULL;    // include cstdlib
```

第一种方法仅在C++11中可用，也是C++编程中最推荐的；第二种是最常见的，直接给指针赋值0，即是空指针；第三种 NULL 在 cstdlib 中定义，其实 NULL 的值就是0；

假设p是一个指针，那么以下做法是错误的，即使zero等于0：

```
int zero = 0;
p = zero;
```

> 一个编程建议是，指针一定要初始化，最好是先有对象（变量），然后再去定义指向这个对象的指针。假设真的要在定义对象前定义指针，不知道让它指向哪里，那就初始化为0或者nullptr。不要让它悬空。

## 赋值

有时候我们会搞混究竟是改变了指针本身，还是改变了指针指向的对象。一个好方法是，**改变的永远是等号左边的**。

```
pi = &val;    // 指针指向了val的地址
*pi = 0;      // 指针没变，但是指针指向的对象值变为0了
```

## void* 指针

```
double mynum = 3.66;
double *pd = &mynum;  // pa指向mynum

int mynum2 = 9;
void *pv = &mynum2;  // pv指向mynum2

pv = &mynum1;    // pv现在又指向mynum1了
```

从上面的例子可以看到， `void *`型指针跟普通指针也没什么区别。但是，`void *`指针可以指向任何类型。当然，它不能用于操作指针所指的对象，因为我们不知道这个对象的类型（`void *`一会儿可指向int型变量的地址，一会儿可以指向double型）

## 指向指针的指针

```
#include <stdio.h>

int main() {
    int val = 1024;  // 一个int
    int *pi = &val;  // 一个指向val的指针
    int **ppi = &pi; // 一个指向pi的指针

    printf("1) val是一个int，值为%d\n\n", val);
    printf("2) pi是一个指向int的指针，pi为%p\n\n",pi);
    printf("3) ppi是一个指向指针的指针，ppi为%p\n",ppi);
    printf("4) *ppi其实就是 2) 中的pi,*ppi为%p",*ppi);
    return 0;
}
```

不要被**ppi 吓到了，其实它就是指向了pi这个对象。只不过恰好pi这个对象也是一个指针罢了。

输出：

```
1) val是一个int，值为1024

2) pi是一个指向int的指针，pi为0x7ffc86a9d6b4

3) ppi是一个指向指针的指针，ppi为0x7ffc86a9d6b8
4) *ppi其实就是 2) 中的pi,*ppi为0x7ffc86a9d6b4
```


可以看到， `*ppi`其实就是`pi`，他们都是0x7ffc86a9d6b4


## 指向常量的指针

C语言中的`const`限定了对象不能被改变，一把用来表示常量。相当于 java 的 `final`。

而指向常量的指针（point to const），不能用于改变其所指对象的值。point to const一般用来存放常量的地址。

有一点值得注意，允许一个指向常量的指针指向非常量，但是却不能通过这个指针操作这个非常量。

```
const double d = 3.33;
const double *pd = &d;

double x = 6.66;
pd = &x;   // ok，但是不能通过pd去改变 x 的值
```

## 常量指针

指针是对象，跟int、double等一样，所以可以用 `const int`来表示常量， 那当然也可以用 `*const int`来表示常量指针啦。

只是，一旦定义了`*const int`，那这个指针必须初始化，且只能指向初始化的这个地方，不能再改变了。

注意下面的定义

```
int i = 6;
int *const p1 = &i;  //常量指针，不能改变p1所指的对象
const int *p2 = &i  
```

---

# p->mem 是什么意思

有时候我们会看到 `p->mem` 这种用法，实际上它等价于 `(*p).mem`。

```
#include <stdio.h>

int main() {

    typedef struct {
        int x;
        int y;
    } Point;

    Point pos;
    pos.x = 10;
    pos.y = 5;
    printf("answer1:%d\n", pos.x * pos.y);

    Point* pPos = &pos;
    (*pPos).x = 15;
    pPos->y = 20;

    printf("answer2:%d\n", pos.x * pos.y);

}
```

输出
```
answer1:50
answer2:300
```

首先定义了一个`Point`结构体，包含 int 类型的 x, y。然后实例化·

我们当然可以用`pos.x = 10`，`pos.y = 5`这样的方式来给结构体的每个变量赋值。

但有时候我们要用指针操作对象，我们可以先定义一个`Point *`类型的指针`pPos`。

然后查看两种用指针间接给结构体赋值的方法：
1. (\*pPos).x = 15;
2. pPos->y = 20;

第一种是先将指针 `pPos` 解引用，让它变成指向的对象（`pos`）， 然后用 对象.成员 的方式来赋值。第二种直接在指针`pPos`上操作，也就是用`->`来表示，对指针指向的结构体对象（`pos`）的某个成员(`y`)进行操作。

换言之，
* `.` 直接成员访问操作符，但操作前需对指针解引用
* `->` 间接成员访问操作符

实质上两种方式是等价的。

---

# 结构体和指针

## 定义结构体

在C语言中，我们可以这样定义结构体：

```C
//方法一：
struct student{
    short age;
    char name[MAXNAME];
    long phoneNumber;
};

struct student s1;  // s1是student类型的一个实例

//方法二：
typedef struct student{
    short age;
    char name[MAXNAME];
    long phoneNumber;
}STUDENT;

STUDENT s2; // s2是student类型的一个实例
```

可见，用方法二比较方便一点。

## 用指针访问结构体成员

```
typedef struct{
    short age;
    char name[MAXNAME];
    long phoneNumber;
}STUDENT;

STUDENT s2; // s2是student类型的一个实例

student *ps = &s2;
ps->age = 6;
printf("%d\n",s2.age);
```

如果要给结构体的 name[MAXNAME] 赋值，下面的做法是错误的

```C
ps->name = "jerry";
```

应该用strcpy函数。

```
char *name = "jerry";
strcpy(ps->name, name);
printf("%s\n",stu1.name);
```

## 结构体偏移量问题

假设现在有一个结构体

```
struct fun{
    int a;
    int b;
    char c;
};
```

我们已知道结构体成员 c 的地址，如何求结构体的起始地址呢？

答案就是：`(char *) & ((struct fun*)0)->c`

```
int main(){
    //实例化一个结构体变量
    struct fun domain;

    //结构体起始地址
    printf("iic:%u\n",&domain);

    //结构体成员 c 的地址
    printf("char c:%u\n", &(domain.c));

    //偏移量
    printf("sub: %d\n\n", (char *) & ((struct fun*)0)->c);

    return 0;
}
```

输出：

```
jerrysheh@ubuntu:~$ ./fun
fun:-1838605600
char c:-1838605592
sub: 8
```
