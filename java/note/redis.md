
# Redis

[TOC level=1-6]:# ""


- [Redis](#redis)
  - [一、Redis 数据类型](#一redis-数据类型)
    - [（1）、字符串类型 string](#1字符串类型-string)
    - [(2)、哈希类型 hash](#2哈希类型-hash)
    - [（3）、列表类型 list](#3列表类型-list)
    - [(4)、集合类型 set](#4集合类型-set)
    - [(5)、有序集合类型 zset （sorted set）](#5有序集合类型-zset-sorted-set)
  - [二、redis持久化](#二redis持久化)
    - [持久化概述](#持久化概述)
      - [1、RDB](#1rdb)
      - [实现方式](#实现方式)
      - [总结：](#总结)
      - [2、AOF](#2aof)
      - [实现方式](#实现方式-1)<!-- @IGNORE PREVIOUS: anchor -->
      - [总结：](#总结-1)<!-- @IGNORE PREVIOUS: anchor -->
  - [三、主从复制（master/slave）](#三主从复制masterslave)
    - [1、面临问题](#1面临问题)
    - [2、解决办法](#2解决办法)
    - [3、什么是主从复制](#3什么是主从复制)
      - [设置方式](#设置方式)
    - [4、容灾处理](#4容灾处理)
    - [5、总结](#5总结)
  - [三、高可用Sentinel哨兵](#三高可用sentinel哨兵)
    - [1、配置](#1配置)


## 一、Redis 数据类型 ##

### （1）、字符串类型 string ###

字符串类型是Redis中最基本的数据类型，它能存储任何形式的字符串，包括二进制数
据，序列化后的数据，JSON化的对象甚至是一张图片。  
常用命令  
**1、set**  
将字符串值 value 设置到 key 中  
**2、get**  
获取 key 中设置的字符串值  
**3、incr**  
  将 key 中储存的数字值加1，如果 key 不存在，则 key 的值先被初始化为 0 再执行 INCR 操作
（只能对数字类型的数据操作）  
**4、decr**  
将 key 中储存的数字值减1，如果 key 不存在，则么 key 的值先被初始化为 0 再执行 DECR 操作
（只能对数字类型的数据操作）  
**5、setex**  
set expire的简写，设置key的值 ，并将 key 的生存时间设为 seconds (以秒为单位)  
**6、setnx**  
setnx 是 set if not exists 的简写，如果key不存在，则 set 值，存在则不设置值  
**7、getset**  
设置 key 的值为 value ，并返回 key 的旧值

### (2)、哈希类型 hash ###

Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。
常用命令  
**1、hset**  
将哈希表 key 中的域 field 的值设为 value  
**2、hget**  
获取哈希表 key 中给定域 field 的值  
**3、hmset**  
  同时将多个 field-value (域-值)对设置到哈希表 key 中  
**4、hmget**  
获取哈希表 key 中一个或多个给定域的值  
**5、hgetall**  
获取哈希表 key 中所有的域和值  
**6、hdel**  
删除哈希表 key 中的一个或多个指定域field  
**7、hkeys**  
查看哈希表 key 中的所有field域  
**8、hvals**  
查看哈希表 key 中所有域的值
### （3）、列表类型 list ###
Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）  
常用命令  
**1、lpush**  
将一个或多个值 value 插入到列表 key 的表头（最左边）  
**2、rpush**  
将一个或多个值 value 插入到列表 key 的表尾（最右边）  
**3、lrange**  
获取列表 key 中指定区间内的元素，0 表示列表的第一个元素，以 1 表示列表的第二个元素， -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。  
**4、lpop**  
从左边获取列表 key 的一个元素，并将该元素移除  
**5、rpop**  
从右边获取列表 key 的一个元素，并将该元素移除  
**6、lindex**  
获取列表 key 中下标为指定 index 的元素  
**7、llen**  
获取列表 key 的长度  
**8、lrem**  
从左到右删除列表中指定个数的并与指定value值相等的value
### (4)、集合类型 set ###
Redis的Set是string类型的无序集合，集合成员是唯一的，即集合中不能出现重复的数据  
常用命令  
**1、sadd**  
将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将不会再加入  
**2、smembers**  
获取集合 key 中的所有成员元素  
**3、sismember**  
判断 member 元素是否是集合 key 的成员  
**4、 scard**  
获取集合里面的元素个数  
**5、 srem**  
删除集合 key 中的一个或多个 member 元素  
**6、 srandmember**  
随机返回集合中的一个元素  
**7、spop**  
随机从集合中删除一个元素  
**8、 smove**  
将 member 元素从一个集合移动到另一个集合
### (5)、有序集合类型 zset （sorted set） ###
Redis 有序集合zset和集合set一样也是string类型元素的集合，且不允许重复的成员。
不同的是zset的每个元素都会关联一个分数（分数可以重复），redis通过分数来为集合中的成员进行从小到大的排序。  
常用命令  
**1、zadd**  
将一个或多个 member 元素及其 score 值加入到有序集合 key 中  
**2、zrem**  
删除有序集合 key 中的一个或多个成员  
**3、zcard**  
获取有序集 key 的元素成员的个数  
**4、 zrank**  
获取有序集 key 中成员 member 的排名，有序集成员按 score 值从小到大顺序排列  
**5、 zrevrank**  
获取有序集 key 中成员 member 的排名，有序集成员按 score 值从大到小顺序排列  
**6、 zrangebyscore**  
获取有序集 key 中，所有 score 值介于 min 和 max 之间的成员  
**7、zrevrangebyscore**  
获取有序集 key 中， score 值介于 max 和 min 之间的所有的成员  
**8、 zcount**  
获取有序集 key 中，所有 score 值介于 min 和 max 之间的成员的个数
**9、zrange**  
获取有序集 key 中，指定区间内的成员，按 score 值从小到大排列  
**10、 zrevrange**  
获取有序集 key 中，指定区间内的成员，按 score 值从大到小排列
## 二、redis持久化 ##
### 持久化概述 ###
持久化可以理解为存储，就是将数据存储到一个不会丢失的地方，如果把数据放在内存中，电脑关闭或重启数据就会丢失，所以放在内存中的数据不是持久化的，而放在磁盘就算是一种持久化。  
Redis的数据存储在内存中，内存是瞬时的，如果linux宕机或重启，又或者Redis崩溃或重启，所有的内存数据都会丢失，为解决这个问题，Redis提供两种机制对数据进行持久化存储，便于发生故障后能迅速恢复数据。
#### 1、RDB ####
Redis Database（RDB），就是在指定的时间间隔内将内存中的数据集快照写入磁盘，数据恢复时将快照文件直接再读到内存。
#### 实现方式 ####
RDB方式的数据持久化，仅需在redis.conf文件中配置即可
* 配置格式：```save <seconds> <changes> ```  
save 900 1  
save 300 10  
save 60 10000
* dbfilename：设置RDB的文件名，默认文件名为dump.rdb
* dir：指定RDB和AOF文件的目录，默认是 ./
#### 总结： ####
* 优点：由于存储的是数据快照文件，恢复数据很方便，也比较快：
* 缺点：会丢失最后一次快照以后更改的数据
* 如果你的应用能容忍一定数据的丢失，那么使用rdb是不错的选择，如果你不能容忍一定数据的丢失，使用rdb就不是一个很好的选择
* 由于需要经常操作磁盘，RDB 会经常 fork 出一个子进程。如果你的redis数据库很大的话，Fork 占用比较多的时间，并且可能会影响 Redis 暂停服务一段时间（millisecond 级别），如果你的数据库超级大并且你的服务器CPU比较弱，有可能是会达到一秒。
#### 2、AOF ####
Append-only File（AOF），Redis每次接收到一条改变数据的命令时，它将把该命令写到一个AOF文件中（只记录写操作，读操作不记录），当Redis重启时，它通过执行AOF文件中所有的命令来恢复数据。
#### 实现方式 ####
AOF方式的数据持久化，仅需在redis.conf文件中配置即可
* appendonly：默认是no，改成yes即开启了aof持久化
* appendfilename：指定AOF文件名，默认文件名为appendonly.aof
* appendfsync：配置向aof文件写命令数据的策略：  
  no：不主动进行同步操作，而是完全交由操作系统来做（即每30秒一次），比较快但不是很安全  
  always：每次执行写入都会执行同步，慢一些但是比较安全  
  everysec：每秒执行一次同步操作，比较平衡，介于速度和安全之间
*  dir：指定AOF和RDB文件的目录
*  auto-aof-rewrite-percentage：当目前aof文件大小超过上一次重写时的aof文件大小的百分之多少时会再次进行重写，如果之前没有重写，则以启动时的aof文件大小为依据
*  auto-aof-rewrite-min-size：允许重写的最小AOF文件大小
#### 总结： ####
*  Append-only File（AOF）是另一个可以提供完全数据保障的方案；
*  AOF 文件会在操作过程中变得越来越大。比如，如果你做一百次加法计算，最后你只会在数据库里面得到最终的数值，但是在你的 AOF 里面会存在 100 次记录，其中 99 条记录对最终的结果是无用的
*   Redis 支持在不影响服务的前提下在后台重构 AOF 文件，让文件得以整理变小
*  可以同时使用这两种方式，redis默认优先加载aof文件
## 三、主从复制（master/slave） ##
### 1、面临问题 ###
通过持久化功能，Redis保证了即使在服务器重启的情况下也不会丢失（或少量丢失）数
据，但是由于数据是存储在一台服务器上的，如果这台服务器出现故障，比如硬盘坏了，也会导致数据丢失。


### 2、解决办法 ###

为了避免单点故障，我们需要将数据复制多份部署在多台不同的服务器上，即使有一台服务器出现故障其他服务器依然可以继续提供服务。

Redis提供了复制（replication）功能来自动实现多台redis服务器的数据同步

### 3、什么是主从复制 ###

通过部署多台redis，并在配置文件中指定这几台redis之间的主从关系，主负责写入数据，同时把写入的数据实时同步到从机器，这种模式叫做主从复制，即master/slave，并且redis默认master用于写，slave用于读，向slave写数据会导致错误

默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

![](https://segmentfault.com/img/bVboOBE?w=378&h=176)

#### 设置方式 ####
* 方式1：修改配置文件，启动时，服务器读取配置文件，并自动成为指定服务器的从服务器，从而构成主从复制的关系

主服务器（master）  
daemonize yes  
port 6380  
pidfile /var/run/redis_6380.pid  
logfile 6380.log  
dbfilename dump6380.rdb

从服务器（slave）  
daemonize yes  
port 6382  
pidfile /var/run/redis_6382.pid  
logfile 6382.log  
dbfilename dump6382.rdb  
slaveof 127.0.0.1 6380

方式2： ./redis-server --slaveof <master-ip> <master-port>，在启动redis时指定当前服务成为某个主Redis服务的从Slave

/usr/local/redis-3.2.9/src/redis-server /usr/local/redis-3.2.9/redis6381.conf --slaveof 127.0.0.1 6380 &


### 4、容灾处理 ###

当Master服务出现故障，需手动将slave中的一个提升为master， 剩下的slave挂至新的master上

>slaveof no one，将一台slave服务器提升为Master （提升某slave为master）

>slaveof 127.0.0.1 6381 （将slave挂至新的master上）


### 5、总结 ###

* 一个master可以有多个slave
* slave下线，读请求的处理性能下降
* master下线，写请求无法执行
* 当master发生故障，需手动将其中一台slave使用slaveof no one命令提升为master，其它slave执行slaveof命令指向这个新的master，从新的master处同步数据
* 主从复制模式的故障转移需要手动操作，要实现自动化处理，这就需要Sentinel哨兵，实现故障自动转移

## 三、高可用Sentinel哨兵 ##

Sentinel哨兵是redis官方提供的高可用方案，可以用它来监控多个Redis服务实例的运行情况。

### 1、配置 ###

复制三份sentinel.conf文件：
* sentinel26380.conf
* sentinel26381.conf
* sentinel26382.conf  
三份sentinel配置文件修改：  
1、修改 port 26380、 port 26381、 port 26382  
2、修改 sentinel monitor mymaster 127.0.0.1 6380 2


```
 格式：Sentinel monitor <name> <masterIP> <masterPort><Quorum投票数> 
```

Sentinel会根据Master的配置自动发现Master的Slave  
Sentinel默认端口号为26379


配置举例  
执行以下三条命令，将创建三个监视主服务器的Sentinel实例：  
./redis-sentinel  ../sentinel26380.conf  
./redis-sentinel  ../sentinel26381.conf  
./redis-sentinel  ../sentinel26382.conf


监控：

* Sentinel会不断检查Master和Slave是否正常
* 如果Sentinel挂了，就无法监控，所以需要多个哨兵，组成Sentinel网络
* 监控同一个Master的Sentinel会自动连接，组成一个分布式的Sentinel网络，互相通信并交换彼此关于被监控服务器的信息
* 当一个Sentinel认为被监控的服务器已经下线时，它会向网络中的其它Sentinel进行确认，判断该服务器是否真的已经下线
* 如果下线的服务器为主服务器，那么Sentinel网络将对下线主服务器进行自动故障转移，通过将下线主服务器的某个从服务器提升为新的主服务器，并让其从服务器转移到新的主服务器下，以此来让系统重新回到正常状态
* 下线的旧主服务器重新上线，Sentinel会让它成为从，挂到新的主服务器下

总结：
 * 主从复制，解决了读请求的分担，从节点下线，会使得读请求能力有所下降，Master下线，写请求无法执行
 * Sentinel会在Master下线后自动执行故障转移操作，提升一台Slave为Master，并让其它Slave成为新Master的Slave
