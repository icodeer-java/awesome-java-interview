
> 在 JDK8 之前，当我们采用多线程的方式向 HashMap 中插入元素的时候，会有一定的概率造成线程死锁。这个问题在面试中也是比较常见的，那么原因是什么呢？“面试宝典” 里面常常会给出如下极简的答案：“**在数据迁移过程中，因为会采用头插法，所以会造成多线程死锁。而 jDK8 之后（包含 8）则采用了尾插法，所以，可以有效的避免这个问题**”。那么，本篇小短文就带着大家来到 **JDK7 的源码**中去深入的寻找更完整的答案。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9lsVjh6LNUKuPVtueaAg2gzL0yhB9BXwYMPYvkoiaVByDjyGhA9Ts2kK7EG6TDeQR7Aosx8IytMVg/640?wx_fmt=png)

put 方法基本流程
----------

首先，判断 table 数组是否为空（即：{}），如果为空，则调用`inflateTable(threshold)`方法初始化一个默认长度为 16 的数组。源码如下所示：

```
if (table == EMPTY_TABLE) {
    inflateTable(threshold);
}

```

其次，如果 key 等于 null，则将其放入 table[0] 所在元素的链表中。源码如下所示：

```
if (key == null) {
    return putForNullKey(value);
}

```

第三，通过 key 进行 rehash 操作，计算出待插入到 table 数组中的位置 i，如果这个元素之前插入过，则更新 value 值，并将旧的 value 值返回出去。源码如下所示：

```
int hash = hash(key);
int i = indexFor(hash, table.length);
for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    Object k;
    if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
        V oldValue = e.value;
        e.value = value;
        e.recordAccess(this);
        return oldValue;
    }
}

```

最后，如果这个元素没插入过，则调用 addEntry 进行添加。源码如下所示：

```
addEntry(hash, key, value, i);

```

addEntry(hash, key, value, i);
------------------------------

在`addEntry`方法中，会涉及到 table 数组扩容 & 数据迁移操作。那么在这个场景下，我就可以看到多线程下如何会造成死锁。

相关的代码就在`addEntry`方法中的`resize(2 * table.length)`方法里，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9lsVjh6LNUKuPVtueaAg2gUMCYljMtcoeZw9Gia783wFAiaNJ1r9o2GJmsTEiaGmR9pPUC2JsWZosEA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9lsVjh6LNUKuPVtueaAg2gVibZa1EGINFX5Nz8Xe5c3vX73fS9dZXDAqLgfbpNFgSCv9VrUpLh5SA/640?wx_fmt=png)

数据迁移逻辑概述
--------

数据迁移真正逻辑就在`transfer(Entry[] newTable, boolean rehash)` 方法中，源码和注释如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9lsVjh6LNUKuPVtueaAg2gu0UpgF8pES0ib8jibKB1a6EpmdabbkzniaDXWslL80u0xHM3s3iaZY7W9Q/640?wx_fmt=png)

当然，这么看起来不是那么直观，下面我会以图示的方式演示如何数据迁移的。

首先，我们需要知道的知识点就是，JDK7 中采取的是**头插法**进行数据迁移，那么迁移后的新旧链表顺序其实就是**相反的**。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9lsVjh6LNUKuPVtueaAg2gjZngAcTYer9WSG0FftpqEKqmibYC2Fz67LkqfTcic0yCY7je8q96CJkQ/640?wx_fmt=png)

数据迁移详解
------

在上一节内容中，我们已经看到了`transfer`方法的源码，那么下面我们就根据源码的内容，演示每一步的数据迁移操作。迁移的场景就是原有数组下标 0 处有一条**链表** **A->B->C**，对其进行迁移。迁移详细步骤如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9lsVjh6LNUKuPVtueaAg2gpql0xPmkIibKbias9ibkF3nrO5G7hzQZeGxDibl4Naudejic6hDD6QVhwEw/640?wx_fmt=png)

从上图中，已经演示了如何通过`transfer`方法中关键的代码内容执行数据迁移了。

多线程死锁场景
-------

既然数据迁移的整个过程我们已经介绍过了，那么还是假设一个场景：有**线程 A** 和**线程 B** 这两条线程。同时对 HashMap 进行数据迁移操作。那么，线程 A 是正常执行的，线程 B 执行过程中慢了些。那么为何会产生死锁呢？详情请看下图：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9lsVjh6LNUKuPVtueaAg2gMIBUVzX09nv5qBORImZ4X9eqdqwI7nicZqr9gnoVuqYeEnA3nyN7O7g/640?wx_fmt=png)

总结
--

如上的代码是针对于 JDK7 的解析，从 JDK8 开始，HashMap 源码改进的内容还是蛮大的，从 **rehash** 的方式再到**加入****红黑树**再到**尾插法**等等。

所以，在面试过程中，面试官想要考察面试者是否有看源码的习惯或者能力时，都会考察变更前和变更后的区别。当然，本篇文章只是举了一个特定的例子，会造成死锁，由于 HashMap 不是线程安全的，所以在多线程场景下问题还是比较多的。

如果想要更加详细的了解非线程安全的 JDK8 版本的 **HashMap 源码解析**，请跳到这篇文章：[长文多图——HashMap 源码解析（包含红黑树）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247484480&idx=1&sn=b859cfac971636339658a3b988c45f7e&chksm=e91146bdde66cfab3fa9e7430c2b74750932aa320cc18a5b1d36072a95966a468262fbf76e8a&scene=21#wechat_redirect)。

如果想更详细的了解 JDK8 版本线程安全的 **ConcurrentHashMap 源****码解析**，请跳到这篇文章：[万字图文——ConcurrentHashMap 源码深度解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247484206&idx=1&sn=3def29ab1ab1b0efe0df53a3d683f268&chksm=e91141d3de66c8c58e10afc8c7e7ac8cc59feaf93da3162ad8e553af317555633efffb133125&scene=21#wechat_redirect)。