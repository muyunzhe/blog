---
title: RNN之LSTM
comments: true
toc: true
mathjax2: true
date: 2017-07-26 17:58:32
categories: [artificial-intelligence,deep-learning]
tags: [deep-learning,Neural Networks]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
---
LSTM是对RNN隐含层的改进。一般所称的LSTM网络全叫全了应该是使用LSTM单元的RNN网络。
 <!--more-->
## RNN存在什么问题
梯度消失或者梯度弥散(vanishing)、梯度爆炸

## LSTM
怎么解决？遗忘门
 LSTM 和普通 RNN 相比, 多出了三个控制器. (输入控制, 输出控制, 忘记控制)
