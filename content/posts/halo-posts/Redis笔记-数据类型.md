---
title: Redis笔记-数据类型
id: 66
date: 2024-09-27 19:10:52
auther: admin
cover: 
excerpt: 基本数据类型及应用场景类型简介特性场景String(字符串)二进制安全可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M---Hash(字典)键值对集合,即编程语言中的Map类型适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需
permalink: /?p=66
categories:
 - 中间件
tags: 
 - redis
---

## 基本数据类型及应用场景

| 类型                 | 简介                                                   | 特性                                                         | 场景                                                         |
| -------------------- | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| String(字符串)       | 二进制安全                                             | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M | ---                                                          |
| Hash(字典)           | 键值对集合,即编程语言中的Map类型                       | 适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) | 存储、读取、修改用户属性                                     |
| List(列表)           | 链表(双向链表)                                         | 增删快,提供了操作某一段元素的API                             | 1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列          |
| Set(集合)            | 哈希表实现,元素不重复                                  | 1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作 | 1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐 |
| Sorted Set(有序集合) | 将Set中的元素增加一个权重参数score,元素按score有序排列 | 数据插入集合时,已经进行天然排序                              | 1、排行榜 2、带权重的消息队列                                |

![](https://tencent-img.feilewu.cn/2021/04/20/98105977-58e24f050b6ba.png)

### 字符串（String）

#### 设置值

 set命令:为键设置值.

```
set key value
```

setex命令:为键设置值,并设置秒级的过期时间.

```
setex key value seconds
```

setpx命令:为键设置值,并设置毫秒秒级的过期时间.

```
setpx key value millisecond
```

setnx命令:向键设置值前判断键是否存在,不存在则设置值存在则不设置.

```
setnx key value
```

setxx命令:与setnx相反,向键设置值前判断键是否存在,存在则设置值存在则不设置.

```
setxx key value
```

setnx和setxx的使用场景:由于redis是单线程命令处理机制,如果多个客户端同时向redis服务使用setnx命令也只会有一个客户端执行成功.setnx可以用于分布式锁的一个解决方案.

#### 计数

incr命令:用于对键的值进行自增操作,返回结果为三种:
  - 值不是整数,返回错误.

  - 值是整数,返回自增后的结果.

  - 值不存在,按照值为`0`自增,返回`1`.

```
incr key   
```

 incrby命令:用于对键的值进行指定自增量的自增操作,返回结果与incr相同.

```
incrby key
```

incrbyfloat命令:用于对键的值进行指定浮点型自增量的自增操作.

```
incrbyfloat key
```

decr命令:用于对键的值进行自减操作.
```
decr key
```
decrby命令:用于对键的值进行指定值进行减值操作.
```
decrby key
```
append命令:用于对键的值进行追加操作,返回追加后的值的得长度.
```
append key
```

strlen命令:用于查看键的值得长度.
```
strlen key
```
setrange命令:用于键的值内指定索引字符的替换,如果参数是多个字符则从指定位置开始逐个替换,返回替换后值的长度.
```
setrange key offset value
```

getrange命令:用于获取键的值指定索引的内容.

```
getrange key start end
```

### 哈希（Hash）

#### 常用操作

插入key-field

```
hget key field
```

删除field

```
hdel key field [field1...]
```

计算field数量

```
hlen key
```

批量设置或获取field-value

```
hmset key field [field...]
hmget key field [field...]
```

判断field是否存在

```
hexists key field
```

获取所有field

```
hkeys key
```

获取所有value

```
hvals key
```

获取所有的field-value

```
hgetall key
```

### 列表（List）

![](https://tencent-img.feilewu.cn/2021/04/20/3044188535-58ea3f8ac1e54.png)

#### 1.常规插入与移除

如上图所示，lpush和lpop表示从列表的头部插入和移除；rpush和rpop则从列表尾部插入和移除。

#### 2.查询

范围查询

```
lrange key start end
```

按索引查询

```
lindex key index
```

求长度

```
llen key
```

删除指定元素

```
lrem key count value
```

`lrem`命令会从列表中找到等于`value`的元素进行删除,根据`count`的不同分为三种:

1. `count>0`,从列表中删除指定数量(`count`)的元素.
2. `count<0`,从列表中删除`count`绝对值数量的元素.
3. `count=0`,删除所有.

#### 3.阻塞操作

移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

```
BLPOP key1 [key2 ] timeout
```
移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
```
BRPOP key1 [key2 ] timeout
```

#### 4. 使用场景

##### 1.消息队列

`redis`的`lpush-brpop`命令组合即可实现阻塞队列,生产者客户端使用`lpush`命令向列表插入元素.消费者客户端使用`brpop`命令阻塞式的"抢"列表中的尾部元素.多个客户端保证消息的负载均衡与可用性.

![](https://tencent-img.feilewu.cn/2021/04/20/2640374334-58fc50a7b5afd.png)

##### 2. 文章列表

每个用户都有属于自己的列表，并且是有序的，也支持按范围查询

### 集合（set）

集合(set)类型也是用来保存多个的字符串元素,但和列表不同的是:它的元素是无序且不可重复的,不能通过索引获取元素.如下图,集合user:1:follows中包含着"his"、"it"、"sports"、"music"四个元素,一个集合最多可以存储(2的32次方-1)个元素.
![](https://tencent-img.feilewu.cn/2021/04/20/2126708661-58fc57b1428ea.png)
#### 集合内操作

添加元素(返回添加的元素个数)

```
sadd key value [value...]
```

删除元素(返回成功的元素个数)

```
srem key value [value...]
```

获取元素个数

```
scard key
```

 判断元素是否在集合中(如果元素存在于集合内则返回1,反之返回0)

```
sismember key value
```

随机从集合中返回指定个数元素

```
srandmember key [count]
```

从集合中随机弹出元素

```
spop key
```

获取集合的所有元素

```
smembers key
```

#### 集合间操作

求交集

```
sinter key [key...]
```

求并集

```
sunion key [key...]
```

求差集

```
sdiff key [key...]
```

#### 应用场景

集合类型比较典型的使用场景是标签(tag).例如一个用户可能对音乐感兴趣,另一个用户对新闻感兴趣,这些想去点就是标签.有了这些数据就可以获得喜欢同一个标签的人,以及用户的共同喜好的标签,这些数据对于用户体验来说比较重要.

### 有序集合（sorted set）

#### 集合内操作

添加成员

```
zadd key score member [score member ...]
```

获取成员个数

```
zcard key
```

获取某个成员的分数

```
zscore key member
```

获取成员排名(zrank命令是从分数从低到高返回排名,zrevrank反之,排名从0开始)

```
zrank key member
zrevrank key member
```

删除成员

```
zrem key member [member...]
```

增加成员分数

```
zincrby key score member
```

获取指定范围的元素(有序集合是按照分值排名的,zrange是由低到高返回,zrevrange反之Z)

```
zrange key start end [withscores] 
zrevrange key start end [withscores]
```

返回指定分数范围的成员

```
zrangebyscore key min max [withscores] [limit offset count]
zrevrangebyscore key min max [withscores] [limit offset count]
```

返回指定分数范围成员个数

```
zcount key min max
```

删除指定排名内的升序元素

```
zremrangebyrank key start end
```

 删除指定分数范围的成员

```
zremrangebyscore key min max
```

#### 集合间操作

 交集

```
zinterstore storeKey keyNum key [key ...] [weights weight [weight...]] [aggregate sum|min|max]
```

- `storeKey`:交集计算结果保存到这个键下.
- `keyNum`:需要做交集的键的个数.
- `key[key ...]`:需要做交集的键.
- `weights weight [weight...]`:每个键的权重,在做交集计算时,每个键中的每个`member`的分值会和这个权重相乘,每个键的权重默认为`1`.
- `aggregate sum|min|sum`:计算成员交集后,分值可以按照`sum`(和)、`min`(最小值)、`max`(最大值)做汇总.默认值为`sum`.

并集

```
zunionstore storeKey keyNum key [key...] [weights weight [weight...]] [aggregate sum|min|max]
```

参数类似于交集

## 字符编码

![](https://tencent-img.feilewu.cn/2021/04/20/1753739797-58e256966df9b.png)