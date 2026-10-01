---
title: 利用Redisson快速实现分布式锁--Redisson的介绍与使用
id: 74
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: 什么是RedissonRedisson 是一个使用 Java编写的Redis 客户端。 其以Redis为基础，构建了一种内存数据网格（ in-memory data grid），并提供了redis支持的分布式Java对象和服务。包括超过50中Java对象和若干服务（Set, Multimap, So
permalink: /?p=74
categories:
 - 分布式
tags: 
 - java
 - redisson
 - 分布式锁
---

## 什么是Redisson

Redisson 是一个使用 Java编写的Redis 客户端。 其以Redis为基础，构建了一种内存数据网格（ **in-memory data grid**），并提供了redis支持的分布式Java对象和服务。包括超过50中Java对象和若干服务（Set, Multimap, SortedSet, Map, List, Queue, Deque, Semaphore, Lock, AtomicLong, Map Reduce, Publish / Subscribe, Bloom filter, Spring Cache, Tomcat, Scheduler, JCache API, Hibernate, MyBatis, RPC, local cache...）。

简单来讲，Redisson可以帮助我们集中精力处理业务，而不必将过多的精力投入到redis的用法上。在项目中我们常常用它来实现redis分布式锁，替代原生redis+lua脚本的方式。

## Maven依赖

项目地址https://github.com/redisson/redisson

```
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.15.5</version>
</dependency>
```

## 配置

### Redis配置

在使用之前，首先得确保[redis](https://www.feilewu.cn/?p=66)的安装与使用。在这里不做赘述。

Redisson支持如下的redis配置方式：

- 单机模式
- 主从模式
- 哨兵模式
- 集群模式
- [复制节点](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.Redis.Groups.html)（**Amazon Web Services (AWS) ElastiCache Cluster and Azure Redis Cache**）

### Java代码

```java
Config config = new Config();
config.useSingleServer()
  .setAddress("redis://127.0.0.1:6379");

RedissonClient client = Redisson.create(config);
```

本文示例采用redis单机模式的配置，首先利用SingleServerConfig构建出单机模式下redis的配置对象，并使用setAddress 方法配置IP和端口信息，除此之外，还可以通过对应的setter方法设置retryAttempts、connectionTimeout 和 clientName等参数。然后，通过SingleServerConfig构建出Config对象，并将其传递给Redisson类的create静态方法，获得RedissonClient对象，后续我们便可以使用其进行各种业务操作。

除了单机模式我们还可以使用其他配置类来配置其他redis部署模式，比如

- *useMasterSlaveServers*  配置主从模式
- *useSentinelServers* 配置哨兵模式
- *useClusterServers*  配置集群模式
- *useReplicatedServers* 配置复制节点模式

### 文件配置

同时，redisson还支持以文件的形式进行配置

```java
Config config = Config.fromJSON(new File("singleNodeConfig.json"));  
RedissonClient client = Redisson.create(config);
```

同时，我们在singleNodeConfig.json文件中写入

```text
{
    "singleServerConfig": {
        "idleConnectionTimeout": 10000,
        "connectTimeout": 10000,
        "timeout": 3000,
        "retryAttempts": 3,
        "retryInterval": 1500,
        "password": null,
        "subscriptionsPerConnection": 5,
        "clientName": null,
        "address": "redis://127.0.0.1:6379",
        "subscriptionConnectionMinimumIdleSize": 1,
        "subscriptionConnectionPoolSize": 50,
        "connectionMinimumIdleSize": 10,
        "connectionPoolSize": 64,
        "database": 0,
        "dnsMonitoringInterval": 5000
    },
    "threads": 0,
    "nettyThreads": 0,
    "codec": null
}
```

如果采用yaml文件配置方式

```yaml
singleServerConfig:
    idleConnectionTimeout: 10000
    connectTimeout: 10000
    timeout: 3000
    retryAttempts: 3
    retryInterval: 1500
    password: null
    subscriptionsPerConnection: 5
    clientName: null
    address: "redis://127.0.0.1:6379"
    subscriptionConnectionMinimumIdleSize: 1
    subscriptionConnectionPoolSize: 50
    connectionMinimumIdleSize: 10
    connectionPoolSize: 64
    database: 0
    dnsMonitoringInterval: 5000
threads: 0
nettyThreads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
```

同时，如果我们想将当前配置保存到文件中，可以使用使用Config类的toJSON()和toYAML()方法

```java
config.toJSON();
config.toYAML();
```

### Redisson的具体使用

Redisson 支持同步、异步和反应式的编程方式，并且其提供的接口都是线程安全的。

RedissonClient 生成的所有实体（对象、集合、锁和服务）均同时具有同步和异步的方法。 一般的，一个方法对应的异步方法通过Async来进行修饰。

以RAtomicLong为例

```java
RAtomicLong myLong = client.getAtomicLong("mylong");
myLong.compareAndSet(0,10);
RFuture<Boolean> isSet = myLong.compareAndSetAsync(0, 10);
isSet.handle((result,exception)->{
    return result;
});
```

可以看到RAtomicLong对应的同步和异步方法，异步与同步不同的是，其返回一个RFuture<Boolean>对象，我们需要注册监听其来对异步返回的结果进行处理。

对于响应式的接口，我们需要先生成响应式的客户端对象*RedissonReactiveClient*：

```java
RedissonReactiveClient client = Redisson.createReactive();
```

后续操作类似，不在赘述。

### 对象支持

Redisson 遵循java.util.concurrent.atomic 包中的规范，实现了分布式的对象，通过Redisson可以对这些存储在redis中的对象进行无锁的、线程安全的、原子的操作。这些对象的单个实例被序列化存储在redis集群中的任何可用节点里，可以被单个应用程序或者多个应用程序访问。

Redisson 将对象与Redis键绑定起来。我们可以通过 RKeys 接口管理这些键，并通过这些键访问Redisson对象。

```java
RKeys keys = client.getKeys();
Iterable<String> allKeys = keys.getKeys();
Iterable<String> keysByPattern = keys.getKeysByPattern('key*')
```

Redisson支持以下几种对象：

- ObjectHolder
- BinaryStreamHolder
- GeospatialHolder
- BitSet
- AtomicLong
- AtomicDouble
- Topic
- BloomFilter
- HyperLogLog

### 集合支持

- Map
- Multimap
- Set
- SortedSet
- ScoredSortedSet
- LexSortedSet
- List
- Queue
- Deque
- BlockingQueue
- BoundedBlockingQueue
- BlockingDeque
- BlockingFairQueue
- DelayedQueue
- PriorityQueue
- PriorityDeque

### 锁和同步器

- Lock
- FairLock
- MultiLock
- ReadWriteLock
- Semaphore
- PermitExpirableSemaphore
- CountDownLatch

#### Lock（分布式锁）

```java
RLock lock = client.getLock("hello");
try {
    lock.tryLock(500,1500, TimeUnit.MILLISECONDS);
} catch (InterruptedException e) {
    e.printStackTrace();
}finally {
    lock.unlock();
}
```

这句代码就可以用来实现最简单的分布式锁了，是不是比原生setnx+lua脚本要方便许多呢。

其中，本质上还是Redisson已经对原生的lua脚本进行了封装。

具体实现我们以哨兵模式为例：

![img](https://tencent-img.feilewu.cn/img/2021/06/07/07-50-22-9bb938f2ba3172a33b3b49912aad14cb-6918995-ffa4b1382b8fc19a-eb1ecd.png)

在如图所示的架构中，应用程序要想获得分布式锁，需要向通过EVAL指令向3个哨兵集群中执行lua脚本，需要多余半数的节点响应成功，才能算获取到分布式锁。

问题点：

失效时间（过期时间）一旦达到失效时间，程序还没有执行完对应的任务，锁自动失效后，其他应用线程将会拿到分布式锁，导致不可预知的错误。这也是基于redis分布式锁的难题，在一定场景下，可以考虑使用Zookeeper来进行实现。

#### **MultiLock**

简单来说，将多个RLock对象进行分组，并这些RLock锁合并成一个大锁，以便对其进行统一的管理，当申请加锁时，可以一次性的将所有小锁的资源上锁，完成任务后，可以一次性的同时释放掉所有上锁的资源。值得注意的是，这个RLock对象可能来自于不同的Redisson实例。为了避免redis节点崩溃而导致的节点锁无法释放的问题，Redisson允许设置leaseTime参数，一旦达到指定的时间间隔，所有的锁将会自动释放，避免发生锁被永远挂起的情况。

