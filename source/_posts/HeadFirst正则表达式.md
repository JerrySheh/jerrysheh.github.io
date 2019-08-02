---
title: HeadFirst正则表达式
categories: 技术&技巧
tags: linux
abbrlink: e36ce161
date: 2017-10-31 08:53:39
---

正则表达式，又称规则表达式。（Regular Expression，在代码中常简写为 regex、regexp 或 RE），计算机科学的一个概念。正则表通常被用来检索、替换那些符合某个模式(规则)的文本。

<!-- more -->

---

# 正则表达式到基本规则

字符|规则|例子
---|---|---
\|转义字符|\\\ , \\(
[xyz]|匹配包含的任意一字符|a[xyz]b,匹配 axb，ayb, azb
[a-z]|匹配指范围内到任意字符|[0-9]
[^xyz]|取反|
[^a-z]|取反|
.|匹配除了\\n之外的单个字符| abc..
^|匹配行首| ^hello
$|匹配行尾| $com
x &#124; y| 匹配x或者y（表达式）|
* | 匹配前面的子表达式零次或多次| zo*，匹配z，zo，zoo
+ | 匹配前面的子表达式一次或多次| zo+，匹配zo，zoo
? | 匹配前面的子表达式零次或一次| do(es)?，匹配do，does
{n,m}|最少匹配n次，最多匹配m次|(o{1,3})，匹配foooood中的3个o
{n}|匹配确定的n次|o{2}，匹配food中的oo，不匹配fod中的o

---

# 实例

## 实例1

匹配以 Str 开头， r 结尾， 中间任意个任意字符

```
Str.*r
```

匹配： Stringbuffer， StringBuilder

## 实例2

匹配所有以 , 结尾，修改成 comment '',

```
查找：
(.*),

替换：
$1 comment '',
```

.表示任意字符，* 表示任意多个，括号用于在下面 $1 保留原内容

效果：
```
原：
not null,
NUMBER(20),

现：
not null comment '',
NUMBER(20) comment '',
```

# Linux中使用 grep 和正则表达式，查找文件

`grep -E --color '[xyz]' filename`
