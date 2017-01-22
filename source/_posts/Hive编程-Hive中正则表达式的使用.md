---
title: Hive编程-Hive中正则表达式的使用
comments: true
categories: [cloud-computing,storage]
tags: [storage,hive]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2016-05-30 18:40:00
---
hive中的正则解析函数：regexp_extract的使用方法与注意事项
<!--more-->

# 函数描述:
`regexp_extract(str, regexp[, idx]) - extracts a group that matches regexp`
返回值: string
说明：将字符串subject按照pattern正则表达式的规则拆分，返回index指定的字符。注意，在有些情况下要使用转义字符
>参数解释:
其中：
str是被解析的字符串
regexp 是正则表达式
idx是返回结果 取表达式的哪一部分  默认值为1。
0表示把整个正则表达式对应的结果全部返回
1表示返回正则表达式中第一个() 对应的结果 以此类推

# 举例：
```
hive> select regexp_extract(‘foothebar’, ‘foo(.*?)(bar)’, 1) from dual;
the

hive> select regexp_extract(‘foothebar’, ‘foo(.*?)(bar)’, 2) from dual;
bar

hive> select regexp_extract(‘foothebar’, ‘foo(.*?)(bar)’, 0) from dual;
foothebar
```
