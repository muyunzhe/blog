---
title: 推荐系统之EE问题与解决思路
comments: true
toc: true
mathjax2: true
categories: [data-mining,recommender-system]
tags: 推荐系统
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2018-01-04 20:26:17
---
推荐系统不可避免会遇到冷启动问题，如何去尝试新用户的兴趣点，尝试到什么时候地步才算真正掌握了用户的兴趣，用户的兴趣发生改变如何灵活的调整推荐策略。这些，都与E&E问题有关，而Bandit算法是解决E&E问题的一种思路
 <!--more-->
## E&E问题
条件：假设我们有K个准备推荐的item，每个item的回报的服从不同的概率分布p_item，且分布参数未知
目标：如果有T次机会推荐，如何制定决策过程从而获取最大的累积回报
### 表现形式
随机式(stochastic bandit): item的回报服从某个固定的概率分布
对抗式(adversarial bandit): item的回报会动态调整，让你选的奖励变低
### 推荐应用
计算机广告和推荐系统中，有很多问题可以抽象为E&E问题：
冷启动问题：假设一个用户对不同类别的内容感兴趣程度不同，那么我们的推荐系统初次见到这个用户时，怎么快速地知道他对每类内容的感兴趣程度？
推荐item选择：假设资源池有若干item（或若干类item），怎么知道该给每个用户展示哪个（类），从而获得最大的点击？是每次都挑效果最好那个么？那么新广告如何才有出头之日？
推荐策略选择：假设我们有了新的推荐策略，有没有比A/B test更快的方法知道它和旧模型相比谁更靠谱？

### E&E算法框架
Exploitation & Exploration
（1）Exploitation：基于已知最好策略，开发利用已知具有较高回报的item（贪婪、短期回报）
Advantage：充分利用已知高回报item
Disadvantage：陷于局部最优，错过潜在更高回报item的机会

（2）Exploration：不考虑曾经的经验，勘探潜在可能高回报的item（非贪婪、长期回报）
Advantage：发现更好回报的item
Disadvantage：充分利用已有高回报item机会减少（如已经找到最好item）

（3）目标：要找到Exploitation & Exploration的trade-off，以达到累计回报最大化。
极端情况下，Exploitation每次选择最高mean回报的item（太“confident”），Exploration每次随机选择一个item（太不“confident”），两个极端都不能达到最终目标，因此，在选择item时，我们不仅要考虑item的mean回报，同事也要兼顾confidence。

## Bandit算法
专治困难选择症
### naive Algorithm
最简单直接的，我们可以通过简单观察法（naive Algorithm）选取item：先对每个item进行一定次数（如100次）选择尝试，计算item的回报率，接下来选择回报率高的item。算法简单直接，但存在以下问题：
item很多导致获取item回报率的成本太大
尝试一定次数（如100次）得到的“高回报”item未必靠谱
item的回报率有可能会随时间发生变化：好的变差、差的变好

### ε-Greedy
选一个(0,1)之间较小的数ε，每次决策以概率ε去勘探Exploration，1-ε的概率来开发Exploitation，基于选择的item及回报，更新item的回报期望，不断循环下去。
这样做的好处在于：
能够应对变化：如果item的回报发生变化，能及时改变策略，避免卡在次优状态
可以控制对Exploration和Exploitation的偏好程度：ε大，模型具有更大的灵活性（能更快的探索潜在可能高回报item，适应变化，收敛速度更快），ε小，模型具有更好的稳定性（更多的机会用来开发利用当前最好回报的item），收敛速度变慢
虽然ε-Greedy算法在Exploration和Exploitation之间做出了一定平衡，但
设置最好的ε比较困难，大则适应变化较快，但长期累积回报低，小则适应变好的能力不够，但能获取更好的长期回报。
策略运行一段时间后，我们对item的好坏了解的确定性增强，但仍然花费固定的精力去exploration，浪费本应该更多进行exploitation机会
策略运行一段时间后，我们已经对各item有了一定程度了解，但没用利用这些信息，仍然不做任何区分地随机exploration（会选择到明显较差的item）

Epsilon-Greedy算法的变体：
1）ε-first strategy：首先进行小部分次数进行随机尝试来确定那些item能获得较高回报，然后接下来大部分次数选择前面确定较高回报的item
2）ε-decreasing strategy: epsilon随着时间的次数逐步降低，开始的时候exploration更多次数，然后逐步降低（ε_t=1/log(itemSelectCnt+0.00001 )），增加exploitation比重(Annealing-Epsilon-Greedy)

### SoftMax
ε-Greedy在探索时采用完全随机的策略，经常会选择一个看起来很差的item，此问题的解决方法之一是，基于我们目前已经知道部分item回报信息，不进行随机决策，而是使用softmax决策找出回报最大的item。
SoftMax利用softmax函数来确定各item的回报的期望概率排序，进而在选择item时考虑该信息，减少exploration过程中低回报率item的选择机会，同时收敛速度也会较ε-Greedy更快。

但是，softmax算法的缺点是：
没有考虑预估item回报率期望的置信度信息，因而选择的高回报率item未必靠谱。
无法事先知道t的大小，因问题而异，同时也不容易与其他算法直接比较
Softmax变体：Annealing-Softmax:
随着策略的运行，我们期望降低T温度参数以减小exploration所占的比例：T=1/log(itemSelectCnt+0.0000001)

### UCB, Upper Confidence Bound
统计学中，我们使用置信区间来度量估计的不确定性/置信性。如我们摇骰子一次得到的点数为2，那么得到均值的估计也是2（实际平均点数是3.5），但显然这个估计不太靠谱，可以用置信区间量化估计的变化性：骰子点数均值为2，其95%置信区间的上限、下限分别为1.4、5.2。

UCB思想是乐观地面对不确定性，以item回报的置信上限作为回报预估值的一类算法，其基本思想是：我们对某个item尝试的次数越多，对该item回报估计的置信区间越窄、估计的不确定性降低，那些均值更大的item倾向于被多次选择，这是算法保守的部分（exploitation）；对某个item的尝试次数越少，置信区间越宽，不确定性较高，置信区间较宽的item倾向于被多次选择，这是算法激进的部分（exploration）。

其中，x_j是item_j的平均回报，n_j是item_j截至当前被选择的次数，n为当前选择所有item的次数。上式反映了，均值越大，标准差越小，被选中的概率会越来越大，起到了exploit的作用；同时哪些被选次数较少的item也会得到试验机会，起到了explore的作用。

与ε-Greedy算法、softmax算法相比，这种策略的好处在于：
考虑了回报均值的不确定性，让新的item更快得到尝试机会，将探索+开发融为一体
基础的UCB算法不需要任何参数，因此不需要考虑如何验证参数（ε如何确定）的问题
UCB1算法的缺点：
UCB1算法需要首先尝试一遍所有item，因此当item数量很多时是一个问题
一开始各item选择次数都比较少，导致得到的回报波动较大（经常选中实际比较差的item）

### Thompson sampling
UCB算法部分使用概率分布（仅置信区间上界）来量化不确定性。而Thompson sampling基于贝叶斯思想，全部用概率分布来表达不确定性。

假设每个item有一个产生回报的概率p，我们通过不断试验来估计一个置信度较高的概率p的概率分布。如何估计概率p的概率分布呢？ 假设概率p的概率分布符合beta(wins, lose)分布，它有两个参数: wins, lose， 每个item都维护一个beta分布的参数。每次试验选中一个item，有回报则该item的wins增加1，否则lose增加1。每次选择item的方式是：用每个item现有的beta分布产生一个随机数b，选择所有item产生的随机数中最大的那个item。

相比于UCB算法，Thompson sampling：
UCB采用确定的选择策略，可能导致每次返回结果相同（不是推荐想要的），Thompson Sampling则是随机化策略。
Thompson sampling实现相对更简单，UCB计算量更大（可能需要离线/异步计算）
在计算机广告、文章推荐领域，效果与UCB不相上下或更好competitive to or better
对于数据延迟反馈、批量数据反馈更加稳健robust
