---
title: CAP定理
id: 65
date: 2024-09-27 19:10:52
auther: admin
excerpt: 概述1998年，加州大学的计算机科学家 Eric Brewer 提出，分布式系统有三个指标。ConsistencyAvailabilityPartition tolerance这三个指标不可能全部做到。Partition tolerance分区容错性大多数分布式系统都分布在多个子网络。每个子网络就叫
permalink: /?p=65
categories:
 - 分布式
tags: 
 - 分布式
---

# 概述

1998年，加州大学的计算机科学家 Eric Brewer 提出，分布式系统有三个指标。

- Consistency
- Availability
- Partition tolerance

这三个指标不可能全部做到。



![bg2018071607](https://tencent-img.feilewu.cn/img/bg2018071607_1618843236535.jpg)

## Partition tolerance

分区容错性

大多数分布式系统都分布在多个子网络。每个子网络就叫做一个区（partition）。分区容错的意思是，区间通信可能失败。

（数据同步失败）

![bg2018071601](https://tencent-img.feilewu.cn/img/bg2018071601_1618843236538.png)

## Consistency

一致性

不管客户端向哪个节点发送请求，得到的结果应该是一致的。

## Availability

可用性

用户只要向集群中任一节点发送了请求，就必须收到响应，否则就不满足可用性。

# 为什么三者不能同时满足

一致性和可用性，为什么不可能同时成立？答案很简单，因为可能通信失败（即出现分区容错）。

如果保证 G2 的一致性，那么 G1 必须在写操作时，锁定 G2 的读操作和写操作。只有数据同步后，才能重新开放读写。锁定期间，G2 不能读写，没有可用性。

如果保证 G2 的可用性，那么势必不能锁定 G2，所以一致性不成立。

综上所述，G2 无法同时做到一致性和可用性。系统设计时只能选择一个目标。如果追求一致性，那么无法保证所有节点的可用性；如果追求所有节点的可用性，那就没法做到一致性。

# 参考

https://www.zhihu.com/question/54105974

http://www.ruanyifeng.com/blog/2018/07/cap.html