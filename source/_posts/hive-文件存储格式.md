---
title: hive 文件存储格式
comments: true
categories: technology
tags: database
keywords: 'hive, database'
date: 2016-12-15 16:39:41
---
创建一个表时可以指定文件存储格式，默认格式是textfile,还有其他sequencefile，rcfile格式,总的来说就是文本、二进制、压缩三种格式
 <!--more-->
## textfile
 textfile为默认格式
 存储方式：行存储
 磁盘开销大 数据解析开销大
 可结合Gzip、Bzip2使用(系统自动检查，执行查询时自动解压)，但是使用这种方式，hive不会对数据进行切分，从无法对数据进行并行操作。

示例：
```
//建表  
create table if not exists textfile_table(
          sourceSystem_en   string,
          sourceSystem_cn    string
) row format delimited fields terminated by '\t' stored as textfile;

//插入数据前操作  
hive> set hive.exec.compress.output=true;  
hive> set mapred.output.compress=true;  
hive> set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;  
hive> set io.compression.codecs=org.apache.hadoop.io.compress.GzipCodec;  
//插入数据  
hive> insert overwrite table textfile_table select * from testfile_table;  
```

## sequencefile
 二进制文件,以<key,value>的形式序列化到文件中
 存储方式：行存储
 可分割 压缩
 一般选择block压缩
 这种二进制文件内部使用Hadoop 的标准的Writable 接口实现序列化和反序列化。优势是文件和hadoop api中的mapfile是相互兼容的。
 SEQUENCEFILE支持三种压缩选择：NONE、RECORD、BLOCK。RECORD压缩率低，一般建议使用BLOCK压缩。
示例:
```
//建表  
create table if not exists seqfile_table(  
          site   string,  
          url    string,  
          pv     bigint,  
          label  string   
) row format delimited fields terminated by '\t' stored as sequencefile;  

//插入数据前设置相关属性  
hive> set hive.exec.compress.output=true;  
hive> set mapred.output.compression.type=BLOCK;  

//插入数据  
insert overwrite table seqfile_table select * from textfile_table;  
```

## rcfile
 存储方式：数据按行分块 每块按照列存储
 压缩快 快速列存取
 读记录尽量涉及到的block最少
 读取需要的列只需要读取每个row group 的头部定义。
 读取全量数据的操作 性能可能比sequencefile没有明显的优势
示例：
```
//建表  
create table if not exists rcfile_table(  
          site   string,  
          url    string,  
          pv     bigint,  
          label  string   
) row format delimited fields terminated by '\t' stored as rcfile;  

插入数据操作：  
set hive.exec.compress.output=true;    
set mapred.output.compress=true;    
//插入数据  
insert overwrite table rcfile_table select * from testfile_table;  
```

## 总结
 SEQUENCEFILE、RCFILE、RCFILE格式的表不能直接从本地文件导入数据，数据要先导入到TEXTFILE格式的表中，然后再从表中用insert导入到SEQUENCEFILE、RCFILE、ORCFILE等。
