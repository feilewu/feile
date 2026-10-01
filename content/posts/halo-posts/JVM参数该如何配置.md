---
title: JVM参数该如何配置
id: 77
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: 
permalink: /?p=77
categories:
 - java
tags: 
 - java
 - jvm
---

## 背景

面试官经常会问：JVM参数该如何设置？

虽然线上项目的JVM参数在我加入团队后就没有动过了，但我们还是得学习下不是，万一后面需要了呢？

## JVM内存模型参数

![image](https://tencent-img.feilewu.cn/img/2021/06/09/14-05-11-78f8eb4e67f1bf3d93cfd257be090683-jvm参数设置-4e9fc7.png)

从图中可以很清晰的看出JVM运行时内存的分布及每一个部分对应的参数

- Xmx表示最大堆内存
- Xms表示最小堆内存
- MaxNewSize表示最大新生代大小
- NewSize表示最小新生代大小
- Xmn 可同时对MaxNewSize和NewSize两个参数进行配置（1.4后才有，不过现在企业一般都采用1.8以上的版本了）

- Xss表示线程栈大小，可以控制JVM可以生成的最大线程数
- MaxPermSize表示最大永生代大小
- PermSize表示最小永生代大小

我们可以看到，没有直接设置老年代的参数，但是可以通过新生代参数和堆空间大小来控制

![image](https://tencent-img.feilewu.cn/img/2021/06/09/14-05-14-0935aa7ab6982d9be69b1e863b7e0476-jvm参数设置-2-86dbe3.png)

再来张简化版的图片。

可以看到JVM大体分为heap区和非heap区，其中heap区包括新生代、老年代；非heap区包括永生代、虚拟机栈和本地方法栈等

## 为什么要将一块堆内存分为新生代和老年代？

将内存区域划分为新生代和老年代，是为了使用不同垃圾回收算法，提高整个系统的垃圾回收效率。

JVM在运行过程中，产生对象的生命周期时不同的，有些对象的存活时间长，有些对象的存货时间短，如果每次垃圾回收，都把所有对象都扫描一遍，势必会造成不必要的性能开销。而将堆内存分为新生代和老年代，可针对不同的区域采用不同的垃圾回收算法，比如新生代中的对象存活时间短，可以采用复制算法完成垃圾收集，而老年代中对象存活时间长，对象数量比较多，适合采用标记清楚和标记整理算法。

而复制算法需要一块对象的暂存区域，所以将新生代分为Eden和2个Survivor区，每一次垃圾收集使用Eden和一个Survivor区。

Eden和Survivor区的大小可以通过XXSurvivorRatio参数来进行设置，其表示Eden大小和survivor区大小的比例

比如有参数-Xmx1024m -Xms1024m -Xmn128m -XXSurvivorRatio=2

我可以分析得到，新生代为128M，Eden和Survivor比例为2，那么可以计算得出Eden大小为64M，两个Survivor区均是32M

## JVM参数设置

### 堆大小配置

堆大小一般与操作系统、物理内存、虚拟内存的大小有关

- Xmx 一般不得大于物理内存
- Xms 初始内存，如果想要增大程序的启动性能，可以增大此参数，一般可以设置为和Xmx持平，防止垃圾收集后重新分配内存
- Xmn 年轻代大小，这个参数控制年轻代和老年代的大小，对性能的影响比较大，sun推荐新生代和老年代的比例为3:5
- Xss 线程堆栈大小，一般默认1M，这个参数设置的越小，JVM就能生成更多的线程（不能超过操作系统的限制）
- NewRatio  老年代和新生代的比值
- SurvivorRatio Eden大小和survivor区大小的比例
- MaxTenuringThreshold 设置对象的年龄，可以控制对象在新生代中的存活次数

### 垃圾回收器的选择及其参数配置

- UseSerialGC    使用串行收集器
- UseParallelGC 使用并行收集器
- UseParalledlOldGC 使用并行年老代收集器
- UseConcMarkSweepGC 使用并发收集器

### 其他配置

此外，JVM提供了不少辅助信息获取到的配置

- PrintGC
- PrintGCDetails
- PrintGCApplicationStoppedTime 垃圾收集期间程序暂停运行的时间
- PrintGCApplicationConcurrentTime 垃圾收集期间，应用程序未中断执行的时间
- PrintHeapAtGC 打印出垃圾收集前后的堆栈信息
- PrintGCTimeStamps 垃圾收集发生时的时间戳
- Xloggc:filename 将日志信息记录到文件中，便于后续分析

上述参数可以组合使用，输出易于分析的信息

