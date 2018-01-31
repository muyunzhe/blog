---
title: 自然语言处理之word2vec模型
comments: true
toc: true
mathjax2: true
date: 2018-01-30 21:27:56
categories: [artificial-intelligence,NLP]
tags: [deep-learning,Neural Networks]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
---
Word2Vec即Word to vector（词汇转向量）。我们希望词义相近的两个单词，在映射之后依然保持相近，词义很远的单词直接则保持很远的映射距离。
 <!--more-->
## Word2Vec结构
![skip-gram模型架构](/img/model.jpg)

## 目标函数
![softmax函数](/img/softmax.png)
极大化似然函数
![极大似然函数](/img/log-likelihood.png)
上面式子是一个标准的概率语言模型的形式，但是上面式子求解起来成本非常高，因为第二项中有个Vocab，也就是字典的大小。
word2vec对上面图进行了改进，不在计算在整个词典中生成当前词的概率，而是从词典中『采样』k个.


## 关于Word2Vec实例总结为6步:

### 1、下载数据；
加载训练文本数据，属于无监督学习，所以文本数据没有标签

### 2、将原词汇数据转换为字典映射；
原词汇数据分词，取出指定的词频排前n的词，低频的词用‘UNK’表示，转换为字典映射。
按频率从高到低排序，现在我们的词汇文本变成了用数字编号替代的格式以及词汇表和逆词汇表。逆词汇只是编号为key,词汇为value。

### 3、为 skip-gram模型 建立一个扫描器；
首先看一下扫描器函数:
defgenerate_batch(batch_size, num_skips, skip_window):
batch_size是指一次扫描多少块，skip_window为左右上下文取词的长短，num_skips输入数字的重用次数。假设我们的扫描器先扫这大段文字的前8个单词，左右各取1个单词，重用次数为2次。

### 4、建立并训练 skip-gram 模型；
这里谈得都是嵌套，那么先来定义一个嵌套参数矩阵。我们用唯一的随机值来初始化这个大矩阵。
```
embeddings = tf.Variable(
                tf.random_uniform([self._options.vocabulary_size, self._options.emb_dim], -1.0, 1.0))
```
对噪声-比对的损失计算就使用一个逻辑回归模型。对此，我们需要对语料库中的每个单词定义一个权重值和偏差值。(也可称之为输出权重 与之对应的 输入嵌套值)。定义如下。
```
nce_weights = tf.Variable(
    tf.truncated_normal([self._options.vocabulary_size, self._options.emb_dim],
                        stddev=1.0 / math.sqrt(self._options.emb_dim)))
nce_biases = tf.Variable(tf.zeros([self._options.vocabulary_size]))
```
我们有了这些参数之后，就可以定义Skip-Gram模型了。简单起见，假设我们已经把语料库中的文字整型化了，这样每个整型代表一个单词。Skip-Gram模型有两个输入。一个是一组用整型表示的上下文单词，另一个是目标单词。给这些输入建立占位符节点，之后就可以填入数据了。
然后我们需要对批数据中的单词建立嵌套向量，TensorFlow提供了方便的工具函数。
```
embed = tf.nn.embedding_lookup(embeddings, self.train_inputs)
```
好了，现在我们有了每个单词的嵌套向量，接下来就是使用噪声-比对的训练方式来预测目标单词。
```
self._loss = tf.reduce_mean(
            tf.nn.nce_loss(weights=nce_weights,
                           biases=nce_biases,
                           labels=self.train_labels,
                           inputs=embed,
                           num_sampled=self._options.num_sampled,
                           num_classes=self._options.vocabulary_size))

```
我们对损失函数建立了图形节点，然后我们需要计算相应梯度和更新参数的节点，比如说在这里我们会使用随机梯度下降法，TensorFlow也已经封装好了该过程。

### 5、开始训练模型；
训练的过程很简单，只要在循环中使用feed_dict不断给占位符填充数据，同时调用 session.run即可。
