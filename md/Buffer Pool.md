> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247484947&idx=1&sn=fa8cb52994b2b8e4eda1727251ac8166&chksm=e91144eede66cdf8a97ec12fb6a3033ff8bbf9074d1c4d9befa844226e5b1402f47707f7ea29&scene=178&cur_album_id=2142543588444995585#rd)

一、缓存的重要性
--------

*   无论是用于存储用户数据的**索引**，还是各种系统**数据**，都是以**页**的形式存放在**表空间**中的。
    
*   所谓**表空间**，只不过是 InnoDB 对一个或几个**实际文件的抽象**。也就是说，我们的数据说到底还是存储在**磁盘**上的。
    

*   但是磁盘读取速度很慢，所以如果需要访问某个页的数据时，InnoDB 会把**完整的页**中的数据**全部加载**到**内存**中。即使只需要访问一个页里面的一条记录，也需要先把整个页的数据加载到内存中。然后就可以在内存中对记录进行读写访问了。
    
*   在读写访问之后，并不着急把该页对应的内存空间释放掉，而是将其缓存起来，如果将来再次访问该页面，就可以减少 I/O 的开销了。
    

二、Buffer Pool 概述  

-------------------

*   为了缓存磁盘中的页，MySQL 服务器**启动时**就向操作系统申请了一片**连续**的内存空间，他们给这片内存起名为——Buffer Pool（缓冲池）。
    
*   默认 Buffer Pool 只有 **128M**，可以在启动服务器的时候配置 **innodb_buffer_pool_size**（单位为字节）启动项来设置自定义缓冲池大小。
    

*   Buffer Pool 对应的一片**连续的内存**被划分为若干个页面，默认也是 **16KB**。该页面称为**缓冲页**。
    
*   为了更好的**管理** Buffer Pool 中的这些缓冲页，InnoDB 为每个缓冲页都创建了**控制块**。它与缓冲页是一一对应的。
    

*   控制块存放到 Buffer Pool 的**前面**，缓冲页存放到 Buffer Pool 的**后面**，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoa6DHyF5icx2qWWOiaXWZbaDZzr7DhzSB6jg2KHrt1rLAw3OhkGVk9BDGQ/640?wx_fmt=png)

三、几种主要的链表  

------------

### 3.1> free 链表

*   Buffer Pool 的初始化过程中，是先向操作系统**申请连续的内存空间**，然后把它划分成若干个【**控制块 & 缓冲页】**对儿。
    
*   当插入数据的时候，为了能够知道哪些缓冲页是**空闲可分配**的，由此产生了 free 链表。
    

*   free 链表是把所有空闲的缓冲页对应的**控制块**作为一个节点放到一个链表中，这个链表便称之为 free 链表。
    
*   free 链表如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoa1EVyxkyJwicT6g7MqVM91x4pJHPrOnzCVfddD2LicpKiaRxqagricVzQDQ/640?wx_fmt=png)【注释】

    其中**基节点**是一块单独申请的内存空间（约占 40 字节）。并不在 Buffer Pool 的那一大片连续内存空间里。

*   磁盘加载页的流程
    

*   首先：从 free 链表中取出一个空闲的控制块（对应缓冲页）。
    
*   其次：把该缓冲页对应的控制块的信息填上（例如：页所在的表空间、页号之类的信息）。
    
*   最后：把该缓冲页对应的 free 链表节点（即：控制块）从链表中移除。表示该缓冲页已经被使用了。
    

### 3.2> flush 链表

*   如果我们**修改了** Buffer Pool 中某个缓冲页的数据，那么它就**与磁盘上的页不一致了**，这样的缓冲页也被称之为**脏页**（**dirty page**）。为了性能问题，我们每次修改缓冲页后，并不着急立刻把修改刷新到磁盘上，而是在**未来的某个时间点进行刷新操作**。
    
*   _那么，如果有了修复发生，不是立刻刷新，那之后再刷新的时，我们怎么知道 Buffer Pool 中哪些页是脏页，哪些页从来没有被修改过呢？_
    

`答：创建一个存储脏页的链表，凡是被修改过的缓冲页对应的**控制块**都会作为节点加入到这个链表中。该链表也被称为flush链表。`

*   flush 链表的结构与 free 链表差不多。
    
*   flush 链表如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoarKK2L8NFLBQOBBkeevHldzlkVuMdl269wEZAwsq2oNRj62ibTZAupdA/640?wx_fmt=png)

*   那么，会存在一个控制块既是 free 链表的节点，也是 flush 链表的节点吗？
    

 答：不会的。因为如果一个缓冲页是空闲的，那它肯定不可能是脏页。反之亦然。

*   后台有**专门的线程**负责每隔一段时间就把脏页刷新到磁盘。这样就不影响用户线程处理正常的请求了。
    

刷新方式有如下**两种**：

1> 从 **flush 链表**中刷新一部分页面到磁盘

*   后台线程会根据当时**系统的繁忙程度**确定刷新速率，**定时**从 flush 链表中刷新一部分页面到磁盘。——即：**BUF_FLUSH_LIST**
    
*   有时后台线程刷新脏页的进度比较慢，导致用户准备加载一个磁盘页到 Buffer Pool 中时**没有可用的缓冲页**。此时，就会尝试查看 **LRU 链表尾部**，看是否存在可以直接释放掉的**未修改**缓冲页。如果没有，则不得不**将 LRU 链表尾部的一个脏页同步刷新到磁盘**（与磁盘交互是很慢的，这会降低处理用户请求的速度）。——即：**BUF_FLUSH_SINGLE_PAGE**
    

2> 从 **LRU 链表的冷数据**刷新一部分页面到磁盘，即：**BUF_FLUSH_LRU**

*   后台线程会**定时**从 LRU 链表的**尾部**开始扫描一些页面，扫描的页面数量可以通过系统变量 innodb_lru_scan_depth 来指定，如果在 LRU 链表中发现脏页，则把它们刷新到磁盘。
    
*   控制块里会存储该缓冲页是否被修改的信息，所以在扫描 LRU 链表时，可以很轻松地获取到某个缓冲页是否是脏页的信息。
    

### 3.3> LRU 链表

*   LRU = Least Recently Used
    
*   由于缓冲区空间有限，如果满了，则需要**把旧的移除掉**，新的加进来。那么移除规则是什么呢？
    

*   我们的目的，肯定是为了把使用频繁的数据保留在缓存中，把使用频率低的数据移除。
    
*   用来记录缓冲页的被使用**热度**。
    

*   Buffer Pool 的**缓冲命中率**（我们当然是期望命中率越高越好）
    

假设我们一共访问了 n 次页，那么被访问的页**已经在 Buffer Pool 中**的次数除以 n，那么就是 Buffer Pool 的缓冲命中率。

*   提高命中率的方法——简单的 LRU 链表
    

【处理逻辑】

*   1> 创建 LRU 链表。
    
*   2> 当要访问某个页时，如果**不在 Buffer Pool**，则把该页从磁盘加载到缓冲池的缓冲页时，就把该缓冲页对应的**控制块**作为节点塞到 LRU 链表的**头部**。
    
*   3> 如果**在 Buffer Pool** 中，则直接把该页对应的控制块移动到 LRU 链表的**头部**。
    

【方案优点】

*   所有最近使用的数据都在链表表头，最近未使用的数据都在链表表尾。
    

【方案缺点】

*   1> 由于**预读**（下面会介绍）的行为，很多预读的页都会被放到 LRU 链表的表头。如果这些预读的页都没有用到的话，这样，会导致很多尾部的缓冲页很快就会被淘汰。
    
*   2> 如果发生**全表扫描**（比如：没有建立合适的索引 or 查询时没有 where 字句），则相当于把原有缓冲页全部都冲刷没了。
    

*   _什么是预读？_
    

*   就是 InnoDB 认为执行当前的请求时，可能会在后面读取某些页面，于是就预先把这些页面加载到 Buffer Pool 中。
    

*   根据触发方式不同，预读可以分为两种：
    

*   **线性预读**
    
    如果**顺序访问**某个区（extent，一个区默认 **64** 个页）的页面超过了 **innodb_read_ahead_threshold**（默认 **56**）的值，就会触发一次异步读取**下一个区中全部的页**到 Buffer Pool 中的请求。
    
*   **随机预读**
    
    如果开启了随机预读功能（默认 **innodb_random_read_ahead=OFF**）
    
    ，如果某个区（extent）有 **13 个连续的页面**都已经被加载到了 Buffer Pool 中，**无论这些页面是不是顺序读取的**，都会触发一次异步读取**本区全部的页**到 Buffer Pool 中的请求。
    

*   综上所述，其实造成 Buffer Pool 命中率低有两种情况：
    

*   1> 加载到 Buffer Pool 中的页**不一定被用到**。
    
*   2> 如果有非常多的使用频率偏低的页被同时加载到 Buffer Pool 中，则可能会把那些使用**频率非常高的页**从 Buffer Pool 中**淘汰掉**。
    

*   为了解决命中率低的问题，InnoDB 把 LRU 链按比例（**innodb_old_blocks_pct**）分成了两段——**young 区域**和 **old 区域**。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoasNrUMvt5vO5w5x83mNud3mCrhicQ38p5G5iaBfWiaIicGRjorDg4icA9y8A/640?wx_fmt=png)

*   LRU 链表如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoat69BicUwLadAeIoTWViajP3W1bPOxBTQibJbCnZiareBQic97CwJiaoXfibWg/640?wx_fmt=png)

*   针对简单 LRU 链表方案缺点的优化
    

*   1> **针对预读的优化**
    
    InnoDB 规定，当磁盘上的某个页在**初次加载**（只是加载，没有涉及读取）到 Buffer Pool 中的某个缓冲页时，该缓冲页对应的控制块会被放到 **old 区域的头部**。这样预读页就只会在 old 区域，不会影响 young 区域中使用比较频繁的缓冲页。
    
*   2> **针对全表扫描的优化**
    
    虽然首次加载放到的是 old 区域的头部，但是由于是全表扫描，会对加载的数据进行访问，那么**第一次访问**的时候，就会将该页放到 young 区域的头部。这样仍然会把那些使用频率比较高的页面给 “排挤” 下去。_那怎么办呢？_
    
*   由于全表扫描有一个特点，就是**它对某个页的频繁访问**且总耗时很短。所以，针对这种情况，InnoDB 规定，在对某个处于 **old 区域**的缓冲页进行**第一次访问**时，就在它对应的**控制块**中记录下这个访问时间，如果后续的访问时间与第一次访问的时间在某个时间间隔内（即：**innodb_old_blocks_time，默认为 1000，单位为 ms**），那么该页面就不会从 old 区域移动到 young 区域的头部，否则将它移动到 young 区域的头部。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoajL7MZTdKP4nNu9EEep3eacz951lq9COoSeYB7xibjhGoM4hiczckeLYg/640?wx_fmt=png)

四、其他补充知识点  

------------

### 4.1> 多个 Buffer Pool 实例

*   在 Buffer Pool 特别大并且**多线程并发访问量**特别高的情况下，单一的 Buffer Pool 可能会影响请求的处理速度。所以，在 Buffer Pool 特别大时，可以把它们拆分成若干个小的 Buffer Pool，每个 Buffer Pool 都称为一个**实例**。它们都是独立的——独立地申请内存空间，独立地管理各种链表。
    
*   可以通过设置 **innodb_buffer_pool_instances** 的值来修改 Buffer Pool 实例的个数
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoaWicc3hm9X7403fOmKMR55qWRiaA6ibzcZYbuC4vwGlKe5GO84kktkich9w/640?wx_fmt=png)

*   每个 Buffer Pool 实例实际占用多少内存空间呢？
    
    **innodb_buffer_pool_size** **÷****innodb_buffer_pool_instances**
    
*   由于管理 Buffer Pool 也是需要性能开销的，所以也并不是实例越多越好；
    
*   InnoDB 规定，当 **innodb_buffer_pool_size 小于 1GB** 时，设置多个实例是无效的，在这种情况下，即使你设置的 **innodb_buffer_pool_instances** 不为 1，那么 InnoDB 默认也会把它改为 1。
    

### 4.2> chunk

*   由于每次**调整 Buffer Pool 的大小**时，都需要重新向操作系统申请一块连续的内存空间，然后将旧的 Buffer Pool 中的内容复制到这一块新空间，但是这种操作是极其耗时的。所以，InnoDB 不再一次性为某个 Buffer Pool 实例向操作系统申请一大片连续的内存空间，而是以一个 **chunk 为单位**向操作系统申请空间。也就是说，一个 Buffer Pool 实例其实是由若干个 chunk 组成的。**一个 chunk 就代表一片连续的内存空间**，里面包含了若干缓冲页与其对应的控制块。
    
*   如图所示，Buffer Pool 包含 2 个实例，每个实例里包含 2 个 chunk：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoaEMFnDa8Apm7LZYdBqqNOXbf5DcrKgoeQ5YNuib7sYictzxO4ajbrxibVg/640?wx_fmt=png)

*   可以通过 **innodb_buffer_pool_chunk_size** 指定 chunk 的大小
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC81vehHtCnCRuWy8G3mDLoasNrN3s1Zw7OzGgkZ6yfWaTRX4nIyCmHsJfQkCcBMmXprfcDpYo8S6A/640?wx_fmt=png)

【注】

134217728/1024/1024=128MB

innodb_buffer_pool_chunk_size 的值并不包含缓冲页对应的控制块的内存空间大小。

### 4.3> 配置 Buffer Pool 时的注意事项

*   innodb_buffer_pool_size 必须是：
    
    innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances 的倍数。否则服务器会自动把 innodb_buffer_pool_size 的值调整为：
    
    【innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances】结果的整数倍。
    

    例如：innodb_buffer_pool_chunk_size=128MB

        innodb_buffer_pool_instances=16

则：innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances=2GB

如果我们设置 innodb_buffer_pool_size=9GB，则会被自动调整为 10GB

*   在服务启动时，如果：
    
    innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances > innodb_buffer_pool_size 的值，那么 innodb_buffer_pool_chunk_size 的值会被服务器自动设置为 innodb_buffer_pool_size **÷** innodb_buffer_pool_instances 的值。
    
    例如：innodb_buffer_pool_chunk_size=256MB
    
                innodb_buffer_pool_instances=16
    

  则：innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances=4GB

如果我们设置 innodb_buffer_pool_size=2GB，因为 2GB < 4GB，则 innodb_buffer_pool_chunk_size 被修改为 128MB。