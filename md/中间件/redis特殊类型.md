
一、geospatial 地理位置
=================

1.1> 概述
-------

可以用于基于**地理位置**的业务场景。比如：查询两地之间的距离，方圆几里存在的地理位置等等。

Redis 提供了`geospatial`相关的`8个`指令，操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxkicobE3aPluuSZh8eOsE0uiclIVVHX0vLKA4fWcTPAlCjxWiapbXCOm3g/640?wx_fmt=png)

1.2> GEOADD（v3.2.0）
-------------------

官方文档：http://www.redis.cn/commands/geoadd.html

指令格式：`GEOADD key longitude latitude member [longitude latitude member ...]`

指令含义：将指定的地理空间位置（纬度、经度、名称）添加到指定的 key 中。

**这些数据将会存储到 Zset**，这样的目的是为了方便使用`GEORADIUS`或者`GEORADIUSBYMEMBER`命令对数据进行**半径查询**等操作。

该命令采用标准格式的参数 x,y，所以**经度必须在纬度之前**。这些坐标的限制是可以被编入索引的，区域面积可以很接近极点但是不能索引。

具体的限制，由 EPSG:900913 / EPSG:3785 / OSGEO:41001 规定如下：

> *   • 有效的**经度**从`-180`度到`180度`。
>     
> *   • 有效的**纬度**从`-85.05112878`度到`85.05112878`度。
>     
> *   • 当坐标位置超出上述指定范围时，该命令将会返回一个错误。
>     

*   操作如下图所示：（经纬度查询 https://jingweidu.bmcx.com）：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxwlwEjicxrUiaPYUNmkMBQnLpmLsNuXb5ujkbuFyGKz6VzhWTfeU6J85w/640?wx_fmt=png)

1.3> GEODIST（v3.2.0）
--------------------

官方文档：http://www.redis.cn/commands/geodist.html

指令格式：`GEODIST key member1 member2 [unit]`

指令含义：返回两个给定位置之间的距离。如果两个位置之间的其中一个不存在， 那么命令返回空值。

指定单位的参数`unit`必须是以下单位的其中一个：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxHF6oXjibqib5WTR5kNDibX2W95ckkwgVhujS0GNPKBsmH12NkVVicDaLBw/640?wx_fmt=png)

如果用户没有显式地指定单位参数， 那么`GEODIST`默认使用**米**作为单位。

`GEODIST`命令在计算距离时会假设地球为完美的球形，在极限情况下，这一假设最大会造成`0.5%`的误差。

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxJEVtI4hSpib0BQWauoZJqcicIBKyRAiaqKytztqScfNmrMBEJho86enSA/640?wx_fmt=png)

1.4> GEOHASH（v3.2.0）
--------------------

官方文档：http://www.redis.cn/commands/geohash.html

指令格式：`GEOHASH key member [member ...]`

指令含义：返回一个或多个位置元素的 Geohash 表示。通常使用表示位置的元素使用不同的技术，使用 Geohash 位置 52 点整数编码。由于编码和解码过程中所使用的初始最小和最大坐标不同，编码的编码也不同于标准。

该命令将返回`11个字符`的`Geohash`字符串，所以没有精度 Geohash。返回的 Geohash 具有以下特性：

> *   • 他们可以缩短右边的字符。它将失去精度，但仍将指向同一地区。
>     
> *   • 它可以在 geohash.org 网站使用，网址 http://geohash.org/
>     
>     。查询例子：http://geohash.org/sqdtr74hyu0

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxSKt41Q6xQRjzmia8MqyyK99HXXkW6yJGwGIERZOpKvOuwQfkLh9hpKg/640?wx_fmt=png)

1.5> GEOPOS（v3.2.0）
-------------------

官方文档：http://www.redis.cn/commands/geopos.html

指令格式：`GEOPOS key member [member ...]`

指令含义：从`key`里返回所有给定位置元素的位置（经度和纬度）

因为`GEOPOS`命令接受可变数量的位置元素作为输入，所以即使用户只给定了一个位置元素，命令也会返回数组回复。

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5Ox0VFLpXnicfWFUQQXjmTlnnxMNarygj5PBiba4lRNDwBPMicR7rkHWbfOw/640?wx_fmt=png)

1.6> GEORADIUS（v3.2.0）
----------------------

官方文档：http://www.redis.cn/commands/georadius.html

指令格式：`GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]`

指令含义：以给定的经纬度为中心，返回键包含的位置元素当中，与中心的距离不超过给定最大距离的所有位置元素。范围可以使用以下其中一个单位：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxHF6oXjibqib5WTR5kNDibX2W95ckkwgVhujS0GNPKBsmH12NkVVicDaLBw/640?wx_fmt=png)

在给定以下可选项时，命令会返回额外的信息：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5Ox3ROHvHsRhKKIZsp902laY7bDicQTkfCDz5Ht7gQLY6GabbuiavfpHSaA/640?wx_fmt=png)

命令默认**返回未排序**的位置元素。通过以下`2个`参数，用户可以指定被返回位置元素的排序方式：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxZRylr3Pcwf56hibnXt2T447HD6tBuicKALFic6QicqrMKb0vpmsUQaaTQA/640?wx_fmt=png)

在默认情况下，GEORADIUS 命令会返回所有匹配的位置元素。虽然**用户可以使用`COUNT <count>`选项去获取前 N 个匹配元素**，但是因为命令在内部可能会需要对所有被匹配的元素进行处理， 所以在对一个非常大的区域进行搜索时，即使只使用 COUNT 选项去获取少量元素，命令的执行速度也**可能会非常慢**。但是从另一方面来说，使用 COUNT 选项去减少需要返回的元素数量，对于减少带宽来说仍然是非常有用的。

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxFlbl5KsLBzDZZjpflHtVocG2aZaYkTjHOexCaL8zQbzcia6wu3ZkEZA/640?wx_fmt=png)

1.7> GEORADIUSBYMEMBER（v3.2.0）
------------------------------

官方文档：http://www.redis.cn/commands/georadiusbymember.html

指令格式：`GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]`

指令含义：这个命令和`GEORADIUS`命令一样，都可以找出位于指定范围内的元素，但是 **GEORADIUSBYMEMBER 的中心点是由给定的位置元素决定的**，而不是像 GEORADIUS 那样，使用输入的经度和纬度来决定中心点指定成员的位置被用作查询的中心。

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5Ox0tFtB19q6qXr5jjoNUia9JQxTUjeAnia0anWpiaFk2fRW9HYzJgHpGC2w/640?wx_fmt=png)

二、hyperloglog 预估集合的基数
=====================

2.1> 概述
-------

hyperloglog 常用的使用场景，一般是**非精准性的统计计数**。比如：统计访问网站的 UV 数，商品评论数或点击量等等。

hyperloglog 是一种用于计算唯一事物的概率数据结构（从技术上讲，这称为预估集合的基数）

**它占用的空间很小，只需要`12KB`的内存，可以存储`2^64`不同的元素数量**。但是它的统计是**有小于`1%`的误差**，所以并**不适合精准统计使用场景**。

Redis 提供了 hyperloglog 相关的`3个`指令，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxTNiaEvianHkcqBU9Eic6lrZdnicRaEsNsu9G3icnwSt0n5hnZE91FdICWFw/640?wx_fmt=png)

2.2> PFADD（v2.8.9）
------------------

官方文档：http://www.redis.cn/commands/pfadd.html

指令格式：`PFADD key element [element ...]`

指令含义：将`element`集合存储到以`key`为变量名的`HyperLogLog`结构中.

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxxmYXH0lNw8A0U55Z22UcbVfKic01DoibaaPU1hUAagHBciaa9QeJdOsPQ/640?wx_fmt=png)

2.3> PFCOUNT（v2.8.9）
--------------------

官方文档：http://www.redis.cn/commands/pfcount.html

指令格式：`PFCOUNT key [key ...]`

指令含义：获得指定`key`为变量名的 HyperLogLog 结构中中元素的个数

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxsiaQM425fyqtOr0jouth8ZrxuWibpXQa1eibGVx45nKkNtjnmXwQWfmaA/640?wx_fmt=png)

2.4> PFMERGE（v2.8.9）
--------------------

官方文档：http://www.redis.cn/commands/pfmerge.html

指令格式：`PFMERGE destkey sourcekey [sourcekey ...]`

指令含义：**将多个 HyperLogLog 合并（merge）为一个新的 HyperLogLog**，合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（`observed set`）的并集。合并得出的 HyperLogLog 会被储存在目标变量（第一个参数）里面，**如果该键并不存在，那么命令在执行之前， 会先为该键创建一个空的**。

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxG31NvFicq7Phk9kx4wticH4ibRPb4rQB1WS1QGDGiarUicHpVl75SzLkh9A/640?wx_fmt=png)

三、bitmap 位图
===========

3.1> 概述
-------

我们可以利用 bitmap 指定其二进制位是`0`或`1`，来实现类似 “`是`”or“`否`” 的相关操作。它的特点也是**占用内存空间特别的小**。比如，我们要记录每个用户当天是否活跃（即：是否登录过系统），那么如果我们要记录他一年的是否登录的记录，只需要 365 个 bit 即可存储。

Redis 提供了位图相关的`7个`指令，我们只针对其中常用的`3个`进行操作演示。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxS4DCnXzaoU8x9FAcMnicZv6eaEWJBHamsqqVZ5uKmQQjMvV7qibpAZvw/640?wx_fmt=png)

3.2> SETBIT（v2.6.0）
-------------------

官方文档：http://www.redis.cn/commands/setbit.html

指令格式：`SETBIT key offset value`

指令含义：**设置**或者**清空**`key`的`value`(字符串) 在`offset`处的`bit`值。那个位置的`bit`要么被设置，要么被清空，**这个由`value`（只能是`0`或者`1`）来决定**。当`key`不存在的时候，就创建一个新的字符串`value`。要确保这个字符串大到在`offset`处有`bit`值。参数`offset`需要大于等于`0`，并且小于`2^32`（限制`bitmap`大小为`512MB`）。当`key`对应的字符串增大的时候，新增的部分`bit`值都是设置为`0`。

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5Ox1PrLdMicEYJ5G52w2bicdvet2l9adZiaQK1cMhKSvqJfibJOjw238qxMhQ/640?wx_fmt=png)

3.3> GETBIT（v2.2.0）
-------------------

官方文档：http://www.redis.cn/commands/getbit.html

指令格式：`GETBIT key offset`

指令含义：获取`key`中某个`offset`位置上的值

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxNHtMrgMHIlyHkFM0FEicgibqDic2tr1xSGSTgxy8MxNzhGCcZMLq5LntQ/640?wx_fmt=png)

3.4> BITCOUNT（v2.6.0）
---------------------

官方文档：http://www.redis.cn/commands/pfmerge.html

指令格式：`BITCOUNT key [start end]`

指令含义：获取`key`中从`start`到`end`范围内`1`的个数

操作如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicPDXjOD2Ilib55oibico9X5OxY0jMicVMMmlYicCicsplTTaWXXcFCpTtYRAdoq7MPIy2SFtqs3vc59J9A/640?wx_fmt=png)

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的**点赞** & **分****享**。

**更多技术干货，欢迎大家关注公众号 “爪哇缪斯”(^o^)/~ 「干货分享，每周更新」**

往期推荐
====

[多图详解阻塞队列——SynchronousQueue](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489434&idx=1&sn=31a8254f722e7150587bbe01fce9e6ee&chksm=e9115567de66dc7145acfe78e082ee925de8c48f3d2b97f36b9c4a14bac320f11e313f2cdc17&scene=21#wechat_redirect)  

[面试官：来说一下，你们是怎么解决分布式场景下的事务问题？](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489231&idx=1&sn=e8df78066484855f2827d12184a5e1bc&chksm=e9115432de66dd246a25113175fe399a31b35e2280c3e36c8adfc4e84f0cae5ac7cbc0ecc9f9&scene=21#wechat_redirect)  

[多线程下 HashMap 死锁问题源码分析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489108&idx=1&sn=3fd8aae8e1cb7f5c4a8edf66f1196608&chksm=e91154a9de66ddbf3fba8aa046a5084b9079de519042fea4b3ff774ae8ab5de1fc16925db343&scene=21#wechat_redirect)  

[男人要 “强”，不要“软弱”，“虚” 得一匹](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489202&idx=1&sn=4709871ef90d656531e10bc7267d98d2&chksm=e911544fde66dd598fb4850605bc0b5cbdd73b90c5c6189b1400d3d56e558295edc24c37d49d&scene=21#wechat_redirect)  

[缪斯小卡片——IO 多路复用](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489040&idx=1&sn=605a50877450d04decedbb53187ba2c7&chksm=e91154edde66ddfbadc12a14f54f65fe59e0b949b4b931e21803a1b8e84dc08be2cd59b512af&scene=21#wechat_redirect)  

[缪斯's Tips——桥接方法](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489061&idx=1&sn=d356fa2bbaeae1880e44595f189e5ba4&chksm=e91154d8de66ddcece9870ea95bf862f511db6f8ff4a6862b4aa6bc86af57e54943655cbb8bc&scene=21#wechat_redirect)