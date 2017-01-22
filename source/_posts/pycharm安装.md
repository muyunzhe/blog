---
title: pycharm安装
comments: true
date: 2016-05-23 11:43:56
categories: [technology,tools]
tags: [tools,python]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
---

PyCharm是JetBrains系列产品的一员，也是现在最好用的IDE。
<!--more-->
PyCharm维持了JetBrains一贯高度智能的作风，简要枚举如下：

独特的本地VCS系统

强大的重构功能

基于上下文的智能代码提示和纠错

可以与IDEA、PhpStorm等IDE共享配置文件

PyCharm社区版免费下载地址：http://www.jetbrains.com/pycharm/

安装完PyCharm后，还需要安装Python解释器：http://www.python.org/getit/

推荐安装最稳定且比较新的版本，比如3.3。同时为了兼容以前的程序，最好下载一个2.7.6备用，两者并不冲突。

打开PyCharm新建第一个项目，此时解释器还处于未配置的状态，通过如下操作告诉PyCharm我们安装了Python的路径：

通过+号增加一个解释器

增加之后PyCharm会智能地提示你安装setuptool和pip，照着提示一路点击就行了。（Python2.7的setuptool安装会报错UnicodeDecodeError: 'ascii' codec can't decode byte 0xc4 in position 33: ordinal not in range(128)，需要手工修改脚本再安装，详情）。

配置完成后填入项目路径新建一个项目，然后新建一个.py文件，写一句helloworld：

此时还无法运行，因为没有配置项目的入口脚本，通过下图的步骤指定一个：

在scrip框里填入你的入口脚本:
![image](https://muyunzhe.github.io/img/hh.jpg)
![image](https://muyunzhe.github.io/img/jj.jpg)

之后就可以点击绿色的播放按钮运行这个项目了。
