---
title: git 创建本地分支管理远程分支
comments: true
toc: true
mathjax2: true
date: 2017-08-09 17:51:00
categories: [technology,tools]
tags: [tools,git]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
---
本地Git仓库和远程仓库的创建及关联
 <!--more-->
### 创建远程仓库
界面操作，没说的
切记:如果我们在创建远程仓库的时候添加了README和.ignore等文件,我们在后面关联仓库后,需要先执行pull操作

###　创建本地分支
git init
注意:Git会自动为我们创建唯一一个master分支
我们能够发现在当前目录下多了一个.git的目录，这个目录是Git来跟踪管理版本库的，千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。

### 添加远程分支
```
git remote add origin git@github.com:muyunzhe/helloTest.git
```
备注:origin就是我们的远程库的名字，这是Git默认的叫法，也可以改成别的;
git@github.com:YotrolZ/helloTest.git是我们远程仓库的路径(这里我们使用的github)

### 本地创建文件并提交
```
vim ***
add .
git commit -m 'xxx'
```

### 拉远程分支
原因上面已经提到了
用命令
```
git checkout -b init
git pull origin init
```

### 本地推送到远程
```
git push --set-upstream origin init
```
备注：这时候已经讲本地的init分支跟远程的init分支建立关联了，以后的push操作在init分支下直接用```git push```就可以默认push到远程的init分支了
