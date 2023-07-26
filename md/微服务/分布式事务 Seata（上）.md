![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ656ST8vS4bDTUNZCdGE71QXA6o48CaGkeAAAj8CsMSrpD16wKXBn9Dpg/640?wx_fmt=png)

〇、前提知识点  

==========

0.1> 事务隔离级别
-----------

*   _什么是事务的隔离性？_
    
    如果**多个事务**访问了**同样的一条数据**，那么会造成数据的一致性问题。这就要求我们使用某种手段来强制让这些事务按照顺序一个一个单独地执行，或者最终执行的效果和单独执行一样。也就是说我们希望让这些事务 “隔离” 地执行，互不干涉。这也就是事务的隔离性。
    
*   当多个事务对同一条数据进行写操作的时候，就会涉及到一致性的问题。我们可以通过串行化来解决。但是却会造成一定的性能损失。我们思考，_是否可以牺牲一部分隔离性来换取性能上的提升呢？_是的，当然可以。不过我们首先需要搞明白，_多个事务在不进行串行化执行的情况下，到底会出现哪些一致性问题？_
    

### 0.1.1> 事务并发执行时遇到的问题  

#### 0.1.1.1> 脏写

*   _什么是脏写？_
    
    如果一个事务 T2 **修改了**另一个**未提交事务** T1 **修改过**的数据，这就意味着发生了脏写现象。
    
*   脏写引发的**一致性**问题：
    
    我们希望 x 值和 y 值始终相同。下面是事务 T1 和事务 T2 对 x 值和 y 值的操作：
    
    **w1[x=1] w2[x=2] w2[y=2] c2 w1[y=1] c1**
    
    最终导致了 x=2 y=1，x 值和 y 值不相同，破坏了一致性需求。
    
*   脏写引发的**原子性和持久性**问题：
    
    比方说有 x=0 和 y=0 这两个数据项，下面是事务 T1 和事务 T2 对 x 值和 y 值的操作：**w1[x=2] w2[x=3] w2[y=3] c2 a1**
    
    由于 T1 执行了回滚操作，即 x=0（将 x 置为最初状态），那么相当于对 T2 对数据库所做的修改进行了**部分回滚**（即：T2 只回滚了对 x 做的修改，而不回滚对 y 做的修改），那么这就影响到了事务的**原子性**。
    
    而 T2 已经提交了，但是却被 T1 的回滚造成了自己修改的数据也被回滚了，破坏了 T2 事务的**持久性**。
    

#### 0.1.1.2> 脏读  

*   _什么是脏读？_
    
    脏读就是指当一个事务 T1 正在访问数据，并且对数据进行了修改，而这种修改**还没有提交**到数据库中，这时，另外一个事务 T2 也访问这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是脏数据 (Dirty Data)，依据脏数据所做的操作可能是不正确的。
    
*   操作流程如下图所示:
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65W74Z4uspPNpPhpQfYfboHB4v08I7FD9YblnyBDMiaSz6MHkFs14XQSA/640?wx_fmt=png)

*   脏读引发的**一致性**问题：
    
    事务 T1 和事务 T2 访问 x 和 y 这两个值，我们一致性需求就是让 x 值和 y 值始终相同，x 和 y 的初始值都是 0。现在并发执行事务 T1 和 T2 如下所示：
    
    **w1[x=1] r2[x=1] r2[y=0] c2 w1[y=1] c1**
    
    很显然 T2 是个只读事务，读取到了事务 T1 未提交事务的值，所以 T2 读到的 x=1，y=0，不符合 x=y 的一致性。数据库的不一致状态是不应该暴露给用户的。
    
*   脏读的严格解释
    
    **w1[x] ... r2[x] ... (a1 and c2 in any order)**
    
    也就是 T1 先修改了数据项 x 的值，然后 T2 又读取到了**未提交事务** T1 针对数据项 x 修改后的值，之后 T1 回滚而 T2 提交。这就意味着 T2 **读到了一个根本不存在的值**。
    

#### 0.1.1.3> 不可重复读  

*   _什么是不可重复读？_
    
    指在一个事务 T1 内，多次读同一数据。在这个事务 T1 还没有结束时，另外一个事务 T2 修改并提交了该同一数据。那么，事务 T1 两次读到的数据可能是不一样的。这样就发生了**在一个事务内两次读到的数据是不一样的**，因此称为是不可重复读。
    
*   操作流程如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65nmNMxbU9ldribHRZQmHg6jRaskqxXueTNPpsAyxZ6F5qia3s8ibLicASoQ/640?wx_fmt=png)

*   不可重复读的**一致性**问题：
    
    事务 T1 和事务 T2 访问 x 和 y 这两个值，我们一致性需求就是让 x 值和 y 值始终相同，x 和 y 的初始值都是 0。现在并发执行事务 T1 和 T2 如下所示：
    
    **r1[x=0] w2[x=1] w2[y=1] c2 r1[y=1] c1**
    
    很显然 T1 是个只读事务，最终读取到的是 x=0，y=1，很显然这是一个不一致的状态，这种不一致的状态是不应该暴露给用户的。
    
*   不可重复读的严格解释
    
    **r1[x] ... w2[x] ... c2 ... r1[x] ... c1**
    
    也就是 T1 先读取了数据项 x 的值，然后 T2 又修改了 x 的值，之后 T2 提交事务，然后 T1 再次读取数据项 x 的值时会得到与第一次读取时不同的值。
    

#### 0.1.1.4> 幻读  

*   什么是幻读
    
    如果一个事务 T1 先根据**某些搜索条件**查询出一些记录，在该事务未提交时，另一个事务 T2 操作了一些符合那些搜索条件的记录（insert、delete、update），就意味着发生了幻读现象。
    
*   操作流程如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65Y2MXr6uKnm0w9Eoz3oGF9sgJvxyM0vlSMyeGNibRzoS34EmxibvleeUw/640?wx_fmt=png)

*   幻读的**一致性**问题：
    
    **r1[P] ... w2[y in P] ... c2 ... r1[P] ... c1**
    
    T1 先读取符合搜索条件 P 的记录，然后 T2 写入了符合搜索条件 P 的记录。之后 T1 再读取符合搜索条件 P 的记录时，会发现两次读取的记录是不一样的。
    

### 0.1.2> SQL 标准中的 4 种隔离级别  

*   四种隔离级别如下表所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65jw4mMW7bbG7sbocVNsq4Sdt73ffaAJY5iclDu01hFKA3a95hXou7pLA/640?wx_fmt=png)

【注】  

*   由于无论哪种隔离级别，**都不允许脏写的情况发生**，所以没有列入到表格中。
    
*   MySQL 与 SQL 标准不同的一点就是，**MySQL 在 REPEATABLE READ 隔离级别下很大程度地避免了幻读现象**。
    

0.2> 分布式场景下数据一致性问题  

---------------------

*   在系统开发过程中，我们绝大部分时间使用的是关系型数据库，那么当数据量不大的时候，我们通过单库单表就可以保证某个业务数据的存储能力了，并且通过数据库对事务的支持，可以很好的解决数据一致性问题。但是，当数据量增大了之后，不仅仅是单表查询效率变低下，而且数据所占用空间的限制，都会促使着我们采用分库分表的解决方案。但是当涉及到分库的时候，却打破了我们依赖数据库控制事务一致性的便捷方式，只能我们自己采用其他方案来进行事务的控制，因此分布式场景下数据一致性问题就显著的突显了出来。
    
*   既然谈到分库，我们就来看一下常见的两种分库方式，即：**水平分库**和**垂直分库**。
    

0.2.1> 水平分库  

--------------

*   还是回到上面的话题，当公司在创业初期，业务刚刚起步，我们的订单表每天能产生 5 万条数据，那么放到一张表中完全可以应对现在的业务需求。
    
*   但是随着公司的发展，业务逐一的在推进，订单量一年后，突增为 100 万，那为了应对公司业务的发展和高速增长的订单量，就会将数据库扩展为 A，B，C 这三个库，并且在每个数据库中按照相同的表结构复制出多份，来平均分摊每天产生的订单。那么这种分库的方式，就是水平分库。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65IvIhk3bDSxoa7yhCKKUvJjHKTD07icu9gA5Q13AIfERVDJYu0BRqJpw/640?wx_fmt=png)

0.2.2> 垂直分库  

--------------

*   随着公司发展，我们都无法避免要从创业初期的单体架构拆分为微服务架构，那么针对不同的业务域会进行服务的拆分，那么面对着代码层面的解耦解决了之后，面对数据层面的解耦如何处理呢？那么，很容易我们就会想到，要将 A 库按照不同的业务域进行拆分，比如：拆分成订单库、商品库存库、支付库等等。那么面对着这种数据库的拆分方式，就是垂直分库了。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65MKYWyM2hN1T8dJAjsA5d0fPicficmYn33EKicJib56OgA9RCFbicAekyKag/640?wx_fmt=png)

一、分布式事务解决方案  

==============

*   我们以快捷支付的方式在商城购买商品为例（即：下完订单自动支付），看一下分布式事务在实现上与非分布式的区别。前提是我们针对订单服务、库存服务和支付服务这 3 个服务都有对应的 3 个数据库，分别是订单库、库存库和支付库。如果我们平时以声明式事务来编码，那么它与本地事务在编码上没什么区别，都是标注一个 @Transaction 注解而已，但是如果以编程式事务来实现的话，就能在写法上看出差异了，伪代码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65zKMZVbIsV66XNK44zAKFvy3whhs0IDfBiaya7nhicfCMHwIEVPBZFxtg/640?wx_fmt=png)

【解释】

*   由于订单，库存和支付是 3 个服务和 3 个数据库，所以，我们需要分别开启这 3 个数据库事务，并且执行完毕后，分别执行 3 次 commit 操作。
    
*   我们假设，如果在调用 payTransaction.commit() 方法时，出现了异常，那么由于 order 和 stock 已经成功的调用了 commit，所以，无法执行 rollback 操作了。而只有 pay 服务可以正常回滚。那么，就造成了分布式事务的不一致性。为了解决类似这样的问题，分布式事务的解决方案就应运而生了。
    

1.1> 刚性事务（也叫强一致性事务）  

----------------------

### 1.1.1> XA

*   为了解决分布式事务一致性问题，X/Open 组织提出了一套名为 X/Open XA（XA 的缩写为：eXtended Architecture）的处理事务架构，其核心内容是定义了全局的**事务管理器**（Transaction Manager，用于协调全局事务）和局部的**资源管理器**（Resource Manager，用于驱动本地事务）之间的通信接口。XA 接口是双向的，能在一个事务管理器和多个资源管理器之间形成通信桥梁，通过协调多个数据源的一致动作，实现全局事务的统一提交或统一回滚。
    
*   XA 并不是 Java 的技术规范（XA 提出的时候 Java 还没有诞生），而是一套跟语言无关的通用规范，所以 Java 中专门定义了 JSR 907 Java Transaction API，基于 XA 模式在 Java 语言中实现了全局事务处理的标准，这也就是我们现在熟知的 JTA（Java Transaction API）。
    
*   JTA 最主要的两个接口如下所示：
    
*   事务管理器接口（javax.transaction.TransactionManager）
    
    这套接口用于为 Java EE 服务器提供容器事务（由容器自动负责事务管理）。JTA 还提供了另外一套 javax.transaction.UserTransaction 接口，用于通过程序代码手动开启、提交和回滚事务。
    
*   满足 XA 规范的资源定义接口（javax.transaction.xa.XAResource）
    
    任何资源（JDBC、JMS 等）如果想要支持 JTA，只要实现 **XAResource 接口**中的方法即可。
    

### 1.1.2> 2PC  

*   为了保证整个事务的一致性，XA 将事务提交拆分成两阶段，即：**二段式提交**（2 Phase Commit，2PC）协议。交互时序示意图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65FkKnpvPIia9BZmnAe4dSfjnvrtn6V6Nic98oaRRXN4lG5NE6J1FzicY7w/640?wx_fmt=png)

*   第一阶段：**准备阶段（投票阶段）**
    

*   协调者询问事务的所有参与者是否准备好提交了，准备好恢复 Prepared，否则恢复 Non-Prepared。
    
*   准备操作是在 redoLog 中记录全部事务提交操作所要做的内容，它与本地事务中真正提交的区别只是暂时**不写入最后一条 Commit Record** 而已，这意味着在做完数据持久化后并不释放持有的锁。
    

*   第二阶段：**提交阶段（执行阶段）**
    

*   如果协调者收到所有事务参与者都回复了 Prepared 消息，则先自己在本地持久化事务状态为 Commit，然后向所有参与者发送 Commit 指令，让所有参与者立即执行提交操作。
    
*   否则，任意一个参与者回复了 Non-Prepared 消息，或者任意一个参与者超时未回复是，协调者将在自己完成事务状态为 Abort 持久化后，向所有参与者发送 Abort 指令，让参与者立即执行回滚操作。
    
*   对数据库来说，提交阶段操作是很轻量级的，仅仅是持久化一条 Commit Record 而已，通常能够快速完成。
    
*   只有收到 Abort 指令时，才需要根据回滚日志清理已提交的数据，这个操作相对负载会重一些。
    

*   2PC 的缺点
    

*   **单点问题**
    
    如果协调者发生了宕机，所有的参与者都会收到影响。如果协调者一直没有恢复，没有正常发送 Commit 或 Rollback 的指令，那么所有的参与者都必须一直等待。
    

*   **性能问题**
    
    由于所有参与者相当于被绑定为一个统一调度的整体，再次期间要经过 2 次远程服务调用，3 次数据持久化（1> 准备阶段写 redoLog；2> 协调者做状态持久化；3> 提交阶段在日志写入提交记录），整个过程将持续到参与者集群中最慢的那一个处理操作结束为止，这导致了 2PC 的性能通常比较差。
    

*   **一致性风险**
    
    尽管提交阶段时间很短，但是这仍是一段明确存在的危险期，如果协调者在发出准备指令后，根据收到各个参与者发回的信息确定事务状态时可以提交的，协调者会先持久化事务状态，并提交自己的事务，如果这个时候网络忽然断开，无法再通过网络向所有参与者发出 Commit 指令的话，就会导致**部分数据（协调者的）已提交，但部分数据（参与者）未提交**，且没有办法回滚，产生数据不一致的问题。
    

### 1.1.3> 3PC  

*   为了缓解 2PC 中协调者的**单点问题**和**准备阶段的性能问题**，后续发展出了 “三段式提交”，即：3PC 协议。交互时序示意图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65reGNTfXXalbbGz2p1YHaVc2lQKpYGzSQ29r7yX8HW91bUmib4ZleSFg/640?wx_fmt=png)

*   3PC 把原本 2PC 中的**准备阶段**再细分为两个阶段，即：**CanCommit 阶段**和 **PreCommit 阶段**。把**提交阶段**改称为 **DoCommit 阶段**。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65RFCZRY28NAxrrUEvDYmAW7x0oOiaU2EslSydSL51Ir5WHGDPdTFAicIg/640?wx_fmt=png)

【解释】

*   CanCommit 阶段是一个询问阶段，即：协调者让每个参与的数据库根据自身状态，评估该事务是否有可能顺利完成。
    
*   将 2PC 的准备阶段分为 CanCommit 阶段和 PreCommit 阶段，主要是因为 2PC 的准备阶段是一个重负载的操作，因为一旦协调者发出开始准备的消息，每个参与者都将马上开始写 redoLog，它们所涉及的数据资源即被锁住，如果此时某一个参与者宣告无法完成提交，那么所有的参与者都做了一轮无用功了。所以，增加一轮询问阶段——接：CanCommit 阶段，如果都得到了正面的响应，那么事务能够成功提交的把握就比较大了，也就减少了所有参与者全部回滚的风险了。
    
*   综上所述，在事务需要回滚的场景中，3PC 的性能要比 2PC 好很多。但是，在事务能够正常提交的场景中，2PC 和 3PC 的性能都很差，甚至由于 3PC 多了一次询问，性能还要更差一些。
    

1.2> 柔性事务（最终一致性事务）  

---------------------

### 1.2.1> 可靠事件队列

*   以快捷支付为例，当用户下单的时候，自动就进行了支付扣款操作。那么创建订单、扣减库存和支付扣款这三个操作就应该是一个原子性的操作。那么如果我们采取可靠事件队列的方式，则流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65GD4gGu28WT516UMDMicAmkEn4KiaO6kAHz5WbzekfgYSGEwksusKPXoQ/640?wx_fmt=png)

【解释】  

*   首先，执行创建订单操作，如果执行成功，就向消息事件表中插入事务数据，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65VrY9a4s4TmdPbY2xZXEQ06IXHfOYhldJgzsxcIleC0MUFSibZvtbshA/640?wx_fmt=png)

【注】创建订单和写入消息事件表，使用同一个本地事务写入订单服务的数据库。  

*   其次，消息服务定时轮询消息事务表，**将状态为 “进行中” 的消息发送到 MQ 中**，并且有对应的库存服务和支付服务进行消费调用操作。如果执行成功，则将响应的状态修改为 “已完成”。当某个事务 ID 下所有的行为状态都是“已完成” 状态时，则表示整个事务总体成功的，即：达到最终一致性的状态。
    
*   如果扣减库存或者账户扣款失败了，那么状态依然是 “进行中”。当消息服务定时轮询再次抓取到状态为“进行中” 的消息时，会再次重试扣减库存或扣款操作。所以，这些接口必须具备**幂等性**，否则就会出现重复操作的问题。
    
*   可靠事件队列的特点就是，如果某个服务无法完成工作，那么就**会一直重试**，直到操作成功或是人工介入。由此可见，可靠事务队列只要第一步业务完成了，后续就没有失败回滚的概念，**只许成功，不许失败**。
    

### 1.2.2>  TCC  

*   TCC 是 “**Try-Confirm-Cancel**” 的缩写，是常见的分布式事务机制。
    
*   在具体实现上，TCC 较为繁琐，它是一种**业务侵入式较强**的事务方案，要求业务处理过程必须拆分为 “**预留业务资源**” 和 “**确认 / 释放消费资源**” 两个子过程。如同 TCC 的名字所示，它分为以下三个阶段：
    

*   **Try（尝试执行阶段）**
    
    完成所有业务可执行性的检查（保障一致性），并且预留好全部需要用到的业务资源（保障隔离性）。
    

*   **Confirm（确认执行阶段）**
    
    不进行任何业务检查，直接使用 Try 阶段准备的资源来完成业务处理。
    
    Confirm 阶段可能会重复执行，因此本阶段执行的操作需要具备幂等性。
    

*   **Cancel（取消执行阶段）**
    
    释放 Try 阶段预留的业务资源。
    
    Cancel 阶段可能会重复执行，因此本阶段执行的操作需要具备幂等性。
    

*   上述我们了解了 TCC 模式，那么下面还是以快捷支付为例，看一下 TCC 的执行过程：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ650UXN3icCIic91c3dOj2XOvhlJxJn8oygW04XsJMs3J2D7su9bKx5VOdA/640?wx_fmt=png)

【总结】  

*   由上述操作过程可见，TCC 其实有点类似 2PC 的准备阶段和提交阶段。
    
*   但 TCC 是在用户代码层面，而不是在基础设施层面，这为它的实现带来了较高的灵活性，可以根据需要设计资源锁定的粒度。
    
*   TCC 在业务执行时只操作预留资源，几乎不会涉及锁和资源的争用，具有很高的性能潜力。
    
*   但 TCC 也带来了更高的开发成本和业务侵人性，即更高的开发成本和更换事务实现方案的替换成本，所以，通常我们并不会完全靠裸编码来实现 TCC，而是基于某些分布式事务中间件 (管如阿里开源的 Seata）去完成，尽量减轻一些编码工作量。
    

### 1.2.3>  SAGA  

*   在一些场景下，如果无法实现冻结、解冻、扣减这样的操作的话，那么 TCC 中的第一步 Try 阶段就无法施行了。此时，我们可以考虑使用另外一种柔性事务——SAGA 事务。
    
*   SAGA 事务大致思路是：把一个大事务分解为可以交错运行的一系列子事务集合。原本 SAGA 的目的是避免大事务长时间锁定数据库资源，后来才发展成将一个分布式环境中的大事务分解为一系列本地事务的设计模式。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ657tOpicT48S12ysEjPpneXkqlViaLWxKSiaWgRqXNyfkwpR3n7HyDvDPwQ/640?wx_fmt=png)

*   SAGA 由两部分操作组成：
    

*   将大事务拆分成若干个小事务，将整个分布式事务 T 分解为 n 个子事务，命名为 T1，T2，T3 ... ，Tn。每个子事务都应该是或者能被视为原子行为。如果分布式事务能够正常提交，其对数据的影响（即：最终一致性）应与连续按顺序成功提交 Ti 等价。
    
*   为每一个子事务设计对应的补偿动作，命名为 C1，C2，C3...，Cn。Ti 与 Ci 必须满足以下条件：
    

*   Ti 与 Ci 都具备幂等性。
    
*   Ti 与 Ci 满足交换律，即：无论先执行 Ti 还是先执行 Ci，其效果都是一样的。
    
*   Ci 必须能成功提交，即：不考虑 Ci 本身提交失败被回滚的情形，如果出现就必须持续重试直至成功，或者被人工介入为止。
    

*   如果 T1 到 Tn 均成功提交，那事务顺利完成。否则，要采取以下两种恢复策略之一
    

*   恢复策略一：**正向恢复**（Forward Recovery）
    
    如果 Ti 事务提交失败，则一直对 Ti 进行重试，知道成功为止（最大努力交付）。这种恢复方式不需要补偿，适用于事务最终都要成功的场景。正向恢复的执行模式为：**T1，T2，...，Ti（失败），Ti（重试），...，Ti+1，...，Tn**。
    

*   恢复策略二：**反向恢复**（Backward Recovery）
    
    如果 Ti 事务提交失败，则一直执行 Ci 对 Ti 进行补偿，直至成功为止（最大努力交付）。这里要求 Ci 必须（在持续重试后）执行成功。
    
    反向恢复的执行模式为：**T1，T2，...，Ti（失败），Ci（补偿），...，C2，C1**。
    

*   与 TCC 相比，SAGA 不需要为资源设计冻结状态和撤销冻结的操作，补偿操作往往要比冻结操作容易实现的多。
    
*   SAGA 必须保证所有子事务都得以提交或者补偿，但 SAGA 系统本身也有可能会崩溃，所以它必须设计成与数据库类似的日志机制（被称为 SAGA Log）以保证系统恢复后可以追踪到子事务的执行情况，比如执行到哪一步或者补偿到哪一步了。
    
*   SAGA 事务通常也不会直接靠裸编码来实现，一般是在事务中间件的基础上完成，例如利用 Seata 的 SAGA 事务模式。
    

### 1.2.4> 基于数据补偿  

*   seata 的 AT 模式就是基于数据补偿来代替回滚思路的。AT 事务是参照了 XA 两段提交协议实现的，但是 AT 并不需要等待所有数据源都返回成功采取执行全局提交，而是通过了拦截 SQL 的方式，生成前后镜像，生成行锁，通过本地事务一起提交到操作的数据源中，相当于自己记录了重做和回滚日志。如果分布式事务成功提交，那后续清理每个数据源中对应的日志数据即可；如果分布式事务需要回滚，就根据日志数据自动产生同用于补偿的 “逆向 SQL”。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65rC8Ih3ANQCghS7mVJicmrial86zziajB6r1AGM8ms1sSRNibnM0JvmPZhw/640?wx_fmt=png)

*   采用 AT 模式，效率要比 2PC 这种阻塞式高很多。但是却**丧失了隔离性**。譬如：在本地事务提交之后，分布式事务完成之前，该数据被补偿之前又被其他操作修改过，即：出现了脏写，那么此时一个分布式事务需要回滚，就不可能在通过自动的逆向 SQL 来实现补偿，只能由人工接入来处理了。但是其实也很难通过人工进行有效处理。所以 Seata 增加了**全局锁**的机制来实现写隔离，要求本地事务提交之前，一定要先拿到针对修改记录的全局锁后才允许提交，如果没有获得全局锁就必须一直等待。这种设计以牺牲一定性能做为代价，避免了两个分布式事务中包含的本地事务修改同一个数据的情况，从而避免脏写。但是这种全局锁的方式性能消耗太大，一般不会这样做，需要根据实际业务场景进行取舍。
    

1.3> 刚性事务与柔性事务对比  

-------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65O2WyHVYiaOb50hngeLYV7URKaKiaRHFbhdFUiarzBaBqIzlIzQqFYfauw/640?wx_fmt=png)

*   一致性保障
    
    XA > TCC = SAGA > 事务消息
    
*   业务友好性
    
    XA > 事务消息 > SAGA > TCC
    
*   性能损耗
    
    XA > TCC = SAGA = 事务消息
    

二、Seata 概述  

=============

2.1> 什么是 Seata？
---------------

*   Seata 是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。
    
*   在 Seata 开源之前，其内部版本在阿里经济体内部一直扮演着应用架构层数据一致性的中间件角色，帮助经济体平稳的度过历年的双 11，对上层业务进行了有力的技术支撑。
    
*   经过多年沉淀与积累，其商业化产品先后在阿里云、金融云上售卖。2019.1 为了打造更加完善的技术生态和普惠技术成果，Seata 正式宣布对外开源，未来 Seata 将以社区共建的形式帮助用户快速落地分布式事务解决方案。
    

2.2> Seata 特色功能
---------------

*   微服务框架支持
    
    目前已支持 Dubbo、Spring Cloud、Sofa-RPC、Motan 和 gRPC 等 RPC 框架，其他框架持续集成中。
    
*   AT 模式
    
    提供无侵入自动补偿的事务模式，目前已支持 MySQL、Oracle、PostgreSQL、TiDB 和 MariaDB。
    
*   TCC 模式
    
    支持 TCC 模式并可与 AT 混用，灵活度更高。
    
*   SAGA 模式
    
    为长事务提供有效的解决方案, 提供编排式与注解式 (开发中)。
    
*   XA 模式
    
    支持已实现 XA 接口的数据库的 XA 模式，目前已支持 MySQL、Oracle、TiDB 和 MariaDB。
    
*   高可用
    
    支持计算分离集群模式，水平扩展能力强的数据库和 Redis 存储模式. Raft 模式 Preview 阶段。
    
*   在实际业务使用中，**以非侵入 AT 模式为主，TCC 模式为辅**。
    

2.3> 逻辑结构  

------------

*   Seata 有 **3 个**角色，分别为：
    

*   **TC (Transaction Coordinator) - 事务协调者**
    
    维护全局事务和分支事务的状态，推进事务两阶段处理。
    
*   **TM (Transaction Manager) - 事务管理器**
    
    与 TC 交互，用于开启、提交、回滚全局事务。
    
*   **RM (Resource Manager) - 资源管理器**
    
    与 TC 交互，负责资源的相关处理，包括分支事务的注册与状态上报。
    

*   其中，TM 和 RM 是以 **SDK** 的形式作为 Seata 的客户端与业务系统继承在一起的，TC 作为 Seata 的服务端独立部署，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ654sbnfv38TIztTTx5a69H5E5guMKnatt2JwwLyCzfKBia1iawsB6h3Zag/640?wx_fmt=png)

*   Seata 处理分布式事务的主要流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65924jKadibWkhgIvNMdicicO9KWnoWXPyDic6zrIu0wqXGMdK9icOIxUtfGQ/640?wx_fmt=png)

【解释】  

*   1> TM 开启全局事务（**TM 向 TC 开启全局事务**）
    
*   2> 事务参与者通过 RM 与资源交互，并注册分支事务（**RM 向 TC 注册分支事务**）
    
*   3> 事务参与者在完成资源操作后，上报分支事务状态（**RM 向 TC 上报分支事务完成状态**）
    
*   4> TM 结束全局事务，事务一阶段结束（**TM 向 TC 提交 / 回滚全局事务**）
    
*   5> TC 推进事务二阶段操作（**TC 向 RM 发起二阶段提交 / 回滚**）
    

三、入门实战  

=========

3.1> AT 模式开发实战
--------------

### 3.1.0> 准备工作

*   根据需求下载对应的 seata 版本
    
    https://github.com/seata/seata/releases
    
*   样例的代码结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65Sib8FRtm2l5FswvIuJF8wnLscqvpsIBGgEab6zDtVWdiaeeUbeQKGL5w/640?wx_fmt=png)

*   创建 MySQL 数据库，库名为 seata，然后执行初始化脚本 db_seata.sql
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65xWB8p88f07ppBgArVgts5RgiaBCFyN1K8qTw2lV4RQBbCWBbWsJEBQQ/640?wx_fmt=png)

【解释】这里创建了 4 张表，分别是 3 张业务表：**t_account（账户表）**、**t_order（订单表）**和 **t_stock（库存表）**；1 张事务日志表：**undo_log**；

*   表中的数据如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ651qGXvjJhEAeXRI0cNOzscJRjIBhpAjGI5I55ZvGZHJoYppzXb5uV6w/640?wx_fmt=png)

*   启动 Nacos，本示例采用单机模式启动，即：.**/startup.sh -m standalone**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65ICiaeCpvxy5B3LWlZN88GAKl5ImYjjoz7UxyKgU0HbjbdKaiaz8NRQRA/640?wx_fmt=png)

*   启动 Seata Server，**./seata-server.sh -p 8091 -h 127.0.0.1 -m file**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65R7sUSbtBiaB9oPZPCAiar5fmfuZ3yHrxhPLbhSUSnrFYYhFBpblkVVCQ/640?wx_fmt=png)

### 3.1.1> 验证全局事务正常提交

*   在 BusinessServiceImpl.handleBusiness() 方法上，开启全局事务注解 **@GlobalTransactional**，表示在该方法中调用的服务都在一个分布式事务中。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ654yGfUaXSwtEZiaBu7GH0WnuqAHpAVG1HokyicZQEdWABs3ogHMdJic0jg/640?wx_fmt=png)

*   启动如下四个服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65Xqciaadibn8PDwW9jrL8pHfwzdqDQdd2pMDG6N3cHUQa1ImjVScasbFg/640?wx_fmt=png) 

*   服务启动完毕后，查看 Nacos 控制台（http://localhost:8848/nacos）就可以看到启动并注册到 Nacos 的所有服务，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65icoZT8KX0rfYqqSmH0JsHTLRz1YattWSxicZRJ0OF15ePKMExm1D8GnA/640?wx_fmt=png)

*   发送请求——http://localhost:8104/business/dubbo/buy，表示购买 2 个水杯，总共话费 100 元操作成功。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ650ichicFYjrvyYKlRG4KWfqO3L48ZyTwJYbhxEte8icZosiaKicHGw6AibwaQ/640?wx_fmt=png)

*   数据库中表情况如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65Gia0BYPnHEMM3HBG3ycqDQLL3P2vz3uH6Jylq0zwal2j3OhFL5qrcdw/640?wx_fmt=png)

*   samples-business 服务的日志内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65PPAJHUzc1qM7saibJibKYwTwibXOvic8MXdavG68icU1oXF477IZsfd1dicA/640?wx_fmt=png)

*   samples-stock 服务的日志内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65iaibHicXomuqxw7ib3icydRMapSXPoxGOm8wTONxPhHhEU1z9GdTkvwBDEA/640?wx_fmt=png)

*   samples-account 服务的日志内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65LzUQVzFcTGCVYOMicVIEKlpticgNicto2to3IvcTs9VeIuUDnyleI95kw/640?wx_fmt=png)

*   samples-order 服务的日志内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65T9EhCvicibhLia1rpdgND4SJARdYg2M5uSv7R6Zovr44wPgx4vxcQlJpg/640?wx_fmt=png)

### 3.1.2> 验证全局事务异常回滚  

*   我们可以通过在业务逻辑中抛异常的方式模拟事务的异常回滚，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65cYK0RTlp3Jx8aNbMXC1aH032TRrANH9dhEXQibtjiaaJic7HZ1LDVzVPg/640?wx_fmt=png)

*   发送请求——http://localhost:8104/business/dubbo/buy，表示购买 2 个水杯，总共话费 100 元，操作失败产生回滚。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ657EoKD1ibhPPNk8wzrNPGLHibSemSFfGdWn1GAe9so33oMUJibEFbkMUhg/640?wx_fmt=png)

*   samples-business 服务的日志内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65R0MjCClPCUiaMa25jAYiausCiboSic91N4hpwsDSn42I4mFJEy7Q0bnBKw/640?wx_fmt=png)

*   samples-stock 服务的日志内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65IVNC2TAhLrTOvjUcWSu354HhJKeV4e4MA6cWNYKuUqhbyhibCMic4xuw/640?wx_fmt=png)

*   samples-account 服务的日志内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65Ric6dmzicia6hqeQSrVhRobMdAcYJQccgic3yebFSYN0zO8PSFbjWZHyUQ/640?wx_fmt=png)

*   samples-order 服务的日志内容如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65yA86gCuUw0ibdic0iaId1wxrppYnDaQrkibLU9MVkWxgRZMt8uz2RWDfzw/640?wx_fmt=png)

3.2> TCC 模式开发实战  

------------------

### 3.2.0> 准备工作

*   创建数据库 **transfer_from_db** 和 **transfer_to_db**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ658jTkwx1IuwHye5hsR5zuXF2fUMsy3FzNgjhAVRlgBFhHllCJK357JA/640?wx_fmt=png)

*   项目代码结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65JXrc5sb4P5kW6ric04Tq6iap6nMTPZP7qdBoficzgLcdJdGxyf3BLGFeg/640?wx_fmt=png)

【解释】

*   FirstTccAction：TCC 扣钱服务接口。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ659ekbXp0Sw1y3cz7bJS2gUib4sXBGgHyG05PlDmt2PBD9TAUMcc3ozVg/640?wx_fmt=png)

*   SecondTccAction：TCC 加钱服务接口。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65eZiaRfbRKVvW9LDiakwdrpSicwfPdkHSnbLgiaFQ1diasgvJy5COqXdmyBA/640?wx_fmt=png)

*   运行 TransferProviderStarter 的 main() 方法，会在数据库中创建相关的表
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65cmSNQuGibLaAWs4nOWg2JRehic4mSIR1kOzicjzvVib9AdW1xqibzom5ia7g/640?wx_fmt=png)

*   数据库中表初始化如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65De2BXlicFKtHXu873Rzzm2Q0BgTCvjwlxleuCd7HLGoygDLjvREfia3Q/640?wx_fmt=png)

【解释】

*   扣钱账户数据库 **transfer_from_db**，其中有两个账户，分别为 A 和 B，并且各自有 100 元的存款，没有冻结金额。
    
*   加钱账户数据库 **transfer_to_db**，其中有一个账户 C，有 100 元的存款，没有冻结金额。
    

### 3.1.1> 验证全局事务正常提交

*   执行 TransferApplication 的 main() 方法，注释掉 doTransferFailed(100, 10)，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65ShHXXQ66ibB48pPGw2ShNicpLLGqvd9xib8je0V2ibAlCQqVIAY4XDLSWg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65piadib7Mt8CP87bCGrNUZuzt6FAxsz9juIlgeefXNoA4eq5qRZf6FzUA/640?wx_fmt=png)  

### 3.1.2> 验证全局事务异常回滚

*   执行 TransferApplication 的 main() 方法，注释掉 doTransferSuccess(100, 10)，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65YsibDtStBwvibUTtUWPm1q02xUSEtv5CFiawibxT7Dx2P1CBuZaxh2BjTA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8srGxqBDhZnBbgNdsZMQ65tYGmjTsk1RyP97fSftQNBzrWaHrAIEzquk43ps7p8ahG5a4l8uYqEA/640?wx_fmt=png)