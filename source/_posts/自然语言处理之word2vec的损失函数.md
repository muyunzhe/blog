---
title: 自然语言处理之word2vec的损失函数
comments: true
toc: true
mathjax2: true
date: 2017-12-30 21:23:14
categories: [artificial-intelligence,NLP]
tags: [deep-learning,Neural Networks]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
---
NCE的主要思想是，对于每一个样本，除了本身的label，同时采样出N个其他的label，从而我们只需要计算样本在这N+1个label上的概率，而不用计算样本在所有label上的概率。而样本在每个label上的概率最终用了Logistic的损失函数。
 <!--more-->

## 结构
![nce_loss结构](/img/nce_loss.jpg)

如果在这里使用Softmax + Cross-Entropy作为损伤函数会有一个问题，Softmax当有几万+的分类时，速率会大大下降。
