---
title: python2跟python3的主要区别
comments: true
toc: true
mathjax2: true
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2017-08-31 18:00:48
categories:
tags:
---
Python的3​​.0版本，常被称为Python 3000，或简称Py3k。相对于Python的早期版本，这是一个较大的升级。
为了不带入过多的累赘，Python 3.0在设计的时候没有考虑向下相容。
许多针对早期Python版本设计的程式都无法在Python 3.0上正常执行。
为了照顾现有程式，Python 2.6作为一个过渡版本，基本使用了Python 2.x的语法和库，同时考虑了向Python 3.0的迁移，允许使用部分Python 3.0的语法与函数。
新的Python程式建议使用Python 3.0版本的语法。
 <!--more-->
### future模块
Python 3.x引入了一些与Python 2不兼容的关键字和特性，在Python 2中，可以通过内置的__future__模块导入这些新内容。如果你希望在Python 2环境下写的代码也可以在Python 3.x中运行，那么建议使用__future__模块。例如，如果希望在Python 2中拥有Python 3.x的整数除法行为，可以通过下面的语句导入相应的模块。

### print函数
虽然print语法是Python 3中一个很小的改动，且应该已经广为人知，但依然值得提一下：Python 2中的print语句被Python 3中的print()函数取代，这意味着在Python 3中必须用括号将需要输出的对象括起来。

### 整数除法
由于人们常常会忽视Python 3在整数除法上的改动（写错了也不会触发Syntax Error），所以在移植代码或在Python 2中执行Python 3的代码时，需要特别注意这个改动。

所以，我还是会在Python 3的脚本中尝试用float(3)/2或 3/2.0代替3/2，以此来避免代码在Python 2环境下可能导致的错误（或与之相反，在Python 2脚本中用from __future__ import division来使用Python 3的除法）。

### Unicode
Python 2有基于ASCII的str()类型，其可通过单独的unicode()函数转成unicode类型，但没有byte类型。
而在Python 3中，终于有了Unicode（utf-8）字符串，以及两个字节类：bytes和bytearrays。

### xrange
在Python 2.x中，经常会用xrange()创建一个可迭代对象，通常出现在“for循环”或“列表/集合/字典推导式”中。
这种行为与生成器非常相似（如”惰性求值“），但这里的xrange-iterable无尽的，意味着可能在这个xrange上无限迭代。
由于xrange的“惰性求知“特性，如果只需迭代一次（如for循环中），range()通常比xrange()快一些。不过不建议在多次迭代中使用range()，因为range()每次都会在内存中重新生成一个列表。

在Python 3中，range()的实现方式与xrange()函数相同，所以就不存在专用的xrange()（在Python 3中使用xrange()会触发NameError）。
Python 3中的range对象中的__contains__方法

另一个值得一提的是，在Python 3.x中，range有了一个新的__contains__方法。

### 触发异常
Python 2支持新旧两种异常触发语法，而Python 3只接受带括号的的语法（不然会触发SyntaxError）：

### 异常处理
Python 3中的异常处理也发生了一点变化。在Python 3中必须使用“as”关键字。

### For循环变量与全局命名空间泄漏
好消息是：在Python 3.x中，for循环中的变量不再会泄漏到全局命名空间中了！

### 比较无序类型
Python 3中另一个优秀的改动是，如果我们试图比较无序类型，会触发一个TypeError。

### 通过input()解析用户的输入
幸运的是，Python 3改进了input()函数，这样该函数就会总是将用户的输入存储为str对象。在Python 2中，为了避免读取非字符串类型会发生的一些危险行为，不得不使用raw_input()代替input()。

### 返回可迭代对象，而不是列表
在xrange一节中可以看到，某些函数和方法在Python中返回的是可迭代对象，而不像在Python 2中返回列表。
由于通常对这些对象只遍历一次，所以这种方式会节省很多内存。然而，如果通过生成器来多次迭代这些对象，效率就不高了。
此时我们的确需要列表对象，可以通过list()函数简单的将可迭代对象转成列表。
下面列出了Python 3中其他不再返回列表的常用函数和方法：
zip()
map()
filter()
字典的.key()方法
字典的.value()方法
字典的.item()方法
