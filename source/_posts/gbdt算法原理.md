---
title: gbdt算法原理
toc: true
mathjax2: true
comments: true
categories: [artificial-intelligence,machine-learning]
tags: [machine-learning,classification]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2017-05-10 10:39:31
---
GBDT(Gradient Boosting Decision Tree) 又叫 MART（Multiple Additive Regression Tree)，是一种迭代的决策树算法，该算法由多棵决策树组成，所有树的结论累加起来做最终答案。
 <!--more-->
## DT：回归树 Regression Decision Tree
切记：GBDT中的树都是回归树，不是分类树

## GB：梯度迭代 Gradient Boosting
GBDT的核心就在于，每一棵树学的是之前所有树结论和的残差，这个残差就是一个加预测值后能得真实值的累加量。

## 工作过程
残差的意思就是： A的预测值 + A的残差 = A的实际值

## 问题总结
### 为何需要GBDT
防止过拟合。 Boosting的最大好处在于，每一步的残差计算其实变相地增大了分错instance的权重，而已经分对的instance则都趋向于0。这样后面的树就能越来越专注那些前面被分错的instance。

### 这不是boosting吧？Adaboost可不是这么定义的。
这是boosting，但不是Adaboost。GBDT不是Adaboost Decistion Tree。就像提到决策树大家会想起C4.5，提到boost多数人也会想到Adaboost。Adaboost是另一种boost方法，它按分类对错，分配不同的weight，计算cost function时使用这些weight，从而让“错分的样本权重越来越大，使它们更被重视”。Bootstrap也有类似思想，它在每一步迭代时不改变模型本身，也不计算残差，而是从N个instance训练集中按一定概率重新抽取N个instance出来（单个instance可以被重复sample），对着这N个新的instance再训练一轮。由于数据集变了迭代模型训练结果也不一样，而一个instance被前面分错的越厉害，它的概率就被设的越高，这样就能同样达到逐步关注被分错的instance，逐步完善的效果。

## GBDT的适用范围
GBDT几乎可用于所有回归问题（线性/非线性），相对logistic regression仅能用于线性回归，GBDT的适用面非常广。亦可用于二分类问题（设定阈值，大于阈值为正例，反之为负例）。
