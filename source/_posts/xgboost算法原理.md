---
title: xgboost算法原理
toc: true
mathjax2: true
comments: true
categories: [artificial-intelligence,machine-learning]
tags: [machine-learning,classification]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2017-05-10 10:39:31
---
xgboost的全称是eXtreme Gradient Boosting。正如其名，它是Gradient Boosting Machine的一个c++实现，作者为正在华盛顿大学研究机器学习的大牛陈天奇。他在研究中深感自己受制于现有库的计算速度和精度，因此在一年前开始着手搭建xgboost项目，并在去年夏天逐渐成型。xgboost最大的特点在于，它能够自动利用CPU的多线程进行并行，同时在算法上加以改进提高了精度。
 <!--more-->
和传统的boosting tree模型一样，xgboost的提升模型也是采用的残差（或梯度负方向），不同的是分裂结点选取的时候不一定是最小平方损失。

 Xgboost引入了二阶导来进行求解，并且引入了节点的数目、参数的L2正则来评估模型的复杂度。
xgboost的特性：
1. 显示的把树模型复杂度作为正则项加到优化目标中。
2. 公式推导中用到了二阶导数，用了二阶泰勒展开。（GBDT用牛顿法貌似也是二阶信息）
3. 实现了分裂点寻找近似算法。
4. 利用了特征的稀疏性。
5. 数据事先排序并且以block形式存储，有利于并行计算。
6. 基于分布式通信框架rabit，可以运行在MPI和yarn上。（最新已经不基于rabit了）
7. 实现做了面向体系结构的优化，针对cache和内存做了性能优化。

 xgboost跟gdbt比较大的不同是目标函数的定义。
 ![目标函数](/img/xgboost_loss_function.jpg)
 注：红色箭头指向的l即为损失函数；红色方框为正则项，包括L1、L2；红色圆圈为常数项。xgboost利用泰勒展开三项，做一个近似，我们可以很清晰地看到，最终的目标函数只依赖于每个数据点的在误差函数上的一阶导数和二阶导数。

## 原理
### 定义树的复杂度
 对于f的定义做一下细化，把树拆分成结构部分q和叶子权重部分w。
 定义这个复杂度包含了一棵树里面节点的个数，以及每个树叶子节点上面输出分数的L2模平方。

### 打分函数计算示例
Obj代表了当我们指定一个树的结构的时候，我们在目标上面最多减少多少。我们可以把它叫做结构分数(structure score)

### 分裂节点
1、暴力枚举

2、近似方法 ，近似方法通过特征的分布，按照百分比确定一组候选分裂点，通过遍历所有的候选分裂点来找到最佳分裂点。
两种策略：全局策略和局部策略。在全局策略中，对每一个特征确定一个全局的候选分裂点集合，就不再改变；而在局部策略中，每一次分裂 都要重选一次分裂点。前者需要较大的分裂集合，后者可以小一点。对比补充候选集策略与分裂点数目对模型的影响。 全局策略需要更细的分裂点才能和局部策略差不多

3、Weighted Quantile Sketch

### 定义损失函数
见上图

### 调参

 参考文章：
 * [xgboost原理]( http://blog.csdn.net/a819825294/article/details/51206410)
