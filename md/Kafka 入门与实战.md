> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247485660&idx=1&sn=5e711ab124015b3fab48d4a46c26ed06&chksm=e9114a21de66c337f42bf4230422ea8fc62a674959ec6dc3ae32e322672ae2a67c30f6c57ef0&scene=178&cur_album_id=2162186087471906816#rd)

    今天我们来聊一聊现在 MQ 中最火爆的 Kafka 吧。关于 Kafka 的内容还是比较多的。本篇大概 15000 左右字，大家根据自己的需求来看吧。本文的大纲如下图所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy69wOvsnnibOMpO3eNb1t1eFibcHErwPVeKf8fjCsiaVicSjaibdiaqJFyCkw/640?wx_fmt=png)

一、消息队列的作用是什么？  

================

1.1> 消息队列的优点
------------

*   **可以实现系统解耦**
    
    假设有 A 系统，那么它会产生出业务数据，这个时候，有 B 系统和 C 系统时需要 A 系统产生的业务数据的。那么，如果没有引入 MQ，就需要在代码中硬编码调用 B 系统和 C 系统的接口来传输数据。那么加入由引入了 D 系统和 E 系统，并且下线了 B 系统，那么就需要修改代码，添加调用 D 系统和 E 系统的代码，并且要删除掉调用 B 系统的代码。这种设计方式，其实就是 A 系统要耦合 B、C、D、E 系统了。违反了低耦合的设计原则。如果我们引入 mq，A 系统只需要把产生的业务数据发送到 MQ 即可，下游哪个系统需要这个数据了，就去订阅 MQ 中的消息。系统 A 从此不用关系到底谁消费了它的数据，它也可以对下游完全不可见。那么就实现了系统间的解耦。
    
*   **实现异步调用**
    
    像我们常见的秒杀系统，假设分为如下几个步骤，分别为：验证功能（用户合法性、防止重复用户下单、防止同用户频繁请求，库存是否足够，活动是否有效...），下单功能，库存扣减功能，支付功能，物流功能，通知功能等等。如果假设每个功能都需要处理 200ms，那么总的耗时就是这些功能之和 200ms*6=12000ms，如果某个系统处理逻辑较多，就会造成整个串行的业务流程耗时更久。那么，其实在秒杀的场景，我们只需要告诉用户是否抢单成功即可，什么支付啊，物流这些，都可以异步去完成，那么如果我们引入 MQ，在用户调用验证功能和下单功能之后，就将抢单结果返回给客户，那么就只需要 400ms 了。响应速度提升了 3 倍。所以，针对不同的业务场景，我们就可以采用异步调用的方式，提升系统的响应速度和用户体验。
    
*   **流量削峰**
    
    假设我们有 3 台服务器，里面部署了我们的服务，且我们假设每台服务器可以抗住 1000qps，那么，当运营同学做了几个促销方案，在活动期间，招揽了大批的用户，本来平时可以抗住 3*1000qps 的系统，瞬间来了 1 万 qps，那么我们怎么办呢？我们当然可以选择在活动前期，先准备好 10 + 的机器，这样就可以抗住 1 万 qps，但是，如果其中某台机器出问题了呢？那么就会造成系统的雪崩。而且准备多台服务器，如果高流量没有持续太久，机器也浪费了。所以，针对这种情况，我们可以采取引入 mq 的方式，如果请求突然增多，消息会积压在 mq 中，而不会瞬间压垮服务器，那么，在服务器端，就起到了流量削峰的作用。
    

1.2> 采用 MQ 的缺点
--------------

*   **系统可用性降低**
    
    如果我们有 A、B、C 三个系统，我们引入 MQ 后，A 系统生产出来的业务数据会作为消息发送给 MQ，然后 B 系统和 C 系统对消息进行消费处理。但是，如果 A、B、C 这三个系统都没有任何异常，而只是 MQ 突然挂掉了，那么就会造成整个服务的可不用。针对这个问题，我们可以采用配置 MQ 集群的方式，提高 MQ 的高可用性。
    
*   **提升系统的复杂度**
    
    系统架构设计的一个普遍共识就是，系统中引入的东西越多，那么系统的复杂度就越高，问题也会越多。比如，我们引入了 MQ，那么需要保证消息不会被重复消费，要处理消息丢失的情况，如果多消息作为一个业务原子性操作，那么如果保证消息的顺序等等。所以，在日常的系统设计中，只有业务场景真的需要异步处理，我们才会去选择 MQ，而不是为了使用 MQ 而去引入 MQ。
    
*   **一致性问题**
    
    还是以上面的场景为例，如果 A 系统发送消息成功后，那么在 A 系统中，会认为整个流程是成功的。但是，在消费端，如果 B 系统或 C 系统有异常，那么整个业务流程其实是失败的。那么 A 系统的成功和消费端的失败就是不一致了。所以，当引入 MQ 的时候，我们就需要针对这种情况去做最终一致性的处理。可以根据不同步骤的成功与否，去做补偿或者业务回滚。
    

二、常用 MQ 介绍  

=============

2.1> Kafka
----------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyNXDW1HckGshA4Ea2TKLp8MeUOjyu1ue089TeolZKx1MrHlSeAGVokw/640?wx_fmt=png)

*   Apache Kafka 是一个分布式消息发布订阅系统。它最初由 LinkedIn 公司基于独特的设计实现为一个分布式的日志提交系统 (a distributed commit log)，之后成为 Apache 项目的一部分。Kafka 性能高效、可扩展良好并且可持久化。它的分区特性，可复制和可容错都是其不错的特性。
    

### 2.1.1> 主要特性

*   快速持久化：可以在 O(1) 的系统开销下进行消息持久化；
    
*   高吞吐：在一台普通的服务器上既可以达到 10W/s 的吞吐速率；
    

*   完全的分布式系统：Broker、Producer 和 Consumer 都原生自动支持分布式，自动实现负载均衡；
    
*   支持同步和异步复制两种高可用机制；
    

*   支持数据批量发送和拉取；
    
*   零拷贝技术 (zero-copy)：减少 IO 操作步骤，提高系统吞吐量；
    

*   数据迁移、扩容对用户透明；
    
*   无需停机即可扩展机器；
    

*   其他特性：丰富的消息拉取模型、高效订阅者水平扩展、实时的消息订阅、亿级的消息堆积能力、定期删除机制；
    

### 2.1.2 > 优点

*   客户端语言丰富：支持 Java、.Net、PHP、Ruby、Python、Go 等多种语言；
    
*   高性能：单机写入 TPS 约在 100 万条 / 秒，消息大小 10 个字节；
    

*   提供完全分布式架构，并有 replica 机制，拥有较高的可用性和可靠性，理论上支持消息无限堆积；
    
*   支持批量操作；
    

*   消费者采用 Pull 方式获取消息。消息有序，通过控制能够保证所有消息被消费且仅被消费一次；
    
*   有优秀的第三方 KafkaWeb 管理界面 Kafka-Manager；
    

*   在日志领域比较成熟，被多家公司和多个开源项目使用。
    

### 2.1.3 > 缺点

*   Kafka 单机超过 64 个队列 / 分区时，Load 时会发生明显的飙高现象。队列越多，负载越高，发送消息响应时间变长；
    
*   使用短轮询方式，实时性取决于轮询间隔时间；
    

*   消费失败不支持重试；
    
*   支持消息顺序，但是一台代理宕机后，就会产生消息乱序；
    

*   社区更新较慢。
    

2.2> RocketMQ
-------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyTFI7zjbEm44X0FXZcPsOvIEwVoDrzlf27Bq1agdMIfj7arsiaiboZRRQ/640?wx_fmt=png)

*   RocketMQ 出自阿里的开源产品，用 Java 语言实现，在设计时参考了 Kafka，并做出了自己的一些改进，消息可靠性上比 Kafka 更好。RocketMQ 在阿里内部被广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理，binglog 分发等场景。
    

### 2.2.1> 主要特性

*   基于 队列模型：具有高性能、高可靠、高实时、分布式等特点；
    
*   Producer、Consumer、队列都支持分布式；
    

*   Producer 向一些队列轮流发送消息，队列集合称为 Topic。Consumer 如果做广播消费，则一个 Consumer 实例消费这个 Topic 对应的所有队列；如果做集群消费，则多个 Consumer 实例平均消费这个 Topic 对应的队列集合；
    
*   能够保证严格的消息顺序；
    

*   提供丰富的消息拉取模式；
    
*   高效的订阅者水平扩展能力；
    

*   实时的消息订阅机制；
    
*   亿级消息堆积 能力；
    

*   较少的外部依赖。
    

### 2.2.2> 优点

*   单机支持 1 万以上持久化队列；
    
*   RocketMQ 的所有消息都是持久化的，先写入系统 PAGECACHE，然后刷盘，可以保证内存与磁盘都有一份数据，而访问时，直接从内存读取。
    

*   模型简单，接口易用（JMS 的接口很多场合并不太实用）；
    
*   性能非常好，可以允许大量堆积消息在 Broker 中；
    

*   支持多种消费模式，包括集群消费、广播消费等；
    
*   各个环节分布式扩展设计，支持主从和高可用；
    

*   开发度较活跃，版本更新很快。
    

### 2.2.3> 缺点

*   支持的 客户端语言不多，目前是 Java 及 C++，其中 C++ 还不成熟；
    
*   RocketMQ 社区关注度及成熟度也不及前两者；
    

*   没有 Web 管理界面，提供了一个 CLI (命令行界面) 管理工具带来查询、管理和诊断各种问题；
    
*   没有在 MQ 核心里实现 JMS 等接口；
    

2.3> RabbitMQ
-------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyLOla2mwj5p2UuV267tsw6w2EvsM4Q8cr5l6sohafqVhHoJreqbicA3g/640?wx_fmt=png)

*   RabbitMQ 于 2007 年发布，是一个在 AMQP(高级消息队列协议) 基础上完成的，可复用的企业消息系统，是当前最主流的消息中间件之一。
    

### 2.3.1> 主要特性

*   可靠性：提供了多种技术可以让你在性能和可靠性之间进行权衡。这些技术包括持久性机制、投递确认、发布者证实和高可用性机制；
    
*   灵活的路由：消息在到达队列前是通过交换机进行路由的。RabbitMQ 为典型的路由逻辑提供了多种内置交换机类型。如果你有更复杂的路由需求，可以将这些交换机组合起来使用，你甚至可以实现自己的交换机类型，并且当做 RabbitMQ 的插件来使用；
    

*   消息集群：在相同局域网中的多个 RabbitMQ 服务器可以聚合在一起，作为一个独立的逻辑代理来使用；
    
*   队列高可用：队列可以在集群中的机器上进行镜像，以确保在硬件问题下还保证消息安全；
    

*   支持多种协议：支持多种消息队列协议；
    
*   支持多种语言：用 Erlang 语言编写，支持只要是你能想到的所有编程语言；
    

*   管理界面：RabbitMQ 有一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面；
    
*   跟踪机制：如果消息异常，RabbitMQ 提供消息跟踪机制，使用者可以找出发生了什么；
    

*   插件机制：提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。
    

### 2.3.2> 优点

*   由于 Erlang 语言的特性，消息队列性能较好，支持高并发；
    
*   健壮、稳定、易用、跨平台、支持多种语言、文档齐全；
    

*   有消息确认机制和持久化机制，可靠性高；
    
*   高度可定制的路由；
    

*   管理界面较丰富，在互联网公司也有较大规模的应用，社区活跃度高。
    

### 2.3.3> 缺点

*   尽管结合 Erlang 语言本身的并发优势，性能较好，但是不利于做二次开发和维护；
    
*   实现了代理架构，意味着消息在发送到客户端之前可以在中央节点上排队。此特性使得 RabbitMQ 易于使用和部署，但是使得其运行速度较慢，因为中央节点 增加了延迟，消息封装后也比较大；需要学习比较复杂的接口和协议，学习和维护成本较高。
    

2.4> ActiveMQ
-------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGysJthLIl4DGic0TnexxUXpIdmQdicBticicB5XqUftVCKkOUqic2Erl1GIVg/640?wx_fmt=png)

*   ActiveMQ 是由 Apache 出品，ActiveMQ 是一个完全支持 JMS1.1 和 J2EE 1.4 规范的 JMS Provider 实现。它非常快速，支持多种语言的客户端和协议，而且可以非常容易的嵌入到企业的应用环境中，并有许多高级功能。
    

### 2.4.1> 主要特性

*   服从 JMS 规范：JMS 规范提供了良好的标准和保证，包括：同步 或 异步 的消息分发，一次和仅一次的消息分发，消息接收和订阅等等。遵从 JMS 规范的好处在于，不论使用什么 JMS 实现提供者，这些基础特性都是可用的；
    
*   连接灵活性：ActiveMQ 提供了广泛的连接协议，支持的协议有：HTTP/S，IP 多播，SSL，TCP，UDP 等等。对众多协议的支持让 ActiveMQ 拥有了很好的灵活性；
    

*   支持的协议种类多：OpenWire、STOMP、REST、XMPP、AMQP；
    
*   持久化插件和安全插件：ActiveMQ 提供了多种持久化选择。而且，ActiveMQ 的安全性也可以完全依据用户需求进行自定义鉴权和授权；
    

*   支持的客户端语言种类多：除了 Java 之外，还有：C/C++，.NET，Perl，PHP，Python，Ruby；
    
*   代理集群：多个 ActiveMQ 代理可以组成一个集群来提供服务；
    

*   异常简单的管理：ActiveMQ 是以开发者思维被设计的。所以，它并不需要专门的管理员，因为它提供了简单又使用的管理特性。有很多中方法可以监控 ActiveMQ 不同层面的数据，包括使用在 JConsole 或者在 ActiveMQ 的 WebConsole 中使用 JMX。通过处理 JMX 的告警消息，通过使用命令行脚本，甚至可以通过监控各种类型的日志。
    

### 2.4.2> 优点

*   跨平台 (JAVA 编写与平台无关，ActiveMQ 几乎可以运行在任何的 JVM 上)；
    
*   可以用 JDBC：可以将数据持久化到数据库。虽然使用 JDBC 会降低 ActiveMQ 的性能，但是数据库一直都是开发人员最熟悉的存储介质；
    

*   支持 JMS 规范：支持 JMS 规范提供的统一接口;
    
*   支持自动重连和错误重试机制；
    

*   有安全机制：支持基于 shiro，jaas 等多种安全配置机制，可以对 Queue/Topic 进行认证和授权；
    
*   监控完善：拥有完善的监控，包括 WebConsole，JMX，Shell 命令行，Jolokia 的 RESTful API；
    

*   界面友善：提供的 WebConsole 可以满足大部分情况，还有很多第三方的组件可以使用，比如 hawtio；
    

### 2.4.3> 缺点

*   社区活跃度不及 RabbitMQ 高；
    
*   根据其他用户反馈，会出莫名其妙的问题，会丢失消息；
    

*   目前重心放到 activemq6.0 产品 Apollo，对 5.x 的维护较少；
    
*   不适合用于上千个队列的应用场景；
    

2.5> ZeroMQ
-----------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyJNKibRJDy84sBxtYY4owBSplrsP6OUf1dLJSgcARgrN11h35T4V2M0w/640?wx_fmt=png)

*   号称史上**最快**的消息队列，它实际**类似于 Socket** 的一系列接口，他跟 Socket 的区别是：普通的 socket 是端到端的（1:1 的关系），而 ZMQ 却是可以 N：M 的关系，人们对 BSD 套接字的了解较多的是点对点的连接，点对点连接需要显式地建立连接、销毁连接、选择协议（TCP/UDP）和处理错误等，而 ZMQ 屏蔽了这些细节，让你的网络编程更为简单。ZMQ 用于 node 与 node 间的通信，node 可以是主机或者是进程。
    
*   引用官方的说法：“ZMQ(以下 ZeroMQ 简称 ZMQ) 是一个简单好用的传输层，像框架一样的一个 socket library，他使得 Socket 编程更加简单、简洁和性能更高。是一个消息处理队列库，可在多个线程、内核和主机盒之间弹性伸缩。ZMQ 的明确目标是 “成为标准网络协议栈的一部分，之后进入 Linux 内核”。现在还未看到它们的成功。但是，它无疑是极具前景的、并且是人们更加需要的 “传统”BSD 套接字之上的一层封装。ZMQ 让编写高性能网络应用程序极为简单和有趣。”
    
*   特点是：
    
    高性能，非持久化
    
    跨平台：支持 Linux、Windows、OS X 等
    
    多语言支持；C、C++、Java、.NET、Python 等 30 多种开发语言
    
    可单独部署或集成到应用中使用
    
    可作为 Socket 通信库使用
    
*   与 RabbitMQ 相比，ZMQ 并不像是一个传统意义上的消息队列服务器，事实上，它也根本**不是一个服务器**，更像一个**底层的网络通讯库**，**在 Socket API 之上做了一层封装**，将网络通讯、进程通讯和线程通讯抽象为统一的 API 接口。支持 “Request-Reply “，”Publisher-Subscriber“，”Parallel Pipeline” 三种基本模型和扩展模型。
    
*   ZeroMQ 高性能设计要点：
    
*   **无锁的队列模型**
    
    对于跨线程间的交互（用户端和 session）之间的数据交换通道 pipe，采用无锁的队列算法 CAS；在 pipe 两端注册有异步事件，在读或者写消息到 pipe 的时，会自动触发读写事件。
    
*   **批量处理的算法**
    
    对于传统的消息处理，每个消息在发送和接收的时候，都需要系统的调用，这样对于大量的消息，系统的开销比较大，zeroMQ 对于批量的消息，进行了适应性的优化，可以批量的接收和发送消息。
    
*   **多核下的线程绑定，无须 CPU 切换**
    
    区别于传统的多线程并发模式，信号量或者临界区， zeroMQ 充分利用多核的优势，每个核绑定运行一个工作者线程，避免多线程之间的 CPU 切换开销。
    

2.6> 对比
-------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGykphCuI2fWRI9yZJc1DV4d8O2gOcsbrW41NqWIlzKPcZXgBsQMr2Riag/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGypDwbWzeicSlV1RPYx2epSPEg4qqoGQVfCGlGhWO2NiaukYQS4SPsCiaNg/640?wx_fmt=png)

*   总结：消息队列的选型需要根据具体应用需求而定，ZeroMQ 小而美，RabbitMQ 大而稳，Kakfa 和 RocketMQ 快而强劲。
    

2.7> 适用场景
---------

### 2.7.1> 从公司基础建设力量角度出发

中小型软件公司，建议选 RabbitMQ

*   首先：erlang 语言天生具备高并发的特性，而且他的管理界面用起来十分方便。他的弊端也很明显，虽然 RabbitMQ 是开源的，然而国内有几个能定制化开发 erlang 的程序员呢？所幸，RabbitMQ 的社区十分活跃，可以解决开发过程中遇到的 bug，这点对于中小型公司来说十分重要。
    
*   其次：不考虑 Kafka 的原因是，一方面中小型软件公司不如互联网公司，数据量没那么大，选消息中间件，应首选功能比较完备的，所以 kafka 排除。
    

*   最后：不考虑 RocketMQ 的原因是，RocketMQ 是阿里出品，如果阿里放弃维护 RocketMQ，中小型公司一般抽不出人来进行 RocketMQ 的定制化开发，因此不推荐。
    

大型软件公司，根据具体使用在 RocketMQ 和 kafka 之间二选一。

*   一方面，大型软件公司，具备足够的资金搭建分布式环境，也具备足够大的数据量。
    

*   针对 RocketMQ，大型软件公司也可以抽出人手对 RocketMQ 进行定制化开发，毕竟国内有能力改 JAVA 源码的人，还是相当多的。
    
*   至于 kafka，根据业务场景选择，如果有日志采集功能，肯定是首选 kafka 了。
    

### 2.7.2> 从业务场景角度出发

*   RocketMQ 定位于非日志的可靠消息传输（日志场景也 OK），目前 RocketMQ 在阿里集团被广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理，binglog 分发等场景。
    
*   Kafka 是 LinkedIn 开源的分布式发布 - 订阅消息系统，目前归属于 Apache 顶级项目。Kafka 主要特点是基于 Pull 的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输。0.8 版本开始支持复制，不支持事务，对消息的重复、丢失、错误没有严格要求，适合产生大量数据的互联网服务的数据收集业务。
    

*   RabbitMQ 是使用 Erlang 语言开发的开源消息队列系统，基于 AMQP 协议来实现。AMQP 的主要特征是面向消息、队列、路由（包括点对点和发布 / 订阅）、可靠性、安全。AMQP 协议更多用在企业系统内，对数据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量的要求还在其次。
    

三、安装  

=======

3.1> 安装 ZooKeeper
-----------------

*   首先进入 Zookeeper 官网 https://zookeeper.apache.org/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy7vHrGOYDVBHdQM3uVTq7zAUjHMPWjSsIcfE9vowEkqrzkTI2Ric5utg/640?wx_fmt=png)

*   下载最新版本 ZooKeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGypzMIvKSXJLgHzPyTkMJFWpR0qNibWcOJCaA1tbjozTb4gTPG4Hn5HIA/640?wx_fmt=png)

*   解压到本地路径，进入 ZooKeeper 的 conf 目录下，复制 zoo_sample.cfg 配置文件，命名为 **zoo.cfg**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyuN5jD2ZBtmgaHycw5jT8chUuc1DguI9b6bXQibHtBBESJMN7qCndncA/640?wx_fmt=png)

*   **zoo.cfg** 配置文件中各配置项的含义如下所示：
    

```
# zookeeper时间配置中的基本单位（毫秒）
tickTime=2000
# 允许follower初始化连接到leader最大时长，它表示tickTime时间的倍数 即：initLimit*tickTime
initLimit=10
# 运行follower与leader数据同步最大时长，它表示tickTime时间倍数 即：syncLimit*tickTime
syncLimit=5
# zookeeper数据存储目录及日志保存目录（如果没有指明dataLogDir，则日志也保存在这个文件中）
dataDir=/tmp/zookeeper
# 对客户端提供的端口号
clientPort=2181
# 单个客户端于zookeeper最大并发连接数
maxClientCnxns=60
# 保存的数据快照数量，之外的将会被清除
autopurge.snapRetainCount=3
# 自动出发清除任务时间间隔，以小时为单位。默认为0，表示不自动清除
autopurge.purgeInterval=1
## Metrics Providers
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

```

*   启动 zookeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy3jvbkENaAIxsBjHnhyVeXy0lKpdLWFvvOiadYCuNnX4ukJlibUQe4AMg/640?wx_fmt=png)

*   查看 zookeeper 的运行状态
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy4IAPnWJ41hHkwHIkapcMkVMNc9Vv3j5EVa80JiaO3pp1EIQ3IaicBG9w/640?wx_fmt=png)

3.2> 安装 Kafka  

----------------

*   首先进入 Kafka 官网下载 kafka http://kafka.apache.org/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy9XqFa9zQrDicjFujB5KjwWI1MwcACCCQC3C6icYhc3jXPYbBUxM6SLYw/640?wx_fmt=png)

*   解压，然后进入 config 目录下，编辑 **server.properties** 配置文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGycqsTsHRqDytWDNaTQRLjC6QrFM0Sh5jZ9f7RhYTiaeDxWDpJONmWyBA/640?wx_fmt=png)

*   server.properties 配置项解析
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyaicTCHF5dza2fYtlMgB0hOUSgsbjd0EuT173iaYWV1DTyrrFAibQ3mk2w/640?wx_fmt=png)

*   我们需要关注如下几个配置内容
    

*   **broker 的序号**
    

broker.id=0

*   **当前 kafka 的监听地址**
    

listeners=PLAINTEXT://localhost:9092

*   **日志的存储路径**
    

log.dirs=/Users/muse/kafka_2.13-3.0.0/kafka-logs

*   **zookeeper 的服务地址**
    

zookeeper.connect=localhost:2181

*   修改完响应配置后，启动 Kafka，并通过查看 kafka 的进程来判断是否启动起来了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy7Y5nvAT0Jszmosz1wYd3638KxB7S1yNKrM7gMDWXODKPE74GQUSvkw/640?wx_fmt=png)

3.3> 安装 EFAK  

---------------

*   首先进入 EFAK 官网下载 Eagle：http://download.kafka-eagle.org/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyJeWE84JHfibniacMAf7dwLhkpHggiczWcB8Fd2iaM9qzmlANduTD2icBxSA/640?wx_fmt=png)

*   修改 EFAK 的 conf 目录下配置文件——**system-config.properties**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyhf11Xnic5O6FRUEN7G1bR5unicmEREY9o3cRiaiaDCDIl9lCjee2dwRWIA/640?wx_fmt=png)

*   配置环境变量 **KE_HOME**，并调用 **source .zshrc** 使其立即生效。（如果不配置的话，启动时会报错）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyB8UlQibd8NZdUjD98Thfu0iajiaTIE3MylTwZxa9zmmyeKibSibRIlnyn8w/640?wx_fmt=png)

*   启动 EFAK
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyLOylhTzckh7tklhK6dj634OpkWc6qtcVl2xSVcdicFfA7HoDVh4KLIA/640?wx_fmt=png)

*   启动成功界面
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGySINd9GfvJ9nVLsuIsq3dGhSZDHv406MQKLsuiarBGzaqS4Gy2VCU8kQ/640?wx_fmt=png)

*   访问 EFAK 界面 http://192.168.1.3:8048，用户名：admin 密码：123456
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGymfA3ZqgCTzRgbVicET1BHz3aec67vGocG9TczicP99NaYh0rYwrTg1aA/640?wx_fmt=png)

*   管理界面如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGypwrY2TJ3wibQW4As6MgJPXBVGIYt6QsjGDCDdh9rkIXnr7nsO6LxVuw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy322Apic1WGIa6ibCWLic02TC86sHvicPApTSdVic209wTBDSBAe5Ga6BiaicA/640?wx_fmt=png)

四、kafka 基础知识  

===============

4.1> kafka 常用的术语如下所示
--------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGywF80ia1mG48uRSQMhkPL0sSByVVkVuTQrlgRYXMGKHRTKDYnKOm6BjQ/640?wx_fmt=png)

4.2> 相关指令
---------

https://kafka.apache.org/documentation/#quickstart

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyftGT7RArWFAXPwjW5ZOBEmiap1gnbjj3Se79hcX0sicoVicia66HXSn0gg/640?wx_fmt=png)

4.3> 创建和查看主题  

---------------

*   **bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic muse --partitions 1 --replication-factor 1**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy3VdicwXsibLVneT5IAA7WgjxC1l06swUdGtpZ1TrW4tedgeoUbD9Yq0w/640?wx_fmt=png)

*   **./kafka-topics.sh --list --bootstrap-server localhost:9092**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyyX0L4ZyZyCbiapWXMaxmgakRASoGG8icbkv89WtV4iah8gzxvEbibHm0XA/640?wx_fmt=png)

4.4> 开启消息发送端  

---------------

*   **./kafka-console-producer.sh --topic muse --bootstrap-server localhost:9092**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy6ia2lVgHe9QJOthqwDXiaeTeia2r3w3RSIgm5f7VsxekUEAnWvvHQ5PGw/640?wx_fmt=png)

*   kafka 自带了一个 producer 的命令客户端——kafka-console-producer.sh，它可以从本地文件中读取内容，或者我们也可以从命令行中直接输入内容，并将这些内容以消息的形式发送到 kafka 集群中。在默认情况下，每一行都会被当做一个独立的消息。
    
*   使用 kafka-console-producer.sh，指定发送到的 topic 和 kafka 服务器的地址
    

*   也可以使用**./kafka-console-producer.sh --topic muse --broker-list localhost:9092**
    

4.5> 开启消息接收端
------------

*   【方式一】从最后一条消息的偏移量 + 1 开始消费，即：启动客户端之前的消息是不会被消费的
    

**./kafka-console-consumer.sh --topic muse --bootstrap-server localhost:9092**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyBWDVUKTTt1N3FS9Zj23NA4wW79bOgCWiaGM8P3t6BwlqsG4Tn39MRzA/640?wx_fmt=png)

*   【方式二】从头开始消费，即：启动客户端之前的消息会被消费
    

**./kafka-console-consumer.sh --topic muse --bootstrap-server localhost:9092 --from-beginning**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGykwOeBt4JkPZ8635QlibkyqUIMicGTfcc94PNl0NTL8d95vANk7px8OoA/640?wx_fmt=png)

*   **消息是会被存储在 kafka 中的文件里的，并且是顺序存储的，消息有偏移量的概念，所以我们可以指定偏移量去读取某个位置的消息。**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGygrDv4kn7OUBcvOSLJJlP0YNibYlv7D8y7T5QcoeFcrjcc9apS6uQGMQ/640?wx_fmt=png)

*   如果说消息会被存储到 kafka 中，那么，_发到 topic=muse 的消息存储到哪里了呢？_
    

答：我们上面介绍 kafka 配置文件 server.properties 时，配置了一个属性为 log.dirs，那么这个路径就是存储路径，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyq3JnwjD03qibXMv6eFMGWzlzKhfNnZib8dFiaQYvtjScRAJjHFXRE0EKg/640?wx_fmt=png)

【解释】其中的 00000000000000000000.log 就是消息存储的文件。  

4.6> 单播消息
---------

*   一个消费组里，只会有一个消费者能够消费到某个 topic 中的消息。
    
*   操作演示
    

*   首先，打开两个窗口，分别执行如下语句开启消费端，那么就在 “museGroup1” 消费组中创建了两个 Consumer
    

**./kafka-console-consumer.sh --topic muse --bootstrap-server localhost:9092 --consumer-property group.id=museGroup1**

*   然后：Producer 端发送 3 条消息，我们发现，只有一个 Consumer 收到了消息。如下所示：
    

*   生产者
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyL69QjcP8b5Td69gibZ9ujutBVMFLMNYlG8WqFPAOMzvMIrBvcz9Th1w/640?wx_fmt=png)

*   消费者 1（消费组：museGroup1）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy3US2kgLdzHApdyax9hZPQOLic05Sbep9XvhG9LzRxpB8NQyXIibn9PpQ/640?wx_fmt=png)

*   消费者 2（消费组：museGroup1）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyCxib7jOiceicIO9aZk4wnKJe9fp7qaaOibL1yYJrlaRVs3A1qhmfepibuuw/640?wx_fmt=png)

*   处理流程如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGykgwiagOK9j5MUM0d3hrIKGZoR39AGVWdYfubEhoaghCXnq0CiaJVngDQ/640?wx_fmt=png)

4.7> 多播消息  

------------

*   当业务场景中，需要同一个 topic 下的消息被多个消费者消费，那么我们可以采用创建多个消费组的方式，那么这种方式就是多播消息。
    

*   操作演示
    

*   首先，打开两个窗口，在其中一个窗口执行如下指令：
    

**./kafka-console-consumer.sh --topic muse --bootstrap-server localhost:9092 --consumer-property group.id=museGroup1**

*   然后：在另一个窗口执行如下指令
    

**./kafka-console-consumer.sh --topic muse --bootstrap-server localhost:9092 --consumer-property group.id=museGroup2**

*   最后，Producer 端发送 3 条消息，我们发现，两个 Consumer 都收到了消息。如下所示：
    

*   生产者
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyAibbicTLaE5xnbjXJRIwye6wXaeZQTUjJTCMPAsicnCKPWy6vicTnKU4SQ/640?wx_fmt=png)

*   消费者 1（消费组：museGroup1）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy8eolB6aopbXdlzia3yW0kn0BG93ypha9gyaFMGlbuORCRVPxTxkWEGQ/640?wx_fmt=png)

*   消费者 2（消费组：museGroup2）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyhHIfM8vpIsGPrvgZAqqUhlicOAm8LcXVZn7EtMFIJiaxVoYfoF8YOzCA/640?wx_fmt=png)

*   处理流程如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyZOoFxvG9t2icfJYnerCI2YrCeFOytyyJjWHvUMn2JS14XlDuc9xP0Pw/640?wx_fmt=png)

4.8> 查看消费组信息  

---------------

*   查看当前主题下有哪些消费组
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyyFBdpcn2ywlM20yshlemMJmMzqXP4IsrM3OCcnzzhqjibp7P5u48Myg/640?wx_fmt=png)

*   查看消费组中的具体信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyibnM4HL6dKoU8Ot6FsCl1vOBicbCicfeKjQFACeCDT8UXqFoCXicB8OmLw/640?wx_fmt=png)

【解释】

*   **CURENT-OFFSET**：当前消费组已消费消息的偏移量。
    
*   **LOG-END-OFFSET**：主题对应分区消息的结束偏移量（水位 HW）。
    
*   **LAG**：当前消费组堆积未消费的消息数量。
    

*   操作演示
    

*   首先，我们先看一下消费组 museGroup1 的具体信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyAgqLq2CUZr9un0YfzJn4N4pbX55peBXShudoto0MgvSsBk1XBtFjng/640?wx_fmt=png)

*   然后，我们关闭 museGroup1 的所有 Consumer，使得这个消费组不具备消费消息的能力
    
*   最后，我们向消费组 museGroup1 中发送 3 条消息，我们再来看一下消费组信息。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyyFNuqdCXgXG3kcuMulKqz2hgOjkGIN1j4k5az41dOnM9YC6TRLUsicg/640?wx_fmt=png)

五、kafka 集群  

=============

5.1> 搭建 kafka 集群
----------------

### 5.1.1> 解压 kafka 压缩包

*   创建 kafka-cluster 目录，并解压 kafka_2.13-3.0.0.tgz 为 3 份 kafka
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyutPGtUlibofc5QwCm7ppBt9n8CWPMF8l3YEhcB0OIAxtDNjpOpIx8EQ/640?wx_fmt=png)

### 5.1.2> 修改配置文件 server.properties  

*   修改 kafka1 的 server.propertis 配置文件
    

```
broker.id=0
listeners=PLAINTEXT://localhost:9092
log.dirs=/Users/muse/kafka-cluster/kafka1/kafka-logs

```

*   修改 kafka2 的 server.propertis 配置文件
    

```
broker.id=1
listeners=PLAINTEXT://localhost:9093
log.dirs=/Users/muse/kafka-cluster/kafka2/kafka-logs

```

*   修改 kafka3 的 server.propertis 配置文件
    

```
broker.id=2
listeners=PLAINTEXT://localhost:9094
log.dirs=/Users/muse/kafka-cluster/kafka3/kafka-logs

```

### 5.1.3> 启动三个节点

*   启动 kafka1，kafka2 和 kafka3
    

```
./kafka-cluster/kafka1/bin/kafka-server-start.sh -daemon ./kafka-cluster/kafka1/config/server.properties
./kafka-cluster/kafka2/bin/kafka-server-start.sh -daemon ./kafka-cluster/kafka2/config/server.properties
./kafka-cluster/kafka3/bin/kafka-server-start.sh -daemon ./kafka-cluster/kafka3/config/server.properties

```

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyW4dHF9PoSaNlzQPibxoSjy6MQS8IyRUibhG5m2Itbibmrb1MSmP895Riag/640?wx_fmt=png)

### 5.1.4> 验证 3 个 Broker 是否启动成功

*   首先，可以通过 **ps -ef | grep kafka** 来查看进程是否启动
    

*   其次：在 zookeeper 中查看 / brokers/ids 下中是否有相应的 brokerId 目录生成
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGypD7Z1rdibym7WSwttMzOm5ia9SWRMrwVDGCKwQLxhiatDbPSCKvScC3Gw/640?wx_fmt=png)

5.2> 分区和副本  

-------------

### 5.2.1> 分区

*   一个主题中的消息量是非常大的，因此可以通过分区的设置，来分布式存储这些消息。并且分区也可以提供消息并发存储的能力。
    
*   比如：给一个 Topic 创建了 4 个分区，那么 Topic 中的消息就会**分别存放**在这 4 个分区中。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyDM0rmtibncdq2ArswMw9KiaHjjYjWPsK8sFdPKEnC1QcjJVkqVNuzH9A/640?wx_fmt=png)

### 5.2.2> 副本  

*   如果分片挂掉了，数据就丢失了。那么为了**提高系统的可用性**。我们把分片复制多个，这就是副本了。
    
*   但是，副本的产生，也会随之带来**数据一致性的问题**，即：有的副本写数据成功，但是有的副本写数据失败。
    

*   Leader
    

*   kafka 的读写操作都发生在 leader 上，leader 负责把数据同步给 follower。
    
*   当 leader 挂掉了，那么经过主从选举，从多个 follower 中选举产生一个新的 leader。
    

*   Follower
    

*   follower 接收 leader 同步过来的数据，它不提供读写（主要是为了保证多副本数据与消费的一致性）
    

### 5.2.3> __consumer_offsets-N

*   kafka 默认创建了一个拥有 50 个分区的主题，名称为：“__consumer_offsets”，
    
*   consumer 会把将消费分区的 offset 提交给 kafka 内部 topic：__consumer_offsets，提交过去的时候，
    
    【key】=**consumerGroupID+topic + 分区号**
    
    【value】= **当前 offset 的值**。
    
*   Kafka 会定期清理 topic 里的消息，最后就保留最新的那条数据。
    
*   因为__consumer_offsets 可能会接收高并发的请求，所以默认分配了 **50 个分区**（可以通过 offsets.topic.num.partitions 进行配置），这样可以通过增加机器的方式应对高并发的业务场景。
    
*   可以通过如下公式确定 consumer 消费的 offset 要提交到哪个__consumer_offsets
    

**hash(consumerGroupID)% 主题 “__consumer_offsets” 的分区数量**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGylhpAicEvWnXBzibiaiak9PJAnW7QiahLbCMpC2gYGuUAVKe1nEcNAUXB0FA/640?wx_fmt=png)

### 5.2.4> 操作演示  

*   为名称为 muse-rp 的 Topic 创建 **2 个分区**（--partitions）**3 个副本**（--replication-factor）
    

**./kafka-topics.sh --create --topic muse-rp  --bootstrap-server localhost:9092 --partitions 2 --replication-factor 3**

*   查看分区和副本分布情况
    

**./kafka-topics.sh --describe --topic muse-rp --bootstrap-server localhost:9092**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyb8sfe997pgAlg9ZINNGM9Jgnsn6grjoWhUjiboMCzgYP7buLmKblc5Q/640?wx_fmt=png)

【解释】  

*   **Isr**
    

“可以同步”和 “已同步” 的节点都会被存入 isr 集合中，如果 isr 中的节点性能较差，那么将会从 isr 集合中被剔除。

当 leader 节点挂掉，需要从 follower 节点列表中选举出主节点时，其实就是从 isr 集合中选举出来的。

*   分区和副本的分布如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyWZhPxv7EMS0SXnMKtjQiaFnj7zwwicHjzuIoWvOxlsmS50mePpic7VbGQ/640?wx_fmt=png)

*   我们随便找一个名为 kafka1 的 Broker，来看一下. log 文件和. index 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGypZRzwove6u4044cXiaueV4064Hw3dFUJO8NngqLwOvc3vXvXcSDSWMA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyxaAicfFKxfQibf2oHu4rkRMsVWu2LAeicqn4DP82TsBZoLZlRKkiaicicaibw/640?wx_fmt=png)

【解释】

*   由于将 Topic=“muse-rp” 的主题指定了 2 个分区
    
*   所以产生了两个目录 **muse-rp-0** 和 **muse-rp-1**，
    
*   里面存储的 **00000000000000000000.log** 就是消息存储的文件。
    
*   其中 **00000000000000000000.index** 存储的是消息的**稀疏索引**。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGycUIOwxgDciawnRfCMe3QricW4Rll6Pe3IXibpHibEetlzk7veYsphcgHeg/640?wx_fmt=png)

5.3> 集群消息的发送与消费  

------------------

### 5.3.1> 消息放送

*   **./kafka-console-producer.sh --topic muse-rp --bootstrap-server localhost:9092 localhost:9093 localhost:9094**
    
*   发送 10 个消息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGywg3mWia1SicSdcR0oRBJ30nM3vCYSx0DK5CNRTby1Z8icuwVib4GDc6qeQ/640?wx_fmt=png)

### 5.3.2> 消息消费  

*   **./kafka-console-consumer.sh --topic muse-rp --bootstrap-server localhost:9092 localhost:9093 localhost:9094 --consumer-property group.id=museGroup1**
    
*   在消息组 museGroup1 中开启第一个消费者
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGy0XMTwwDJKPQsH8m7Xaiay3ymv1bMgCDYZRn6Xa8G7lk8aDgrm52PVHQ/640?wx_fmt=png)

*   在消息组 museGroup1 中开启第二个消费者
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyQpnrFczNfUKx8UDYXADt5IlqJzpicRiau4eD90AU6tu7MPDFjib2qkxpA/640?wx_fmt=png)

### 5.3.3> 总结

*   一个 partition 只能被一个消费组中的一个消费者消费，这样设计的目的是保证消息的有序性，但是在多个 partition 的多个消费者消费的总顺序性是无法得到保证的。
    
*   partition 的数量决定了消费组中 Consumer 的数量，建议同一个消费组中的 Consumer 数量不要超过 partition 的数量，否则多余的 Consumer 就无法消费到消息了
    

*   但是，如果消费者挂掉了，那么就会触发 rebalance 机制，会由其他消费者来消费该分区。
    
*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyF9LbjhrRr2oTnRPxOIulILv2fomLTFEfhVXiayMerv7FHIYK4w9oUhw/640?wx_fmt=png)

六、利用 Kafka 的 Java Client 实现消息收发  

==================================

*   项目中引入 kafka-clients 的依赖（也可以直接引入 spring-kafka 的依赖，里面包含了 kafka-clients）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyqPBlwVUbjm9Ecx1XUsJFNj48hMzyd8ALfSxVbQy9AzEz2SPn06ibklA/640?wx_fmt=png)

6.1> 生产者端  

------------

### 6.1.1> 初始化配置

*   创建配置对象 Properties
    

```
Properties properties = new Properties();

```

*   配置 kafka 的 Broker 列表
    

```
properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094");

```

*   发出消息持久化机制参数
    

```
properties.put(ProducerConfig.ACKS_CONFIG, "1");

```

【解释】

*   acks=0
    

表示 producer **不需要等待**任何 broker 确认收到消息的 ACK 回复，就可以继续发送下一条消息。性能最高，但是最容易丢失消息。

*   acks=1
    

表示至少**等待 leader 已经成功**将数据写入本地 log，但是不需要等待所有 follower 都写入成功，就可以继续发送下一条消息。

这种情况下，如果 follower 没有成功备份数据，而此时 leader 又挂掉，则消息就会丢失。

*   acks=-1/all
    

需要等待所有 **min.insync.replicas**（默认为 1，推荐配置 >=2）这个参数配置的副本个数都成功写入日志。

这种策略会保证只要有一个备份存活就不会丢失数据。这是最强的数据保证。

*   配置失败重试机制
    

```
properties.put(ProducerConfig.RETRIES_CONFIG, 3); // 失败重试3次
properties.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, 300); // 重试间隔300ms

```

【解释】  

*   发送失败重试的次数，默认是**间隔 100ms**。
    
*   重试能保证消息发送的可靠性，但是也可能造成消息重复发送，所以需要在**消费者端做好幂等性处理**。
    
*   配置缓存相关信息
    

```
properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 32*1024*1024); // 设置发送消息的本地缓冲区，默认值为32MB
properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16*1024); //  设置批量发送消息的大小，默认值为16KB
properties.put(ProducerConfig.LINGER_MS_CONFIG, 10); // batch等待时间，默认值为0，表示消息必须立即被发送

```

【解释】  

*   Producer 的消息会先发送到本地缓冲区（BUFFER_MEMORY_CONFIG），而不是发送一次消息连接一次 kafka。
    
*   kafka 本地线程会从缓冲区去取数据（BATCH_SIZE_CONFIG），然后批量发送到 Broker，即：一个批次满足 16KB 就会发送出去。
    
*   LINGER_MS_CONFIG 的默认值为 0，表示消息必须立即被发送，但这样会影响性能。  
    设置 10ms 也就是说 Producer 消息发送完后会进入本地的 batch 中；如果 10ms 内，这个 batch 满足了 16KB，那么就会随着 batch 一起被发送出去。  
    如果 10ms 内，batch 没满，那么也必须要把消息发送出去，不能让消息的发送延迟时间太长。
    

*   配置 key 和 value 的序列化实现类
    

```
properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

```

### 6.1.2> 同步消息发送

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyeTiapfMk50D7u8pGBpcydzJIV2jeYlajcfal5uPMoJnPA8N2eM0OuTQ/640?wx_fmt=png)

*   同步消息发送代码如下所示
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyicEPMg2J7FxaXBvSIgJib5ctiano4pRUvXugziczjs7IkUeJE8jyDKjNZA/640?wx_fmt=png)

### 6.1.3> 异步消息发送  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyZIjs6ULicRPK3IVpJ38js3Imundfr5UHicxgxBzvVjYFVJiaia40gHMiaOQ/640?wx_fmt=png)

6.2> 消费者端  

------------

### 6.2.1> 初始化配置

*   创建配置对象 Properties
    

```
Properties properties = new Properties();

```

*   配置 kafka 的 Broker 列表
    

```
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094");

```

‍

*   配置消费组
    

```
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "museGroup");

```

*   offset 的重置策略
    

```
properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

```

【解释】

*   offset 的重置策略——例如创建一个新的消费组，offset 是不存在的，如何对 offset 赋值消费  
    **latest**：默认值，只消费自己启动之后发送到主题的消息。  
    **earliest**：第一次从头开始消费，以后按照消费 offset 记录继续消费。
    

*   心跳相关配置
    

```
/** Consumer给Broker发送心跳的时间间隔 */
properties.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 1000);
/** 如果超过10秒没有接收到消费者的心跳，则会把消费者踢出消费组，然后重新进行rebalance操作，把分区分配给其他消费者 */
properties.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10*1000);

```

*   poll 相关配置
    

```
/** 一次poll最大拉取消息的条数，可以根据消费速度的快慢来设置 */
properties.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500);
/** 如果两次poll的时间超出了30秒的时间间隔，kafka会认为整个Consumer的消费能力太弱，会将它踢出消费组。将分区分配给其他消费者 */
properties.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 30*1000);

```

*   配置 key 和 value 的反序列化实现类
    

```
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

```

### 6.2.2> 自动提交 offset

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyjxicjiaQQtntN8s9qLkj5UZxu2tY7IGkoX7aDEkjmKqefTnVuF6KF64Q/640?wx_fmt=png)

*   自动提交 offset
    

*   当消费者向 Broker 的 log 中 poll 到消息后，默认情况下，会向 broker 中名称为 “__consumer_offsets” 的 Topic 发送 offset 偏移量。
    
*   **自动提交会出现丢失消息的情况**
    

因为如果 Consumer 还没消费完 poll 下来的消息就自动提交了偏移量，那么此时如果 Consumer 挂掉了，那么下一个消费者会从已经提交的 offset 的下一个位置开始消费消息。那么之前没有被消费的消息就丢失了。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyVL82vic2kfNC3CkBiacxmzibgYcGqKXdPIxCNnicrQdM8OY6DoxzY9cDjw/640?wx_fmt=png)

### 6.2.3> 手动提交 offset  

*   手动提交 offset
    

当消费者从 kafka 的 Broker 日志文件中 poll 到消息并且消费完毕之后。再手动提交当前的 offset。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyNGs4vAXn5vhxR6vMTMfLaibGw415GEgdYDm6ecY7n4Sia7IlNNePibMNA/640?wx_fmt=png)

七、SpringBoot 集成  

==================

6.1> 创建 Spring 项目
-----------------

*   创建 SpringBoot 项目，引入 kafka 依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyKia4FWnP9u95ianIzfUzwEbtWUHulwWr3t0MAAXB907icL5zBqS6f3aRQ/640?wx_fmt=png)

*   我们看到，SpringBoot 自动引入的 kafka 版本就是我们所采用的最新版本 3.0.0
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyuxeUKDADicfgEC72wibuCnFj1sRtf5TW2ibeFDFdj1h82CGO5XgHiaR7Aw/640?wx_fmt=png)

6.2> 生产者端
---------

*   初始化生产者配置
    

```
spring:
  kafka:
    producer:
      bootstrap-servers: 127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094 # kafka集群
      acks: 1 # 应答级别：多少个分区副本备份完成时，向生产者发送ack确认（可选0、1、-1/all）
      retries: 3 # 重试次数
      buffer-memory: 33554432 # 生产端缓冲区大小
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      batch-size: 16384 #批量大小，默认16kb
      properties:
        # 提交延时，当Producer积累的消息达到batch-size或者接收到消息linger.ms后，生产者就会将消息提交给kafka
        # linger.ms等于0，表示没当接收到一条消息的时候，就提交给kafka，这个时候batch-size上面配置的值就无效了
        linger.ms: 0

```

*   同步消息发送
    

```
private final static String TOPIC_NAME = "muse-rp";
private final static Integer PARTITION_ONE = 0;
private final static Integer PARTITION_TWO = 1;
@Resource
private KafkaTemplate<String, Message> kafkaTemplate;
/**
 * 同步阻塞——消息发送
 */
public void testBlockingSendMsg() throws Throwable {
    Message message;
    for (int i=0; i< 5; i++) {
        message = new Message(i, "BLOCKING_MSG_SPRINGBOOT_" + i);
        ListenableFuture<SendResult<String, Message>> future = kafkaTemplate.send(TOPIC_NAME, PARTITION_ONE,
                "" + message.getMegId(), message);
        SendResult<String, Message> sendResult = future.get();
        RecordMetadata recordMetadata = sendResult.getRecordMetadata();
        log.info("---BLOCKING_MSG_SPRINGBOOT--- [topic]={}, [partition]={}, [offset]={}", recordMetadata.topic(),
                recordMetadata.partition(), recordMetadata.offset());
    }
}

```

*   异步消息发送
    

```
/**
 * 异步回调——消息发送
 */
public void testNoBlockingSendMsg() {
    Message message;
    for (int i=0; i< 5; i++) {
        message = new Message(i, "NO_BLOCKING_MSG_SPRINGBOOT_" + i);
        ListenableFuture<SendResult<String, Message>> future = kafkaTemplate.send(TOPIC_NAME, PARTITION_TWO,
                "" + message.getMegId(), message);
        future.addCallback(new ListenableFutureCallback<SendResult<String, Message>>() {
            @Override
            public void onFailure(Throwable ex) {
                log.error("消息发送失败！", ex);
            }
            @Override
            public void onSuccess(SendResult<String, Message> result) {
                RecordMetadata recordMetadata = result.getRecordMetadata();
                log.info("---NO_BLOCKING_MSG_SPRINGBOOT---[topic]={}, [partition]={}, [offset]={}",
                        recordMetadata.topic(), recordMetadata.partition(), recordMetadata.offset());
            }
        });
    }
}

```

6.3> 消费者端
---------

*   初始化消费者配置
    

```
spring:
  kafka:
    consumer:
      group-id: museGroup # 消费组id
      enable-auto-commit: true # 是否自动提交offset
      auto-commit-interval: 1000 # 提交offset延时（接收到消息后多久提交offset）
      # 当kafka中没有初始offset或offset超出范围时，将自动重置offset
      # earliest：第一次从头开始消费，以后按照消费offset记录继续消费
      # latest（默认）：只消费自己启动之后发送到主题的消息。
      auto-offset-reset: latest
      max-poll-records: 500 # 批量消费每次最多消费多少条记录
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        session.timeout.ms: 120000 # 消费会话超时时间（超过这个时间consumer没有发送心跳，就会触发rebalance操作）
        request.timeout.ms: 180000 # 消费请求超时时间
        spring:
          json:
            trusted:
              packages: com.muse.kafkademo.entity
    listener:
      ack-mode: record

```

*   消费消息
    

```
private final static String TOPIC_NAME = "muse-rp";
private final static String CONSUMER_GROUP_NAME = "museGroup";
/**
 * 消息消费演示
 */
@KafkaListener(topics = TOPIC_NAME, groupId = CONSUMER_GROUP_NAME)
// public void listenGroup(ConsumerRecord<String, Message> record, Acknowledgment ack) {
public void listenGroup(ConsumerRecord<String, Message> record) {
    log.info(" [topic]={}, [partition]={}, [offset]={}, [key]={}, [value]={}", record.topic(),
            record.partition(), record.offset(), record.key(), record.value());
    // ack.acknowledge(); 需要enable-auto-commit: false才可以
}

```

*   消息消费多配置参数演示
    

```
/**
 * 消息消费多配置参数演示
 */
@KafkaListener(
        groupId = "xxGroup",                    // 消费组
        concurrency = "3",                      // 组里创建3个Consumer
        topicPartitions = {
            @TopicPartition(topic = "xxTopic1", // 针对主题"xxTopic1"，要消费分区0和分区1
                    partitions = {"0", "1"}),
            @TopicPartition(topic = "xxTopic2", // 针对主题"xxTopic2"，要消费分区0和分区1，并且针对分区1，offset被设置为100
                    partitions = "0",
                    partitionOffsets = @PartitionOffset(partition = "1", initialOffset = "100"))
})
public void listenGroupPro(ConsumerRecord<String, Message> record) {
    Message message = record.value();
    log.info("msgId={}, content={}", message.megId, message.content);
}

```

八、Kafka 集群概念补充
==============

8.1> Controller
---------------

*   Kafka 集群中的 Broker 在 ZK 中创建临时序号节点，序号最小的节点也就是最先创建的那个节点，将作为集群的 Controller，负责管理整个集群中的所有分区和副本的状态。控制器的作用如下：
    

*   当某个分区的 leader 副本出现故障时，由控制器负责为该分区选举除新的 leader 副本。
    
*   当检测到某个分区的 ISR 集合发生变化时，由控制器负责通知所有 broker 更新其元数据信息。
    
*   当使用 kafka-topics.sh 脚本为某个 topic 增加分区数量时，同样还是由控制器负责让新分区被其他节点感知到。
    

8.2> Rebalance 机制
-----------------

*   当消费者没有指明分区消费时，消费组里的消费者和分区关系发生了变化，那么就会触发 rebalance 机制。
    
*   这个机制会重新调整消费者消费哪个分区。
    

*   在触发 rebalance 机制之前，消费者消费那个分区有 3 中策略：
    

*   1> **range**
    

通过公式来计算某个消费者消费那个分区。

*   2> **轮询**
    

大家轮流对分区进行消费。

*   3> **sticky**
    

在触发 rebalance 之后，在消费者消费的原分区不变的基础上进行调整。

8.3>HW 和 LEO
------------

*   HW（HighWatermark）俗称高水位，取一个 partition 对应的 ISR 中最小的 LEO（log-end-offset）作为 HW；
    
*   Consumer 最多只能消费到 HW 所在的位置，每个副本都有 HW，Leader 和 Follower 各自负责更新自己的 HW 的状态。
    

*   对于 Leader 新写入的消息，Consumer 不能立刻消费，Leader 会等待该消息被所有 ISR 中的副本同步后更新 HW，此时消息才能被 Consumer 所消费。这样就能保证如果 Leader 所在的 broker 失效，该消息仍然可以从新选举的 Leader 中获取。
    
*   具体逻辑如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8IDNLNiaGIfs4icCgSnvxCGyfia7zkPS2O7Ocd8cI854tT6ehzAQpWbCwbjnS7n2GpGuc9eQlA5oaDw/640?wx_fmt=png)

九、Kafka 常见面试题  

================

9.1> 如何防止消息丢失
-------------

*   针对发送方
    
    ack1 或者 - 1/all 可以防止消息丢失，如果要做到 99.9999%，ack 要设置为 all，并把 min.insync.replicas 配置成分区备份数
    
*   针对接收方
    
    把自动提交修改为手动提交。
    

9.2> 如何防止消息的重复消费
----------------

*   一条消息被消费者消费多次，如果为了不消费到重复的消息，我们需要在消费端增加幂等性处理，例如：
    

*   通过 mysql 插入业务 id 作为主键，因为主键具有唯一性，所以一次只能插入一条业务数据。
    
*   使用 redis 或 zk 的分布式锁，实现对业务数据的幂等操作。
    

9.3> 如何做到顺序消费
-------------

*   针对发送方
    
    在发送时将 ack 配置为非 0，确保消息至少同步到 leader 之后再返回 ack 继续发送。但是，只能保证分区内部的消息是顺序的，而无法保证一个 Topic 下的多个分区总的消息是有序的。
    
*   针对接收方
    
    消息发送到一个分区中，值配置一个消费组的消费者来接收消息，那么这个 Consumer 所接收到的消息就是有顺序的了，不过这也就牺牲掉了性能。
    

9.4> 如何解决消息积压的问题
----------------

*   消息积压会导致很多问题，比如：磁盘被打满、Producer 发送消息导致 kafka 性能过慢，然后就有可能发生服务雪崩。解决的方案如下所示：
    

*   方案 1：提升一个 Consumer 的处理能力。即：在一个消费者中启动多个线程，让多线程加快消费的速度。
    
*   方案 2：提升总体 Consumer 的处理能力。启动多个消费组，增加 Consumer 的数量从而提高消费能力。
    
*   方案 3：如果业务运行，设定某个时间内，如果消息仍没有被消费，那么 Consumer 收到消息后，直接废弃掉，不执行下面的业务逻辑。
    

9.5> 如何实现延迟队列
-------------

*   应用场景：订单创建成功后如果超过 30 分钟没有付款，则需要取消订单。
    
*   步骤一：创建一个表示 “订单 30 分钟未支付” 的 Topic
    
    order_not_paid_30m：延迟 30 分钟的消息队列。
    
*   步骤二：Producer 发送消息的时候，消息内容要带上订单生成的时间 create_time。
    
*   步骤三：Consumer 消费 Topic 中的消息，如果发现 now 减去 create_time 不足 30 分钟，则不去消费下次，记录当前的 offset，不去消费当前以及之后的消息。
    
*   步骤四：通过记录的 offset 去获取消息，如果发现消息已经超过 30 分钟且订单状态是 “未支付”，那么则将订单状态设置为 “取消”，然后或许下一个 offset 的消息。