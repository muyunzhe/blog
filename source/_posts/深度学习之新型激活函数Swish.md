---
title: 深度学习之新型激活函数Swish
comments: true
toc: true
mathjax2: true
date: 2017-11-10 14:44:49
categories: [artificial-intelligence,deep-learning]
tags: [deep-learning,Neural Networks]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
---
近日，谷歌大脑团队提出了新型激活函数 Swish，团队实验表明使用 Swish 直接替换 ReLU 激活函数总体上可令 DNN 的测试准确度提升。此外，该激活函数的形式十分简单，且提供了平滑、非单调等特性从而提升了整个神经网络的性能。
 <!--more-->
## 概念
 激活函数 Swish:f(x) = x · sigmoid(x)
 图 1 展示的是 Swish 函数的图像：
 ![图 1：Swish 激活函数](/img/Swish 激活函数.jpg)
 和 ReLU 一样，Swish 无上界有下界。与 ReLU 不同的是，Swish 是平滑且非单调的函数。事实上，Swish 的非单调特性把它与大多数常见的激活函数区别开来。
 Swish 的导数:
 ![Swish 的导数](/img/Swish 的导数.jpg)
 Swish 的一阶导和二阶导如图 2 所示。输入低于 1.25 时，导数小于 1。Swish 的成功说明 ReLU 的梯度不变性（即 x > 0 时导数为 1）在现代架构中或许不再是独有的优势。事实上，实验证明在使用批量归一化（Ioffe & Szegedy, 2015）的情况下，我们能够训练出比 ReLU 网络更深层的 Swish 网络。
 ![Swish 的一阶导数与二阶导数](/img/Swish 的一阶导数与二阶导数.jpg)
 原论文指出当全连接网络在 40 层以内时，Swish 只稍微优于 ReLU 激活函数，但当层级增加到 40 至 50 层时，使用 Swish 函数的测试准确度要远远高于使用 ReLU 的测试准确度。


 
