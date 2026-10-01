---
title: Spark的运行模式
id: 84
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: 
permalink: /?p=84
categories:
 - spark
tags: 
 - 大数据
---

## Standalone模式
### 架构
![2021-09-21-19-47-21](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-19-47-21.png)

### Standalone运行过程

![2021-09-21-19-52-16](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-19-52-16.png)

1. SparkContext连接到Master，向Master注册并申请资源（CPU Core and Memory）。
2. Master根据SparkContext的资源申请要求和Worker心跳周期内报告的信息决定哪个Worker上分配资源，然后在该Worker上获取资源，然后启动Executor，Executor向SparkContext注册。
3. SparkContext将Application代码发送到Executor。同时，SparkContext解析Application代码，构建DAG图，并提交给DAGScheduler分解成Stage，然后以Stage（或者称为TaskSet）提交给TaskScheduler，TaskScheduler负责将Task分配到相应的Worker，最后提交给Executor执行。
4. Executor会建立Executor线程池，开始执行Task，并向SparkContext报告，直至Task完成。
5. 所有Task完成后，SparkContext向Master注销，释放资源。

## Spark on Yarn模式

### Yarn-Cluster模式 

![2021-09-21-19-53-14](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-19-53-14.png)

1. Client向Yarn中提交应用程序，包括ApplicationMaster程序、启动ApplicationMaster的命令、需要在Executor中运行的程序等。
2. ResourceManager收到请求后，在集群中选择一个NodeManager，为该应用程序分配第一个Container，要求它在这个Container中启动应用程序的ApplicationMaster，其中ApplicationMaster进行SparkContext等的初始化。
3. ApplicationMaster向ResourceManager注册，这样用户可以直接通过ResourceManage查看应用程序的运行状态，然后它将采用轮询的方式通过RPC协议为各个任务申请资源，并监控它们的运行状态直到运行结束。
4. 一旦ApplicationMaster申请到资源后，便与对应的NodeManager通信，要求它在获得的Container中启动Executor，启动后会向ApplicationMaster中的SparkContext注册并申请Task。
5. ApplicationMaster中的SparkContext分配Task给Executor执行，Executor运行Task并向ApplicationMaster汇报运行的状态和进度，以让ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。
6. 应用程序运行完成后，ApplicationMaster向ResourceManager申请注销并关闭自己。

### Spark on Yarn-Client 模式
![2021-09-21-19-58-37](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-19-58-37.png)

1. Client向Yarn的ResourceManager申请启动ApplicationMaster。同时，在SparkContext初始化中将创建DAGScheduler和TaskScheduler。
2. ResourceManager收到请求后，在集群中选择一个NodeManager，为该能够用程序分配第一个Container，要求它在这个Container中启动应用程序的ApplicationMaster。
3. Client中的SparkContext初始化完毕后，与ApplicationMaster建立通讯，向ResourceManager注册，根据任务信息向ResourceManager申请资源。
4. 一旦ApplicationMaster申请到资源后，便与对应的NodeManager通信，要求它在获得的Container中启动Executor，启动后会向Client中的SparkContext注册并申请Task。
5. Client中的SparkContext分配Task给Executor执行，Executor运行Task并向Driver汇报运行的状态和进度，以让Client随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。
6. 应用程序运行完成后，Client的SparkContext向ResourceManager申请注销并关闭自己。

### Yarn-Cluster与Yarn-Client的区别

它们的区别就是ApplicationMaster的区别：Yarn-Cluster中ApplicationMaster不仅负责申请资源，并负责监控Task的运行状况，因此可以关掉Client；而Yarn-Client中ApplicationMaster仅负责申请资源，由Client中的Driver来监控调度Task的运行，因此不能关掉Client。

### Spark on Yarn与MapReduce on Yarn的区别
Spark的高性能一定程度上取决于它采用的异步并发模型（这里指server/driver端采用 的模型），这与Hadoop 2.0（包括Yarn和MapReduce）是一致的。Hadoop 2.0自己实现了类似Actor的异步并发模型，实现方式是epoll+状态机，而Spark则直接采用了开源软件Akka，该软件实现了Actor模型，性能非常高。尽管二者在server端采用了一致的并发模型，但在任务级别（特指Spark任务和MapReduce任务）上却采用了不同的并发机制：Hadoop MapReduce采用了多进程模型，而Spark采用了多线程模型。
      
注意，这里的多进程和多线程，指的是同一个节点上多个任务的运行模式。无论是MapReduce和Spark，整体上看，都是多进程：MapReduce应用程序是由多个独立的Task进程组成的；Spark应用成的运行环境是由多个独立的Executor进程构建的临时资源池构成的。
      
多进程模型便于细粒度控制每个任务占用的资源，但会消耗较多的启动时间，不适合运行低延迟类型的作业，这是MapReduce广为诟病的原因之一。而多线程模型则相反，该模型使得Spark很适合运行低延迟类型的作业。总之，Spark同节点上的任务以多线程的方式运行在一个JVM进程中，可带来以下好处：
1. 任务启动速度快，与之相反的是MapReduce Task进程的慢启动速度，通常需要1s左右；
2. 同节点上所有任务运行在一个进程中，有利于共享内存。这非常适合内存密集型任务，尤其对于那些需要加载大量词典的应用程序，可大大节省内存；
3. 同节点上所有任务可运行在一个JVM进程（Executor）中，且Executor所占资源可连续被多批任务使用，不会在运行部分任务后释放掉，这避免了每个任务重复申请资源带来的时间开销，对于任务数目非常多的应用，可大大降低运行时间。与之对比的是MapReduce中的Task：每个Task单独申请资源，用完后马上释放，不能被其他任务重用，尽管1.0支持JVM重用在一定程度上弥补了该问题，但2.0尚未支持该功能。
   

尽管Spark的多线程模型带来了很多好处，但同样存在不足，主要有：
 - 由于节点上所有任务运行在一个进程中，因此，会出现严重的资源争用，难以细粒度控制每个任务占用资源。与之相反的是MapReduce，它允许用户单独为Map Task和Reduce Task设置不同的资源，进而细粒度控制任务占用资源量，有利于大作业的正常平稳运行。

## MapReduce和Spark运行模型

### MapReduce多进程模型
![2021-09-21-20-03-00](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-20-03-00.png)

1. 每个Task运行在一个独立的JVM进程中；
2. 可单独为不同类型的Task设置不同的资源量，目前支持内存和CPU两种资源；
3. 每个Task运行完后，释放所占用的资源，这些资源不能被其他Task复用，即使是同一个作业相同类型的Task。也就是说，每个Task都要经历“申请资源->运行Task->释放资源”的过程。

### Spark多线程模型 

![2021-09-21-20-03-43](https://tencent-img.feilewu.cn/vscode-images/2021-09-21-20-03-43.png)

1. 每个节点上可以运行一个或多个Executor服务（一个节点上可以有多个Container）；
2. 每个Executor配有一定数量的slot，表示该Executor中可以同时运行多少个ShuffleMap Task或者Reduce Task；
3. 每个Executor单独运行在一个JVM进程中，每个Task则是运行在Executor中的一个线程（每个线程可以通过轮询的方式运行每个Task）；
4. 同一个Executor内部的Task可共享内存，比如通过函数SparkContext.broadcast广播的文件或者数据结构只会在每个Executor中加载一次，而不会像MapReduce那样，每个Task加载一次；
5. Executor一旦启动后，将一直运行，且它的资源可以一直被Task复用，直到Spark程序运行完成后才释放退出。
   
### 总结

总体上看，Spark采用的是经典的scheduler/workers模式，每个Spark应用程序运行的第一步是构建一个可重用的资源池，然后在这个资源池里运行所有的ShuffleMap Task和Reduce Task（注意，尽管Spark引擎内部只用两类Task便可表示出一个复杂的应用程序，即ShuffleMap Task和Reduce Task），而MapReduce应用程序则不同，它不会构建一个可重用的资源池，而是让每个Task动态申请资源，且运行完后马上释放资源。

