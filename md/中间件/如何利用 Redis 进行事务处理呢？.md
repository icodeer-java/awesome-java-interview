
一、概述
----

事务的本质，其实就是一组命令的集合。**一个事务中的所有命令都会按照命令的顺序去执行，而中间不会被其他命令加塞**。

Redis 提供了事务相关的 **5 个指令**，分别是：`DISCARD`、`EXEC`、`MULTI`、`UNWATCH`和`WATCH`。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8iaHASEB2QSo1xiaIfsicC013OfTmqRUsOpBibm52hNqdZJyGh0Lf4QibXUQosia9V3g2gCo3LiagEhPWYw/640?wx_fmt=png)

下面我们就对 Redis 的事务操作一一的进行介绍。

二、MULTI（v1.2.0）
---------------

**指令格式**：`MULTI`

**操作逻辑**：标记一个`事务块的开始`。随后的指令将在执行 EXEC 时作为一个原子执行。简而言之，我们`可以使用MULTI来开启一个事务`。

**操作演示**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8iaHASEB2QSo1xiaIfsicC013ER6I8KViaMuTia2WsDzqGfwWgmhS1jzZLKQuInXyLvXGUjicfjXhiaezcQ/640?wx_fmt=png)

**操作解释**：我们发现，在事务中每次执行一条指令，就会返回`QUEUED`，表明指令已经存入了这个事务的执行队列中了。但是需要注意的一点是，`只是放入了事务队列，但并没有去执行`。那什么时候会执行呢？那就来看一下下个指令 EXEC。

三、EXEC（v1.2.0）
--------------

**指令格式**：`EXEC`

**操作逻辑**：执行事务中所有在排队等待的指令并将链接状态恢复到正常。当使用 WATCH 时，只有当被监视的键没有被修改，且允许检查设定机制时，EXEC 会被执行。简而言之，`我们可以使用EXEC来提交一个事务`。

**操作演示**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8iaHASEB2QSo1xiaIfsicC013YRiad3t5icvuR5AkXktBEWkcm76OoTjLeBnMa9DdcAm5MsrJianicOQSpA/640?wx_fmt=png)

**操作解释**：调用完 EXEC 之后，正确执行的都会`返回OK`，并且当我们再次查询 account 里面的金额的时候，也正确的返回了 1100。这就说明，一个事务内的指令是按照顺序执行的。

四、DISCARD（v2.2.0）
-----------------

**指令格式**：`DISCARD`

**操作逻辑**：刷新一个事务中所有在排队等待的指令，并且将连接状态恢复到正常。`如果已使用WATCH，DISCARD将释放所有被WATCH的key`。

**操作演示**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8iaHASEB2QSo1xiaIfsicC013ZtWW4VWBwxUSWxAtKB4hEKh2evkiazjGicsg5COKvwDDHD7Y7zmUpwXw/640?wx_fmt=png)

五、WATCH（v2.2.0）
---------------

**指令格式**：`WATCH`

**操作逻辑**：标记所有指定的`key被监视起来`，在事务中有条件的执行。`可以利用WATCH实现Redis的乐观锁`。

**操作演示 1**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8iaHASEB2QSo1xiaIfsicC013oe97icqX4hyhnCG0ApeNSvrVVdgXmwIbLlMOFLYZ0S1n6bW9mBzcvew/640?wx_fmt=png)

**操作演示 2**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8iaHASEB2QSo1xiaIfsicC013oPOLyq9jjM82cGD7LzEHmD24lWJxLLgmwjb52Tv3V3cwBTmqh4LX7w/640?wx_fmt=png)

**操作解释**：当执行了 EXEC 指令之后，`watch就被隐式的执行了unwatch`。如果需要再次监控，就需要再次调用 WATCH 指令。

六、UNWATCH（v2.2.0）
-----------------

**指令格式**：`UNWATCH`

**操作逻辑**：刷新一个事务中已被监视的所有 key。`如果执行EXEC或者DISCARD，则不需要手动执行UNWATCH` **操作演示**：（略）

七、事务中异常的处理
----------

### 7.1> 命令语法错误

针对语法错误，`会导致整个事务执行被中断`

**操作演示**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8iaHASEB2QSo1xiaIfsicC013Mr2outXiaGIdibzK5ib5hfaTORiaDYElBSjZ6411thvgYa9RgOcbqBxu8g/640?wx_fmt=png)

### 7.2> 运行操作错误

针对执行中的异常，只会导致该条指令的执行失败，而不会影响事务中其他的指令

**操作演示**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8iaHASEB2QSo1xiaIfsicC013dwZ4LDheR0VHNscaE8US8j8RdmT79mKZvw0GEGy3pHS86AnZmMQ6Ow/640?wx_fmt=png)

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的 **点赞** & **分享** 。

更多技术干货，欢迎大家关注公众号 “**爪哇缪斯**” ~ \(^o^)/ ~ 「干货分享，每天更新」

往期推荐
----

[图解 LeetCode——1224. 最大相等频率（难度：困难）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490100&idx=1&sn=777a51fffb9ab729cb2c31f7625b5f94&chksm=e91158c9de66d1dfaff683db848ba3a51b01be7a59ec05718530b9a99cabc9fe2b0eaaf7f4c7&scene=21#wechat_redirect)  

[图解 LeetCode——1302. 层数最深叶子节点的和（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490083&idx=1&sn=613619fc9059d72251eb418b636d6384&chksm=e91158dede66d1c85825f2f9852593515da729e90b239bd0572a1c65e4129204cafabf1cbdc1&scene=21#wechat_redirect)  

[图解 LeetCode——1656. 设计有序流（难度：简单）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490064&idx=1&sn=33f68307bed4fcfcc16a453fea27c0be&chksm=e91158edde66d1fb03a334bf874ad14803661b113a1666d33e98a79914ba313b738983938622&scene=21#wechat_redirect)  

[图解 LeetCode——1422. 分割字符串的最大得分（难度：简单）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490049&idx=1&sn=7aaf6fd7e1790ff29801fa717d3bb6e8&chksm=e91158fcde66d1ea2e93c50dbba5e1b9d9f0cfffd38712010ad77bc3b28b530188062a0ace9f&scene=21#wechat_redirect)  

[图解 LeetCode——768. 最多能完成排序的块 II（难度：困难）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490036&idx=1&sn=31627c58400f54bbbdfc4d66d48c8857&chksm=e9115b09de66d21f291feac74cf55e1cd6169c07b99dbd58d8aac79f71564d388591349f04ba&scene=21#wechat_redirect)  

[图解 LeetCode——1282. 用户分组（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490019&idx=1&sn=d62f5851451816c210bedb530cf7a334&chksm=e9115b1ede66d2087f99282d7d30de6ac98cc4244e271b0d384bd37fc9c55acd6e2dbb1bb64d&scene=21#wechat_redirect)  

[图解 LeetCode——1417. 重新格式化字符串（难度：简单）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247490004&idx=1&sn=bdb2b9778e42c237bcdefa13fd754150&chksm=e9115b29de66d23ff74705e1c464c4138d2fd2e85fbe35ccf8e9a444bc704407b58433bc5bb7&scene=21#wechat_redirect)  

[图解 LeetCode——640. 求解方程（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489985&idx=1&sn=bdbecad9e81a49a4b36def4e4cc69f05&chksm=e9115b3cde66d22a1c861fce2e2fd42fc28e6747ed4a4093e4259427c97847f6bdbc64f51969&scene=21#wechat_redirect)