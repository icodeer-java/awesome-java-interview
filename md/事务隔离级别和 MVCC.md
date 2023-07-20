> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247487000&idx=1&sn=528fe29bbc97357166b3168a5441c67b&chksm=e9114ce5de66c5f3f7cf83173195a8b6e3abb766e27a5a8c7b8fe46c8e4a3065a0f8fda4f482&scene=178&cur_album_id=2142543588444995585#rd)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDpF1DNnxWmfW18h1encZZZAyY8rFtIaY6JRUJga4zwlsd50lUaM6GNA/640?wx_fmt=png)

一、事务隔离级别  

-----------

*   _什么是事务的隔离性？_
    
    如果**多个事务**访问了**同样的一条数据**，那么会造成数据的一致性问题。这就要求我们使用某种手段来强制让这些事务按照顺序一个一个单独地执行，或者最终执行的效果和单独执行一样。也就是说我们希望让这些事务 “隔离” 地执行，互不干涉。这也就是事务的隔离性。
    
*   当多个事务对同一条数据进行写操作的时候，就会涉及到一致性的问题。我们可以通过串行化来解决。但是却会造成一定的性能损失。我们思考，_是否可以牺牲一部分隔离性来换取性能上的提升呢？_是的，当然可以。不过我们首先需要搞明白，_多个事务在不进行串行化执行的情况下，到底会出现哪些一致性问题？_
    

### 1.1> 事务并发执行时遇到的问题

#### 1.1.1> 脏写

*   _什么是脏写？_
    
    如果一个事务 T2 **修改了**另一个**未提交事务** T1 **修改过**的数据，这就意味着发生了脏写现象。
    
*   脏写引发的**一致性**问题：
    
    我们希望 x 值和 y 值始终相同。下面是事务 T1 和事务 T2 对 x 值和 y 值的操作：
    
    **w1[x=1] w2[x=2] w2[y=2] c2 w1[y=1] c1**
    
    最终导致了 x=2 y=1，x 值和 y 值不相同，破坏了一致性需求。
    
*   脏写引发的**原子性**和**持久性**问题：
    
    比方说有 x=0 和 y=0 这两个数据项，下面是事务 T1 和事务 T2 对 x 值和 y 值的操作：**w1[x=2] w2[x=3] w2[y=3] c2 a1**
    
    由于 T1 执行了回滚操作，即 x=0（将 x 置为最初状态），那么相当于对 T2 对数据库所做的修改进行了**部分回滚**（即：T2 只回滚了对 x 做的修改，而不回滚对 y 做的修改），那么这就影响到了事务的**原子性**。
    
    而 T2 已经提交了，但是却被 T1 的回滚造成了自己修改的数据也被回滚了，破坏了 T2 事务的**持久性**。
    

1.1.2> 脏读  

*   _什么是脏读？_
    
    脏读就是指当一个事务 T1 正在访问数据，并且对数据进行了修改，而这种修改**还没有提交**到数据库中，这时，另外一个事务 T2 也访问这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是脏数据 (Dirty Data)，依据脏数据所做的操作可能是不正确的。
    
*   操作流程如下图所示:
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDUDaLGc8vK9kXPKhbkPQlNHkuaPnkDU1LHVMfOG2tgzLHKNNqs8ctNw/640?wx_fmt=png)

*   脏读引发的**一致性**问题：
    
    事务 T1 和事务 T2 访问 x 和 y 这两个值，我们一致性需求就是让 x 值和 y 值始终相同，x 和 y 的初始值都是 0。现在并发执行事务 T1 和 T2 如下所示：
    
    **w1[x=1] r2[x=1] r2[y=0] c2 w1[y=1] c1**
    
    很显然 T2 是个只读事务，读取到了事务 T1 未提交事务的值，所以 T2 读到的 x=1，y=0，不符合 x=y 的一致性。数据库的不一致状态是不应该暴露给用户的。
    
*   脏读的严格解释
    
    **w1[x] ... r2[x] ... (a1 and c2 in any order)**
    
    也就是 T1 先修改了数据项 x 的值，然后 T2 又读取到了**未提交事务** T1 针对数据项 x 修改后的值，之后 T1 回滚而 T2 提交。这就意味着 T2 **读到了一个根本不存在的值**。
    

#### 1.1.3> 不可重复读

*   _什么是不可重复读？_
    
    指在一个事务 T1 内，多次读同一数据。在这个事务 T1 还没有结束时，另外一个事务 T2 修改并提交了该同一数据。那么，事务 T1 两次读到的数据可能是不一样的。这样就发生了**在一个事务内两次读到的数据是不一样的**，因此称为是不可重复读。
    
*   操作流程如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDbSiagIWVAQX3XX9PoED37f3BtgsNiaXkEBI0z5EhIa6TicfbO1HXLjwEQ/640?wx_fmt=png)

*   不可重复读的**一致性**问题：
    
    事务 T1 和事务 T2 访问 x 和 y 这两个值，我们一致性需求就是让 x 值和 y 值始终相同，x 和 y 的初始值都是 0。现在并发执行事务 T1 和 T2 如下所示：
    
    **r1[x=0] w2[x=1] w2[y=1] c2 r1[y=1] c1**
    
    很显然 T1 是个只读事务，最终读取到的是 x=0，y=1，很显然这是一个不一致的状态，这种不一致的状态是不应该暴露给用户的。
    
*   不可重复读的严格解释
    
    **r1[x] ... w2[x] ... c2 ... r1[x] ... c1**
    
    也就是 T1 先读取了数据项 x 的值，然后 T2 又修改了 x 的值，之后 T2 提交事务，然后 T1 再次读取数据项 x 的值时会得到与第一次读取时不同的值。
    

#### 1.1.4> 幻读

*   什么是幻读
    
    如果一个事务 T1 先根据**某些搜索条件**查询出一些记录，在该事务未提交时，另一个事务 T2 操作了一些符合那些搜索条件的记录（insert、delete、update），就意味着发生了幻读现象。
    
*   操作流程如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDlFzH2KicSFRRKhIIq68P74iaL8ibfPQqSPhxBY3m9PuJobtXeicbp42hicw/640?wx_fmt=png)

*   幻读的**一致性**问题：
    
    **r1[P] ... w2[y in P] ... c2 ... r1[P] ... c1**
    
    T1 先读取符合搜索条件 P 的记录，然后 T2 写入了符合搜索条件 P 的记录。之后 T1 再读取符合搜索条件 P 的记录时，会发现两次读取的记录是不一样的。
    

### 1.2> SQL 标准中的 4 种隔离级别

*   四种隔离级别如下表所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDfN8Ficx8vmhoxfLLibkZQO1XGibtIYYB2NcGzCNpgvGtJ17cdHErpJKEw/640?wx_fmt=png)

【注】  

*   由于无论哪种隔离级别，**都不允许脏写的情况发生**，所以没有列入到表格中。
    
*   MySQL 与 SQL 标准不同的一点就是，**MySQL 在 REPEATABLE READ 隔离级别下很大程度地避免了幻读现象**。
    

### 1.3> MySQL 中支持的 4 种隔离级别

*   不同的数据库厂商对 SQL 标准中规定的 4 种隔离级别的支持不一样。比如：
    

*   Oracle 就只支持 **READ COMMITTED** 和 **SERIALIZABLE** 这两种隔离级别。
    
*   MySQL 支持以上四种，且默认的隔离级别为 **REPEATABLE READ**。
    

*   设置事务的隔离级别
    
    **SET** [**GLOBAL** | **SESSION** | **什么也不加**] **TRANSACTION ISOLATION LEVEL** [**READ UNCOMMITTED** | **READ COMMITTED** | **REPEATABLE READ** | **SERI****ALIZABLE**];
    

*   **GLOBAL**
    
    只对执行完该语句之后**新产生的会话**起作用；
    
    对**当前已经存在的会话无效**。
    
*   **SESSION**
    
    对当前会话所有的**后续事务**有效。
    
    该语句可以在已经开启的事务中执行，但不会影响当前正在执行的事务。
    
    如果在事务之间执行，则对后续的事务有效。
    
*   **什么也不加**
    
    只对当前会话中**下一个**即将开启的事务有效。
    
    下一个事务执行完毕，**后续事务将恢复到之前的隔离级别**。
    
    该语句**不能在已经开启的事务中执行**，否则会报错。
    

*   查看隔离级别
    
    **show variables like 'transaction_isolation';**
    
    **select @@transaction_isolation;**
    

如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDNMSkIVUPfA4Wefo6H2EeVmlWevhhozZS6UfwJDru6VTdK3icPCqrJHA/640?wx_fmt=png)

**_注意，transaction_isolation 是在 MySQL 5.7.20 版本中引入的，用来替换 tx_isolation。如果大家使用的是之前版本的 MySQL，请将其替换为 tx_isolation。_**  

*   查看 mysql 版本
    
    **select @@version;**
    
*   查看是否自动提交
    
    **show variables like 'autocommit';**
    
    **# set autocommit=0;**
    
*   需要隔离级别
    
    **set session transaction isolation level serializable;**
    
    **set session transaction isolation level read committed;**
    
    **set session transaction isolation level read uncommitted;**
    
    **set session transaction isolation level repeatable read;**
    

二、MVCC 原理  

------------

### 2.1> 版本链

*   **聚簇索引**记录中都包含下面两个必要的**隐藏列**（row_id 并不是必要的，在创建的表中**有主键**或者**有不允许为 NULL 的 UNIQUE 键**时，都不会包含 row_id 列）。
    

*   **trx_id**
    
    一个事务每次对某条聚簇索引记录**进行改动**时（即：insert、delete、update），都会把该事务的事务 id 赋值给 trx_id 隐藏列。
    
*   **roll_pointer**
    
    每次对某条聚簇索引记录**进行改动**时，都会把旧的版本写入到 undo 日志中。这个隐藏列就相当于一个**指针**，可以通过它找到该**记录修改前**的信息。
    

*   假设执行 insert into tb_student(id, number, name, age) values(5, 1, 'muse', 20)，假设当前执行插入操作的事务 id 是 80，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDMa1RX7t37qKtOOwvOFBUZyFrR7Z8SJDTHdoffOQkV6qARXPrdrXJiag/640?wx_fmt=png)

【注】实际上 insert undo 日志只在事务回滚时发生作用。**当事务提交后，该类型的 undo 日志就没有用了**，它占用的 Undo Log Segment 也会被系统回收。虽然真正的 insert undo 日志占用的存储空间被回收了，**但是 roll_pointer 的值并不会被清除**。roll_pointer 属性占用 7 个字节，第一个比特就标记着它指向的 undo 日志的类型；如果比特值为 1，就表示它指向的 undo 日志属于 TRX_UNDO_INSERT 大类，也就是该 undo 日志为 insert undo 日志。

*   两个事务 T1 和 T2 分别对 id=5 的数据进行更新操作，版本链示意图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDFiaP1a8xPCSZlHglAd7tIWaQ90X1S8YHIV5ThOnxZynsicicV6TwFHp9A/640?wx_fmt=png)

【注】有一点需要注意，我们在 UPDATE 操作产生的 undo 日志中，**只会记录一些索引列以及被更新的列信息**，并**不会记录所有列的信息**；上图中，记录完整信息，是为了促进理解。

*   什么是版本链
    
    在每次更新该记录后，都会将旧值放到一条 undo 日志中。随着更新次数的增多，**所有的版本都会被 roll_pointer 属性连接成一条链表**，这个链表就称之为**版本链**。
    
*   版本链的**头节点**就是当前记录的**最新值**。
    
*   每个版本中还包含生成该版本时对应的**事务 id**。
    
*   什么是 MVCC
    
    我们利用这个**记录的版本链**和 **ReadView** 来控制并发事务访问相同记录时的行为，我们把这种机制称之为**多版本并发控制**（Multi-Version Concurrency Control）
    

### 2.2> ReadView  

*   不同隔离级别，访问记录方式
    

*   **READ UNCOMMITTED**：由于可以读到未提交事务修改过的记录，所以直接读取记录的最新版本就好了。
    
*   **SERIALIZABLE**：InnoDB 规定使用**加锁**的方式来访问记录。
    
*   **READ COMMITTED** **&** **REPEATABLE READ**：必须保证读到**已经提交**的事务修改过的记录。
    

*   什么是 ReadView？
    
    也叫**一致性视图**，用来判断版本链中的哪个版本是当前事务可见的。ReadView 包含 4 个比较重要的内容：
    

*   **m_ids**：在生成 ReadView 时，当前系统中**活跃的读写事务**的事务 id 列表。
    
*   **min_trx_id**：在生成 ReadView 时，当前系统中活跃的读写事务中**最小的事务 id**；也就是 m_ids 中的最小值。
    
*   **max_trx_id**：在生成 ReadView 时，系统应该分配给**下一个事务的事务 id** 值。
    
*   **creator_trx_id**：**生成该 ReadView** 的事务的事务 id。
    

**只有在对表中的记录进行改动时（即：insert、delete、update）才会为事务分配唯一的事务 id，否则一个事务的事务 id 值都默认为 0。**

*   如何通过 ReadView 来判断记录的某个版本是否可见？
    
    _trx_id：表示__被访问记录上的事务版本 ID——trx_id。_
    

*   如果 **trx_id** **==** **creator_trx_id**，则表明当前事务在访问它**自己修改**过的记录，所以该版本**可以**被当前事务访问。
    
*   如果 **trx_id <****min****_****trx_id**，则表明生成该版本的事务在当前事务**生成 ReadView 之前**已经提交了，所以该版本**可以**被当前事务访问。
    

*   如果 **trx_id >=****max****_****trx_id**，则表明生成该版本的事务在当前事务**生成 ReadView 之后**才开启，所以该版本**不可以**被当前事务访问。
    
*   如果 **trx_id in****m_ids**，说明创建 ReadView 时生成该版本的**事务还是活跃的**，该版本**不可以**被访问。
    

*   如果 **trx_id not in****m_ids**，说明创建 ReadView 时生成该版本的**事务已经被提交**，该版本**可以**被访问。
    
*   如果某个版本的数据对当前事务不可见，那就顺着版本链**找到下一个版本**的数据，并继续执行上面的步骤来判断记录的可见性，以此类推，直到版本链中的最后一个版本。
    

*   READ COMMITTED 和 REPEATABLE READ 隔离级别之间一个非常大的区别就是**——****它们生成 ReadView 的时机不同！！**
    

*   前提说明
    
    再次强调，在事务执行过程中，只有在第一次真正修改记录时（比如使用 insert、delete、update 语句时），才会分配一个唯一的事务 id；而且这个事务 id 是递增的。所以我们才在 T2 中更新了别的表（tb_user）的记录，目的是为它分配事务 id。
    
*   **READ COMMITTED**——在一个事务中，【每次读取】数据前都生成一个 ReadView！！！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDCsZaNmFlRa8xQbuhkaFFdZEGHiabxw4mU5tyM6z7fqI1E5O0UNqlRRg/640?wx_fmt=png)【执行步骤】

BEGIN;  # 开启事务

      select name from tb_student where id=5;  # 生成 ReadView，由于 T1 和 T2 都未提交事务，所以查询得到 name 的值为 “muse”

*   步骤 1：在执行 select 语句时先生成一个 ReadView：
    
    **m_ids=[10, 20]**
    
    **min_trx_id=10**
    
    **max_trx_id=21**
    
    **creator_trx_id=0** （_新开启的事务没有对任何记录进行修改，则系统不会为它分配唯一的事务 id，默认为 0_）
    

*   步骤 2：最新版本的 name=“b” 且 trx_id=10，由于 10 在 m_ids 列表中，所以不符合可见性要求；根据 roll_pointer 跳到下一个版本。
    
*   步骤 3：下一个版本的 name=“a” 且 trx_id=10，由于 10 在 m_ids 列表中，所以不符合可见性要求；根据 roll_pointer 跳到下一个版本。
    

*   步骤 4：下一个版本的 name=“muse” 且 trx_id=2，由于 2 小于 min_trx_id(10)，所以符合可见性要求；最后返回给用户 name=“muse” 的记录。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheID7tUfZ0J1icfFdpnRM9dQKAQRib11DgluaWcfQFRbgV5wtVtWhACSv0Zw/640?wx_fmt=png)【执行步骤】

T1 提交了事务，T2 也开始更新了学生数据表中 id=5 的数据。

BEGIN;  # 开启事务

      select name from tb_student where id=5;  # 生成 ReadView，由于 T1 和 T2 都未提交事务，所以查询得到 name 的值为 “muse”

     select name from tb_student where id=5;  # 再次生成新的 ReadView，由于 T1 提交事务，所以查询得到 name 的值为 “b”

*   步骤 1：第二次执行 select 语句时又会单独生成一个 ReadView。
    
    **m_ids=[20]**（_由于 T1 已经提交了，所以新生成的 ReadView 中 trx_id=10 的活跃事务 id 就不在 m_ids 列表中了_）
    
    **min_trx_id=20**
    
    **max_trx_id=21**
    
    **creator_trx_id=0**
    

*   步骤 2：最新版本的 name=“d” 且 trx_id=20，由于 20 在 m_ids 列表中，所以不符合可见性要求；根据 roll_pointer 跳到下一个版本。
    
*   步骤 3：下一个版本的 name=“c” 且 trx_id=20，由于 20 在 m_ids 列表中，所以不符合可见性要求；根据 roll_pointer 跳到下一个版本。
    

*   步骤 4：下一个版本的 name=“b” 且 trx_id=10，由于 10 小于 min_trx_id(20)，所以符合可见性要求；最后返回给用户 name=“b” 的记录。
    

*   **REPEATABLE READ**——在一个事务中，只在【第一次读取】数据时生成一个 ReadView！！！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheIDCsZaNmFlRa8xQbuhkaFFdZEGHiabxw4mU5tyM6z7fqI1E5O0UNqlRRg/640?wx_fmt=png)【执行步骤】

    与 READ COMMITTED 步骤一样。略

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicgBJmah5rx7dQc2VnuheID7tUfZ0J1icfFdpnRM9dQKAQRib11DgluaWcfQFRbgV5wtVtWhACSv0Zw/640?wx_fmt=png)

【执行步骤】

    BEGIN;  # 开启事务

    select name from tb_student where id=5;  # 生成 ReadView，由于 T1 和 T2 都未提交事务，所以查询得到 name 的值为 “muse”

    select name from tb_student where id=5;  # 不生成 ReadView 了，复用第一次的，虽然 T1 提交事务，ReadView 还是旧的，所以查询得到 name 的值还为 “muse”

*   步骤 1：第二次执行 select 语句时，会复用之前的 ReadView。
    
    **m_ids=[10,20]**（即使后续 T1 提交了，也依然不会生成新的 ReadView，trx_id=10 这个事务 id 依然在 m_ids 中作为活跃事务 id）**min_trx_id=10**
    
    **max_trx_id=21**
    
    **creator_trx_id=0**
    

*   步骤 2：最新版本的 name=“d” 且 trx_id=20，由于 20 在 m_ids 列表中，所以不符合可见性要求；根据 roll_pointer 跳到下一个版本。
    
*   步骤 3：下一个版本的 name=“c” 且 trx_id=20，由于 20 在 m_ids 列表中，所以不符合可见性要求；根据 roll_pointer 跳到下一个版本。
    

*   步骤 4：下一个版本的 name=“b” 且 trx_id=10，由于 10 在 m_ids 列表中，所以不符合可见性要求；根据 roll_pointer 跳到下一个版本。
    
*   步骤 5：下一个版本的 name=“a” 且 trx_id=10，由于 10 在 m_ids 列表中，所以不符合可见性要求；根据 roll_pointer 跳到下一个版本。
    

*   步骤 6：下一个版本的 name=“muse” 且 trx_id=2，由于 2 小于 min_trx_id(10)，所以符合可见性要求；最后返回给用户 name=“muse” 的记录。
    

### 2.3> 二级索引与 MVCC  

*   我们知道，只有在聚簇索引记录中才有 trx_id 和 roll_pointer 隐藏列。如果某个查询语句使用二级索引来执行查询，该如何判断可见性呢？有如下两步
    

BEGIN;  # 开启事务

select name from tb_student where name='muse';  

*   步骤 1：二级索引页面的 **Page Header** 部分有一个名为 **PAGE_MAX_TRX_ID** 的属性，它代表着修改该二级索引页面的最大事务 id 是什么。当 select 语句访问某个二级索引记录时，首先会看一下 min_trx_id 是否大于该页面的 PAGE_MAX_TRX_ID，如果大于，则说明该页面中的所有记录都对该 ReadView 可见；否则就得执行步骤 2，在回表之后再判断可见性。
    
*   步骤 2：利用二级索引记录中的主键值进行回表操作，得到对应的聚簇索引记录后再按照前面讲过的方式找到对该 ReadView 可见的第一个版本，然后判断该版本中相应的二级索引列（name）的值是否与利用该二级索引查询时的值（“muse”）相同。
    

### 2.4> MVCC 小结  

*   所谓的 MVCC 指的就是在使用 READ COMMITTED 和 REPEATABLE READ 这两种隔离级别的事务执行普通的 SELECT 操作时，访问记录的版本链的过程。
    
*   这样可以使不同事务的读 - 写、写 - 读操作并发执行，从而提升性能。
    

*   READ COMMITTED 和 REPEATABLE READ 这两种隔离级别有一个很大的不同点——就是生成 ReadView 的时间不同。
    
*   READ COMMITTED 在每一次进行普通 SELECT 操作前都会生成一个 ReadView。
    

*   REPEATABLE READ 只会在第一次进行普通 SELECT 操作前生成一个 ReadView。