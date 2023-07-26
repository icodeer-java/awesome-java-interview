> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490185&idx=1&sn=a768adafebfc554293cb1c445f19021f&chksm=e9115874de66d1626fd8550b709c4e24d6195717e545aca5b8b5b3542aec86be40af9f466c8b&scene=178&cur_album_id=2451364959101059073#rd)

一、 概述
-----

主从复制，是指将一台 Redis 服务器的数据复制到其他的 Redis 服务器。前者称为主节点（`Master`/`Leader`），后者称为从节点（`Slave`/`Follower`）；数据是从主节点复制到从节点的。

其中，**主节点负责写数据**（当然有读的权限），**从节点负责读数据**（它没有写数据的权限）。

默认的配置下，**每个 Redis 都是主节点**。

一个主节点可以有多个从节点，但是**一个从节点只能有一个主节点**，即：主从节点是 **1 对 N** 的关系。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf93QJibL0y02ib8icwcr8icZSmYDx0zyLpSTibaK28rL5ATdntefvHUn1WflQ/640?wx_fmt=png)

### 1.1> 主从复制的用处

**(1) 数据冗余**：主从复制实现了数据的备份，实际上提供了数据冗余的实现方式。

**(2) 故障恢复**：当主节点出现异常时，可以由从节点提供服务，实现快速的故障恢复，实际上提供了服务冗余的实现方式。

**(3) 负载均衡**：在主从复制的基础上，配合读写分离，可以`由主节点提供写服务，由从节点提供读服务，分担服务器的负载`；在写少读多的业务场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器是并发量。

**(4) 高可用**：哨兵配合主从复制，可以是实现 Redis 集群的高可用。

二、环境搭建
------

创建`redis-cluster`目录，然后复制 3 份 redis（也可以一个 redis 三份不同的配置文件，启动的时候，读取相应的配置文件），如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9bTL08TOmOvVqMuaJ2SbThQz2r4T2juiciakyianKoWlNu1Aj7l61HVFoA/640?wx_fmt=png)

分别修改它们的 redis.conf 配置文件，如下所示：

**redis-6380/redis.conf**

```
port 6380
pidfile /var/run/redis-6380.pid
logfile "redis-6380.log"
dbfilename dump-6380.rdb
daemonize yes

```

**redis-6381/redis.conf**

```
port 6381
pidfile /var/run/redis-6381.pid
logfile "redis-6381.log"
dbfilename dump-6381.rdb
daemonize yes
# 如果不通过修改配置文件，也可以在客户端中输入“SLAVEOF 127.0.0.1 6380”即刻生效！！
# 也可以在客户端中输入“SLAVEOF NO ONE”来断开主从关系
replicaof 127.0.0.1 6380 

```

**redis-6382/redis.conf**

```
port 6382
pidfile /var/run/redis-6382.pid
logfile "redis-6382.log"
dbfilename dump-6382.rdb
daemonize yes
replicaof 127.0.0.1 6380

```

分别启动这 3 个 redis 服务：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9OcTCYtbaxnMUM77DS31nR5SFKTyxLGxsqgkMLGFRxtvc37wbPiaksWA/640?wx_fmt=png)

开启 3 个客户端，来连接这 3 个 redis 服务实例；**利用`ping`查看服务是否正常，并且通过`info replication`查看自己在集群中的角色**

**redis-6380**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9hLFanpibcHqc5pFF8kq4wdicWAho2rRK7GMbtHiazMy5h2FNfHujM12UQ/640?wx_fmt=png)

**redis-6381**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9wf2jH7owUgJjsUjcSowPUWYFNXRic0tO43ABubyfCgn3IXR9makropA/640?wx_fmt=png)

**redis-6382**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9UibqYV1icC31f3k3GHpg5vHaias8OY1oeOtTkYZ6VQQw8DyTPx7TYrtTg/640?wx_fmt=png)

三、相关特性
------

### 3.1> 从节点是只读的

我们测试一下**主节点 redis-6380** 的读写操作，结果是**读写都 ok**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9Ra6W7GkbhlDVP2rLywHDfxNPqC4BrIVibbuge09L3JMTOt65If29XRA/640?wx_fmt=png)

我们测试一下**从节点 redis-6381** 的读写操作，发现**不能执行写入操作，但是可以读取数据**，其中 muse 是我们在 6380 主节点中添加的

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9My4voyHjYEmpSmptmklEQxicBIZaUzXQ6v4mQohbH4qs2olgzUw1bDQ/640?wx_fmt=png)

我们测试一下从**节点 redis-6382** 的读写操作，也一样是**只读的**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9A876aQicOyh2htRsYlFaa3Gs24gajjThZk6VAOSaYwDXUGuP0cvT27A/640?wx_fmt=png)

### 3.2> 主节点意外宕机

我们**关闭主节点 6380 的服务**，查看从节点的对外服务是否收到影响

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9jdELe5hjcRn2RhqaSagARxFpkusrpOZicHR3MK9SHGqN4S1vZNFEIZA/640?wx_fmt=png)

测试两个从节点是否可以对外正常的提供服务。如下所示，我们可以看到，**从节点依然可以对外提供只读的服务**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9XckAHZoWj2RfYLX88Dbo7clftbexRSDWrWDWgHRrXGHc8wRv0AfVPg/640?wx_fmt=png)

虽然主节点挂掉了，但是这两个**从节点并不会自动的成为主节点**，他们依然是从节点的角色。我们可以通过`info replication`来确认一下

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf91xHtUnSU85qQ7l40fZ8YyZDpbn05foZrL9DTdRNP5tUzzORrGdNLibA/640?wx_fmt=png)

我们**重新启动主节点**，并且添加数据，我们来确认一下，这两个从节点会不会依然能够获得主节点同步过来的新数据

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9oibe5iadAcKC6j31HBz26U13FTCwe4qxoaHbicohSofqmZCKz6h8jIdMA/640?wx_fmt=png)

> 【解释】我们发现，两个从节点都可以获取到新添加的 bob。说明，**只要主节点再次成功启动，主从结构依然可以自动的建立起来**。

四、实现原理
------

Redis 的主从复制可以分为两个阶段：

> *   • **sync** 阶段
>     
> *   • **command propagate** 阶段。
>     

### 4.1> sync 阶段

当**从节点启动后**，会发送`sync`指令给主节点，要求**全量同步数据**。具体步骤如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9Fa56rAaVN2iaInKZ3akkEWLwjhdA5WYnOE43sXCicwS0cVicDicvDVRRKA/640?wx_fmt=png)

> **步骤 1**：Slave 启动后，连接 Master 节点并发送 sync 指令。  
> **步骤 2**：Master 节点接到 sync 指令后，会执行`BGSAVE`指令，生成`RDB`文件。此外，在 Master 节点生成 RDB 文件时，会将此后客户端执行的`增删改操作都存入缓冲区`。  
> **步骤 3**：文件生成后，会发送给 Slave 节点，Slave 节点接收到后，会`删除所有旧的数据，然后加载RDB数据`，实现数据`全量同步`操作。  
> **步骤 4**：当 Slave 数据加载完毕后，Master 会将`缓冲区的指令`发送给 Slave  
> **步骤 5**：由 Slave 去`执行缓冲区新增的指令`。

### 4.2> command propagate 阶段

该阶段属于**命令传播阶段**。

上面我们介绍了，Slave 节点通过 sync 指令请求 Master 节点全量数据的同步操作。那么，如果**后续 Master 节点接收到新的增删改操作**，也需要 Slave 节点接收同步的更新，那么这种就是 command propagate

### 4.3> psync 指令

当主从节点都正在运行的时候，出现了网络抖动，造成连接断开，那么当网络恢复，两个节点再次建立起连接的时候。从节点发送`sync`指令后，主节点依然需要重新生成 RDB，并对从节点进行全量数据的同步造成。那么这中间的耗时是非常严重的，并且传输备份文件也会对网络带宽造成很大的消耗。那么为了解决这个问题，从 Redis 2.8 开始，引入了 **psync 指令来代替 sync 指令**

psync 指令会根据不同的情况，来确定执行`全量重同步`还是`部分重同步`

**全量重同步**

> *   • 当从节点是`第一次`与主节点建立连接的时候，那么就会执行全量重同步，这个同步过程与上面我们介绍的`sync阶段`+`command propagate阶段`一样
>     

**部分重同步**

> *   • 从节点的**复制偏移量**无法在**复制积压缓冲区**中找相应待同步的数据  
>     
> *   • 主节点与从节点不是第一次同步（根据 Redis **节点 ID** 判断）
>     

**什么是复制偏移量？**

> *   • Master 节点和 Slave 节点都保存着一份赋值偏移量。
>     
> *   • 当 Master 节点每次向 Slave 节点发送 n 字节数据的时候，就会在 Master 节点偏移量加上 n；而 Slave 节点每次接收到 n 个字节的时候，也会在 Slave 节点偏移量上加 n。
>     
> *   • 在命令传播阶段，Slave 节点会定期的发送心跳`REPLCONF ACK{offset}` 指令，这里的 offset 就是 Slave 节点的 offset。当 Master 节点接收到这个心跳指令后，会对比自己的 offset 和命令里的 offset，如果发现有数据丢失，那么 Master 节点就会推送丢失的那段数据给 Slave 节点。如下图所示：
>     
>     ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCib2ahZFIzT6sOad01iaYbNf9efeCP2RfF6uIoJ3tr7Vkv2CEnmpyRuia6zA1ZT2NANugXWwZry6CwyA/640?wx_fmt=png)
>     

**什么是复制积压缓冲区？**

> *   • 复制积压缓冲区是由主节点维护的一个固定长度（默认 **1MB**）的队列。
>     
> *   • 它存储了每个字节值与对应的复制偏移量。
>     
> *   • 因为复制积压缓冲区的大小是固定的，所以它保存的是主节点近期执行的写命令。当从节点将 offset 发送给主节点后，主节点便会根据 offset 与复制积压缓冲区的大小来决定是否可以使用部分重同步。`如果offset之后的数据仍然在复制积压缓冲区内，则执行部分重同步；否则还是执行全量重同步`。
>     

**什么是节点 ID？**

> *   • Redis 节点服务启动之后，就会产生一个**用来唯一标识 Redis 节点的 ID**。
>     
> *   • 当 Master 节点与 Salve 节点进行第一次连接同步的时候，Master 节点会将 ID 发送给 Slave 节点，**Slave 节点接收到会，会对其进行保存**。那么当主从服务之间发生了中断重连的时候，Slave 服务器会将这个 ID 发送给 Master 服务器，Master 服务器会拿自己的 ID 进行对比，**如果相同，则说明主从之前是连接过的。否则，则说明是第一次建立的连接。那么，就需要全量去同步数据了**。
>     

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的 **点赞** & **分享** 。

更多技术干货，欢迎大家关注公众号 “**爪哇缪斯**” ~ \(^o^)/ ~ 「干货分享，每天更新」

往期推荐
----

[如何利用 Redis 进行事务处理呢？](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490123&idx=1&sn=4f4e958af4df1afbcef8923f8fd96e7f&chksm=e91158b6de66d1a07f19998c6d46de480e99137d434443b09d3a12d593f32ecfa6f03fc35a24&scene=21#wechat_redirect)  

[你说啥？Redis 中除了五大数据类型，还有特殊数据类型！](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489663&idx=1&sn=67f4752147c2e58c496e1258098b9103&chksm=e9115a82de66d39404f3c873da470af94ab2cfe6e71fad9fc38f82be19da61b5d4005dc099b8&scene=21#wechat_redirect)  

[（三）DDD 上下文映射图——老师，我俩可是纯洁的男女关系！](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489855&idx=1&sn=5449ec4d350d27202e84aee82d9958ad&chksm=e9115bc2de66d2d4f056d051f77b6975525a4f6848c9c493fe490c40b0ed444ca0db2169f926&scene=21#wechat_redirect)  

[不了解阻塞队列，怎么跟面试官侃大山？](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489781&idx=1&sn=45c9fee860a58e7a488029d9c19aa013&chksm=e9115a08de66d31e2519996f8c7f0935567669cc17f2a7629fda58b5d7c026b58c926252dbfe&scene=21#wechat_redirect)  

[面试官：设计原则有哪些？什么是里式替换原则？](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489707&idx=1&sn=83dbb8fed1a3eca746c730f6068b333c&chksm=e9115a56de66d3407c3eb06446224697e28a2c76e5791bbc2000383475dbaec1d5573efc46a4&scene=21#wechat_redirect)  

[图解 LeetCode——1302. 层数最深叶子节点的和（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490083&idx=1&sn=613619fc9059d72251eb418b636d6384&chksm=e91158dede66d1c85825f2f9852593515da729e90b239bd0572a1c65e4129204cafabf1cbdc1&scene=21#wechat_redirect)  

[图解 LeetCode——1224. 最大相等频率（难度：困难）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490100&idx=1&sn=777a51fffb9ab729cb2c31f7625b5f94&chksm=e91158c9de66d1dfaff683db848ba3a51b01be7a59ec05718530b9a99cabc9fe2b0eaaf7f4c7&scene=21#wechat_redirect)  

[图解 LeetCode——768. 最多能完成排序的块 II（难度：困难）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490036&idx=1&sn=31627c58400f54bbbdfc4d66d48c8857&chksm=e9115b09de66d21f291feac74cf55e1cd6169c07b99dbd58d8aac79f71564d388591349f04ba&scene=21#wechat_redirect)  

[图解 LeetCode——1282. 用户分组（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490019&idx=1&sn=d62f5851451816c210bedb530cf7a334&chksm=e9115b1ede66d2087f99282d7d30de6ac98cc4244e271b0d384bd37fc9c55acd6e2dbb1bb64d&scene=21#wechat_redirect)