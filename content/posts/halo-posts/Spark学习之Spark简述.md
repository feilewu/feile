---
title: Spark学习之Spark简述
id: 83
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: 
permalink: /?p=83
categories:
 - spark
tags: 
 - scala
 - 大数据
---

## Spark生态

![](https://gitee.com/pfxu/images/raw/master/2021/09/20/20210920220010.png)

Spark Core：包含Spark的基本功能；尤其是定义RDD的API、操作以及这两者上的动作。其他Spark的库都是构建在RDD和Spark Core之上的

Spark SQL：提供通过Apache Hive的SQL变体Hive查询语言（HiveQL）与Spark进行交互的API。每个数据库表被当做一个RDD，Spark SQL查询被转换为Spark操作。

Spark Streaming：对实时数据流进行处理和控制。Spark Streaming允许程序能够像普通RDD一样处理实时数据

MLlib：一个常用机器学习算法库，算法被实现为对RDD的Spark操作。这个库包含可扩展的学习算法，比如分类、回归等需要对大量数据集进行迭代的操作。

GraphX：控制图、并行图操作和计算的一组算法和工具的集合。GraphX扩展了RDD API，包含控制图、创建子图、访问路径上所有顶点的操作。

## Spark的运行流程

![](https://gitee.com/pfxu/images/raw/master/2021/09/20/20210920213758.png)

构建Spark Application的运行环境，启动SparkContext

SparkContext向资源管理器（可以是Standalone，Mesos，Yarn）申请运行Executor资源，并启动StandaloneExecutorbackend，

Executor向SparkContext申请Task

SparkContext将应用程序分发给Executor

SparkContext构建成DAG图，将DAG图分解成Stage、将Taskset发送给Task Scheduler，最后由Task Scheduler将Task发送给Executor运行

Task在Executor上运行，运行完释放所有资源

## Spark与Hadoop

MapReduce采用硬盘保存临时数据，而Spark采用内存保存临时数据

![](https://gitee.com/pfxu/images/raw/master/2021/09/20/20210920215917.png)