
概述
==

Redis3.0 开始引入了去中心化分片集群 Redis Cluster。

传统的 Redis 集群是基于**主从复制 + 哨兵**的方式来实现的。但是集群中都**只有一个主节点提供写服务**。

Redis Cluster 则采用**多主多从**的方式，支持开启多个主节点，每个主节点上可以挂载多个从节点。

Cluster 会**将数据进行分片**，将数据分散到多个主节点上，而每个主节点都可以对外提供读写服务。这种做法使得 Redis 突破了单机内存大小限制，扩展了集群的存储容量。并且 Redis Cluster 也具备高可用性，因为每个主节点上都至少有一个从节点，当主节点挂掉时，Redis Cluster 的故障转移机制会将某个从节点切换为主节点。

Redis Cluster 是一个**去中心化的集群**，每个节点都会与其他节点保持互连，使用 **gossip 协议**来交换彼此的信息，以及探测新加入的节点信息。并且 Redis Cluster 无需任何代理，客户端会直接与集群中的节点直连。

分片方式
====

方案 1：哈希取模
---------

这种方式就类似我们使用 HashMap 时选址的方式，只要 hash 计算出来的值够散列，那么每个 key 都可以均匀的分散到 N 个节点上。

但是它存在的问题就是，如果要扩容或缩容，会导致 key 重新计算存储位置，从而导致缓存失效。

方案 2：一致性哈希
----------

一致性哈希算法将整个哈希值空间组织成一个虚拟的圆环，其范围为`0 ~ 2^32-1`，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8U8gzZkY6ukEdFLXNI5CI3eYHltyaALIicJf5ej2cgslfSBM6thZkKP1Q/640?wx_fmt=png)

**一致性哈希算法的原理：**

> 我们会先对 Key 计算它的 hash 值，从而确定它在环上的位置。然后从该位置沿着环**顺指针**地走，找到的第一个节点，便是这个 Key 应该存放的服务器节点的位置。

当我们向集群中增加或减少节点时，就无需像哈希取模算法那样，对整个集群 Key 的位置进行重新计算。**一致性哈希算法将增减节点的影响限制在相邻的节点上**，比如：我们在`node2`与`node4`之间增加一个节点`node5`，则只有`node4`中的一部分数据会迁移到新增节点上；如果我们想要将`node4`节点置为下线状态，则`node4`节点的数据只会迁移到`node3`中，其他节点无影响。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8UiaNGzVGp6rNPdFREUl9LHlnj7Hia6pTrNOMMh6eUWKib6icKG0hXlBwX5Q/640?wx_fmt=png)

**一致性哈希算法的缺点：**

> 当节点比较少时，增删节点对单个节点的影响会很大，从而导致出现数据不均衡的情况。拿上图来举例，当我们删除任意一个节点，都会导致集群中的某一个节点的数据量由总数据的 `1/4` 变为 `1/2`。

方案 3：哈希槽
--------

该方案在一致性哈希的理论基础上，引入了**虚拟节点**这一概念。原本是由实际节点来 “抢占” **哈希环**的位置，现在则是将虚拟节点分配给实际节点，然后由虚拟节点来抢占。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8US1veHfCl5c0MticQzzF1qwX4jRodCaFUvCu5zq1ib29sRxCLQOSCN0Jw/640?wx_fmt=png)

在引入了虚拟节点这一概念后，数据到实际节点的映射关系就变成了数据到虚拟节点，再由虚拟节点到实际节点了。Redis 集群便是采用了这种方案。一个集群包含 16384 个哈希槽（hash slot）也就是 **16384 个虚拟节点**。譬如，我们的集群有三个节点，那么：

> *   • **Master1 节点**负责处理`0～5460`号 slot
>     
> *   • **Master2 节点**负责处理`5461～10922`号 slot
>     
> *   • **Master3 节点**负责处理`10923～16383`号 slot
>     

当我们在集群中新增了一个节点`Master4`，那么集群**只需要将 Master1，Master2，Master3 中负责的一部分 hash slot 分配给 Master4 节点就可以了**；

如果要移除某一个节点，也只需要将该节点负责的 hash slot 分配给其他的节点即可。这样集群便实现了良好的可扩容性。同时，由于存在`16384`个虚拟节点，那么这些 hash slot 在哈希环上可以分布均匀，从而实现负载均衡。

搭建集群（3 主 3 从）
=============

由于 Redis Cluster 要求必须要**至少 6 个节点**，所以我们就以配置 **3 主 3 从**为例：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8UqF95yGSrlBMtLHVHgryibt5kkZkW2mClprQxhLzAjnBhlx9UEjibotFA/640?wx_fmt=png)

修改 redis-6390.conf~redis-6395.conf 配置文件

```
# 配置集群节点对应的端口号(分别为6390，6391，6392，6393，6394，6395)
port 6390
# 守护进程开启
daemonize yes
# 关闭保护模式
protected-mode no
# 将集群开启
cluster-enabled yes
cluster-config-file nodes-6390.conf(分别为6390，6391，6392，6393，6394，6395)

```

启动集群中 redis 服务

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8UicPfZAZguz3w5wbym51S7sgSQdHLzXF9MBRWwq2ORJVlXk0w7ZfOavg/640?wx_fmt=png)

**分配主从（`--cluster-replicas 1`表示创建一主一从）**

> ./redis-cli --cluster create 127.0.0.1:6390 127.0.0.1:6391 127.0.0.1:6392 127.0.0.1:6393 127.0.0.1:6394 127.0.0.1:6395 --cluster-replicas 1

执行完毕结果如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8UZgxlDmXFZt8opzJPyS6WMHt1aVK55LhT69RjyJjQ7UhCgpaN0xK9ug/640?wx_fmt=png)

我们来登录`6390`，看看他在集群中的角色信息是什么

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8UZXgwMjxOeqTysOiciazzksfGjTBtaWywTCRy795CATBpl13ZTCM8jc9Q/640?wx_fmt=png)

我们在 6390 客户端中添加一条记录，我们发现，它根据 key 值确认了 slot=12965，然后将数据存储到了 6392 这个节点上，并且客户端也切换为 6392 了

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8UoQl6aWwCHQFOjktTpplYaYUoWh3deIunAw9FC8piaXzCD0gFWOuSMGA/640?wx_fmt=png)

部署过程中可能出现的异常
============

配置完集群后，可能会报如下错误，这说明 16384 个槽位没有分配完

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8UtxyOyK5gF3QAyo4LKZZzCdzAjazaTx7vG9kkaT1XvBMx9ibyzJvx01w/640?wx_fmt=png)

**我们通过如下指令就可以进行检查和修复**

> redis-cli **--cluster check** 172.17.0.2:6379 redis-cli **--cluster fix** 172.17.0.2:6379 #官方修复功能

修复后的结果如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8Ria1sMWOoib76XvvbXHTQ8Uxcd2Iz5uXkpAibqJfs68v17FdolxZV4RNE8qqDDTP7ic1ic2iaqZETR3Sg/640?wx_fmt=png)

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的 **点赞** & **分享** 。

更多技术干货，欢迎大家关注公众号 “**爪哇缪斯**” ~ \(^o^)/ ~ 「干货分享，每天更新」

往期推荐
----

[Redis 常见场景问题和解决方案](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490198&idx=1&sn=cf425d7f006166f80291b3ced0ef154d&chksm=e911586bde66d17dd4ed1f7513dd73418803c4795c445df262a7fe73418c24a119ee462824dc&scene=21#wechat_redirect)  

[只要五分钟！带你了解 Redis 主从复制](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490185&idx=1&sn=a768adafebfc554293cb1c445f19021f&chksm=e9115874de66d1626fd8550b709c4e24d6195717e545aca5b8b5b3542aec86be40af9f466c8b&scene=21#wechat_redirect)  

[如何利用 Redis 进行事务处理呢？](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490123&idx=1&sn=4f4e958af4df1afbcef8923f8fd96e7f&chksm=e91158b6de66d1a07f19998c6d46de480e99137d434443b09d3a12d593f32ecfa6f03fc35a24&scene=21#wechat_redirect)  

[你说啥？Redis 中除了五大数据类型，还有特殊数据类型！](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489663&idx=1&sn=67f4752147c2e58c496e1258098b9103&chksm=e9115a82de66d39404f3c873da470af94ab2cfe6e71fad9fc38f82be19da61b5d4005dc099b8&scene=21#wechat_redirect)  

[图解 LeetCode——782. 变为棋盘（难度：困难）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490225&idx=1&sn=709c0adb241ab5d3c5cdbc9e249c8d2a&chksm=e911584cde66d15a29814006b1c2c4183a12a631482629d4e87920a3e7d39b1037cd483d287b&scene=21#wechat_redirect)  

[图解 LeetCode——1282. 用户分组（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490019&idx=1&sn=d62f5851451816c210bedb530cf7a334&chksm=e9115b1ede66d2087f99282d7d30de6ac98cc4244e271b0d384bd37fc9c55acd6e2dbb1bb64d&scene=21#wechat_redirect)  

[图解 LeetCode——768. 最多能完成排序的块 II（难度：困难）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490036&idx=1&sn=31627c58400f54bbbdfc4d66d48c8857&chksm=e9115b09de66d21f291feac74cf55e1cd6169c07b99dbd58d8aac79f71564d388591349f04ba&scene=21#wechat_redirect)  

[图解 LeetCode——636. 函数的独占时间（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489949&idx=1&sn=e64b8c05e61310b0a77dd00d4d1ae273&chksm=e9115b60de66d276ce625c63d09d04a338749e0c4c9ba96526d064f4078f39c7cb86da1e01c3&scene=21#wechat_redirect)