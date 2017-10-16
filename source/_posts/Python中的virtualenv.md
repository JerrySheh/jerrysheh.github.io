---
title: Python 中的 virtualenv
date: 2017-10-16 21:16:30
tags: Python
---

开发 Python 应用程序的时候，需要安装（import）各种各样的第三方包。默认情况下，都会被安装到Python 3的 site-packages 目录下面。比如，我的第三方包统一安装在目录`C:\Program Files\Python36\Lib\site-packages`下面。

但是，当我们开发多个项目的时候，如果应用A需要jinja 2.7，而应用B需要jinja 2.6怎么办？

virtualenv 就是用来为特定的应用程序创造一套独立的运行环境的。

如何你用的是Python 3，还可以直接创建虚拟环境（见第二部分）

<!-- more -->

# 一. 使用 virtualenv

## 1. 安装

```
pip3 install virtualenv
```

## 2. 创建一个工程目录

```
mkdir webApp
cd webApp
```

## 3. 创建独立Python运行环境

```
virtualenv venv
```

## 4. 进入该环境

Linux / Mac
```
source venv/bin/activate
```

Windows
```
.\venv\Scripts\activate
```

此时命令行前面出现了 `(venv)` 表示已经进入独立的虚拟Python运行环境

```
(venv) D:\Python\new\venv>
```


此时可以用 pip 安装该项目所需的 Python 包

```
(venv) D:\Python\new\venv> pip install jinja2
```


## 5. 退出venv环境

```
(venv) D:\Python\new\venv> deactivate

D:\Python\new>
```

## 6. 删除venv环境

删除目录即可


---

# 二. 使用 Python 3 自带的venv

virtualenv 可用在 python 2 或 python 3， 但如果是 python 3 项目， 其实还可以使用 python 3 自带的  venv

## 1. 创建虚拟环境

```
python -m venv myvenv
```

默认是干净的环境，如果虚拟环境中需要使用系统的环境，可用

```
python -m venv --system-site-packages myvenv
```
使虚拟环境指向系统环境包目录（非复制），在系统环境pip新安装包，在虚拟环境就可以使用。

## 2. 激活虚拟环境

不同的命令行工具有不同的激活方法：

bash/zsh
```
source <venv>/bin/activate
```

fish
```
<venv>/bin/activate.fish
```

csh/tcsh
```
<venv>/bin/activate.csh
```

cmd
```
<venv>\Scripts\activate.bat
```

PowerShell
```
<venv>\Scripts\Activate.ps1
```

这里的 `<venv>` 指的是刚刚执行创建虚拟环境的目录

## 3. 关闭虚拟环境

```
deactivate
```

## 4. 删除虚拟环境

直接删除目录即可
