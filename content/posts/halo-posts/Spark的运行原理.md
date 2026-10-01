---
title: Spark的运行原理
id: 85
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: Spark基本概念RDD：是弹性分布式数据集（Resilient Distributed Dataset）的简称，是分布式内存的一个抽象概念，提供了一种高度受限的共享内存模型。DAGScheduler：有向无环图调度器基于DAG划分Stage并以TaskSet的形式提交Stage给TaskSched
permalink: /?p=85
categories:
 - spark
tags: 
 - 分布式
 - 大数据
---

## Spark基本概念

RDD：是弹性分布式数据集（Resilient Distributed Dataset）的简称，是分布式内存的一个抽象概念，提供了一种高度受限的共享内存模型。
![2022-03-29-20-49-45](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-49-45.png)

DAGScheduler：有向无环图调度器基于DAG划分Stage并以TaskSet的形式提交Stage给TaskScheduler；负责将作业拆分成不同阶段的具有宽窄依赖关系的多批任务；最重要的任务之一就是：计算作业和任务的依赖关系，指定调度逻辑。在SparkContext初始化的过程中被实例化，一个SparkContext对应一个DAGScheduler。 
![2022-03-29-20-50-13](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-50-13.png)

DAG：是Directed Acyclic Graph（有向无环图）的简称，反映RDD之间的依赖关系。
![2022-03-29-20-48-38](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-48-38.png)
窄依赖：父RDD每一个分区最多被一个子RDD的分区所用；表现为一个父RDD的分区对应于一个子RDD的分区，或两个父RDD的分区对应于一个子RDD的分区。
![2021-09-21-19-40-39](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-19-40-39.png)

宽依赖：父RDD的每个分区都可能被多个子RDD分区所使用，子RDD分区通常对应所有的父RDD分区
![2021-09-21-19-41-36](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-19-41-36.png)

常见的宽窄依赖有：map、filter、union、mapPartitions、mapValues、join（父RDD是hash-partitioned：如果joinAPI之前被调用的RDD API是宽依赖【存在shuffle】，而且两个join的RDD的分区数量一致，join结果的RDD分区数量也一样，这个时候join API是窄依赖）。

常见的宽依赖有：groupByKey、partitionBy、reduceByKey、join（父RDD不是hash-partitioned：除此之外的，RDD的join API都是宽依赖）。

Driver Program：控制程序，负责为Application构建DAG图。
![2022-03-29-20-50-31](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-50-31.png)

Cluster Manager：集群资源管理中心，负责分配计算资源。

Worker Node：工作节点，负责完成具体计算。
![2022-03-29-20-50-45](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-50-45.png)

Executor：是运行在工作节点（Worker Node）上的一个进程，负责运行Task，并为应用程序存储数据。
![2022-03-29-20-51-01](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-51-01.png)

Application：用户编写的Spark应用程序，一个Application包含多个Job。
![2022-03-29-20-51-13](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-51-13.png)

Job：作业，一个Job包含多个RDD及作用于相应RDD上的各种操作。
![2022-03-29-20-51-26](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-51-26.png)

Stage：阶段，是作业的基本调度单位，一个作业会分为多组任务，每组任务被称为“阶段”。

TaskScheduler：任务调度器 将TaskSet提交给Worker（集群）运行并汇报结果；负责每个具体任务的实际物理调度。
![2022-03-29-20-51-45](https://tencent-img.feilewu.cn/vscode-images/2022-03-29-20-51-45.png)

Task：任务，运行在Executor上的工作单元，是Executor中的一个线程。

TaskSet：任务集 由一组关联的，但相互之间没有Shuffle依赖关系的任务所组成的任务集。
 - 1） 一个Stage创建一个TaskSet；
 - 2） 为Stage的每个RDD分区创建一个Task，多个Task分装成TaskSet。
![2021-09-21-19-34-22](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-19-34-22.png)

总结：Application由多个Job组成，Job由多个Stage组成，Stage由多个Task组成。Stage是作业调度的基本单位。

整体结构
![2021-09-21-19-39-09](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-19-39-09.png)


## Spark作业执行过程

- 我们使用spark-submit提交一个Spark作业之后，这个作业就会启动一个对应的Driver进程。根据你使用的部署模式（deploy-mode）不同，Driver进程可能在本地启动，也可能在集群中某个工作节点上启动。而Driver进程要做的第一件事情，就是向集群管理器（可以是Spark Standalone集群，也可以是其他的资源管理集群，美团•大众点评使用的是YARN作为资源管理集群）申请运行Spark作业需要使用的资源，这里的资源指的就是Executor进程。YARN集群管理器会根据我们为Spark作业设置的资源参数，在各个工作节点上，启动一定数量的Executor进程，每个Executor进程都占有一定数量的内存和CPU core。

 - 在申请到了作业执行所需的资源之后，Driver进程就会开始调度和执行我们编写的作业代码了。Driver进程会将我们编写的Spark作业代码分拆为多个stage，每个stage执行一部分代码片段，并为每个stage创建一批Task，然后将这些Task分配到各个Executor进程中执行。Task是最小的计算单元，负责执行一模一样的计算逻辑（也就是我们自己编写的某个代码片段），只是每个Task处理的数据不同而已。一个stage的所有Task都执行完毕之后，会在各个节点本地的磁盘文件中写入计算中间结果，然后Driver就会调度运行下一个stage。下一个stage的Task的输入数据就是上一个stage输出的中间结果。如此循环往复，直到将我们自己编写的代码逻辑全部执行完，并且计算完所有的数据，得到我们想要的结果为止。

 - Spark是根据shuffle类算子来进行stage的划分。如果我们的代码中执行了某个shuffle类算子（比如reduceByKey、join等），那么就会在该算子处，划分出一个stage界限来。可以大致理解为，shuffle算子执行之前的代码会被划分为一个stage，shuffle算子执行以及之后的代码会被划分为下一个stage。因此一个stage刚开始执行的时候，它的每个Task可能都会从上一个stage的Task所在的节点，去通过网络传输拉取需要自己处理的所有key，然后对拉取到的所有相同的key使用我们自己编写的算子函数执行聚合操作（比如reduceByKey()算子接收的函数）。这个过程就是shuffle。

 - 当我们在代码中执行了cache/persist等持久化操作时，根据我们选择的持久化级别的不同，每个Task计算出来的数据也会保存到Executor进程的内存或者所在节点的磁盘文件中。

 - 因此Executor的内存主要分为三块：第一块是让Task执行我们自己编写的代码时使用，默认是占Executor总内存的20%；第二块是让Task通过shuffle过程拉取了上一个stage的Task的输出后，进行聚合等操作时使用，默认也是占Executor总内存的20%；第三块是让RDD持久化时使用，默认占Executor总内存的60%。

 - Task的执行速度是跟每个Executor进程的CPU core数量有直接关系的。一个CPU core同一时间只能执行一个线程。而每个Executor进程上分配到的多个Task，都是以每个Task一条线程的方式，多线程并发运行的。如果CPU core数量比较充足，而且分配到的Task数量比较合理，那么通常来说，可以比较快速和高效地执行完这些Task线程。
