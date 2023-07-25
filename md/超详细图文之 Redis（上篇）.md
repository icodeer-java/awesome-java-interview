> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247485288&idx=1&sn=6e5c9d236f83595264d85fa143043cfb&chksm=e9114595de66cc837fd976b1e17fc2f99e72a9e276f3b68a108d89be2f6cd0ecdd205561d319&scene=178&cur_album_id=2162186087471906816#rd)

*   关于 Redis 的专题，将会由**上篇**和**下篇**这两篇文章来讲解，竭力收录了大部分关于 Redis 入门、使用、部署、面试题等知识点。本文为上篇的部分，内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaf7LFJibxSoaF5G5XibyFQtcAia1HEujonwo3ChOIYA2HAVVHLVoa2gZfg/640?wx_fmt=png)

*   那么闲话少叙，我们步入正题吧~
    

一、Redis 安装和启动  

================

*   Redis Download 官网
    
    https://redis.io/download
    

1.1> 单机安装
---------

*   下载、解压、编译
    

```
$ wget http://download.redis.io/releases/redis-6.2.6.tar.gz
$ tar xzf redis-6.2.6.tar.gz
$ cd redis-6.2.6
$ make

```

*   查看 redis 开头的命令
    

```
$ cd /home/muse/redis/redis-6381/src
$ ll | grep redis

```

*   启动服务端
    

```
$ src/redis-server --daemonize yes

```

*   查看 redis 进程是否启动
    

```
ps -ef | grep redis

```

*   启动客户端
    

```
$ src/redis-cli

```

*   清空 DB 所有内容
    

```
127.0.0.1:6379> FLUSHALL

```

*   查看 DB 中数据数量
    

```
127.0.0.1:6379> DBSIZE

```

1.2> 集群安装（一主二从）
---------------

*   复制 redis
    

```
$ cp redis-6379 redis-6380
$ cp redis-6379 redis-6381

```

*   复制自定义配置文件
    

```
$ cd redis-6379
$ mkdir master_slave
$ cp redis.conf ./master_slave/
$ vim ./master_slave/redis.conf

```

*   redis-6379/master_slave/redis.conf 配置文件内容
    

```
port 6379
bind 127.0.0.1
requirepass "myredis"
daemonize yes
logfile "6379.log"
dbfilename "dump-6379.rdb"

#如若master设置了认证密码，那么所有redis数据节点都配置上masterauth属性
masterauth "myredis"

```

*   redis-6380/master_slave/redis.conf 配置文件内容
    

```
port 6380
bind 127.0.0.1
requirepass "myredis"
daemonize yes
logfile "6380.log"
dbfilename "dump-6380.rdb"

#如若master设置了认证密码，那么所有redis数据节点都配置上masterauth属性
masterauth "myredis"
slaveof 127.0.0.1 6379

```

*   redis-6381/master_slave/redis.conf 配置文件内容
    

```
port 6381
bind 127.0.0.1
requirepass "myredis"
daemonize yes
logfile "6381.log"
dbfilename "dump-6381.rdb"

#如若master设置了认证密码，那么所有redis数据节点都配置上masterauth属性
masterauth "myredis"
slaveof 127.0.0.1 6379

```

*   节点服务端启动
    

```
$ cd /home/muse/redis
$ ./redis-6379/src/redis-server ./redis-6379/master_slave/redis.conf 
$ ./redis-6380/src/redis-server ./redis-6380/master_slave/redis.conf 
$ ./redis-6381/src/redis-server ./redis-6381/master_slave/redis.conf

```

*   节点客户端启动
    

```
$ redis-cli -h 127.0.0.1 -p 6379 // 主客户端可read & write （如果是本机启动，也可以不指定 -h 127.0.0.1 ）
$ redis-cli -h 127.0.0.1 -p 6380 // 从客户端只可读 read only
$ redis-cli -h 127.0.0.1 -p 6381 // 从客户端只可读 read only

```

*   登录授权
    

```
127.0.0.1:6379> keys *
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth myredis
OK

```

*   查看主从关系
    

```
$ ./redis-6379/src/redis-cli -h 127.0.0.1 -p 6379 info replication
or
127.0.0.1:6380> info replication

```

1.3> 使用 Redis 的网页测试工具
---------------------

*   网页测试工具
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaWmchH3iaA4bx5iaO1rqLRboclZOqga8ARe2PicmCjj2pqooia3ePibde4cg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaHVCFkKeia4Cgxxl3DoKlEg6vSbNGU3XxtHADZXrrt1Lt9fRdvB7SbxA/640?wx_fmt=png)

二、Redis 基础
==========

*   Redis 有几种基本数据结构？分别是什么？分别什么情况下会使用？
    
    这是一道 Redis 必考的面试题，针对这个问题，我们引出下面的课程内容，一共几种呢？**5 种**！！分别是 **string（字符串）**，**list（列表）**，**set（集合）**，**zset（有序集合）**，**hash（字典）**。那么我们针对这五种技术数据结构，来一一介绍一下。
    

2.1> Redis 对象
-------------

*   首先在介绍 String 类型之前，我们先了解一下什么是 redis 对象，如下所示：
    

```
redis.h
/*
 * redis对象
 */
typedef struct redisObject {
    // 对象的类型（取值范围：REDIS_STRING, REDIS_LIST, REDIS_HASH, REDIS_SET, REDIS_ZSET）
    unsigned type:4;
    // 对象的编码（取值范围：REDIS_ENCODING_INT, REDIS_ENCODING_EMBSTR, REDIS_ENCODING_RAW, REDIS_ENCODING_HT, REDIS_ENCODING_LINKEDLIST,REDIS_ENCODING_ZIPLIST,REDIS_ENCODING_INTSET,REDIS_ENCODING_SKIPLIST）
    unsigned encoding:4;
    // 指向底层实现数据结构的指针
    void *ptr;
    unsigned notused:2;     /* Not used */
    unsigned lru:22;        /* lru time (relative to server.lruclock) */
    int refcount;   
} robj;

```

*   type——对象类型
    
    每当我们在 redis 中新建一个键值对的时候，我们至少会创建两个对象，一个是 key 的对象，一个是 value 的对象。key 对象总是一个字符串类型的对象，而值对象，则会是 5 种对象类型中的任意一种。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaUyQ600Q1TEp9xLp89eEibddsGzIoniaM0JStjV2I8dQe1HgECmI8cz3w/640?wx_fmt=png)

*   可以通过 TYPE key 命令查看当前 value 的类型，如下所示：
    

```
127.0.0.1:6379> set muse java
OK
127.0.0.1:6379> type muse
string

```

```
127.0.0.1:6379> rpush queue 1 3 4 5
(integer) 4
127.0.0.1:6379> type queue
list

```

*   encoding——编码
    
    ptr 指向对象的底层实现数据结构，而这些数据结构由 encoding 属性决定。通过 encoding 属性来设定对象所使用的编码，而不是为特定类型的对象关联一种固定的编码，极大地提升了 Redis 的灵活性和效率，因为 Redis 可以根据不同的使用场景来为一个对象设置不同的编码，从而优化对象在某一场景下的效率。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaewXTroWBVmIeKJDKYC96WtvJtIOlDyo7iaTMXQNzsLJwJfYhYmQlyRw/640?wx_fmt=png)

*   其中，每种类型的对象都至少使用了**两种**不同的编码。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaHc4E7tkAjtgPyCER8Wc8Y4W4omX48FiaDpib1XgKx5hRU1yBGxFicr5SQ/640?wx_fmt=png)

*   redis3.2 新增了 List 编码类型——quickList；可以通过 **OBJECT ENCODING key** 命令查看当前 value 的类型，如下所示： 
    

```
127.0.0.1:6379> OBJECT encoding muse
"embstr"

```

2.2> string（字符串）
----------------

### 2.2.1> 概述

*   适用场景：缓存业务信息，且只是根据 key 直接获取缓存 value，不需要排序，去重等功能。
    
*   string 是 Redis 中最简单的数据结构，但是却是大家日常使用频率最高的数据结构。它使用简单，并且扩展性非常的强。我们可以设置普通的字符串，也可以设置 json。存取速度也是最块的。
    

*   字符串存储的底层结构其实就是字符数组。
    

*   这个字符串是动态的，是可以修改的。内部采用预分配冗余空间的方式来减少内存的频繁分配。
    

*   当存储的字符串大小 <=1M 的时候，都是翻倍扩容。
    
*   如果存储的字符串大小 > 1M 的时候，则每次只扩容 1M 空间。
    
*   字符串最大不能超过 512M。
    

### 2.2.2> 基本操作

*   赋值 no_1 为 muse
    

```
set no_1 muse

```

*   获取 no_1 的 value 值
    

```
get no_1

```

*   删除 no_1
    

```
del no_1

```

*   设置 no_1 10 秒后过期
    

```
expire no_1 10
setex no_1 10 muse

```

注意：注意，假如 no_1 已存在，那么该语句会更新 no_1 的 value 为 muse，并且过期时间设置为 10 秒

*   查看 no_1 还有多久过期
    

```
ttl no_1

```

*   如果 no_1 存在，就不赋值
    

```
setnx no_1 bob

```

*   计算 value 值长度
    

```
strlen numbers

```

*   计数
    

```
set score 0
incr score
incrby score 10
incrby score -5

```

*   批量操作
    

```
mset no_1 muse no_2 john no_3 bob
mget no_1 no_2 no_3

```

### 2.2.3> 内部实现

*   字符串对象可以使用 **int**、**raw**、**embstr** 这三种 encoding。那么分别什么情况下会选择不同 encoding 呢？
    

#### a> int

*   如果保存的是可以用 long 类型表示的整数值，那么 encoding 为 int。 
    

```
127.0.0.1:6379> set numbers 100
OK
127.0.0.1:6379> OBJECT encoding numbers
"int

```

```
127.0.0.1:6379> set numbers 11111111111111111111111111111111111111111111111111111111111111111111111111
OK
127.0.0.1:6379> OBJECT encoding numbers
"raw"

```

*   数据结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaIsNysXnlIu9OHo5ee7WadDPRPDPlicZoWqOlGCibdO0dUz5qHStnRe9A/640?wx_fmt=png)

#### b> embstr

*   保存的值小于 40，则使用 embstr
    
*   embstr 编码是专门用于保存**短字符串**的一种优化编码方式，它与 raw 编码的区别是，raw 编码会调用两次内存分配函数来分别创建 redisObject 和 sdshdr，而 embstr 编码则通过**调用一次内存分配函数**来分配一块连续的空间包含 redisObject 和 sdshdr。并且 embstr 编码的字符串对象的所有数据都保存再一块连续的内存里面，所以执行速度更快。
    

```
127.0.0.1:6379> set num 123456789012345678901234567890123456789
OK
127.0.0.1:6379> STRLEN num
(integer) 39
127.0.0.1:6379> OBJECT encoding num
"embstr"

```

*   Redis 没有为 embstr 编码的字符串对象提供修程序，所以，**embstr 是只读的**。如果我们对其进行修改，其实是先转换成 raw，再执行修改命令。所以，修改后 embstr 就会变为 raw 编码的字符串对象了。
    

```
127.0.0.1:6379> set name muse
OK
127.0.0.1:6379> OBJECT encoding name
"embstr"
127.0.0.1:6379> APPEND name boy
(integer) 7
127.0.0.1:6379> OBJECT encoding name
"raw"

```

*   数据结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiazLASribWex6Q2zcGZC6EtLALWwsnSolmI5FLvTBHSReSWwNia3qQEalA/640?wx_fmt=png)

#### c> raw  

*   保存的值大于等于 40，则使用 raw
    

```
127.0.0.1:6379> set num 1234567890123456789012345678901234567890
OK
127.0.0.1:6379> STRLEN num
(integer) 40
127.0.0.1:6379> OBJECT encoding num
"raw"

```

*   数据结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaPvJt9piaGlZKJqGFaBGtCficAkCKAeicib3LBMVN4IfJCicIBksXZxictqhQ/640?wx_fmt=png)

#### d> 编码转换规则  

*   如果将原本保存的整数值转换为字符串值，那么字符串对象的编码也将从 int 变为 raw。
    

```
127.0.0.1:6379> set num 1234
OK
127.0.0.1:6379> OBJECT encoding num
"int"
127.0.0.1:6379> APPEND num a
(integer) 5
127.0.0.1:6379> get num
"1234a"
127.0.0.1:6379> OBJECT encoding num
"raw"

```

2.3> list（列表）
-------------

### 2.3.1> 概述

*   适用场景：消息队列。
    
*   它的特点就是内部元素有序、重复，并且插入和删除很快 O(1)，但是查找却很慢 O(n)。功能支持队列和栈操作。
    

### 2.3.2> 具体操作

*   左侧插入
    

```
lpush msg_queue msg1 msg2 msg3

```

*   右侧插入
    

```
rpush msg_queue msg1 msg2 msg3

```

*   左侧弹出
    

```
lpop msg_queue

```

*   右侧弹出
    

```
rpop msg_queue

```

*   查看列表长度
    

```
llen msg_queue

```

*   查看列表中某个 index 的值   下标从 0 开始，要注意的是：由于这个操作是要遍历列表，所以 index 越大，效率越差。
    

```
lindex msg_queue 2

```

*   按范围查看队列信息，-1 倒数第一个，-2 倒数第二个
    

```
lrange msg_queue 0 -1 #查看全部msg_queue列表中的信息
lrange msg_queue 0 4 #查看msg_queue列表中从下标0到下标4的5个元素的信息

```

*   ltrim 保留某区间的列表
    

```
ltrim msg_queue 0 3 #保留msg_queue列表中从下标0到下标3的4个元素的信息
ltrim msg_queue 1 0 #清空msg_queue列表，随着最后一个元素被删除，msg_queue也随之被删除

```

### 2.3.3> 内部实现

*   列表对象的编码支持 **ziplist** 和 **linkedlist** 两种。
    

#### a> ziplist

*   ziplist 编码列表对象，采用**压缩列表**实现。每个列表节点保存一个列表中的元素。
    

```
127.0.0.1:6379> RPUSH testlist a b c
(integer) 3

```

*   数据结构如下： 
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaQuyicDSKLWyrsveuoMECko7a5xVnwBic7FJ0ScRW4cEtF6x2Sciawpibow/640?wx_fmt=png)

#### b> linkedlist  

*   linkedlist 编码列表对象，采用双向链表作为底层实现，每个列表节点保存一个列表中的元素。
    
*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefia1VaevcvJyvh2M84HibJn0HB87keIaaXtvrQVV4fT4yutJFic5tmR5pXA/640?wx_fmt=png)

#### c> 编码转换规则  

*   同时满足一下两个条件时，是 ziplist 类型，否则为 linkedlist 类型
    

*   条件 1：列表中所有元素**长度**都小于 66 字节。
    
*   条件 2：列表中元素的**个数**小于 512 个。 
    

```
127.0.0.1:6379> RPUSH testlist a b c
(integer) 3
127.0.0.1:6379> OBJECT encoding testlist
"ziplist"
127.0.0.1:6379> RPUSH testlist 12345678901234567890123456789012345678901234567890123456789012345
(integer) 5
127.0.0.1:6379> OBJECT encoding testlist
"linkedlist"

```

2.4> set（集合）
------------

### 2.4.1> 概述

*   适用场景：存储有去重需求的数据，比如：针对一篇文章用户进行点赞操作。
    
*   它的特点是内部元素无序且不重复。它的内部实现相当于一个特殊的字典，字典中所有的 value 指都为 NULL。
    

### 2.4.2> 具体操作

*   添加元素到集合中
    

```
sadd likeset bob muse john muse

```

*   查看集合中的所有元素 输出顺序是乱序的
    

```
smembers likeset

```

*   但是如果添加的是纯数字  输出顺序是有序的
    

```
127.0.0.1:6379> SADD numbers 2 1 4 6 8 3 4 5
(integer) 1
127.0.0.1:6379> SMEMBERS numbers
"1"
"2"
"3"
"4"
"5"
"6"
"8"

```

*   查询 muse 是否在集合中
    

```
sismember likeset muse

```

*   查询集合的长度
    

```
scard likeset

```

*   取出集合中的一个元素
    

```
spop likeset

```

*   删除
    

```
DEL bigset

```

2.4.3> 内部实现  

*   集合对象的编码可以是 **intset** 或 **hashtable**。
    

#### a> intset

*   intset 编码集合对象使用整数集合作为底层实现，集合对象包含的所有元素都被保存在整数集合里面。 
    

```
127.0.0.1:6379> SADD number 1 2 3 4
(integer) 4
127.0.0.1:6379> OBJECT encoding number
"intset"

```

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaaFqD80odI7tttkia5BZrlickWkOu0Slo4cibOVg3mfgXml15VmpoF3pDQ/640?wx_fmt=png)

#### b> hashtable  

*   底层字典作为底层实现，每个键都是一个字符串对象，每个字符串对象包含了一个集合元素，而字典的值则全部被设置为 NULL。
    
*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiadBvs2Dk0Twxo7QFFlgCzQ2eCguNzm4OZZrRiaHg5K7tWaAKxNtmc5mw/640?wx_fmt=png)

#### c>  编码转换规则  

*   当集合对象同时满足以下两个条件时，对象使用 intset 编码，否则使用 hashtable 编码
    

*   条件 1：集合对象保存的所有元素都是**整数**值。
    
*   条件 2：集合对象保存的元素数量**不超过 512** 个。 
    

```
127.0.0.1:6379> SADD alphabet a b c d
(integer) 4
127.0.0.1:6379> OBJECT encoding alphabet
"hashtable"
127.0.0.1:6379> EVAL "for i=1, 512 do redis.call('SADD', KEYS[1], i) end" 1 bigset
(nil)
127.0.0.1:6379> SCARD bigset
(integer) 512
127.0.0.1:6379> OBJECT encoding bigset
"intset"
127.0.0.1:6379> SADD bigset 513
(integer) 1
127.0.0.1:6379> SCARD bigset
(integer) 513
127.0.0.1:6379> OBJECT encoding bigset
"hashtable"

```

2.5> zset（有序集合）
---------------

### 2.5.1> 概述

*   适用场景：存储有去重且有序的数据，比如：学生的高考成绩。
    
*   它的内部采用 “跳跃列表” 实现。根据 score 进行排序。
    

### 2.5.2> 具体操作

*   添加元素到 sat_score 中 注意，111 muse 会覆盖 134 muse
    

```
zadd sat_score 134 muse 122 bob 156 john 111 muse

```

*   查看 muse 的 score 值
    

```
zscore sat_score muse

```

*   正序输出 输出从下标 0 到下标 - 1（即倒数第一个）中的所有元素从小到大
    

```
zrange sat_score 0 -1
zrangebyscore sat_score -inf +inf  # 与上面输出效果一样，输出score>=负无穷，score<=正无穷的所有元素

```

*   倒序输出
    

```
zrevrange sat_score 0 -1

```

*   查看 sat_score 中的元素个数
    

```
zcard sat_score 注意，是按照正序排序输出的，从0开始

```

*   获得 sat_score 中 score>=2 且 score<=5 的元素，正序排列
    

```
zrangebyscore sat_score 2 5
zrangebyscore sat_score 0 -1  会返回empty array，而并不是展示所有列表。因为score>=0 且score<=-1 是不存在的
zrangebyscore sat_score (2 5   半括号，表示小于。不加半括号，表示小于等于
zrangebyscore sat_score -inf 5   inf代表infinite:无穷的，-inf代表负无穷 +inf代表正无穷
zrangebyscore sat_score 2 inf withscores 参数WITHSCORES：表示输出时带出score值，正序排列

```

*   获得 sat_score 中 score<=5 且 score>=2 的元素，倒序排列
    

```
zrevrangebyscore sat_score 5 2

```

*   删除 sat_score 中的元素 muse
    

```
zrem sat_score muse

```

2.5.3> 内部实现：  

*   有序集合编码的内部实现可以是 **ziplist** 或 **skiplist**
    

#### a> ziplist

*   ziplist 使用压缩列表作为底层实现，第一个节点保存元素的成员（member），而第二个节点则保存元素的分值（score）。压缩列表内的集合元素按分值从小到大进行排序。
    

```
127.0.0.1:6379> zadd sat_score 134 muse 122 bob 156 john 
(integer) 3
127.0.0.1:6379> OBJECT encoding sat_score
"ziplist"

```

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaSJCiaJ6NUT6eup6nria5wpf8Xib9L6dpyYFODic61F72ZCo4CFaqRred7w/640?wx_fmt=png)

#### b> skiplist  

*   skiplist 编码的有序集合采用 zset 结构作为底层实现，一个 zset 同时包含一个字典和一个跳跃表。
    

```
redis.h
/*
 * redis对象
 */
typedef struct zset {
    // 跳跃表
    dict *dict;
    // 字典
    zskiplist *zsl;
} zset;

```

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiakqvBRVq7FPuFoSS0KfobjSQERPv7BPtZ96H7RcV98jOgqIUp2SiamZQ/640?wx_fmt=png)

2.6> hash（字典）  

----------------

### 2.6.1> 概述

*   适用场景：存储无序字典的数据，比如：适合存储对象类型。比如存储猪肉价格。
    
*   它的内部采用数组 + 链表的结构，类似 java 里的 HashMap。
    

*   hash 的 key 值只能是字符串。将对象存储为 hash 结构可以针对需要来获取部分数据，而不是将整个对象获取。减少网络资源浪费。
    
*   rehash 采用了渐进式的策略。
    

### 2.6.2> 具体操作

*   添加元素到 pig 中
    

```
hset pig leg "26rmb/kg"
hset pig ear "30rmb/kg"
hset pig tongue "40rmb/kg"
hset pig trotters "60rmb/kg"

```

*   查询 pig 中猪耳朵的价格
    

```
hget pig ear

```

*   批量添加
    

```
hmset boys_weight muse 126g bob 99 john 179 tom 155

```

*   批量获取元素
    

```
hmget boys_weight muse john

```

*   获得 pig 中元素个数
    

```
hlen pig

```

*   查询 pig 中所有元素
    

```
hgetall pig

```

### 2.6.3> 内部实现：

*   哈希对象编码支持 **ziplist** 和 **hashtable** 两种。
    

#### a> ziplist

*   ziplist 编码底层使用压缩列表实现，当有新的键值对要加入到哈希对象时，会先将 key 值从队尾推入压缩列表中，再将这个 key 对应的 value 值**从队尾推入**压缩列表中；所以，同一键值对的两个节点总是紧挨在一起的，**key 在前**，**value 在后**。
    

```
127.0.0.1:6379> hset muse name muse
(integer) 1
127.0.0.1:6379> hset muse age 20
(integer) 1
127.0.0.1:6379> hset muse sex male
(integer) 1
127.0.0.1:6379> hgetall muse
1) "name"
2) "muse"
3) "age"
4) "20"
5) "sex"
6) "male"
127.0.0.1:6379> OBJECT encoding muse
"ziplist"

```

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaTseNgl52TNG8axBHRO8VroDo79yPtS6Aw8TIXnKrMHwtfmvCpic4tQg/640?wx_fmt=png)

#### b> hashtable  

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiao0nKvPTc1rYBItXfzvb8KTr9lmNAPdusrXvc5SnlUFriabluKibZIHbA/640?wx_fmt=png)

#### c>  编码转换规则  

*   同时满足两个条件时是 ziplist 编码类型，否则为 hashtable 编码类型
    

条件 1：哈希对象中所有键值对中，key 和 value 的**长度**均小于 46 字节。

条件 2：哈希对象中键值对的**个数**小于 512 个。

```
127.0.0.1:6379> hset muse name 1234567890123456789012345678901234567890123456789012345678901234
(integer) 0
127.0.0.1:6379> OBJECT encoding muse
"ziplist"
127.0.0.1:6379> hset muse name 12345678901234567890123456789012345678901234567890123456789012345
(integer) 0
127.0.0.1:6379> OBJECT encoding muse
"hashtable"

```

2.7> Redis 相关命令集合
-----------------

*   https://redis.io/commands#
    

三、Redis 数据结构篇  

================

3.1> SDS 简单动态字符串
----------------

### 3.1.1> 概述

*   SDS 与 C 字符串的不同之处
    
    SDS（simple dynamic string），**简单动态字符串**。是由 Redis 自己创建的一种表示字符串的抽象类型。
    
    C 字符串是不可被修改的。但是 SDS 是动态可以被修改的。
    
*   SDS 的定义
    

```
sds.h/sdshdr
struct sdshdr {
    // 记录buf数组中已使用的字节长度
    unsigned int len;
    // 记录buf数组中未使用的字节长度
    unsigned int free;
    // java中的char占2个字节（Unicode表示）；C语言中占1个字节（ASCII表示），由于汉字是2个字节，所以无法保存
    char buf[];
};

```

*   数据结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaBqSs9hUBXkzCaWQT5B9qCGiaIU2gXT3qvzxGYXp5icA2LPlbM5G0LeGw/640?wx_fmt=png)

最后一位遵循 C 字符串的空字符（'\0'）结尾的规则，目的是，可以直接使用 C 字符串的函数。其中 len 计数不包含‘\0’。  

### 3.1.2> 为什么 Redis 使用 SDS 而不是 C 字符串

*   第一点：C 字符串**没有记录字符长度**，每次都需要遍历，所以复杂度为 O(n)。SDS 的 len 记录了当前字符串的长度，所以获取字符串长度的复杂度为 O(1)
    
*   第二点：C 字符串**无法杜绝缓冲区溢出**。比如执行 strcat 函数时，如果没有指定足够的内存，那么拼接后会造成缓冲区溢出。SDS 在进行修改时，会先查看空间是否足够，如果不够了，那么它的 API 会自动的进行空间扩展。
    

如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaNpMj4mqVpRjZ5dC9xOkLEIibenJdeqq1s6hH4Dg4Wcia88cYN74dJ4vw/640?wx_fmt=png)

*   第三点：C 字符串存在内存重分配的性能损耗；SDS 采用**空间预分配**和**惰性空间释放**来减少性能损耗。
    
*   _什么是空间预分配？_
    

*   1> 如果对 SDS 进行修改后，SDS 的长度（len 的长度）**小于 1MB** 的时候，那么程序分配和 len 属性同样大小的未使用空间（free）。
    
*   2> 如果**大于 1MB**，那么程序会分配 1MB 的未使用空间（free）。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaeqKBqBH0tQ0sdSVdicBk6Y8BHXRCDRhWan4lGMHoLYSEsQnEj44x6MQ/640?wx_fmt=png)

*   _什么是惰性空间释放？_
    

*   当有缩短 SDS 字符串操作时，程序并不立即把空闲出来的字节释放掉，而是使用 free 属性将这个空闲的字节记录起来，等待将来使用。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiafDiaf57wmFQeicWW2JyRwxceAtibcxzYUxicyIArLwKJOzDIx9CNBicEticQ/640?wx_fmt=png)

*   第四点：C 字符串只能保存文本数据，并且字符串里面**不能包含空字符**，否则就会被误认为是字符串结尾。
    

*   SDS 则采用二进制来保存数据，并且它使用 len 属性来判断字符串末尾而不是空字符。所以，它不仅可以保存文本数据，也可以保存任意格式的二进制数据，如：图片、音频、视频、压缩文件这样的二进制数据。
    

3.2> list 双向链表
--------------

*   链表的特点是高效的**删除**和**新增**节点来灵活的调整链表中的元素顺序。
    
*   由于 C 语言没有内置链表，所以 Redis 自己构建了链表的实现。
    

*   Redis 基本数据结构中的 REDIS_LIST，底层的实现之一就采用的链表。即：当包含了**很多元素**，或者元素中有比**较长的字符串**时，就会采用链表作为 REDIS_LIST 的底层实现。
    

```
adlist.h
/*
 * 双向链表节点
 */
typedef struct listNode {
    // 前节点
    struct listNode *prev;
    // 后节点
    struct listNode *next;
    // 本节点的值
    void *value;
} listNode;

```

```
adlist.h
/*
 * 双向链表
 */
typedef struct list {
    // 表头节点
    listNode *head;
    // 表尾节点
    listNode *tail;
    // 节点复制函数
    void *(*dup)(void *ptr);
    // 节点释放函数
    void (*free)(void *ptr);
    // 节点对比函数
    int (*match)(void *ptr, void *key);
    // 双向链表长度
    unsigned long len;
} list;

```

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaMvlkNoS3l8v1MPal7ocEWc210H3R4QtVmM3NEX7Ibj3R81IDSxkM7Q/640?wx_fmt=png)

*   特征如下：
    

*   **双端**：具有 prev 和 next 指针，获取某个节点的前置 / 后置节点的复杂度为 O(1)。
    
*   **无环**：头节点的 prev=NULL，尾节点的 next=NULL，对链表的访问以 NULL 为终点。
    

*   **带表头 / 表尾指针**：list 结构中包含 head 指针和 tail 指针，所以获得链表头节点 / 尾节点的复杂度为 O(1)。
    
*   **多态性**：可以通过设置 dup、free、match 这三个不同类型特定函数，保存各种不同类型的节点值。
    

3.3> dict 字典
------------

*   字典又被称为符号表、关联数组、映射（map）。是一种用于**保存键值对**的抽象数据结构。
    
*   C 语音并没有内置这种数据结构，因为 Redis 构建了自己的字典实现。
    

*   字典是哈希键的底层实现之一，当一个哈希键包含的键值对比较多，又或者键值对中的元素都是比较长的字符串时，Redis 就会使用字典作为哈希键的底层实现。
    

### 3.3.1> 哈希表

```
dict.h
/*
 * 哈希表结构
 */
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值，总是等于size-1，为什么？因为sizemask最大值为size-1，最小值为0，正好是size的大小。
    unsigned long sizemask;
    // 哈希表已存在的节点数量
    unsigned long used;
} dictht;

```

```
dict.h
/*
 * 哈希表节点结构
 */
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    // 下一个哈希节点，单向链表
    struct dictEntry *next;
} dictEntry;

```

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaMmOJCcC8fvG94pIYVt7WYxuU6lY8yr9ibggau9iaQyRZOAy1icacCUobg/640?wx_fmt=png)

### 3.3.2> 字典

```
/*
 * 字典结构
 */
typedef struct dict {
    // 字典类型相关函数
    dictType *type;
    // 传递给dictType函数的可选参数
    void *privdata;
    // 哈希表 一般情况下，数据会存储再ht[0]中，ht[1]只会在对ht[0]进行rehash时使用
    dictht ht[2];
    // rehash索引，它记录了rehash目前的进度（当rehash不在进行时，值为-1）
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    // 当前运行的迭代数值
    int iterators; /* number of iterators currently running */
} dict;

```

```
/*
 * 一系列用于操作特定类型键值对的函数，Redis会为用途不同的字典设置不同的类型特定函数
 */
typedef struct dictType {
    // 计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);
    // 复制键的函数
    void *(*keyDup)(void *privdata, const void *key);
    // 复制值的函数
    void *(*valDup)(void *privdata, const void *obj);
    // 对比键的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    // 销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);
    // 销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);
} dictType;

```

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiadp7icsppuAf26scxn9uOvuzflxfLGjfB1xHicv5hkicUOFCebmzDXVKUg/640?wx_fmt=png)

### 3.3.3> 哈希算法  

*   步骤一：使用字典设置的哈希函数，计算键 key 的哈希值。（注：Redis 使用 **MurmurHash2 算法**来计算键的哈希值）
    
    hash = dict -> type -> hashFunction(key);
    
*   步骤二：使用哈希表的 sizemask 属性和哈希值，计算出索引值  
    
    index = hash & dict -> ht[x].sizemask;
    
*   键冲突
    
    Hash 表使用链地址法来解决键冲突。每个节点都有 next 指针，可以构成一条单向链表。因为该链表没有指向表尾节点的指针，所以，处于速度考虑，新添加的节点会添加到链表的表头位置，添加复杂度为 O(1)。
    
*   rehash——重新散列
    
    随着哈希表中节点数量的增多或者减少，为了让**负载因子**（load factor）维持在一个合理的范围，要对哈希表的大小进行相应的扩展或收缩。
    
*   负载因子
    
    load_factory=ht[0].used / ht[0].size
    
*   扩展条件
    
    以下任意一个被满足时，开始执行扩展操作：
    

*   case-1：服务器当前**没有执行** BGSAVE 或 BGREWRITEAOF 命令，并且哈希表的**负载因子 >=1**。
    
*   case-2：服务器当前**正在执行** BGSAVE 或 BGREWRITEAOF 命令，并且哈希表的**负载因子 >=5**。
    

*   _为什么会因为 BGSAVE 或 BGREWRITEAOF 命令是否执行，而导致不同的负载因子阀值？_
    
    在执行 BGSAVE 或 BGREWRITEAOF 命令的过程中，Redis 需要创建子进程，而大多数操作系统都采用 **copy-on-write** 技术来优化子进程的使用效率，所以在子进程存在期间，服务器会提高执行扩展操作所需的负载因子，从而尽可能的避免在子进程存在的期间进行哈希表扩展操作，这样可以避免不必要的内存写入操作，从而最大限度的**节约内存**。
    
*   收缩条件
    
    当哈希表的**负载因子小于 0.1** 时，程序自动开始对哈希表执行收缩操作。
    

### 3.3.4> rehash 过程

*   步骤一：为字典的 ht[1] 哈希表分配空间，规则如下：
    
    扩展操作：ht[1] 的空间大小为第一个大于等于 **ht[0].used*****_2_** 的 2 的 n 次幂。比如：ht[0].used=3，则 ht[1].size 的大小扩展为 8。
    
    收缩操作：ht[1] 的空间大小为第一个大于等于 **ht[0].used** 的 2 的 n 次幂。比如：ht[0].used=3，则 ht[1].size 的大小收缩为 4。
    
*   步骤二：将 ht[0] 中的所有键值对 rehash（重新计算键的哈希值和索引值，然后放置到 ht[1] 哈希表相应的位置上）到 ht[1] 中。
    
*   步骤三：释放 ht[0]，将 ht[1] 设置为 ht[0]，并为 ht[1] 创建一个新的空白哈希表，为下一次 rehash 做准备。
    

### 3.3.5> 渐进式 rehash  

*   为 ht[1] 分配空间（按照上面介绍的扩展操作分配空间来分配，即：第一个大于等于 **ht[0].used*****_2_** 的 2 的 n 次幂）
    
*   在字典中**维护 rehashidx（索引计数器变量）=0**，表示 rehash 准备开始工作。
    

*   当进行增、删、改、查操作时，除了执行指定操作之外，还要顺带将 ht[0] 的值 rehash 到 ht[1] 上。其中，**新增操作，一律会被保存到 ht[1] 里面，而不会保存到 ht[0] 中**。而删、改、查操作都会在两个哈希表上进行。比如查找操作，会先查找 ht[0] 里面的元素，如果没找到，则继续查找 ht[1] 中的元素。
    
*   当一个元素 rehash 完成后，**rehashidx 自增 1**。
    

*   当最终全都 rehash 完毕 i 后，**rehashidx 被赋值为 - 1**，表示 rehash 操作完毕。
    
*   h[1] 与 h[0] 互换，互换完毕后，清除 ht[1]
    

3.4> skiplist 跳跃表
-----------------

*   跳跃表是一种**有序**数据结构。
    
*   Redis 使用跳跃表作为有序集合键的底层实现之一，如果一个有序集合包含的元素数量比较多，或者有序集合中元素的成员是比较长的字符串时，Redis 就会使用跳表来作为有序集合的底层实现。
    

*   Redis 只在两个地方用到了跳跃表：
    

*   1> 实现有序集合键。
    
*   2> 在集群节点中用作内部数据结构。
    

*   数据结构定义如下：
    

```
/*
 * 跳表
 */
typedef struct zskiplist {
    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;
    // 表中节点的数量（表头节点的层数不计算在内）
    unsigned long length;
    // 跳表中最大的层数（表头节点的层数不计算在内）
    int level;
} zskiplist;

```

```
/*
 * 跳表节点
 */
typedef struct zskiplistNode {
    // robj，即：redisObject
    robj *obj;
    // 分值
    double score;
    // 跳表后置节点
    struct zskiplistNode *backward;
    // 跳表层
    struct zskiplistLevel {
        // 跳表前置节点
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];
} zskiplistNode;

```

*   数据结构如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaGsHsYOQrtdEHODQoBhgez9w6MuOS6JB6DgYRyoN5le6E54OVg4sAYA/640?wx_fmt=png)

*   每次创建一个新的跳跃表节点的时候，程序都会根据**幂次定律**（power law， 越大的数出现的概率越小）随机生成一个介于 **1 和 32 之间**的值作为 level 数组的大小，这个大小就是层的高度。
    

3.5> intset 整数集合
----------------

略  

四、Redis 应用篇  

==============

4.1> 分布式锁
---------

*   使用场景  
    两个人同时操作一个银行账户，一个取钱，一个存钱。
    
*   实现要点
    

*   1> 加锁和解锁的 key 必须是唯一的。
    
*   2> 不能永久加锁，一定要有过期时间。
    
*   3> 一定要保证加锁与设置过期时间的原子性。
    
*   4> 要支持过期续租。
    

*   原理
    
    采用 **set key value [EX seconds|PX milliseconds] [NX|XX] [KEEPTTL]**
    
    redisVersion>=2.6.12 时，添加了 EX，PX，XX 选项
    
    redisVersion>=6.0 时，添加了 KEEPTTL 选项
    
    Note: Since the SET command options can replace SETNX, SETEX, PSETEX, it is possible that in future versions of Redis these three commands will be deprecated and finally removed.
    
*   实践举例：
    

```
set distribute_lock muse1 EX 20 NX

```

*   加锁操作
    

```
public static boolean tryLock(String key, String uniqueId, int seconds) {
    return "OK".equals(jedis.set(key, uniqueId, "NX", "EX", seconds));
}

```

*   解锁操作：
    

```
public static boolean releaseLock(String key, String uniqueId) {
    String luaScript = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "return redis.call('del', KEYS[1]) else return 0 end";
    return jedis.eval(
        luaScript, 
        Collections.singletonList(key), 
        Collections.singletonList(uniqueId)
    ).equals(1L);
}

```

*   分布式锁面临的问题
    
*   **锁过期**问题描述：A 线程抢到锁，锁的过期时间为 10 秒钟，但是 A 由于某种原因，执行了 15 秒，那么锁会过期消失，而 A 并没有执行完毕。
    

*   解决方案：加大加锁时间，过期时间续租。
    
*   实现方式：在 A 线程抢到锁后，立刻开启后台自旋线程，根据设置阈值去检测是否锁已经释放（比如 10*2/3 秒后），如果，依然持有锁，则进行加锁续租 10 秒，继续自旋。如果已经不持有锁了，则线程终止。
    

*   **重叠解锁**问题描述：在锁过期场景下，10 秒后 B 线程成功抢到锁，B 线程需要执行 8 秒。但是，当 A 线程执行完（5 秒后），调用解锁操作，将 B 线程的锁解开了。但是 B 线程并未执行完毕。以此类推。出现了线程彼此错位解锁的场景。
    

*   解决方案：每个线程有自己的唯一标识。抢锁时，在 value 里插入该唯一标识，解锁时，验证如果 value 中的值与线程的唯一标识相同才可以成功解锁，从而防止被其他线程解锁。使用 Lua 保证执行的原子性。
    

*   **单点问题**问题描述：A 线程抢到锁后，由于主节点还未及时将内容同步到从节点，然后就意外挂掉了；从节点会成为主节点，但是从节点中并没有 A 线程的加锁信息，造成该锁依然可以被其他线程抢占。  
    解决方案：Redlock 算法。
    

4.2> 消息队列
---------

### 4.2.1> 实时消息队列

*   使用场景：
    
    异步发送消息，实时消费
    
*   实现要点： 
    
    使用 list 来实现。
    
    rpush——>blpop  或  lpush——>brpop
    
    长时间阻塞读，会被识别为空闲连接，redis 服务会主动断开连接，此时需要做异常捕获处理，再建立连接。
    

### 4.2.2> 延时消息队列

*   使用场景：
    
    异步发送消息，实时消费
    
*   实现要点： 
    

*   采用 zset 来实现，插入队列是，通过将发送时间赋值 score，来进行发送时间的排序。
    
*   采用 jedis.zrangeByScore(delayQueue, 0, System.currentTimeMills(), 0, 1) 来获取一个时间段的 1 条消息。
    
*   采用 jedis.zrem(delayQueue, 消息)，如果执行成功，说明消费成功。解析消息进行业务处理。
    
*   没抢到消息，继续轮训。
    
*   建议采用 Lua 脚本，将 zrangeByScore 获取一条消息与 zrem 删除这条消息这两个操作原子化，以免多个线程同时获取到 1 条消息，然后再操作 zrem 的时候，其他消息白白执行一次。
    

4.3> 位图
-------

*   使用场景：  
    适合 0，1 这种 boolean 类型状态的存储业务场景，可以避免数据量大的时候，占用太多的 redis 存储空间。  
    比如记录咱们上班是否迟到的记录。
    
*   返回字符的 ASCII 码的二进制：  
    **bin(x)** 返回一个整数 int 或者长整数 long int 的二进制表示。  
    **ord(x)** 返回一个字符（长度为 1 的字符串）作为对应的 ASCII 数值。
    

*   举例： 
    

```
>>> bin(ord('m'))   // 返回m的ASCII的二进制
'0b1101101'

```

*   ASCII 码
    
    ASCII (American Standard Code for Information Interchange): 美国信息交换标准代码）是基于 [拉丁字母] 的一套电脑 [编码] 系统，主要用于显示现代 [英语] 和其他 [西欧] 语言。它是最通用的信息交换标准，并等同于 [国际] 标准 ISO/IEC 646。ASCII 第一次以规范标准的类型发表是在 1967 年，最后一次更新则是在 1986 年，到目前为止共定义了 128 个字符 [1] 。
    
    常见 ASCII 码的大小规则：0~9<A~Z<a~z。
    
*   使用语法：
    
    **SETBIT key offset value**
    
    **GETBIT key offset**
    
    **BITCOUNT key [start end]**
    
    **BITPOS key bit [start] [end]****BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]**  
    
*   举例：
    

*   设置第一位是 1
    

```
setbit bitmap 0 1

```

*   获取第一位
    

```
getbit bitmap 0

```

*   获取计算给定字符串中，被设置为 1 的比特位的数量
    

```
set bitmap muse
bitcount bitmap         查询“muse"中，被设置为1的数量
bitcount bitmap 0 0     查询第一个字符“m“中，被设置为1的数量
bitcount bitmap 1 1     查询第二个字符“u“中，被设置为1的数量
bitcount bitmap 2 -1    查询从第三个字符到末尾字符（即：”se“）中，被设置为1的数量

```

*   获得某个区间内，0 或 1 第一次出现的 index
    

```
bitpos bitmap 1 0    查询第一个字符“m“中，第一个被设置为1的index
bitpos bitmap 0 1    查询第二个字符“m“中，第一个被设置为0的index
bitpos bitmap 1 2 -1 查询从第三个字符到末尾字符（即：”se“）中，第一个被设置为1的index

```

*   自 3.2.0 起可用，时间复杂度：O（1）用于指定的每个子命令
    

4.4> HyperLogLog
----------------

*   在文章 Redis（下）有详细介绍
    

4.5> 布隆过滤器
----------

*   在文章 Redis（下）有详细介绍
    

五、缓存常见问题
========

*   在文章 Redis（下）有详细介绍
    

六、Redis 持久化  

==============

*   由于 Redis 是内存数据库，所有的数据都保存在内存中，如果不采用持久化的方式，那么当 Redis 服务器退出的时候，所有数据都会丢失。我们介绍两种 redis 持久化的方式：**RDB** 和 **AOF**。
    

6.1> RDB
--------

*   RDB 持久化支持**手工执行**和服务器**定期执行**，RDB 持久化产生的文件是一个经过**压缩的二进制文件**，对应文件为 dump.rdb，因为它保存在磁盘上，所以可以用它来还原数据库中的数据。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefia4Dp6wrpoANZvZfxIghCibfxJzFOTsokiaP0SbMCQf6dk6IicdibZIbSZmQ/640?wx_fmt=png)

*   保存手工执行 RDB 保存有两个命令：**SAVE** 命令和 **BGSAVE** 命令。
    
*   其中 SAVE 命令**会阻塞** Redis 服务器进程，直到 RDB 文件生成完毕，都会一直处于阻塞状态，不能处理任何的 Redis 命令请求。 
    

```
127.0.0.1:6379> SAVE
OK

```

*   BGSAVE 命令会 **fork 一个子进程**来生成 RDB 文件，Redis 服务器进程不受影响，可以继续处理命令请求。 
    

```
127.0.0.1:6379> BGSAVE
Background saving started

```

*   数据加载
    
*   当服务器启动的时候，**RDB 自动执行加载**，没有专门的命令来加载 RDB 文件。
    
*   只要 Redis 启动时检测到 RDB 文件的存在，那么就会自动载入 RDB 文件。
    
*   加载过程中，会一直处于**阻塞状态**，直到加载完毕为止。
    
*   由于 AOF 文件的更新频率一般比 RDB 文件的更新频率高，所以，如果服务期开启了 AOF 持久化功能，那么就**优先加载 AOF 文件**，否则，加载 RDB 文件。
    

```
>./redis-server
8143:M 12 Sep 10:36:28.061 * DB loaded from disk: 0.001 seconds

```

*   自动间隔性保存——saveparam
    

由于 BGSAVE 命令执行时是不阻塞主服务进程的，所以 Redis 允许用户通过 save 选项来使服务器**每隔一段时间自动执行一次 BGSAVE 命令**。

*   设置自动保存，**redis.conf 文件中**，服务器 save 的默认选项如下所示:
    

```
save 900 1 // 900秒内，执行1次修改操作。
save 300 10 // 300秒内，执行10次修改操作。
save 60 10000 // 60秒内，执行10000次修改操作。

```

*   满足任意一个，即会执行 BGSAVE 命令。 
    

```
redis.h
/*-----------------------------------------------------------------------------
 * Global server state
 *----------------------------------------------------------------------------*/
struct redisServer {
  ...
  // 保存saveparam的数组
  struct saveparam *saveparams;   /* Save points array for RDB */
  // 修改计数器，记录上一次成功执行SAVE或BGSAVE后，数据进行了多少次修改（包括写入、删除、更新等操作）。
  // set name "muse"   dirty计数器+1
  // sadd name "bob" "tom" "john" "sam" dirty计数器+4
  long long dirty;                /* Changes to DB from the last save */
  // 上一次执行保存的时间，记录上一次成功执行SAVE或BGSAVE的时间
  time_t lastsave;                /* Unix time of last successful save */
  ...
}

```

```
redis.h
struct saveparam {
    // 执行的秒数
    time_t seconds;
    // 修改的次数
    int changes;
};

```

*   数据结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaJD34OOmhcgf2DDibdEWdCPhTu9neqkKUTHKUhCKT3Oht6zOXKfOufDw/640?wx_fmt=png)

*   BGSAVE 执行条件检测器——serverCron
    
    serverCron 默认**每 100 毫秒**执行一次条件验证，如果符合保存条件，则执行 BGSAVE 命令。
    
    它会通过 dirty 和 lastsave（间隔时间 = 当前时间 - lastsave 时间）这两个参数，来判断是否执行 BGSAVE 命令。
    

6.2> AOF
--------

*   AOF（Append Only File）持久化，它是通过**保存 Redis 执行命令**来记录数据库数据变更。持久化流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaUszpsdHTtBJjibTLjAticqb4NVtuqTByfBrr9wzeiafK299JcV1sILhicg/640?wx_fmt=png)

### 6.2.1> 持久化  

*   如何开启 AOF 和配置 AOF 路径——redis.conf 文件：
    

```
appendonly yes // 默认为no 打开为yes
appendfilename "appendonly.aof" // 默认配置，路径为 ..../redis-3.0.0/src/appendonly.aof

```

*   查看 AOF 文件存储信息
    

```
127.0.0.1:6379> set name muse
OK
127.0.0.1:6379> rpush chats a b c
(integer) 3
127.0.0.1:6379> BGREWRITEAOF // 异步执行一个AOF文件重写操作
Background append only file rewriting started

```

*   AOF 文件内容（Redis 命令的请求协议格式）：cat  [安装路径]/redis-3.0.0/src/appendonly.aof
    

```
*2
$6
SELECT
$1
0
*3
$3
SET
$4
name
$4
muse
*5
$5
RPUSH
$5
chats
$1
a
$1
b
$1
c

```

*   AOF 的持久化分为三步：**命令追加**（append）——> **文件写入**——> **文件同步**（sync）三个步骤。
    
*   追加
    

如果打开 AOF 后，每次执行完一个写命令之后，都会把写命令以请求协议格式保存到 **aof_buf 缓冲区**的末尾。 

```
redis.h
/*-----------------------------------------------------------------------------
 * Global server state
 *----------------------------------------------------------------------------*/
struct redisServer {
  ...
  // AOF缓冲区
  sds aof_buf;      /* AOF buffer, written before entering the event loop */ 
  ...
}

```

*   写入与同步
    

*   Redis 的服务器进程就是一个**事件循环**，在这个循环中：
    
*   **文件事件**负责接收客户端的命令请求和发送给客户端执行结果回复；
    
*   **时间事件**负责执行如 serverCron 这种需要定时运行的函数。
    
*   当每次一个事件循环结束之前，都会调用 **flushAppendOnlyFile** 函数，来判断是否需要将 aof_buf 缓冲区中的内容写入和同步到 AOF 文件中。
    

*   流程如下图所示：  
    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaYZumMQDD4M5P8emibgbPot2xp9niaHu7jKeBiaic8DlURNQvqy0muuvpbw/640?wx_fmt=png)
    
*   其中，flushAppendOnlyFile 函数的行为由 appendfsync 配置决定：redis.conf 文件
    

```
# If unsure, use "everysec".
# appendfsync always
appendfsync everysec
# appendfsync no

```

*   参数含义解释如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaVPsuj0qXYYDeQicB78LsONYKaDtQwg1GmuNO33aO7roxiauSue4At7gw/640?wx_fmt=png)

*   AOF 持久化的效率和安全性
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiam0KAEYAMfe0T17zNC2nPeca59Zrq2DVItvkf3bzlCefUO1Upg2ClOw/640?wx_fmt=png)

*   _那什么叫写入？什么叫同步？有什么区别呢？_
    

*   为了提高文件的写入效率，在现代操作系统中，当用户调用 write 函数时，将一些数据写入到文件的时候，操作系统通常会将写入数据暂时保存在一个内存缓冲区里面，等到缓冲区的空间被填满、或者超过了指定的时限之后，才真正地将缓冲区中的数据写如到磁盘里面。
    
*   这种做法虽然提高了效率，但也为写入数据带来了安全问题，因为如果计算机发生停机，那么保存在内存缓冲区里面的写入数据将会丢失。
    
*   为此，操作系统提供了 fsync 和 fdatasync 两个同步函数，它们可以强制让操作系统立即将缓冲区中的数据写入到磁盘里，从而确保写入数据的安全性。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefia1Lfgg2FLiccT3frKbwywR5zRoibN6OgQhu97Nb5yODiaw9dljYFMSgddA/640?wx_fmt=png)

### 6.2.2> 数据库 AOF 加载

*   当 Redis 服务启动并读取 AOF，即可恢复关闭前的数据状态。加载流程如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiaaKc3gjdrvAh60CkLEebMYTkNLWJNDkmZaU0xicGh5q0drlEzjEKiav5g/640?wx_fmt=png)

*   其中，Fack Client（伪客户端）的执行命令效果与带网络的效果完全一样。由于载入 AOF 时，命令来源于 AOF 文件，而不是网络连接传递过来的命令，所以，建立了一个没有网络连接的伪客户端。
    

### 6.2.3> AOF 重写

*   _大家考虑一下 AOF 有什么缺陷没有？_
    

*   由于时间推移，AOF 会越来越大，不仅仅会对 Redis 服务器造成影响，也会对还原时间造成大量的浪费。比如我们考虑消息队列的场景，执行了如下语句：
    

```
127.0.0.1:6379> LPUSH number 1 2 3
(integer) 3
127.0.0.1:6379> RPOP number
"1"
127.0.0.1:6379> RPOP number
"2"
127.0.0.1:6379> LPUSH number 4 5
(integer) 3
127.0.0.1:6379> RPOP number
"3"
127.0.0.1:6379> LRANGE number 0 -1
1) "5"
2) "4"

```

*   这个场景，AOF 就生成了 6 条命令语句进行保存。针对该情况，Redis 提供了 **AOF 文件重写（rewrite）功能**。
    
*   AOF 文件重写的原理
    

*   通过读取当前 Redis 数据库的状态（而不是旧的 AOF 文件），来生成新的命令语句，保存在新的 AOF 文件中，来替换老的 AOF 文件。
    
*   如上面的例子来说，在老的 AOF 文件中，会生成 6 条语句，但是最终的数据结果则为 5 和 4，那么可以生成一条新的命令语句：LPUSH number 5 4，并保存到新的 AOF 文件中。
    

*   AOF 重写触发——redis.conf
    

```
auto-aof-rewrite-percentage 100 // 比上次重写后的体量增加了100%时自动触发重写
auto-aof-rewrite-min-size 64mb // 在aof文件体量超过64MB时触发重写

```

*    _AOF 重写有哪些问题？_
    

*   由于 AOF 重写会进行大量写操作，所以调用函数线程**长时间阻塞**，主线程无法执行其他操作。
    
*   针对这个问题，Redis 采取了将 AOF 重写程序放到**子进程**里执行，这样，服务进程不会被阻塞，并且由于开启的是进程而不是线程，所以不用使用锁机制来保证数据的安全性。
    

*   _开启子进程执行 AOF 又有什么问题？_
    

*   由于重写期间，服务器进程还要继续执行 redis 命令请求，所以会造成 AOF 重写文件保存的数据状态与真是数据库数据状态不一致。
    

*   _那与真是数据库数据状态不一致，该怎么办呢？_
    

*   为了解决这个问题，Redis 服务器设置了一个 **AOF 重写缓冲区**，这个缓冲区在服务器创建子进程之后开始使用。
    
*   当一个 redis 命令被服务进程接收且需要被执行时，除了执行这个命令之外，同时还会将这个命令发送给 AOF 缓冲区和 AOF 重写缓冲区。
    
*   当子进程完成 AOF 重写后，它会向父进程发送一个信号，要求父进程调用一个信号处理函数：
    

*   首先：阻塞主服务进程，将 AOF 重写缓冲区的内容写入到新的 AOF 文件中。
    
*   其次：新的 AOF 覆盖旧的 AOF。
    
*   最后：恢复主服务进程，可以继续接收并执行 redis 命令。
    

*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9UxLxdsh9FficamCIehbefiagbEZjMRCJ1QxBpoCXibED3KOQG0iclDN0K0QoYiaPBq9F7Sa93UZVhicBg/640?wx_fmt=png)