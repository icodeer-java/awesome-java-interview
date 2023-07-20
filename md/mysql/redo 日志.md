
![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn868bdYSvEYfbRhlwaD7MVst63HibHrd8WvYzjvaHobbaagcicDCEBEZtbw/640?wx_fmt=png)

一、什么是 redo 日志  

----------------

### 1.1> 关于 MySQL 故障产生的问题

*   问题
    
    如果我们只在内存的 Buffer Pool 中修改了页面，假设在**事务提交后**突然发生了某个**故障**，导致内存中的**数据都失效了**，那么这个已经提交的事务在数据库中所做的更改也就丢失了。针对这种问题，怎么处理呢？
    
*   解决方案一：
    
    **在事务提交时，把该事务修改的所有页面都刷新到磁盘。**
    

【缺点】

*   1> **刷新一个完整的数据页****太浪费了**
    
    虽然我们只修改了一条记录，但是会将这条记录所在的页（16KB）都刷新到磁盘上，会造成大量磁盘 I/O 的浪费。
    
*   2> **随机 I/O** **刷新起来比较慢**
    
    **一个事务**可能包含**很多语句**，即使是**一条语句**也可能修改**许多页面**，并且该事务修改的这些页面可能**并不相邻**。这就意味着将某个事务修改的 Buffer Pool 中的页面刷新到磁盘时，需要进行很多的**随机 I/O**。而随机 I/O 要比顺序 I/O 慢，尤其是机械硬盘。
    

*   解决方案二：
    
    **在事务提交时，只需要把修改的内容记录一下就好了。**
    
    例如：“将第 0 号表空间第 100 号页面中偏移量为 1000 处的值更新为 2。”
    
*   redo 日志的定义
    
    因为在系统因崩溃而重启时需要按照上述内容所记录的步骤重新更新数据页，所以上述内容也成为**重做日志**（redo log）。
    

【优点】

*   1> redo 日志占用的**空间非常小**；
    
*   2> redo 日志是**顺序写入****磁盘**的；
    

在执行事务的过程中，每执行一条语句，就可能产生若干条 redo 日志，这些日志是按照产生的**顺序写入磁盘**的，也就是使用**顺序 I/O**。

二、redo 日志格式  

--------------

*   redo 日志本质上只是记录了一下事务对数据库进行了哪些**修改**，因此针对不同修改场景，定义了**多种类型**的 redo 日志，但是绝大部分类型的 redo 日志都有如下的**通用格式。**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86bDjMXgcLMhxPUkOvOjPt6N6wQatj9yEML3vBAiaM2icSo7XfIBtCeXicw/640?wx_fmt=png)

*   在 MySQL5.7.22 版本中，共有 53 种不同的类型。  
    
*   【表空间 ID + 页号】，可以定位到 redo 日志相关的页。
    

### 2.1> 简单的 redo 日志类型——物理日志

*   _什么是物理日志？_
    
    在对页面的**修改是极其简单**的情况下（下面会有例子），redo 日志中只需要记录一下在**某个页面**的**某个偏移量**处**修改了几个字节的值**、具体**修改后的内容是啥**就好了。
    

【场景举例】

*   如果某张表**没有主键**，并且没有定义不允许存储 **NULL 值**的 **UNIQUE 键**，那么 InnoDB 会自动为表添加一个名为 **row_id 的隐藏列**作为主键。
    
*   为这个 row_id 隐藏列进行赋值的方式如下：
    

*   内存中维护一个**全局变量**，当向某个包含 row_id 隐藏列的表中插入一条记录时，就会把这个全局变量的值当做新记录的 row_id 的值，并且把这个全局变量 **+1**；
    
*   每当这个全局变量的值为 **256 的倍数**时，就会将该变量的值刷新到**系统表空间页号为 7** 的页面中一个名为 **Max Row Id** 的属性中。（这个写入操作，实际上是在 **Buffer Pool** 中完成的，我们需要把这次对这个页面的修改以 redo 日志的形式记录下来）
    

*   当系统启动时，会将这个 **Max Row Id** 属性加载到内存中，并将该值**加上 256** 之后赋值给前面提到的全局变量（因为在系统上次关机时，如果内存中的全局变量没有到达 256 的倍数，而没有刷新到 BufferPool，那么就会出现该全局变量的值可能大于磁盘页面中 Max Row ID 属性的值）
    

*   这种对页面修改是极其简单的。所以该 redo 日志即为**物理日志**。
    

*   物理日志的几种不同类型
    

*   MLOG_1BYTE（type=1）
    
    表示在**页面的某个偏移量**处写入 **1** **字节**的 redo 日志类型。
    

*   MLOG_2BYTE（type=2）
    
    表示在**页面的某个偏移量**处写入 **2 字节**的 redo 日志类型。
    

*   MLOG_4BYTE（type=4）
    
    表示在**页面的某个偏移量**处写入 **4 字节**的 redo 日志类型。
    

*   MLOG_8BYTE（type=8）
    
    表示在**页面的某个偏移量**处写入 **8 字节**的 redo 日志类型。由于 **Max Row ID** 占用 8 字节的空间，所以在修改页面中的这个属性时，会记录一条类型为 MLOG_8BYTE 的 redo 日志
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86QWc2CsyyVSN94ibAbTNcFATINQ4SiahcC7ND1k3XmKvXic8ePMrEicSQxQ/640?wx_fmt=png)

*   MLOG_WRITE_STRING（type=30）
    

表示在**页面的某个偏移量**处写入**一个字节序列**。    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn863WWGc7IJUk6OtzyaDCZQVYQJnJngrX8pmz8FhwiaIVKnm6TTqDGZCyA/640?wx_fmt=png)

【注】只要在【数据占用的字节数】处填入 1、2、4、8 这些数字，就可以分别替代 MLOG_1BYTE、MLOG_2BYTE、MLOG_4BYTE、MLOG_8BYTE 这些类型的 redo 日志。但是，为了节省空间，所以，才特殊定义了这 4 个类型。  

### 2.2> 复杂一些的 redo 日志类型  

*   执行一条 INSERT 语句涉及的更新内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86pSjQC0nQx782Yme3pnS2sefPS5zXGU0wibA1EeIQ21fiaJ4S1WBCw6fg/640?wx_fmt=png)【注】**用户数据**指的就是聚簇索引和二级索引对应的 B + 树。

*   可能更新 **Page Directory** 中的**槽信息**。
    
*   可能更新 **Page Header** 中的各种**页面统计信息**。
    

*   **PAGE_N_DIR_SLOTS**：表示的槽数量可能会更改。
    
*   **PAGE_HEAP_TOP**：代表的还未使用的空间最小地址可能会更改。
    

*   **PAGE_N_HEAP**：代表的本页面中的记录数量可能会更改。
    

*   可能更新记录的**单向链表**
    

*   数据页中的记录按照序列从小到大的顺序组成的一个单向链表，每插入一条记录，还需要更新上一条记录的记录头信息中的 next_record 属性来维护这个单向链表。
    

*   还有其他需要**更新的内容**。
    

*   综上所述：我们想要说明的一点就是——在把一条记录插入到一个页面时，**需要更改的地方非常的多**。这时，如果使用前面介绍的简单的物理 redo 日志来记录这些修改，可能会有如下两种解决方案
    

*   方案一：**在每个修改的地方都记录一条 redo 日志**
    
    这种方式的缺点是显而易见的，因为被修改的地方实在太多了，可能 redo 日志占用的空间都要比整个页面占用的空间多。
    

*   方案二：**将整个页面第一个被修改的字节到最后一个被修改的字节之间****所有的数据****当成一条物理 redo 日志中的具体内容**
    
    这种方案所涉及的数据中，会掺杂很多本来没有被修改的数据，这样都加到 redo 日志中，太浪费空间了。
    

*   为了解决上面的问题，我们来介绍一下新的 redo 日志类型
    

*   **MLOG_REC_INSERT**（type=9）
    
    表示**插入**一条使用**非紧凑行格式**（REDUNDANT）的**记录**时，redo 日志的类型。
    

*   **MLOG_COMP_REC_INSERT**（type=38）
    
    表示**插入**一条使用**紧凑行格式**（COMPACT、DYNAMIC、COMPRESSED）的**记录**时，redo 日志的类型。
    

*   **MLOG_COMP_PAGE_CREATE**（type=58）
    
    表示**创建**一个存储**紧凑行格式**记录的**页面**时，redo 日志的类型。
    

*   **MLOG_COMP_REC_DELETE**（type=42）
    
    表示**删除**一条使用**紧凑行格式**的**记录**时，redo 日志的类型。
    

*   **MLOG_COMP_LIST_START_DELETE**（type=44）
    
    表示在从某条给定记录开始**删除**页面中一系列使用**紧凑行格式**的**记录**时，redo 日志的类型。
    

*   **MLOG_COMP_LIST_END_DELETE**（type=43）
    
    与 MLOG_COMP_LIST_START_DELETE 类型的 redo 日志**相呼应**，表示**删除一系列记录**，直到 MLOG_COMP_LIST_END_DELETE 类型的 redo 日志对应的**记录为止**。
    

*   **MLOG_ZIP_PAGE_COMPRESS**（type=51）
    
    表示**压缩一个数据页**时，redo 日志的类型。
    

*   还有很多很多种类型，这里就不列举了，等用到时再说。
    

*   上面这些类型的 redo 日志包含两个层面的意思：
    

*   从**物理层面**来看
    
    这些日志都指明了对哪个**表空间**的哪个**页**进行修改。
    

*   从**逻辑层面**来看
    
    在系统崩溃后重启时，并不能直接根据这些日志中的记载，在页面内的某个偏移量处恢复某个数据，而是需要**调用一些事先准备好的函数**，在执行完这些函数后才可以将页面恢复成系统崩溃前的样子。
    

*   上面解释可能有些懵，我们还是以 MLOG_COMP_REC_INSERT 类型的 redo 日志为例，解释一下物理层面和逻辑层面到底是啥意思。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn867NNugGNUNpAMPpyLaa5qaGs7Fwf0tSjN5VrN9B5JjUkicKicFl10qia9A/640?wx_fmt=png)

三、Mini-Transaction  

---------------------

### 3.1> 以组的形式写入 redo 日志

*   在执行语句的过程中产生的 redo 日志，被 InnoDB 划分成了若干个**不可分割的组**。比如：
    

*   更新 **Max Row ID** 属性时产生的 redo 日志为一组，是不可分割的。
    
*   向**聚簇索引**对应 B + 树的页面中插入一条记录时产生的 redo 日志为一组，是不可分割的。
    
*   向**二级索引**对应 B + 树的页面中插入一条记录时产生的 redo 日志为一组，是不可分割的。
    
*   还有其他的一些不可分割的组。
    

*   什么是**不可分割**呢？
    
    我们以向某个索引对应的 B + 树中插入一条记录为例进行解释。
    

*   **乐观插入**
    
    该数据页剩余的空闲空间相当**充足**，足够容纳这一条待插入记录。这样很简单，直接把记录插入到这个数据页中，然后记录一条 MLOG_COMP_REC_INSERT 类型的 redo 日志就好了。
    
*   **悲观插入**
    
    该数据页剩余的**空间不足**，那么就涉及到了**页分裂**操作——即：创建一个叶子节点，把原先数据页中的一部分记录复制到这个新的数据页中，然后再把记录插入进去；再把这个叶子节点插入到叶子节点链表中，最后还要在内节点中添加一条目录项记录来指向这个新创建的页面。很显然，这个过程需要对多个页面进行修改，着意味着会产生多条 redo 日志。还要涉及修改各种段、区的统计信息，修改各种链表的统计信息等（比如：FREE 链表、FREE_FRAG 链表等），反正**总共需要记录的 redo 日志有二三十条**。
    

*   InnoDB 认为，向某个索引对应的 B + 树中插入一条记录的过程必须是原子的，不能说插入了一半之后就停止了。否则就会形成一棵不正确的 B + 树。所以他们规定在执行这些需要**保证原子性**的操作时，必须以**组**的形式来记录 redo 日志。
    
*   在进行恢复时，针对某个组中的 redo 日志，要么把全部的日志都恢复，要么一条也不恢复。
    
*   如何把这些 redo 日志划分到一个组里呢？
    
    在该组中的最后一条 redo 日志后面加上一条特殊类型的 redo 日志。该类型的 redo 日志的名称为 **MLOG_MULTI_REC_END**，结构很简单，只有一个 type 字段（type=31）。所以，某个需要保证原子性的操作所产生的一系列 redo 日志，必须以一条类型为 **MLOG_MULTI_REC_END** 的 redo 日志结尾，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn860dHMOToiaTylDP6Va1z3FqD693CxJb2p8TjOLqSSEiaZjjVMA7Fk1Slw/640?wx_fmt=png)

*   当系统因崩溃而重启恢复时，只有解析到类型为 **MLOG_MULTI_REC_END** 的 redo 日志时，才认为解析到了一组完整的 redo 日志，才会进行恢复；否则直接放弃前面解析到的 redo 日志。
    
*   有些需要保证原子性的操作**只生成一条** redo 日志，那是否也需要 MLOG_MULTI_REC_END 结尾呢？
    
    答：不是的。为了勤俭节约，通过 type 字段即可表示。因为一个 type 字段其实占用 1 个字节（8 位）。所以，当**最高位为 1** 的时候，代表这个需要保证原子的操作且只产生了一条**单一的 redo 日志**。否则，就表示这个需要保证原子性的操作产生了一系列的 redo 日志。而剩下的 7 个位，足够可以表达所有的 redo 类型日志（redo 日志有几十种）。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86RSYuVNOWQPVsZ11XWwYkbJnuE4zxWXWCIrSgiaJj0vEcUBribBRZLibjw/640?wx_fmt=png)

### 3.2> Mini-Transaction 的概念

*   什么是 MTR
    
    对底层**页面**进行一次**原子访问**的过程被称为一个 Mini-Transaction（MTR）。
    
    比如，前文说的修改 Max Row ID 的值，就算是一个 MTR；
    
    比如，向某个索引对应的 B + 树中插入一条记录的过程也算是一个 MTR；
    
*   事务、语句、MTR、redo 日志之间的关系
    
    1 个事务可以包含 n 条 SQL 语句；
    
    1 条 SQL 语句可以包含 n 个 MTR；
    
    1 条 MTR 可以包含 n 条 redo 日志；
    
*   关系如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86pYicHoB3mDzoJqwDQwpYR4sxoTnOscFgYbkvhqTZtbT3gjtJH35aD3Q/640?wx_fmt=png)

四、redo 日志的写入过程  

-----------------

### 4.1> redo log block

*   什么是 redo log block
    
    为了更好地管理 redo 日志，设计 InnoDB 的大叔把通过 MTR 生成的 redo 日志都放在了大小为 **512 字节**的页中。
    
    为了与前文提到的表空间中的页进行区别，我们这里把用来存储 redo 日志的页称为 block。（其实 “页” 和“block”的意思差不多）
    
*   一个 redo log block 的结构示意图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn860icyEHibemjibTUALHu3rKIPRic7TXP9hQZLI5pgc7tzvZhvLQ65XC7FAg/640?wx_fmt=png)

【解释】

*   **【4 字节****】LOG_BLOCK_HDR_NO**
    
    每个 block 都有一个**大于 0** 的唯一编号，该属性就表示该**编号值**。
    
*   **【2 字节】LOG_BLOCK_HDR_DATA_LEN**
    
    表示 block 中已经**使用了多少字节**；
    
    初始值为 **12**（因为 log block body 从第 12 个字节处开始）。
    
    随着往 block 中写入的 redo 日志越来越多，该属性值页跟着增长。
    
    如果 log block body 已经被全部写满，那么该属性的值被**设置为 512**。
    
*   **【2 字节】LOG_BLOCK_FIRST_REC_GROUP**
    
    代表该 block 中**第一个 MTR** 生成的 redo 日志记录组的偏移量，其实也就是这个 block 中第一个 MTR 生成的第一条 redo 日志记录的偏移量（如果一个 MTR 生成的 redo 日志横跨了好多个 block，那么最后一个 block 中的 LOG_BLOCK_FIRST_REC_GROUP 属性就表示这个 MTR 对应的 redo 日志结束的地方，也就是下一个 MTR 生成的 redo 日志开始的地方）。
    
*   **【4 字节】LOG_BLOCK_CHECKPOINT_NO**
    
    表示 checkpoint 的序号。
    
*   **【4 字节】LOG_BLOCK_CHECKSUM**
    
    表示该 block 的校验值，用于正确性校验。
    

### 4.2> redo 日志缓冲区  

*   与 Buffer Pool 类似，写入 redo 日志时也**不能直接写到磁盘**中，实际上在服务器启动时就向操作系统申请了一大片称为 **redo log buffer**（redo 日志缓冲区）的**连续内存空间**，也可以将其简称为 log buffer。这片内存空间被划分成若干个连续的 redo log block。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86ic6nbeo03XnicTFbnEulLKNQibk4RxPQibCPPCt4oZhIDXfLXWskzvuic3A/640?wx_fmt=png)

*   **innodb_log_buffer_size** 来指定 log buffer 的大小。在 MySQL 5.7.22 版本中，该启动选项的默认值为 **16MB**。
    

### 4.3> redo 日志写入 log buffer

*   向 log buffer 中写入 redo 日志的过程是**顺序写入**的。
    
*   **buf_free**
    
    全局变量，该变量指明后续写入的 redo 日志应该写到 log buffer 中的哪个位置。
    
*   一个 MTR 执行过程中可能产生若干条 redo 日志，这些 redo 日志是一个不可分割的组，所以**并不是每生成一条 redo 日志就将其插入到 log buffer 中**，而是将每个 MTR 运行过程中产生的日志**先暂时存到一个地方**；当该 MTR 结束的时候，再将过程中产生的一组 redo 日志全部复制到 log buffer 中。
    
*   log buffer 结构示意图
    
    事务 T1 的两个 MTR：mtr_t1_1 和 mtr_t1_2
    
    事务 T2 的两个 MTR：mtr_t2_1 和 mtr_t2_2
    
    不同的事务是可能并发执行的，所以 T1、T2 的 MTR 可能是交替执行的。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86zWI0ibjdDFSK7m2yBtFUskqV4JxHB5nAnpGsVEkQ7ZFOImkyQGtHw1Q/640?wx_fmt=png)

五、redo 日志文件  

--------------

### 5.1> redo 日志刷盘时机

*   MTR 运行过程中产生的一组 redo 日志在 MTR 结束时会被复制到 log buffer 中。可是这些日志总在内存里也不是办法，在一些情况下它们会被刷新到磁盘中。
    
*   哪些情况下会被刷新到磁盘中呢？
    

*   1> log buffer 空间**不足 50%** 的时候
    
*   2> **事务提交的时候**
    
    引入 redo 日志后，虽然在事务提交时可以不把修改过的 Buffer Pool 页面立即刷新到磁盘，但是为了保证持久性，必须要把页面修改时所对应的 **redo 日志刷新到磁盘**。否则假如系统崩溃后，无法将该事务对页面所做的修改恢复过来。
    
*   3> 后台有一个线程，大约以**每秒一次**的频率将 log buffer 中的 redo 日志刷新到磁盘。
    
*   4> **正常关闭服务器**时
    
*   5> 做 **checkpoint** 时
    

### 5.2> redo 日志文件组  

*   **MySQL 的数据目录**（使用 show variables like 'datadir'; 可查看）下默认有名为 **ib_logfile0** 和 **ib_logfile1** 的两个文件，log buffer 中的日志在默认情况下就是刷新到这两个磁盘文件中。可以通过下面几个启动选项来调节：
    

*   1> **innodb_log_group_home_dir**
    
    指定了 redo 日志文件所在的目录，默认值为当前的数据目录。
    
*   2> **innodb_log_file_size**
    
    指定了每个 redo 日志文件的大小，在 MySQL 5.7.22 版本中的默认值为 **48MB**。
    
*   3> **innodb_log_files_in_group**
    
     指定了 redo 日志文件的个数，默认值为 2，最大值为 100。
    

*   redo 日志文件组示意图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86HIZqDkBEfsiamvtKDrJcy5ib7amibRLHVgJXOhdhwjuS6vO1TxuibQ6wUA/640?wx_fmt=png)

【注】按箭头的**顺序**进行日志写入。

### 5.3> redo 日志文件（磁盘上的文件）格式  

*   由于 log buffer 本质是一片**连续**的内存空间，被划分成若干个 **512 字节**大小的 block；将 log buffer 中的 redo 日志刷新到磁盘的本质就是把 block 的镜像**写入日志文件**中，所以 redo 日志文件其实也是由若干个 512 字节大小的 block 组成的。如下图所示：
    
*   redo 日志文件结构图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86lCSweyicefYkE7Gk2ibqgw4jrFlwKY8RUeNKLNT2Iia5rtVqSefla6QXA/640?wx_fmt=png)

【注】  

*   redo 日志文件组中，每个文件的**大小都一样**，**格式页一样**。
    
*   前 2048 个字节（也就是前 4 个 block），用来存储一些**管理信息**。
    

*   从第 2048 字节往后的字节，用来存储 log buffer 中的 **block 镜像**。
    

*   所以前面所说的循环使用 redo 日志文件，其实是从每个日志文件的前 2048 个字节开始算起的。
    
*   log file header 的参数说明如下：
    

*   1> 【4B】LOG_HEADER_FORMAT
    
    redo 日志的版本，在 MySQL 5.7.22 中永远为 1。
    
*   2> 【4B】LOG_HEADER_PAD1
    
    用于字节填充，没什么实际意义
    
*   3> **【8B】LOG_HEADER_START_LSN**
    
    **标记本 redo log 文件偏移量为 2048 字节处对应的 lsn 值。**
    
*   4> 【32B】LOG_HEADER_CREATOR
    
    标记本 redo 日志文件的创建者是谁。正常运行时该值为 MySQL 的版本号，如 “SQL 5.7.22”
    
*   5>【4B】LOG_BLOCK_CHECKSUM
    
    本 block 的校验值；所有 block 都有该值，我们不用关心。
    

*   checkpoint1 或 checkpoint2 的参数说明如下：
    

*   1> **【8B】LOG_CHECKPOINT_NO**
    
    **服务器执行 checkpoint 的编号，每执行一次 checkpoint，该值就加 1。**
    
*   2> **【8B】LOG_CHECKPOINT_LSN**
    
    **服务器在结束 checkpoint 时对应的 lsn 值；系统在崩溃后恢复时将从该值开始。**
    
*   3> **【8B】LOG_CHECKPOINT_OFFSET**
    
    **上个属性中的 lsn 值在 redo 日志文件组中的偏移量。**
    
*   4> **【8B】LOG_CHECKPOINT_LOG_BUF_SIZE**
    
    **服务器在执行 checkpoint 操作时对应的 log buffer 的大小。**
    
*   5> 【4B】LOG_BLOCK_CHECKSUM
    
    本 block 的校验值；所有 block 都有该值，我们不用关心。
    

*   系统中 **checkpoint** 的相关信息其实只存储在 redo 日志文件组的**第一个日志文件中**。
    

六、log sequence number  

------------------------

*   什么是 lsn
    
    lsn 全称为 log sequence number，是一个**全局变量**，用来**记录当前总共已经写入的 redo 日志量**。lsn 初始值为 **8704**，也就是说，一条 redo 日志也没写入的时候，lsn 的值就是 8704。
    
*   MTR 写入 log buffer 后，lsn 的变化示意图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn868r27AEkffsjO2L5lCYayoa9fdofEnVb1b6VPj3iaRicjg45SPAS2AiadQ/640?wx_fmt=png)

### 6.1> flushed_to_disk_lsn  

*   如何知道有哪些日志被刷新到磁盘中了
    
    一个名为 **buf_next_to_write** 的全局变量，用来标记当前 log buffer 中已经有哪些日志被刷新到磁盘中了。如下所示： 
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86ibJcBSr5HPWibwPPMt4hWRTT7mf9QBMjAemEMbcAfBia78cLFqpo720cg/640?wx_fmt=png)

*   演示一下 block 写入 redo 日志文件的过程
    
*   首先：系统在第一次启动后，向 **log buffer 中**写入了 mtr_1(8,716～8,916)、mtr_2(8,916～9,948)、mtr_3(9,948~10,000) 这 3 个 MTR 产生的 redo 日志。此时的 lsn 已经增长到了 10,000，由于没有刷新操作，此时 flushed_to_disk_lsn 的值仍为初始值 8,704。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86pYB8zQ4L6Usa3MtJic0ByFLLrXPjetUV0kBTIug7AWMOSSDt4X9RwIA/640?wx_fmt=png)

*   随后：将 log buffer 中的 block 刷新到 **redo 日志文件中**。假设讲 mtr_1 和 mtr_2 的 redo 日志刷新到磁盘，那么 flush_to_disk_len 就应该增长 mtr_1 和 mtr_2 写入的日志量，即：增长到 9,948。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86AVwL9wKJqIMcRj20iaqfD361F87ITPREufFvt7mITicMt0uXC5vHJkoQ/640?wx_fmt=png)

### 6.2> lsn 值与 redo 日志文件组中的偏移量的对应关系  

*   lsn 值和 redo 日志文件组偏移量的对应关系
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86123mGBROltibu7icktk9kSuBo35QnPtpJkPKUAy5MdniczY4HbY8KGOMg/640?wx_fmt=png)【注】偏移量从 2,048 开始；lsn 值从 8,704 开始；

### 6.3> flush 链表中的 lsn  

*   一个 MTR 代表对底层页面的一次原子访问，在访问过程中可能会产生一组不可分割的 redo 日志；在 MTR 结束时，会把这一组 redo 日志写入到 log buffer 中。除此之外，在 MTR 结束时还有一件非常重要的事情要做，就是把在 MTR 执行过程中修改过的页面加入到 Buffer Pool 的 flush 链表中。
    
*   第一次修改某个已经加载到 Buffer Pool 中的页面时，就会把这个页面对应的**控制块**插入到 flush 链表的头部；之后再次修改该页面时，由于它已经在 flush 链表中，所以就**不再次插入**了。也就是说，flush 链表中的脏页是按照页面的**第一次修改时间**进行排序的。
    

*   在这个过程中，会在缓冲页对应的控制块中记录两个关于页面何时修改的属性：
    

*   oldest_modification
    
    **第一次修改** Buffer Pool 中的某个缓冲页时，就将修改该页面的 MTR **开始时对应的 lsn 值**写入这个属性。
    
*   newest_modification
    
    **每修改**一次页面，都会将修改该页面的 MTR **结束时对应的 lsn** 值写入这个属性。也就是说，该属性表示页面最近一次修改后对应的 lsn 值。
    

*   下面进行举例演示，如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86H07fg9nEVbpO3sqsdcvFnC5KRO3wQFGXoZGsH19SM6lvcxyAZvq3Cg/640?wx_fmt=png)【解释】

*   页 b 第二次被修改了，所以只改变 newest_modification 的值，oldest_modification 的值不变化。
    

*   所以，综上所述，在 flush 链表中，前面的脏页修改的时间比较晚，后面的脏页修改的时间比较早。
    

七、checkpoint  

---------------

*   如果 redo 日志对应的脏页已经刷新到磁盘中了，那么它也就失去了存在的意义了。它所占用的磁盘空间就可以被后续的 redo 日志所重用。
    
*   针对如下图：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86Aqw1oxKx2IMwy1zCcDF7IhYb0RVxAHbOnNwmkFJkad94ReiagSlNMCw/640?wx_fmt=png)【解释】

*   mtr1 和 mtr2 生成的 redo 日志虽然已经写到磁盘上的 log file 中了，但是它们修改的脏页仍然留在 Buffer Pool 中，所以它们的 redo 日志不可以被覆盖。
    
*   随着系统运行，如果页 a 从 Buffer Pool 中刷到了磁盘上，那么页 a 对应的控制块就会**从 flush 链表中移除掉**。而且，它的 redo 日志占用的空间就可以被覆盖掉了。
    

*   InnoDB 通过全局变量 **checkpoint_lsn**，来表示当前系统中可以被覆盖的 redo 日志总量是多少。这个变量的初始值也是 **8704**（因为 lsn 的初始值就是 8704）。
    
*   比如，现在页 a 被刷新到了磁盘上，mtr1 生成的 redo 日志就可以被覆盖了，所以可以进行一个增加 checkpoint_lsn 的操作。我们把这个过程称为**执行一次 checkpoint**。
    
*   执行一次 checkpoint 可以分为两个步骤
    

*   步骤一：计算当前系统中**可以被覆盖**的 redo 日志对应的 lsn 值最大是多少。比如，当前系统中页 a 已经被刷新到磁盘了，那么 **flush 链表的尾节点**就是页 c。该节点就是当前系统中最早修改的脏页了，它的 oldest_modification=8916，我们就把 8916 赋值给 checkpoint_lsn（也就是说在 redo 日志对应的 lsn 值小于 8916 时，就可以被覆盖掉）。
    
*   步骤二：将 checkpoint_lsn 与对应的 redo 日志文件组偏移量以及此次 checkpoint 的编号写到日志文件的管理信息（checkpoint1 或者 checkpoint2）中 **checkpoint_no** 变量，用来统计目前系统执行了多少次 checkpoint，每执行一次 checkpoint，该变量就 + 1；前面我们也说过，计算一个 lsn 值对应的 redo 日志文件组偏移量是很容易的，所以可以计算出该 checkpoint_lsn 在 redo 日志文件组中对应的偏移量 **checkpoint_offset**；然后将 checkpoint_lsn、checkpoint_no、checkpoint_offset 这 3 个值写到 redo 日志文件组的管理信息中。
    

*   每个 redo 日志文件都有 2048 字节的管理信息，但是上述关于 checkpoint 的信息只会被写到日志文件组中的**第一个日志文件**的管理信息中。
    
*   _不过，应该存储到 checkpoint1 还是 checkpoint2 中呢？_
    
    答：当 checkpoint_no 的值为偶数时，就写到 checkpoint1 中；是奇数时，就写到 checkpoint2 中。
    
*   redo 日志文件组中各个 lsn 值的关系
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86eZbKECFiaBPJod0TJNEHcUvZicOvBsWOl2VEOfiaCTV7HfEib74pZjpaVA/640?wx_fmt=png)

八、用户线程批量从 flush 链表中刷出脏页  

--------------------------

*   一般情况下，针对 Buffer Pool 中的刷脏页操作，都是**后台线程**对 **LRU 链表**和 **flush 链表**进行刷脏页操作的，主要是因为刷脏操作比较慢，不想影响用户线程处理请求。
    
*   但是，如果修改操作十分频繁，导致 redo 日志操作频繁，系统 lsn 值增长过快。如果后台线程的刷脏操作不能将脏页快速刷出，系统将无法及时执行 checkpoint，可能就需要**用户线程**从 flush 链表中把那些**最早修改的脏页**（oldest_modification 较小的脏页）同步刷新到磁盘。这样这些脏页对应的 redo 日志就没有用了，然后就可以去执行 checkpoint 了。
    

九、查看系统中的各种 lsn 值  

-------------------

*   可以使用 **SHOW ENGINE INNODB STATUS \G** 命令查看当前各种 lsn 值的情况。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8wPbkg45p0pEzwkLrAnn86YGJJmjFsXPDWAZ0assbaYGicgswNd2t3riaZUN8XQNhLwPwVIibGG325A/640?wx_fmt=png)

*   Log sequence number
    
    lsn 值，即：当前系统已经写入的 redo 日志量。包括写入到 log buffer 中的 redo 日志量。
    
*   Log flushed up to
    
    flushed_to_disk_lsn，即：写入磁盘 log file 文件的 redo 日志量。
    
*   Pages flushed up to
    
    表示 flush 链表中被最早修改的那个页面对应的 oldest_modification 属性值。
    
*   Last checkpoint at
    
    checkpoint_lsn 值。
    

十、innodb_flush_log_at_trx_commit 的用法  

---------------------------------------

*   为了保证事务的持久性，用户线程在事务提交时，需要将该事务执行过程中产生的所有 redo 日志都刷新到磁盘中。这个规则我们可以通过系统变量 **innodb_flush_log_at_trx_commit** 来进行配置修改，该变量有如下 3 个可选值:
    

*   0：表示在事务提交时，不立即向磁盘同步 redo 日志，这个任务交给后台线程来处理；
    
*   1：表示在事务提交时，需要将 redo 日志同步到磁盘。（默认值）
    
*   2：表示在事务提交时，需要将 redo 日志写到操作系统的缓冲区中，但并不需要保证将日志真正刷新到磁盘。如果操作系统挂掉了，则数据丢失。
    

十一、崩溃恢复  

----------

### 11.1> 确认恢复的起点

*   我们只需要把 checkpoint1 和 checkpoint2 这两个 block 中的 checkpoint_no 值读出来并比较一下大小，哪个 checkpoint_no 值更大，就说明哪个 block 存储的就是最近一次的 checkpoint 信息。这样就能拿到最近发生的 checkpoint 对应的 checkpoint_lsn 值以及它在 redo 日志文件组中的偏移量 checkpoint_offset。
    

### 11.2> 确认恢复的终点  

*   前面说过，redo 日志是顺序写入的，写满一个 block 之后再往下一个 block 中写，所以，根据 block 的 log block header 部分中有一个名为 LOG_BLOCK_HDR_DATA_LEN 的属性值，该值记录了当前 block 中使用咯多少字节的空间。对于被填满的 block 来说，该值永远为 512，如果不为 512，则表示此次崩溃恢复中需要扫描的最后一个 block。
    
*   综上所述：当因崩溃而恢复系统时，**只需要从 checkpoint_lsn 在日志文件组中对应的偏移量开始，一直扫描 redo 日志文件中的 block，直到某个 block 的 LOG_BLOCK_HDR_DATA_LEN 值不等于 512 为止**。
    

### 11.3> 怎么恢复  

*   使用哈希表
    

*   根据 redo 日志的 spaceID 和 page number 属性计算出哈希值，把 spaceID 和 page number 相同的 redo 日志放到哈希表的同一个槽位中。如果哈希值相同，则使用链表连接起来。（链表按照生成的先后顺序连接，redo1——>redo2——redo3... ——>redoN）
    
*   这样就可以在恢复过程中，针对同一个页面一次性的修复好，避免了很多读取页面的随机 I/O，加快恢复速度。
    

*   跳过已经刷新到磁盘中的页面
    

*   对于 lsn 值不小于 checkpoint_lsn 的 redo 日志，它所对应的脏页不能确定是否已经刷到磁盘中。
    
*   原因是在最近执行的一次 checkpoint 后，后台线程可能又不断地从 LRU 链表和 flush 链表中将一些脏页刷出 Buffer Pool。
    
*   解决办法：
    
    在 File Header 中有一个称为 FIL_PAGE_LSN 的属性，该属性记载了最近一次修改页面时对应的 lsn 值（其实就是页面控制块中的 newest_modification 值）。如果在执行了某次 checkpoint 之后，有脏页被刷新到磁盘中，那么该页对应的 FIL_PAGE_LSN 代表的 lsn 值肯定大于 checkpoint_lsn 的值。所以符合这种情况的页面，就不需要进行恢复了。