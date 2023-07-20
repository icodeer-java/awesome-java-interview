
    上周写了一篇 MySQL——锁（一），本来想继续（二）、（三) 这么的排下去，但是觉得这么写文章，内容还是太分散了。所以，这篇就算是完整版吧。文章里的内容算是一个读书笔记，关于锁的入门和面试应急应该还是有些用处的。下面是文章内容设计的目录：  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaKWDl5b83mSDU9b3KticHORJIMzpTp0tSQibSJWIW2wyCATaTwq9bXRLQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJia9d6uu0qdMYRdYXoiaRpCzvL41N3HBVbobqSgqvnD882ib60qrODcgVlw/640?wx_fmt=png)

一、解决并发事务带来的问题  

================

1.1> 写 - 写情况
------------

*   由于任何一种隔离级别都不允许脏写（写 - 写）的现象发生，所以，当**多个未提交事务**相继对**一条记录**进行改动的时候，就需要让它们**排队**执行。
    
*   这个排队的过程其实是通过为该记录**加锁**来实现的。这个锁本质上是一个**内存中的结构**。
    

*   写 - 写具体操作流程如下：
    

*   1> 一开始是没有**锁结构**与记录进行关联的，即：下图第一个图例所示。
    
*   2> 当一个事务 T1 想对这条记录进行改动时，会看看**内存中有没有与这条记录关联的锁结构**，如果没有，就会在内存中**生成一个锁结构**与这条记录相关联，即：下图第二个图例所示。我们把该场景称之为**获取锁成功** / **加锁成功**。
    
*   3> 此时又来了另一个事务 T2 要访问这条记录，发现这条记录已经有一个锁结构与之关联了，那么 T2 也会生成一个锁结构与这条记录关联，不过锁结构中的 **is_waiting 属性值为 true**，表示需要等待。即：下图第三个图例所示。我们把该场景称之为**获取锁失败** / **加锁失败**。
    
*   4> 事务 T1 提交之后，就会把它生成的锁结构释放掉，然后检测一下还有没有与该记录关联的锁结构。结果发现了事务 T2 还在等待获取锁，所以把事务 T2 对应的锁结构的 **is_waiting 属性设置为 false**，然后把该事务对应的线程唤醒，让 T2 继续执行。
    

*   整体流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibAnol8HUF5H7WPuFenoCCicEhymOicMLZkLsXYnHvWe9ibqticb2nRMtiaibAGtypF48ic9ccoib3icnB7o8g/640?wx_fmt=png)

1.2> 读 - 写或写 - 读情况  

---------------------

*   为了避免在 “读 - 写” 或“写 - 读”情况下避免脏读、不可重复读、幻读现象，有如下两种可选的解决方案：
    

*   1> 读操作使用多版本并发控制（MVCC），写操作进行加锁。
    
*   2> 读、写操作都采用加锁的方式。
    

*   MySQL 与 SQL 标准不同的一点就是，MySQL 在 REPEATABLE READ 隔离级别下很大程度地避免了幻读现象。
    

1.3> 一致性读  

------------

*   一致性读 / 一致性无锁读 / 快照读
    

     定义：事务**利用 MVCC** 进行的读取操作。

*   所有**普通的 SELECT 语句**在 READ COMMITTED 或 REPEATABLE READ 隔离级别下都算是一致性读。比如：
    

```
select * from student;
select * from student s left join address a on s.addr_id = a.id;

```

*   一致性读并**不会对表中的任何记录进行加锁**操作，其他事务可以自由地对表中的记录进行改动。
    

1.4> 锁定读  

-----------

在使用加锁的方式来解决读写问题的时候，由于既要允许读 - 读情况不受影响，又要使写 - 写或读 - 写情况中的操作互相阻塞，所以 MySQL 给锁分为以下两类：

*   共享锁（S 锁）
    

 **Shared Lock**：在事务要**读取**一条记录时，需要先获取该记录的 **S 锁**。

*   独占锁（X 锁）
    

 **Exclusive Lock**：在事务要**修改**一条记录时，需要先获取该记录的 **X 锁**。

*   S 锁和 X 锁的兼容关系
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibAnol8HUF5H7WPuFenoCCictMcyslmC9YAIs0t2nx1FOq8MOk4Z5nwuWFa1GeyBYjicwN85ib6fLIkQ/640?wx_fmt=png)

【举例解释】

*   情况 1：事务 T1 首先获取了一条记录的 **S 锁**
    

如果事务 T2 也要获得这条记录的 S 锁，那么此时，T2 是可以获得这条记录的 S 锁。如果事务 T2 要获得这条记录的 X 锁，那么操作会被阻塞，直到事务 T1 提交之后将 S 锁释放掉为止。

*   情况 2：事务 T1 首先获取了一条记录的 **X 锁**
    

那么无论事务 T2 要获得这条记录的 S 锁还是 X 锁，T2 都会被阻塞，直到事务 T1 提交之后将 X 锁释放掉为止。

*   锁定读的语句
    

        对读取的记录**加 S 锁**

 **SELECT ...** **LOCK IN SHARE MODE;**

        对读取的记录**加 X 锁**

 **SELECT ...** **FOR UPDATE;**

1.5> 写操作  

-----------

*   DELETE
    

      先在 B + 树中**定位**到这条记录的位置，然后获取这条记录的 **X 锁**，最后再执行 **delete mark** 操作。

*   INSERT
    

      一般情况下，新插入的一条记录受**隐式锁**保护，不需要在内存中为其生成对应的锁结构。

*   UPDATE
    

     分为如下 3 种情况：

*   **未修改主键**并且被更新的列在修改前后所占用的**存储空间未发生变化**
    

先在 B + 树中**定位**到这条记录的位置，然后获取这条记录的 **X 锁**，最后在原记录的位置进行修改操作。

*   **未修改主键**并且被更新的列在修改前后所占用的**存储空间发生变化**
    

先在 B + 树中**定位**到这条记录的位置，然后获取这条记录的 **X 锁**，之后将原记录彻底删除掉（即：把记录彻底移入垃圾链表），最后再插入一条新记录。

*   **修改主键**
    

相当于在原记录上执行 **DELETE 操作**之后再来一次 **INSERT 操作**。加锁操作就需要按照 DELETE 和 INSERT 的规则进行了。

二、多粒度锁  

=========

*   上面提到的锁都是针对记录的，可以称为**行级锁** / **行锁**；对一条记录加行锁，影响的只是该行记录而已，所以行锁的粒度比较细。
    
*   如果一个事务在表级别进行加锁，就称为**表级锁** / **表锁**。它会影响表中的所有数据，锁的粒度比较粗。
    

*   **表锁**也可以分为**共享锁（S 锁）**和**独占锁（X 锁）**
    

*   情况 1：一个事务给表加了 S 锁
    

    其他事务**可以**继续获得该**表** / 该表中的某些**记录**的 **S 锁**。

    其他事务**不可以**继续获得该**表** / 该表中的某些**记录**的 **X 锁**。

*   情况 2：一个事务给表加了 X 锁
    

    其他事务**不可以**继续获得该**表** / 该表中的某些**记录**的 **X 锁**或 **S 锁**。

*   每当要对表上 S 锁的时，需要表中的记录和表没有 X 锁；当要对表上 X 锁的时候，需要表中的记录和表即没有 X 锁也没有 S 锁。那么，表上的锁比较好判断，_记录上的锁怎么判断呢？_总不能一行一行的来判断是不是有 X 锁或者 S 锁把。那么，为了解决这个问题，InnoDB 提出了**意向锁**的概念。即：
    

*   意向共享锁（**IS 锁**）
    

ntention Shared Lock：当事务准备在某条**记录**上加 S 锁时，首先需要在**表级别**加一个 IS 锁。

*   意向独占锁（**IX 锁**）
    

Intention Exclusive Lock：当事务准备在某条**记录**上加 X 锁时，首先需要在**表级别**加一个 IX 锁。

*   **IS 锁**和 **IX 锁**是**表级锁**，它们的提出仅仅**为了在之后加表级别的 S 锁和 X 锁时可以快速判断表中的记录是否被上锁**，以避免用遍历的方式来查看表中有没有上锁的记录；也就是说，其实 **IS 锁和 IX 锁是兼容的**，**IX 锁和 IX 锁是兼容的**，**IS 和 IS 也是兼容的**。兼容性关系如下所示：
    

*   表级别的锁的兼容性
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibAnol8HUF5H7WPuFenoCCicd4BqicFtibnJyI1oQCGibdehuhhVBFAFyemW7uy7gxHm5RpuJV4LxkxvQ/640?wx_fmt=png)

三、MySQL 中的行锁和表锁  

==================

3.1> 其他存储引擎中的锁
--------------

*   对于 MyISAM、MEMORY、MERGE 这些存储引擎来说，它们**只支持表级锁**，而且这些存储引擎并**不支持事务**。
    

3.2> InnoDB 存储引擎中的锁
-------------------

*   InnoDB 存储引擎**既支持表级锁也支持行级锁**
    

### 3.2.1>  InnoDB 中的表级锁  

#### a> S 锁、X 锁

*   InnoDB 存储引擎提供的**表级** S 锁或者 X 锁相当 “鸡肋”，在对某个表执行 SELECT、INSERT、DELETE、UPDATE 语句时，**InnoDB 存储引擎是不会为这个表添加表级别的 S 锁或者 X 锁的**，只会在一些特殊情况下（比如系统崩溃恢复时）用到。
    
*   在对某个表执行 DDL 语句时，其他事务在对这个表并发执行 DML 语句时，会发生阻塞；反之亦然。这个过程其实是通过在 server 层使用一种称为**元数据锁**（Metadata Lock，MDL）的东西来实现的，也不会使用 S 锁和 X 锁。
    

*   DDL 语句在执行时会**隐式提交**当前会话中的事务
    

     原因：DDL 语句的执行一般都会在若干个特殊事务中完成。在开启这些特殊事务前，需要将当前会话中的事务提交掉。

*   虽然表级 S 锁或 X 锁相当鸡肋，不过我们还是可以手动获取一下的，比如在系统变量 **autocommit=0**、**innodb_table_lock=1** 时，可以按照下面来写语句：
    

*   **LOCK_TABLES t READ**
    

对表 t 加表级别的 S 锁。

*   **LOCK_TABLES t WRITE**
    

对表 t 加表级别的 X 锁。

#### b> IS 锁、IX 锁

*   IS 锁和 IX 锁是表级锁，它们的提出仅仅**为了在之后加表级别的 S 锁和 X 锁时可以快速判断表中的记录是否被上锁**，以避免用遍历的方式来查看表中有没有上锁的记录；
    

#### c> AUTO-INC 锁

*   系统自动给 AUTO_INCREMENT 修饰的列进行递增赋值的实现方式主要有下面两个：
    

*   AUTO-INC 锁
    

执行插入语句时就加一个表级别的 AUTO-INC 锁，然后为每条待插入记录的 AUTO_INCREMENT 修饰的列分配递增的值。

在该语句执行结束后，再把 AUTO-INC 锁释放掉。这样一来，一个事务在持有 AUTO-INC 锁的过程中，其他事务的插入语句都要被阻塞。

AUTO-INC 锁的作用范围只是**单个插入**语句，在插入语句执行完成后，这个锁就被释放了。

*   轻量级锁
    

在通过 AUTO_INCREMENT 获得修饰的列的值时获取这个轻量级锁，就把该轻量级锁释放掉，而不需要等到整个插入语句执行完后才释放锁。

*   innodb_autonic_lock_mode 系统变量，用来控制到底使用上述两种方式中的哪一种
    

*   innodb_autonic_lock_mode=0
    

一律采用 AUTO_INC 锁。

*   innodb_autonic_lock_mode=2
    

一律采用轻量级锁。

*   innodb_autonic_lock_mode=1
    

两种方式混着来，即：插入记录的数量**确定时**采用轻量级锁，**不确定时**采用 AUTO-INC 锁。

其中，不确定插入记录数量的情况。

例如：INSERT...SELECT、REPLACE...SELECT、LOAD DATA。

### 3.2.2>  InnoDB 中的行级锁  

*   行级锁，也称为记录锁，顾名思义就是在记录上加锁。
    
*   不过行锁也根据不同的类型分为了多种。也就是说，即使对同一条记录加行锁，如果记录的类型不同，起到的功效也是不同的。
    

#### a> Record Lock

*   官方名称：LOCK_REC_NOT_GAP
    

*   也称为**记录锁**：也就是仅仅负责把一条记录锁上的锁。
    
*   记录锁也分为：**S 型**记录锁和 **X 型**记录锁。
    

*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibAnol8HUF5H7WPuFenoCCictCAXzU4dxmb2JU6ibrxImQd56f8KyJPT4fFI5Pse5ia6tpxvW22Wl1CQ/640?wx_fmt=png)

#### b> Gap Lock

*   官方名称：LOCK_GAP
    
*   也称为 **gap 锁**：锁住了指定的记录以及记录前面的间隙，防止其间插入新记录。
    
*   gap 锁的提出仅仅是为了**防止插入幻象记录**（即：幻读现象）而提出的。                                                                                                                  
    
*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibAnol8HUF5H7WPuFenoCCicwIjpryVLz9FQ3NhAW7y3flCrw5zgdzWicZjKojctozyGZdnhXDbabxg/640?wx_fmt=png)【解释】

      意味着不允许别的事务在 no 值为 5 的**记录前面的间隙**插入新记录，即：no 列的值在区间（3，5）的新记录是不允许立即插入的，当 gap 锁释放才可以插入。

*   如何锁定 no 值为 5 之后的记录呢？
    

      为 Supremum 记录加一个 gap 锁，则可以阻止其他事务插入 no 值在区间（5, +∞）的新纪录。

#### c> Next-Key Lock

*   官方名称：LOCK_ORDINARY
    
*   也称为 **next-key 锁**：本质就是一个**记录锁** +**gap 锁**的合体。它既能保护该条记录，又能阻止别的事务将新纪录插入到被保护记录前面的间隙中。
    

*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibAnol8HUF5H7WPuFenoCCicq8giaefCUuykHxlavicXc8mal27TPJjvvCtTrjgEM6UYWUro2EeYyCDA/640?wx_fmt=png)

#### d> Insert Intention Lock

*   官方名称：LOCK_INSERT_INTENTION
    
*   也称为**插入意向锁**：事务在等待时也需要在内存中生成一个锁结构，表明有事务想在某个间隙中插入新记录，但是现在处于等待状态。
    

*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibAnol8HUF5H7WPuFenoCCicrlyYFGcXnJS5ia1zYIy3epVWrKwyhJsjwvN4mXkVXfR2wKMPed1cIew/640?wx_fmt=png)【解释】

 **type 属性**，用来表明该锁的类型。

    由于 T1 持有 no=9 的 gap 锁（即：no 等于 5~9 之间不能插入记录），所以 T2 和 T3 分别想插入 no=6 和 no=7 的两条记录时会生成插入意向锁的锁结构并且处于等待状态。

    本质上就是把插入意向锁对应锁结构的 is_waiting 置为 true。

    当 T1 提交后会把 gap 锁释放掉，这时候，**T2 和 T3 之间也并不会相互阻塞**，他们可以同时获取到 number 值为 9 的插入意向锁，然后执行插入操作。

*   事实上**插入意向锁并不会阻止别的事务继续获取该记录上任何类型的锁**，就是这么鸡肋。
    

#### d> 隐式锁

*   一般情况下，执行 INSERT 语句是不需要在内存中生成锁结构的。
    
*   但是也会有例外，比方说：一个事务首先插入了一条记录 (此时并没有与该记录关联的锁结构），然后另一个事务执行如下操作：
    

*   立即使用 SELECT... LOCK IN SHARE MODE 语句读取这条记录 （也就是要获取这条记录的 S 锁），或者使用 SELECT ... FOR UPDATE 语句读取这条记录（也就是要获取这条记录的 X 锁），该咋办？如果允许这种情况的发生，那么可能出现**脏读现象**。
    
*   立即修改这条记录（也就是要获取这条记录的 X 锁），该咋办？如果允许这种情况的发生，那么可能出现**脏写现象**。
    

*   解决办法——使用事务 id，我们把聚簇索引和二级索引中的记录分开看一下
    

*   场景 1：对于**聚簇索引**
    

有一个 **trx_id** 隐藏列，该隐藏列记录着最后改动该记录的事务 id。在当前事务中新插入一条聚簇索引记录后，该记录的 trx_id 隐藏列代表的就是当前事务的事务 id。如果其他事务此时想对该记录添加 S 锁或者 X 锁，首先会看一下**该记录的 trx_id 隐藏列代表的事务是否是当前的活跃事务**。如果不是的话就可以正常读取：如果是的话，那么就帮助当前事务创建一个 X 锁的锁结构，该锁结构的 is_waiting 属性为 false：然后为自己也创建一个锁结构，该锁结构的 is_ waiting 属性为 true，之后自己进入等待状态。

*   场景 2：对于**二级索引**
    

本身并没有 trx_id 隐藏列，但是在二级索引页面的 Page Header 部分有一个 **PAGE_MAX_TRX_ID** 属性，该属性代表对该页面做改动的最大的事务 id。如果 **PAGE_MAX_TRX_ID 属性值小于当前最小的活跃事务 id**，那就说明对该页面做修改的事务都己经提交了，否则就需要在页面中定位到对应的二级索引记录，然后通过回表操作找到它对应的聚筷索引记录，然后再重复情景 1 的做法。

*   综上所述，隐式锁起到了**延迟生成锁结构**的用处。即：一般情况不生成隐式锁，如果发生上述冲突的锁操作，则采用隐式锁结构来保护记录。
    

3.3> InnoDB 锁的内存结构  

---------------------

*   前文已经介绍过，对一条记录加锁的本质就是在内存中创建一个锁结构跟这条记录相关联，_那么如果我们在操作一个事务的时候，对应多条记录的时候，是不是要针对多条记录生成多个内存的锁结构呢？_比如我们执行 select * from tb_user for update 的时候，tb_user 表中如果存在 1 万条数，那么难道要生成 1 万个内存的锁结构吗？那当然不会是这样的。其实，如果符合以下几个条件，那么这些记录的锁就可以放到一个内存中的锁结构里了，条件如下所示：
    

*   1> 加锁操作时在同一个事务中。
    
*   2> 需要被加锁的记录在同一个页中。
    
*   3> 需要加锁的类型是一致的。
    
*   4> 锁的等待状态是一致的。
    

*   那么，我们说了这么多次的锁结构，它到底是怎么组成的呢？它其实主要是由 6 部分组成的。分别为：锁所在的事务信息、索引信息、表锁或行锁信息、type_mode、其他信息和与 heap_no 对应的比特位。如下图所示：
    

    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJialluvkkccXHJSXNV5n1P49AkVXkccwRN1icljCqN9VrdI1SibwJYycZSQ/640?wx_fmt=png)

下面我们就针对上面的锁结构图进行解释：

*   **锁所在的事务信息**
    

一个锁结构对应一个事务，那么这里就存储着锁对应的事务信息。它其实只是一个指针，可以通过它获取到内存中关于该事务的更多信息，比如：事务 id 是多少。

*   **索引信息**
    

对于行级锁来说，这里记录的就是加锁的记录属于哪个索引。

*   **表锁 / 行锁信息**
    

对于表锁，主要是来记录对哪张表进行的加锁操作以及其他的信息。

对于行锁，内容包括 3 部分：

*   Space ID：记录所在的表空间 ID。
    
*   Page Number：记录所在的页号。
    

*   n_bits：一条记录对应一个 bit，那么当我们对多条记录进行加锁操作的时候，就会对应多个 bit，那么这个值就是用来记录有多少个 bit 的，而具体哪条记录对应哪个 bit，是在【_与 heap_no 对应的比特位_】这块内容中有 mapping 映射的。但是，大家需要注意的是，并不是有多少条记录 n_bits 的值就是多少。为了之后在页面中插入新记录的时候也不至于重新分配锁结构，n_bits 的值一般都比页面中的记录条数多一些。
    

*   **type_mode**
    

它是由 32 个 bit 组成的，分别为：lock_mode、lock_type、lock_wait 和 rec_lock_type，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJianE3IcFNbG4x4Y3pBV455eCGPr2kh008iadlayMGXMebyk9DrZuwlryA/640?wx_fmt=png)

*   **其他信息**
    

为了更好的管理系统运行过程中生成的各种锁结构，而设计了各种哈希表和链表。

*   **与 heap_no 对应的比特位**
    

如果是行级锁，会通过这部分的比特位来对应 n_bit 属性的值。在每条记录的头信息中保存一个叫做 heap_no 的属性，它是用来表示记录在堆中的相对位置的。即：Infimum 的 heap_no 为 0，Supermum 的 heap_no 为 1，然后插入记录的 heap_no 依次类推，每次加一。那么，这个所谓的 “与 heap_no 对应的比特位” 就是一个 bit 与 heap_no 的对应关系。以 n_bit=16 为例，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiav6FCgCwvhrgrsFKPRGs0VL02RyhqcVIBp1k8H1oric6OFFdjicUfKsvA/640?wx_fmt=png)

*   我们介绍完了锁的结构，因为涉及的都是理论，所以还是摆脱不了不直观的问题。那么我们就以一个例子来把上面的理论知识串起来吧。
    

*   我们假设要开启一个事务 T1，往 tb_user 表（已存在 5 条记录）中表空间为 67，页号为 3 的页面上，插入一个 number=15 的记录（number 是主键），并位这个记录加 S 锁，那么我们分析一下它所生成的行级锁结构是怎么样的？
    

*   1> 由于开启的是事务 T1，所以【锁所在的事务信息】指的就是 T1 这个事务。
    
*   2> 由于要直接对 number 这个聚簇索引加锁，所以【索引信息】值的就是 PRIMARY 索引。
    
*   3> 由于是行级锁，所以在【表锁 / 行锁信息】中【Space ID】等于 67，【Page Number】等于 3，【n_bits】等于 72
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJialaMbMmRLemn0HS9kGEdYy4sBnUTfgNJdtOygnzAcTkz7Jt97mEC1Uw/640?wx_fmt=png)

其中，n_recs 包含伪记录（Infimum 和 Supermum）共 2 条记录和正常记录 5 条记录，共 7 条记录。那么根据上面的公式计算，得出如下结果 n_bits=(1+((7+64)/ 8))*8=72。

*   4>【type_mode】由四部分组成，其中 lock_mode=LOCK_S=2，lock_type=LOCK_REC=32，lock_wait=NOT_WAITING=0，rec_lock_type=LOCK_REC_NOT_GAP=1024; 那么组合在一起就是 type_mode=2|32|0|1024=1058。
    
*   5>【其他信息】不做重点讨论。
    
*   6> 【与 heap_no 对应的比特位】，因为之前已经存在 5 条记录了，所以 number=15 对应的 no_heap=7，它对应的 bit 位，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiasfNicGQccCOOjfZbraspElcWtC6H0ic38gGILYcctiaGfQvzqsTJicdHDQ/640?wx_fmt=png)

那么，综上所述：锁结构为如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaviardQdHjiao8eNSUIR3JCE5f5Vl5cYsRW5UGzOHbJ1g5ibYsUB4cW28Q/640?wx_fmt=png)

四、语句加锁分析
========

*   上面我们已经了解了加锁方面的理论知识了，那么下面我们就以示例的方式，加深对加锁的理解，在介绍示例之前，我们先看一下 tb_user 表的表结构和表中存在的数据：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiatic8oNqdjWATibEjyq5rH1kVWoTSmhKWvKSMeibhE6N1ZM2Q6Xg1oVyVA/640?wx_fmt=png)

【解释】

*   在 tb_user 表中，有三个字段，分别为学号（number），学生姓名（name）和学生年龄（age）这三个字段。那么其中 number 字段为聚簇索引字段，name 字段为二级索引字段。
    
*   那么针对与语句加锁分析，我们根据以下 4 类进行分析，分别为：普通的 SELECT 语句，锁定读的语句，半一致性读的语句和 INSERT 语句。那么下面我们就针对这四类语句进行加锁分析。
    

4.1> 普通的 SELECT 语句  

---------------------

*   普通的 select 语句其实针对于不同的隔离级别，会有不同的处理方式。如下表所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaseVp8gRutDe7k15x1AZgXAAff5u79vgCeNPK1ia66icGL23Itia74R4sw/640?wx_fmt=png)

*   _那么当事务的隔离级别为 REPEATABLE READ 时，为什么说是很大程度上避免了幻读呢？那什么情况下，依然会出现幻读的情况？_那我们针对这个问题，去演示一下出现幻读的场景。
    

*   首先我们先查询 tb_user 表，发现里面只有如下 3 条数据
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaR4FtWibqvHBRcHMphCENrFPriblqTnibuPvOpvalqETFPyGj3dWgtmRpQ/640?wx_fmt=png)
    
*   然后开启 T1 事务，查询 id=4 的数据，发现什么也没有查询到。
    

        ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaVVAFyubGtNTCECIX3zCdYsrib0HicmVT6LAQRM9fYJxXwaRZALlSDAJA/640?wx_fmt=png)

*   此时，我们开启 T2 的事务，并插入一个 id 等于 4 的数据，然后提交事务
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaTcJzcng0ylHmlqPunEcKywcfgl5ibgG3IcCfyadFnkt827RImRhCj1g/640?wx_fmt=png)  
    
*   那么我们再把视角转移到 T1 的事务，我们再次查询 id=4 的这条记录，发现查询不到，那么当我们针对 id=4 的记录执行了 update 操作之后，再去查询 id=4 的数据，发现竟然查询到了。那么就出现了幻读的现象。
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiazPp8TxMXcicyqibWCC1W3coQXlO13BfPibFOaSxtuicUGcsysiaAEUmQ1JQ/640?wx_fmt=png)  
    
*   _那这是为什么呢？_原因就是当 T2 执行完插入 id=4 且提交了事务之后，id=4 的这条记录上的 trx_id=T2，那么当 T1 针对 id=4 的这条记录执行了 update 操作后，这条记录上的 trx_id 就变为了 T1，那么再查询的时候，就可以把 id=4 的记录查询出来了。此时的 MVCC 就无法完全禁止幻读了。但是，还是可以很大程度上避免幻读。
    

4.2> 锁定读的语句  

--------------

### 4.2.1> 流程概述

*   针对锁定读的语句，其实可以归类为以下四种语句：
    

*   语句 1：**SELECT ... LOCK IN SHARE MODE;**
    
*   语句 2：**SELECT ... FOR UPDATE;**
    
*   语句 3：**UPDATE ...**
    
*   语句 4：**DELETE ...**
    

【解释】因为语句 3 和语句 4 在操作 update 和 delete 之前，都要隐式的去查找相应的数据，所以也可以认为是一种锁定读。

锁定读的过程如下所示：

*   步骤 1：快速在 B + 树中定位到该扫描区间（即 select 的查询区间）中的第一条记录，把该记录作为当前记录。
    
*   步骤 2：根据不同的隔离级别，为当前记录加不同类型的锁
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaxqia60iacmMVovK7pxBRdhBianT5YJ4zyFJfxsHyYpwYic82OsLrC6Pjbw/640?wx_fmt=png)
    
*   步骤 3：判断**索引条件下推**（ICP：Index Condition Pushdown）的条件是否成立。如果符合索引条件下推，则执行步骤 4，否则，则获取记录所在的单向链表的下一条记录，并做为新的记录，跳到步骤 2 继续执行。另外，本步骤还会判断当前记录是否符合扫描区间的边界条件，如果超出了扫描边界，则跳过步骤 4 和步骤 5，直接向 server 层返回查询完毕。注意，**步骤 3 不会释放锁**。
    
*   ICP：只适用于二级索引，且只适用于 select 语句。它是用来把查询中与被使用索引有关的搜索条件**下推到存储引擎中**去判断，而不是返回到 server 层再去判断。ICP 只是为了减少回表次数，也就是减少读取完整的聚簇索引记录的次数，从而减少 I/O 操作。
    
*   步骤 4：执行回表操作，获取到对应的聚簇索引记录，并加锁。
    
*   步骤 5：判断边界条件是否成立，如果还在边界内，则执行步骤 6，否则，如果隔离级别为 READ UNCOMMITTED 或 READ COMMITTED，则要释放掉加在该记录上面的锁，如果隔离级别为 REPEATABLE READ 或 SERIALIZABLE，则不去释放记录上面的锁。
    
*   步骤 6：server 层判断其余搜索条件是否成立。如果不满足搜索条件，也要像步骤 5 中描述的那样，根据不同的隔离级别来确定对当前记录是否加锁 or 释放锁。
    
*   步骤 7：获取当前记录所在单向链表的下一条记录，并跳到步骤 2。
    

### 4.2.2> SELECT ... LOCK IN SHARE MODE 示例  

那么针对上面的步骤描述，我们通过几个示例的演示，加深一下上面步骤的理解。

*   【示例一】针对聚簇索引 number 作为搜索条件，隔离级别为 READ UNCOMMITTED 或 READ COMMITTED，执行 select * from tb_user where number >2 AND number <=7 AND age=25 LOCK IN SHARE MODE;
    

*   步骤 1：首先扫描区间为 (2,7] 中的第一条记录，即：number=3。
    
*   步骤 2：为 number=3 的记录加 S 行的记录锁。
    
*   步骤 3：由于查询条件为聚簇索引，所以不符合 ICP。
    
*   步骤 4：由于查询条件为聚簇索引，所以不需要回表。
    
*   步骤 5：扫描区间为 (2,7]，当前区间为 number=3，符合扫描区间
    
*   步骤 6：server 层判断 number=3 记录上面的其他条件，它的 age=11，不满足查询条件，所以释放掉该记录上的锁。
    
*   步骤 7：获取 number=3 记录所在单向链表的下一条记录，即：number=5，继续执行步骤 2 的操作，下面针对其他 number 的操作就不在赘述了。最终加锁结果如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaxSF920xZMkKkC6l6bwgRGkNEtpGkYXa3EnMrbzKeNCXJbPia6n2ZxGg/640?wx_fmt=png)

*   【示例二】针对聚簇索引 number 作为搜索条件，隔离级别为 REPEATABLE READ 或 SERIALIZABLE，执行 select * from tb_user where number >2 AND number <=7 AND age=25 LOCK IN SHARE MODE;
    

*   示例二与示例一的区别只在于隔离级别上。那么从上面我们介绍步骤原理的时候，也说过，如果是 READ COMMITTED 或 SERIALIZABLE 的隔离级别的话，如果不满足条件是不会解锁的。所以，我们具体步骤就不再赘述了，可以参照实例一中的具体步骤，我们就来看一下加锁情况变成了怎样？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaJqL9KS7y1JkN9fs2yBnhnt90sa6bIJBXOMkALJDuYzcn6V8SCnicYJw/640?wx_fmt=png)

*   【示例三】针对二级索引 name 作为搜索条件，隔离级别为 READ UNCOMMITTED 或 READ COMMITTED，执行 select * from tb_user  FORCE INDEX(idx_name) where name >= 'james' AND name <='tom' AND age=20 LOCK IN SHARE MODE;
    

*   步骤 1：首先扫描区间为 ['james','tom'] 中的第一条记录，即：name='james'。
    
*   步骤 2：为 name='james'的二级索引记录加 S 型的记录锁。
    
*   步骤 3：由于查询条件为二级索引，所以符合 ICP。
    
*   步骤 4：执行回表操作，找到相应的聚簇索引记录，也就是 number=9，然后为该聚簇索引记录加一个 S 型的记录锁。
    
*   步骤 5：扫描区间为 ['james','tom']，当前区间为 name='james'，符合扫描区间
    
*   步骤 6：server 层判断 number=9 记录上面的其他条件，它的 age=11，不满足查询条件，所以释放掉该记录在二级索引和聚簇索引上的锁。
    
*   步骤 7：获取 name='james'记录所在单向链表的下一条记录，即：name='john'，继续执行步骤 2 的操作，下面针对其他 name 的操作就不在赘述了。最终加锁结果如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaUfujGXxeKOcAYr3ichSNyxV1n1Qg56usdcBo9DI02ndx9x84B0gakzQ/640?wx_fmt=png)

*   【示例四】针对二级索引 name 作为搜索条件，隔离级别为 REPEATABLE READ 或 SERIALIZABLE，执行 select * from tb_user  FORCE INDEX(idx_name) where name >= 'james' AND name <='tom' AND age=20 LOCK IN SHARE MODE;
    

*   示例四与示例三的区别只在于隔离级别上。那么从上面我们介绍步骤原理的时候，也说过，如果是 READ COMMITTED 或 SERIALIZABLE 的隔离级别的话，如果不满足条件是不会解锁的。所以，我们具体步骤就不再赘述了，可以参照实例三中的具体步骤，我们就来看一下加锁情况变成了怎样？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiankT7SYNCzib79JlX7I2lsLYvTXPPw6pyowyz3DRxWxneFrA5dYjib2Pg/640?wx_fmt=png)

### 4.2.3> SELECT ... FOR UPDATE 示例  

*   SELECT ... FOR UPDATE 语句的加锁过程与 SELECT ... LOCK IN SHARE MODE 语句类似，区别是为记录加 X 锁。
    

### 4.2.4> UPDATE ... 示例  

*   加锁方式与 SELECT ... FOR UPDATE 语句加锁类似。只不过如果更新了二级索引列，那么所有被更新的二级索引记录在更新之前都需要加 X 型记录锁。
    
*   【示例一】隔离级别为 READ UNCOMMITTED 或 READ COMMITTED，执行 update tb_user set name = 'unknown' where number >2 AND number <=7 AND age<25;
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiatJVJ5rcyTG2SfZLYTOvftBWMesYiciaMJsvyCghv2zS31S01LNL39hxA/640?wx_fmt=png)

【解释】

*   由于更新了 name 列，而 name 列又是一个索引列，所以在更新前也需要为 idx_name 二级索引中对应的记录加锁。
    

*   【示例二】隔离级别为 REPEATABLE READ 或 SERIALIZABLE，执行 update tb_user set name = 'unknown' where number >2 AND number <=7 AND age<25;
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaLA7ic1ynIULXkPmUnckFQ2M8b9byeAUWhHjNR2OD5ggs9IWPQiaK9icwA/640?wx_fmt=png)

### 4.2.5> DELETE ... 示例  

*   与 UPDATE 的处理方式相同，当表中包含二级索引，那么二级索引记录在被删除之前都需要加 X 型记录锁。
    

### 4.2.6> 补充说明  

*   对于 UPDATE 和 DELETE 语句来说，在对被更新或者被删除的二级索引记录加锁的时候，实际上加的是**隐式锁**，但是效果与 X 型记录锁一样。
    
*   对于隔离级别为 READ UNCOMMITTED 和 READ COMMITTED 的情况，采用的是一种称为**半一致读**的方式来执行 UPDATE 语句。
    

#### a> 二级索引精准匹配加锁流程  

*   当隔离级别为 **READ UNCOMMITTED 和 READ COMMITTED** 的情况，如果匹配的模式为**精准匹配**，那么将不会为扫描区间后面的下一条记录加锁。比如我们执行 select * from tb_use where name='muse' for update。那么加锁情况如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaKSgyrNqlf611nfAdxlalN9C8hmkciarf8IRFJ9tT5XStr9R7ibErDGtw/640?wx_fmt=png)

*   当隔离级别为 **REPEATABLE READ 或 SERIALIZABLE** 的情况，如果匹配的模式为**精准匹配**，那么会为扫描区间后面的下一条记录加 **gap 锁**。比如我们执行 select * from tb_use where name='muse' for update。那么加锁情况如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaMFib9KLeLIF8NEJoL4pQ6icvCFtRy6yHWCgstBd3F11t4JWvkiaSGgjQA/640?wx_fmt=png)

#### b> 二级索引找不到记录  

*   当扫描区间中**没有记录**，且为**精确查找**，隔离级别为 **REPEATABLE READ 或 SERIALIZABLE**，那么也要为扫描区间后面的下一条记录加一个 **gap 锁**。比如执行：select * from tb_user where name='moon' FOR UPDATE。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJia2IW2NzP1dXYQsTk5PrmgQ1cfd15LnFhozyDJC2wx8CL5HGXcKyUib7A/640?wx_fmt=png)

*   当扫描区间中**没有记录**，且**不是精确查找**，隔离级别为 **REPEATABLE READ 或 SERIALIZABLE**，那么也要为扫描区间后面的下一条记录加一个 **next-key 锁**。比如执行：select * from tb_user where name>'m' and name<'t' FOR UPDATE。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiavhk3VIIQbCLKTYEPaiaBw8sBdiapyObkRDNC9rHxvNw88ia1PHO3YB37w/640?wx_fmt=png)

#### c> 左闭区间加锁  

*   当隔离级别为 **REPEATABLE READ 或 SERIALIZABLE**，使用聚簇索引，并且扫描区间为左闭区间，如果定位到的第一个聚簇索引记录的 number 值正好与扫描区间中最小的值相同，那么会为该聚簇索引记录加 X 类型的记录锁。例如：select * from tb_user where number>=3 FOR UPDATE；加锁情况如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaJz5cMicJsNlpTxkwrdxY03AehAIW7UDV4rrYXwDjmLroCKbLeicbVTcQ/640?wx_fmt=png)

#### d> 自右向左扫描加锁  

*   当隔离级别为 **REPEATABLE READ 或 SERIALIZABLE**，从右向左的顺序扫描记录，会给匹配到的第一条记录的下一条记录加 gap 锁。例如：select * from tb_user where name >='john' and name <=tom and age=20 order by name DESC FOR UPDATE;
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaAhdjCtmPypfcF57c5wmn5HcuZP3xD0zO4CNoG4M3rIJeF9RicIncNaw/640?wx_fmt=png)

【解释】

*   在 tony 记录加 gap 锁，目的是为了防止有 name='tom'的记录插入。
    

4.3> 半一致性读的语句  

----------------

*   当隔离级别为 **READ UNCOMMITTED** 或 **READ COMMITTED** 且执行 **UPDATE** 语句时，将会使用半一致读。
    
*   _那么什么是半一致读呢？_
    
    就是当 UPDATE 语句读取到已经被其他事务加了 **X 锁**的记录时，InnoDB 会将该记录的**最新提交版本**读出来，然后判断该版本是否与 UPDATE 语句中的搜索条件相匹配。如果不匹配，则不对该记录加锁，从而跳到下一条记录；如果匹配，则再次读取该记录并对其进行加锁。这样做的目的就是让 UPDATE 语句尽量少被别的语句阻塞。
    
    比如 tb_user(number, name, age) 表中有三条记录，分别为 (1, 'a', 10),(2, 'b', 20),(3, 'c', 30)。那么 id=1 被 T1 事务锁住了，当我们在事务 T2 中执行 update ... where number>=1 and number<=3 and age > 10 的时候，会先判断被锁住的这个 id=1 的记录 age 是不是大于 10，我们发现这条记录的 age 等于 10，所以不满足 update 的 where 条件，所以就无需等待，继续去更新 id 为 2 和 3。
    

*   T1 开启事务，对 id=2 的记录加 X 型记录锁。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaSfb05SoJHhRpEcsz8saicVOt5Qibibness3AZ3t86Xe7K54mSUAicUWOnA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaRur0fZ19HF0ydCBDnSJshGQkjBFSIkK989TyZgQGtBapZ6iaor8UY6A/640?wx_fmt=png)

*   T2 开启事务，执行 update tb_user set name='bob1' where id >= 2 and id <=3 and name<>'bob'; 结果发现，update 操作并未被阻塞。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaafGRYCW4s48afK0LAxNY0icGkcISHA0CmZiclh7XBW9fCzYAQxkIQIibQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiavXpejpibZuTX6dg7drTVtD23uwoict6mByXh6UsicDia3F2ic6BhCxibR4Nw/640?wx_fmt=png)

*   那么我们把 T1 和 T2 的隔离级别都修改为 REPEATABLE-READ
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaI1Gb0knaE5ZT3DRxfdLETJfoWhHHX5Lll2uklW6RkhoibP5tlBwM8WA/640?wx_fmt=png)

*   此时我们再在 T1 事务中执行
    
    select * from tb_user where id=2 for update;
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJia0jOmMcuRcibCGj5HF3waB8HmG4aDBwEUjibibWkNzquQib1wfZDpybxv7Q/640?wx_fmt=png)

*   开启 T2 事务，执行同样的 update 语句——update tb_user set name='bob1' where id>=2 and id<=3 and name<>'bob'; 结果发现，update 操作被阻塞了，等一会儿之后就会报超时。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaSYqAk6hu3KgcD1orwsAcovM6yN6DsvJ18N3MdfgiaSZMsl9mo9uRHIw/640?wx_fmt=png)

4.4> INSERT 语句  

-----------------

*   insert 语句在一般情况下不需要在内存中生成锁结构，只是单纯依靠隐式锁保护插入的记录。
    
*   不过在当前事务插入一条记录之前，需要先定位该记录在 B + 树中的位置。如果该位置的下一条记录已经被加了 gap 锁或 next-key 锁，那么当前事务就会为该记录加上插入意向锁，并且事务进入等待状态。
    
*   下面介绍在执行 insert 语句时，会在内存中生成锁结构的两种特殊情况。
    

### 4.4.1> 重复键  

*   当插入记录的主键与已存在的主键列值重复的时候，会引发插入报错。但是在报错之前，会对该主键值加 S 锁操作，具体如下所示：
    

*   当隔离级别为 **READ UNCOMMITTED** 或 **READ COMMITTED** 时，加 S 型**记录锁**；
    
*   当隔离级别为 **REPEATABLE READ 或 SERIALIZABLE** 时，加 S 型 **next-key 锁**；
    

*   如果与唯一二级索引重复，那么无论是什么隔离级别，都会对已经存在的 B + 树中的那条唯一二级索引记录加 **next-key** 锁。
    
*   另外，在使用 INSERT...ON DUPLICATE KEY... 这样的语法来插入记录时，如果遇到主键或者唯一二级索引列的值重复，会对 B + 树中已存在的相同键值的记录加 **X 锁**，而不是 S 锁。
    

### 4.4.2> 外键检查  

*   待插入记录的外键在主表中能找到
    

    在插入成功之前，无论当前事务的隔离级别是什么，只需要直接给主表对应的那条记录加 **S 型记录锁**即可。

*   待插入记录的外键在主表中找不到
    

*   当隔离级别为 **READ UNCOMMITTED** 或 **READ COMMITTED** 时，并不对记录加锁；
    
*   当隔离级别为 **REPEATABLE READ 或 SERIALIZABLE** 时，对主表查询不到的那个键值附近加 **gap 锁**；
    

五、查看事务加锁情况  

=============

5.1> 获取锁的相关信息
-------------

### 5.1.1> INNODB_TRX

*   该表存储了 InnoDB 存储引擎**当前正在执行的事务信息**。
    

*   我们开启 T1 的事务，并进行查询操作，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiafK2rtgdI4lMAspjFFjKuMwSoicnmDA23tt6lTAPzicticfxlncXuuAicyw/640?wx_fmt=png)

*   然后执行查询操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiafupRFdwVd0ehOJaChethas1ttrsw6COiaN7ibjI0mg3rp2xn7A7fzfsw/640?wx_fmt=png)

### 5.1.2> DATA_LOCKS  

*   如果一个事务想要获取某个锁但未获取到，则会记录该锁的信息。
    
*   如果一个事务获取到了某个锁，但是这个锁阻塞了别的事务，则会记录该锁的信息。
    

*   需要注意一点就是，只有当系统中发生了某个事务因为获取不到锁而被阻塞的情况发生时，该表中才会有记录。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaa3YPrt4M97fgJVTypuNCTia9bZWZywzAz1fjuia5IsspXibQHhr7lVkOg/640?wx_fmt=png)

*   我们在 T1 中开启一个事务，并且执行 select...for update；
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJia1wUWfH5gQT8qbjtFlcY93DmAnIzDHqib4509bqyPW05gzJUalsxYAXQ/640?wx_fmt=png)

*   然后在 T2 中开启事务，也对同一条记录进行 select...for update 操作。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiazYUO1jWBzP4AyiaT15hLsv2NsD6X4amtV19VCMR1TNUBSWQ8zy8q1RA/640?wx_fmt=png)

*   执行 select * from performance_schema.data_locks\G; 语句，查看锁的信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiaXezDwaJld7lnfLLdpLVpZH37kicEz3nREX1O8bkicotETR5icYXQLQIrg/640?wx_fmt=png)

### 5.1.3> DATA_LOCK_WAITS  

*   我们可以通过 data_locks_waits 看到阻塞和请求的两个事务 id 信息，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicBd8icwKDibjXU5J26eb2qJiamsKWkek1hGdRL3TYKuw8brf7hLnMlaadAFrQEM025Bjg3ibqeFAh5AQ/640?wx_fmt=png)

【解释】

*   **REQUESTING_ENGINE_TRANSACTION_ID**
    

表示因为获取不到锁而被阻塞的事务 id。

*   **BLOCKING_ENGINE_TRANSACTION_ID**
    

表示因为获取到别的事务需要的锁而导致对方被阻塞的事务 id。