---
title: spark源码编译安装与IntelliJ源码阅读环境配置
comments: true
categories: technology
tags: database
date: 2016-06-14 15:00:07
---
Spark是UC Berkeley AMP lab所开源的类Hadoop MapReduce的通用的并行计算框架，Spark基于map reduce算法实现的分布式计算，拥有Hadoop MapReduce所具有的优点；但不同于MapReduce的是Job中间输出和结果可以保存在内存中，从而不再需要读写HDFS，因此Spark能更好地适用于数据挖掘与机器学习等需要迭代的map reduce的算法。
<!--more-->
# 安装scala
1、下载scala压缩包
scala官网地址http://www.scala-lang.org/download/
2、建立目录，解压文件到所建立目录
3、添加环境变量
```
/*编辑配置文件bashrc (该配置文件只对当前用户有效)*/
$ vim ~/.bashrc
/*在文件的结尾添加如下：×/
#add scala configure
export SCALA_PATH=/home/zhongjuliu/servers/scala-2.11.8
export PATH=$PATH:$SCALA_PATH/bin
/*按esc 输入 :wq 保存并退出×/
```
或者编辑/etc/profile配置文件，使得配置对所有用户有效
4、测试，观察结果版本号是否一致
```
$ scala -version
Scala code runner version 2.11.2 -- Copyright 2002-2013, LAMP/EPFL
```
# 安装git
此处省略几个字
# 安装IntelliJ IDEA（包含scala插件）
下载IDEA安装文，可以到Jetbrains官网http://www.jetbrains.com/idea/download/ 选择最新的安装文件。由于以后的练习需要在Linux开发Scala应用程序，选择安装scala插件。
# spark源码的下载与编译
## 源码下载
spark的github地址
https://github.com/apache/spark
gie clone命令：
```
git clone git://github.com/apache/spark.git
```
## 源码编译
编译的目的是生成指定环境下运行Spark本身或开发Spark Application的JAR包，本次编译的目的生成运行在hadoop2.2.0上的Spark JAR包。
虽然Spark源代码本质上是使用Maven或SBT进行编译，但重点推荐使用SBT进行编译，原因如下：
spark本身是使用scala语言编写的，SBT是scala的编译工具
spark的官方文档从0.9.0开始没提供maven的编译命令，所以以后版本会不会提供pom还难说
spark的部署工具make-distribution.sh内嵌SBT编译命令
### maven编译
先设置参数，后执行命令
需事先安装好maven3.04或maven3.05，并设置要环境变量MAVEN_HOME，将$MAVEN_HOME/bin加入PATH变量。然后将源代码复制到指定目录，然后进入该目录，由于MAVEN工具默认的内存比较小，需要先调大其占用的内存上限：
```
export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"
```
然后打包：
```
mvn -Pyarn -Dhadoop.version=2.2.0 -Dyarn.version=2.2.0 -DskipTests clean package
或者
mvn clean assembly:assembly
```
### sbt编译
* sbt的安装
Run the following from the terminal to install sbt (You’ll need superuser privileges to do so, hence the sudo).
```
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
sudo apt-get update
sudo apt-get install sbt
```
* sbt编译
由于其构建信息集成自maven，maven中的profiles也是有效的，所以可以使用下面的命令进行编译、打包：
```
sbt -Pyarn -Phadoop-2.6 -Phive -Phive-thriftserver assemly
或者
build/sbt clean assembly
或者构建**自定义hadoop版本**
sbt clean assembly -Phive -Phadoop-2.4 -Pyarn -Dhadoop.version=2.4.0.2.1.4.0-632
```
这里使用的是spark自带的build目录下的sbt
如果要运行测试，最好先进行打包：
```
sbt -Pyarn -Phadoop-2.6 -Phive -Phive-thriftserver assemly
sbt -Pyarn -Phadoop-2.6 -Phive -Phive-thriftserver test
```
>编译spark项目所需内存较大，如果用下载的sbt默认的内存是-Xmx1024m，会发生堆内存溢出。最好使用源代码中自带的build/sbt，它把内存设置为了-Xmx2048m，这样编译不会内存溢出。

### 用spark自带的dev/make-distribution.sh生成spark部署包
编译完源代码后，虽然直接用编译后的目录再加以配置就可以运行spark，但是这时目录很庞大，部署起来不方便，所以需要生成部署包。
spark源码根目录下带有一个脚本文件make-distribution.sh可以生成部署包，该脚本会使用MAVEN进行编译，然后打成一个tgz包。其参数有：
--tgz：在根目录下生成 spark-$VERSION-bin.tar.gz，不加参数是不生成tgz文件，只生成/dist目录。
--hadoop VERSION：打包时所用的Hadoop版本号，不加参数时为1.0.4。
--with-yarn：是否支持Hadoop YARN，不加参数时为不支持yarn。
--with-tachyon：是否支持内存文件系统Tachyon，不加参数时为不支持，此参数spark1.0.0-SNAPSHOT之后提供。
如果要生成spark支持yarn、hadoop2.2.0的部署包，只需要将源代码复制到指定目录，进入该目录后运行：
```
./make-distribution.sh --hadoop 2.2.0 --with-yarn --tgz
```
如果要生成spark支持yarn、hadoop2.2.0、techyon的部署包，只需要将源代码复制到指定目录，进入该目录后运行：
```
 ./make-distribution.sh --hadoop 2.2.0 --with-yarn --with-tachyon --tgz
 ```
或者部署**自定义版本**
```
./make-distribution.sh --tgz --with-tachyon -Phadoop-2.4 -Pyarn -Phive -Dhadoop.version=2.4.0.2.1.4.0-632
```
生成在部署包位于根目录下，文件名类似于spark-1.0.0-SNAPSHOT-hadoop_2.2.0-bin.tar.gz。
值得注意的是：make-distribution.sh已经带有SBT编译过程，所以不需要先编译再打包。

# spark的IDEA开发环境配置
## 创建工程
1.在IDEA菜单栏选择File->New Project，选择创建Scala项目；
2.在项目的基本信息填写项目名称、项目所在位置、Project SDK和Scala SDK，在这里设置项目名称为class3，关于Scala SDK的安装可直接点击create按钮，进入之后download。
3.设置Modules，创建该项目后，可以看到现在还没有源文件，只有一个存放源文件的目录src以及存放工程其他信息的杂项。通过双击src目录或者点击菜单上的项目结构图标打开项目配置界面，在Modules设置界面中，分别设置main->scala目录为Sources类型
4.配置Library
选择Library目录，添加Scala SDK Library，这里选择scala-2.10.4版本
添加Java Library，这里选择的是在$SPARK_HOME/lib/spark-assembly-1.1.0-hadoop2.2.0.jar文件

## 项目运行
### 直接运行
第一步 编写代码
在src->main->scala下创建class3包，在该包中添加SogouResult对象文件

第二步 编译代码
代码在运行之前需要进行编译，可以点击菜单Build->Make Project或者Ctrl+F9对代码进行编译，编译结果会在Event Log进行提示，如果出现异常可以根据提示进行修改.

第三步 运行环境配置
SogouResult首次运行或点击菜单Run->Edit Configurations打开"运行/调试 配置界面"
运行SogouResult时需要输入搜狗日志文件路径和输出结果路径两个参数，需要注意的是HDFS的路径参数路径需要全路径，否则运行会报错：
l  搜狗日志文件路径：使用上节上传的搜狗查询日志文件hdfs://hadoop1:9000/sogou/SogouQ1.txt
l  输出结果路径：hdfs://hadoop1:9000/class3/output2

第四步 运行结果查看
启动Spark集群，点击菜单Run->Run或者Shift+F10运行SogouResult，在运行结果窗口可以运行情况。当然了如果需要观察程序运行的详细过程，可以加入断点，使用调试模式根据程序运行过程。
使用如下命令查看运行结果，该结果和上节运行的结果一致
```
hadoop fs -ls /class3/output2
hadoop fs -cat /class3/output2/part-00000 | less
```

### 打包运行
上个例子使用了IDEA直接运行结果，在该例子中将使用IDEA打包程序进行执行
编写代码
在class3包中添加Join对象文件
第一步   配置打包信息
在项目结构界面中选择"Artifacts"，在右边操作界面选择绿色"+"号，选择添加JAR包的"From modules with dependencies"方式，出现如下界面，在该界面中选择主函数入口为Join

第二步   填写该JAR包名称和调整输出内容
【注意】的是默认情况下"Output Layout"会附带Scala相关的类包，由于运行环境已经有Scala相关类包，所以在这里去除这些包只保留项目的输出内容

第三步   输出打包文件
点击菜单Build->Build Artifacts，弹出选择动作，选择Build或者Rebuild动作

第四步   复制打包文件到Spark根目录下
```
cd /home/hadoop/IdeaProjects/out/artifacts/class3
cp LearnSpark.jar  /app/hadoop/spark-1.1.0/
ls /app/hadoop/spark-1.1.0/
```

第五步   运行查看结果
通过如下命令调用打包中的Join方法，运行结果如下：
```
cd /app/hadoop/spark-1.1.0
bin/spark-submit --master spark://hadoop1:7077 --class class3.Join --executor-memory 1g LearnSpark.jar hdfs://hadoop1:9000/class3/join/reg.tsv hdfs://hadoop1:9000/class3/join/clk.tsv
```
