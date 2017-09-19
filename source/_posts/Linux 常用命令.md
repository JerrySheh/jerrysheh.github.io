---
title: linux 常用命令
date: 2017-09-12 19:28:00
updated:  2017-09-12 10:40:00
tags: linux
---

# linux 常用命令

<!-- more -->
## 常用命令

  命令| 说明
  ---|-----
  ls | 显示当前目录下文件
  ls -a|显示当前目录包括隐藏的文件
  mkdir | 创建目录
  rmdir | 删除空目录
  pwd | 显示当前目录
  cd| 切换目录
  touch|新建文件
  echo | 创建带有内容的文件
  cat| 查看文件内容
  cp | 拷贝
  mv | 移动或重命名
  rm | 删除文件
  rm -r| 删除包括子目录
  rm -f| 强制删除
  apropos -e “list directory contents”| 精确反查带有”“list directory contents”功能的命令
  whatis ls| 显示ls命令的作用
  man ls| 显示ls命令的手册（space翻页 j下行 k上行 /关键字搜索 n下一个关键字 shift+n上一个关键字）


  ## 快捷操作

  命令| 说明
  ---|-----
  ctrl+a | 定位到命令开头
  ctrl+e| 定位到命令结尾
  ctrl+ ←| 定位到上一个单词


## vim

  命令| 说明
  ---|-----
  :q | 退出
  :q! | 强制退出
  :wq | 保存并退出
  :set number | 显示行号
  :set nonumber | 隐藏行号
  /apache | 在文档中查找apache, n 下一个，shift+n 上一个
  yyp | 复制光标所在行，并粘贴
  h | 左移
  j | 下一行
  k | 上一行
  l | 右移

## 正则表达式

  命令| 说明
  ---|-----
  ×| 匹配所有
  ？| 匹配一个字符
