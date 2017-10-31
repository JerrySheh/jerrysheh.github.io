---
title: 给自己的 git 备忘
date: 2017-09-04 15:43:46
updated: 2017-09-04 17:04:01
tags: git
---



# 给自己的 git 备忘

## git使用流程


### 1. 于一个目录下，初始化git


`git init`


### 2. 新建/修改文件后，把修改内容 add 到 git 上面


`git add .`


#### 三个 add 的区别

  `git add . `
  他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。

  `git add -u `
  他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区。add -u 不会提交新文件（untracked file）。（git add --update的缩写）

  `git add -A `
  是上面两个功能的合集（git add --all的缩写）

### 3. 提交到仓库


`git commit -m "modified"`


### 4. 关联一个远程仓库


`git remote add origin git@github.com:JerrySheh/repository_name.git`


### 5. 推送到远程

`git push origin branch_name`



如果远程已经有文件，需要先pull


`git pull origin master`


<!-- more -->
<br />
***

## git 分支管理


#### · 新建并切换到分支
`git checkout -b branch_name`


#### · 把新建的本地分支 push 到远程
`git push origin branch_name:branch_name`


#### · 删除远程分支
`git push origin -d branch_name`


<br />
***
## git 撤销


#### · 查看更改日志，找到你想返回去的commit_id （一般第一条是你搞错了的，第二条就是上次你想返回去的id）
`git log`

#### ·  撤销提交和修改过的代码
`git reset --hard commit_id`

#### · 只撤销提交，不撤销修改过的代码（可以直接通过git commit 重新提交对本地代码的修改。）
`git reset commit_id `
