---
title: Zookeeper概述与安装
id: 4
date: 2024-09-27 19:10:52
auther: admin
cover: 
excerpt: 特性ZooKeeper 解决的本质问题消息传递：延迟性，先发送的消息，不一定先到达；消息传递：丢失性，发送的消息，可能丢失；节点崩溃：分布式系统内，任何一个节点都可能崩溃。如何保证分布式一致性顺序一致性：同一个 Client 发起的事务请求，严格按照发起顺序执行原子性：事务请求，要么应用到所有节点，
permalink: /?p=4
categories:
 - 分布式
tags: 
 - 分布式
 - cap
 - zab
---

## 特性
### ZooKeeper 解决的本质问题

1. 消息传递：延迟性，先发送的消息，不一定先到达；
1. 消息传递：丢失性，发送的消息，可能丢失；
1. 节点崩溃：分布式系统内，任何一个节点都可能崩溃。
### 如何保证分布式一致性

- 顺序一致性：同一个 Client 发起的事务请求，严格按照发起顺序执行
- 原子性：事务请求，要么应用到所有节点，要么一个节点都没有应用
- 单一视图：Client 无论连接到哪个节点，看到的服务端数据都是一致的（Note：不准确，其实是最终一致性）
- 可靠性：事务一旦执行成功，状态永久保留
- 实时性：事务一旦执行成功，Client 并不能立即看到最新数据，但 ZooKeeper 保证最终一致性





官方文档：[https://zookeeper.apache.org/doc/r3.6.2/zookeeperStarted.html](https://zookeeper.apache.org/doc/r3.6.2/zookeeperStarted.html)
## 安装（centos 7）
### 下载
```shell
wget https://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz
tar zvxf apache-zookeeper-3.6.2-bin.tar.gz
cd apache-zookeeper-3.6.2-bin
```
### 配置
#### 单机模式

1. 创建配置文件 conf/zoo.cfg
```shell
vi conf/zoo.cfg
```

2. 写入配置
```shell
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

- **_tickTime_** : the basic time unit in milliseconds used by ZooKeeper. It is used to do heartbeats and the minimum session timeout will be twice the tickTime.
- **_dataDir_** : the location to store the in-memory database snapshots and, unless specified otherwise, the transaction log of updates to the database.
- **_clientPort_** : the port to listen for client connections
3. 启动
```shell
bin/zkServer.sh start
```

4. 连接
```shell
bin/zkCli.sh -server 127.0.0.1:2181
```
```shell
Connecting to localhost:2181
log4j:WARN No appenders could be found for logger (org.apache.zookeeper.ZooKeeper).
log4j:WARN Please initialize the log4j system properly.
Welcome to ZooKeeper!
JLine support is enabled
[zkshell: 0]
```
