---
title: HeadFirst正则表达式
date: 2017-10-31 08:53:39
tags: linux
---

正则表达式，又称规则表达式。（英语：Regular Expression，在代码中常简写为regex、regexp或RE），计算机科学的一个概念。正则表通常被用来检索、替换那些符合某个模式(规则)的文本。 ——百度百科


<!-- more -->

---

# 一、正则表达式到基本规则

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


# Linux中使用 grep 和正则表达式，查找文件

`grep -E --color '[xyz]' filename`
