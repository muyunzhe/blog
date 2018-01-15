---
title: 机器学习维度规约之PCA
comments: true
toc: true
mathjax2: true
date: 2017-09-21 18:29:16
categories: [artificial-intelligence,machine-learning]
tags: [machine-learning,PCA, LDA]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
---
PCA（Principal Component Analysis）是一种常用的数据分析方法。PCA通过线性变换将原始数据变换为一组各维度线性无关的表示，可用于提取数据的主要特征分量，常用于高维数据的降维.
 <!--more-->
## 算法步骤：
* 第一步，分别求每列的平均值，然后对于所有的样例，都减去对应的均值。
* 第二步，求特征协方差矩阵。
* 第三步，求协方差的特征值和特征向量。
* 第四步，将特征值按照从大到小的顺序排序，选择其中最大的k个，然后将其对应的k个特征向量分别作为列向量组成特征向量矩阵。

* 第五步，将样本点投影到选取的特征向量上。假设样例数为m，特征数为n，减去均值后的样本矩阵为DataAdjust(m*n)，协方差矩阵是n*n，选取的k个特征向量组成的矩阵为EigenVectors(n*k)。

有两篇文章写的不错：
<http://www.jianshu.com/p/673d4fe72362>
<https://zhuanlan.zhihu.com/p/21580949>


## PCA与Autoencoder的关系
当AutoEncoder只有一个隐含层的时候，其原理相当于主成分分析（PCA）。但是 AutoEncoder 明显比PCA的效果更好一点，尤其在图像上。 当AutoEncoder有多个隐含层的时候，每两层之间可以用RBM来pre-training，最后由BP来调整最终权值。
