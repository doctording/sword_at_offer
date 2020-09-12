---
title: "Redis"
layout: page
date: 2020-09-06 18:00
---

[TOC]

# Redis

* 基于内存的
* 数据是K（String类型）,V（有5种基本数据结构：String,Hash,List,Set,ZSet，注意：memcache value就是string,没有数据类型）
* 单线程(工作线程是单线程的)
* 连接使用（inux中epoll）多路复用
* 主从，集群，持久化等
* 串行化/原子操作

## KV的数据类型说明

比如V即value数据类型是个数组，目前想取出下标是2的元素

1. 没有数据类型，那么客户端要取出全量存储的数据，然后在客户端完成计算（memcache）：
2. 有数据类型，在取数据的时候，服务端完成计算，返回最后结果给客户端（redis）：计算的需求在数据需求产生，IO减少了

## 持久化

* 快照（snapshot）：这种持久化可能会丢失某些时刻的数据
* AOF（append of file）只追加日志文件：将所有redis的写命令记录到日志文件中

### 快照（RDB方式）

redis默认开启的持久化方式，将某一时刻的所有数据保存到磁盘上，保存的文件为.rdb，也称为RDB方式

#### 客户端命令(推荐bgsave)生成快照

客户端可以使用`bgsave`命令创建一个快照，redis主进程会**fork**出一个子进程，fork在linux下如果采用copy-on-write(cow)技术，即刚开始子进程与redis主进程是共享资源的，当主进程有写操作后，则会有page fault, 子进程就会有自己的进程资源空间，子进程完成快照生成

也可以使用save命令，redis主进程负责快照生成，期间不响应任何客户端请求，客户端都将被阻塞，直接快照生成完毕

#### redis.conf中配置save选项

* after 900 sec (15 min) if at least 1 key changed
* after 300 sec (5 min) if at least 10 keys changed
* after 60 sec if at least 10000 keys changed

```cpp
save 900 1
save 300 10
save 60 10000
```

任意一个save选项满足了，则会触发`bgsave`命令

#### shutdown（关闭redis服务器）

内部执行的是redis save命令

### AOF

可以将所有写命令记录到磁盘中，做到不丢失数据

redis.conf中设置`appendonly yes`生成`appendonly.aof`文件

aof文件写入的时机问题（类比mysql redo log持久化策略）

#### appendfysnc 三种选项

redis.conf中的appendfysnc是对redis性能有重要影响的参数之一。可取三种值：always、everysec【推荐】和no。
1. 设置为always时，会极大消弱Redis的性能，因为这种模式下每次write后都会调用fsync（Linux为调用fdatasync）
2. 如果设置为no，则write后不会有fsync调用，由操作系统自动调度刷磁盘，性能是最好的，但可能会丢失不定数量的数据
3. everysec为最多每秒调用一次fsync，这种模式性能并不是很糟糕，一般也不会产生毛刺，即使系统崩溃，那么最多丢失1秒的数据（某些业务允许丢失，比如热搜排行）

## 过期key的删除策略？

* 定时删除: 在设置键的过期时间的同时，创建一个定时器（timer），让定时器在键的过期时间来临时，立即执行对键的删除操作. 即从设置key的Expire开始，就启动一个定时器，到时间就删除该key；这样会对内存比较友好，但浪费CPU资源

* 惰性删除:放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；如果没有过期，就返回该键。 即平时不处理，在使用的时候，先检查该key是否已过期，已过期则删除，否则不做处理；这样对CPU友好，但是浪费内存资源，并且如果一个key不再使用，那么它会一直存在于内存中，造成浪费

* 定期删除:每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。即设置一个定时任务，比如10分钟删除一次过期的key；间隔小则占用CPU,间隔大则浪费内存

## 最大内存&淘汰策略

1. 在配置文件redis.conf 中，可以通过参数`maxmemory <bytes>`来设定最大内存
2. 当现有内存大于 maxmemory 时，便会触发redis主动淘汰内存方式，通过设置`maxmemory-policy`
* volatile-lru:利用LRU算法移除设置过过期时间的key(LRU:最近使用 Least Recently Used)
* allkeys-lru:利用LRU算法移除任何key（和上一个相比，删除的key包括设置过期时间和不设置过期时间的），通常使用该方式
* volatile-random:移除设置过过期时间的随机key
* allkeys-random:无差别的随机移除
* volatile-ttl:移除即将过期的key(minor TTL)
* noeviction:不移除任何key，只是返回一个写错误 ，默认选项

## redis使用场景

1. 手机验证码功能（redis String类型设置超时时间）
2. 具有时效性的一些功能，比如订单未支付关闭
3. 分布式系统中的Session共享
4. 利用zset(排序的，每个元素有个分数)实现类似排行榜功能
5. 分布式缓存
6. 还是利用过期时间，存储认证之后的token信息，token也有时效性
7. 分布式锁

### 分布式缓存

本地缓存：存在应用服务器内存中
分布式缓存：存在当前应用服务器之外的外部的内存存储中

#### 分布式缓存的更新模式

* LRU/LFU/FIFO
* 超时剔除
* 主动更新

#### 失效策略

#### 淘汰策略

#### 缓存击穿，雪崩等问题

1. 缓存空对象
2. 布隆过滤器

## redis架构

### 普通的redis主从结构

普通的主从模式
1. 当主节点挂掉后，那么整个redis就不能提供服务了
2. 从节点仅仅用是同步主节点数据

在slave上写操作会报错(配置：`slave-read-only yes`)

```java
(error) READONLY You can't write against a read only slave.
```

当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。

* slave宕机，只需重启slave机器，自动完成同步数据
* master宕机，先在命令行执行SLAVEOF NO ONE命令，断开主从关系并将slave变成master继续服务，然后将主库重新启动后，执行SLAVEOF命令，将其设置为从库

### 哨兵（sentinel）模式

![](../../content/db_cache/imgs/redis2.webp)

哨兵的两个作用:
* 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
* 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。

![](../../content/db_cache/imgs/redis3.webp)

哨兵执行的工作：
* 监控(Monitoring): 哨兵(sentinel) 会不断地检查你的Master和Slave是否运作正常。
* 提醒(Notification):当被监控的某个 Redis出现问题时, 哨兵(sentinel) 可以通过 API 向管理员或者其他应用程序发送通知。
* 自动故障迁移(Automatic failover):当一个Master不能正常工作时，哨兵(sentinel) 会开始一次自动故障迁移操作,它会将失效Master的其中一个Slave升级为新的Master, 并让失效Master的其它Slave改为复制新的Master; 当客户端试图连接失效的Master时,集群也会向客户端返回新Master的地址,使得集群可以使用Master代替失效Master。

### Redis集群

redis 3.0 之后支持redis集群，解决redis单点问题

* 主从架构
* 读写分离
* 可支持水平扩展的读高并发架构

* 所有的redis节点之间进行检测(PING-PONG)
* 某个节点fail是通过集群只能够半数以上检测失败才确定的
* 客户端与redis节点直连，没有proxy层，客户端只需要连接上某一个节点即可
* redis集群把所有的物理节点映射到[0-16383]slot上，集群维护`node<-->slot<-->value`（node是物理节点，每个节点都有一个范围的slot, CRC-16算法，Key会crc操作到定位到某个物理节点上）

![](../../content/db_cache/imgs/redis4.png)

## redis单线程模型

### redis架构&IO多路复用

Redis基于Reactor模式开发了自己的网络事件处理器，被称为文件事件处理器，由套接字、I/O多路复用程序、文件事件分派器（dispatcher），事件处理器四部分组成

![](../../content/db_cache/imgs/redis5.png)

### 使用单线程的原因？

* Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了。而性能上，普通笔记本能轻松处理每秒几十万的请求。【官方说明】

详细原因：
1. 不需要各种锁的性能消耗：redis支持多种数据结构，不需要加锁，不会出现因为加锁导致的性能开销，死锁问题
2. CPU消耗少，单线程多进程集群方案：因为CPU不会成为瓶颈，单线程不会有上线文切换操作；但是如果CPU成为Redis瓶颈，可以考虑多起几个Redis进程，Redis是key-value数据库，不是关系数据库，数据之间没有约束。只要客户端分清哪些key放在哪个Redis进程上就可以了

### Redis并发竞争key的问题？

Redis是一种单线程机制的nosql数据库，基于key-value，数据可持久化落盘。由于单线程所以Redis本身并没有锁的概念，多个客户端连接并不存在竞争关系，但是利用jedis等客户端对Redis进行并发访问时会出现问题，比如：同时有多个子系统去set一个key，就出现并发竞争key的问题？

如何解决：
* 分布式锁
* 消息队列：在并发量过大的情况下,可以通过消息中间件进行处理,把并行读写进行串行化
