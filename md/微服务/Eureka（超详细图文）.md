

    在微服务中，服务注册与发现的重要性不言而喻，现在市面上流行的除了 Dubbo 之外，也包含 SpringCloud Alibaba 的 Nacos 和今天我们要介绍的 SpringCloud Netflix 的 Eureka，那么还是老套路，先看一下本篇文章的大纲吧。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2Wnt72jf7WBqcYB6oJH2xGcEgK8Pa3f6mqOz7HzqRicGQ4P5PibYMcvbfA/640?wx_fmt=png)

一、服务的演变
=======

1.1> 单体服务
---------

*   一个项目中包含了所有的服务。如下图所示，即：所有服务都在 service 模块中： 
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WxCXLqibFJEXqz5AyDNvsHymTjlltGibceflMZcX7Jib52wWGqXSyzZtgQ/640?wx_fmt=png)

*   优点：
    

*   1> 架构简单，清晰。服务之间调用方便，快捷。
    
*   2> 服务部署简单。部署量少。运维量很低。
    

*   缺点：
    

*   1> 随着项目进展时间变长，整个项目的**代码复杂度越来越高**，很容易一个小改动，导致很大的系统问题。
    
*   2> 由于代码量很大，**编译和部署会越来越慢**，甚至 20~30 分钟都很正常。
    
*   3> 由于有的功能属于大量运算的 **CPU 密集型**模块，有的是大量读写磁盘的 **IO 密集型**模块。但是由于融合在了一个项目中，无法针对单个功能模块进行扩展。那么只能 CPU 和内存和磁盘都要提升，资源投入很大。
    
*   4> 如果我们想要针对已有项目改变技术选型，那么就需要针对整个项目进行修改，工作量将会巨大。
    

1.2> 微服务  

-----------

*   微服务的核心就是把传统的单体服务，**根据业务拆分**为一个个的服务，实现解耦；每个服务都可以提供特定业务的功能，每个服务都能够单独部署，也可以拥有自己的数据库。这样的一个个的小服务就是微服务。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WoyjXM0MhDWWTFg1qTQFx4P2A3icEkSVArjmzGHbArq0AI9R9Qwwx95g/640?wx_fmt=png)

*   优点：
    

*   1> 每个服务足够小，足够内聚，**专注于一个业务功能点**提供服务。代码更容易理解。
    
*   2> 有**代码修改**或**部署上线**，只会影响对应的微服务，而不会是整个服务。
    
*   3> 可针对服务是计算型还是 IO 型进行针对性的**硬件升级**。
    
*   4> 可以针对某些**高吞吐服务**进行硬件升级或者服务横向扩容，而不是对所有服务都升级。节约投入成本。
    

*   缺点：
    

*   1> 极大的**增加了运维工作量**，以前几个 war 包，现在可能需要部署几百个。
    
*   2> 微服务之间的互相调用，会**增加通讯成本**。
    
*   3> 分布式事务问题会引出**数据一致性**的问题。
    
*   4> 服务增多，如果管控成百上千的服务。如何准确并快速**定位问题**。
    

二、SpringCloud 微服务生态圈  

=======================

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WdZ86Cg1fWl4zjOYfgPTkia6ajTeUJ7nOVFkycicMBD2I0Vjqym1z17MA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WiamD2Vx02VwtfTibs7SuF34pjnWcCFX5iczVDk8gF0WZROhsKyibictnJ1A/640?wx_fmt=png)

三、什么是服务治理  

============

3.1> 定义说明
---------

*   服务治理可以说是微服务架构中**最为核心和基础**的模块，它主要用来实现各个微服务实例的**自动化注册**与**发现**。
    
*   在传统的系统部署中，服务运行在一个**固定的已知的 IP 和端口上**，如果一个服务需要调用另一个服务，那么可以通过地址直接调用。但是，在虚拟化或者容器化的环境中，**服务实例的启动和销毁是很频繁的**，那么**服务地址也是在动态变化**的。
    
*   如果需要将请求发送到动态变化的服务实例上，至少需要两个步骤：
    

*   **1>** **服务注册**
    
    存储服务的主机和端口信息。
    
*   **2>** **服务发现**
    
    允许其他用户发现服务信息（在注册阶段存储的）。
    

*   服务治理的好处
    

*   **1>** 可以不用了解架构的部署拓扑环境，**只通过服务的名字就能够使用服务**，提供了一种服务发布与查找的协调机制。
    
*   **2>** 关键能力：
    
    服务注册、服务目录、服务查找
    
*   **3>** 其他能力：
    
    监控监控、多种查询、实时更新、高可用性等
    

3.2> 服务发现的两种表现方式  

-------------------

### 3.2.1> 客户端——服务发现 （client-side service discovery）

*   客户端通过查询服务注册中心，获取可用服务的实际网络地址（IP&PORT）。然后通过负载均衡算法来选择一个可用的服务实例，并将请求发送至该服务。
    
*   在服务启动的时候，向服务注册中心注册服务；在服务停止的时候，向服务注册中心注销服务。服务注册的一个典型实现方式就是通过 **heartbeat 机制（心跳机制）**定时刷新。Netflix OSS 就是使用客户端发现方式的一个很好的例子。Netflix Eureka 是一个服务注册中心。它提供一个管理和查询可用服务的 REST API。负载均衡功能是通过 Netflix Ribbon（是一个 IPC 客户端）和 Netflix Eureka 共同实现的。
    

*   如图下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W41ClTspiahRmODtAVlWUZQhhsMRbk49NUgI2xIBicU1ia03U9amibTjwDg/640?wx_fmt=png)

### 3.2.2> 服务端——服务发现 （server-side service discovery）  

*   客户端向 load balancer 上发送请求。load balancer 查询服务注册中心，找到可用的服务，然后转发请求到该服务上。和客户端发现一样，服务都要到注册中心进行服务的注册和销毁。
    
*   AWS 的弹性负载均衡（Elastic Load Balancer——ELB）就是服务端发现的一个例子。ELB 通常是用于为外网服务提供负载均衡的。当然，你也可用使用 ELB 为内部虚拟私有云（VPC）提供负载均衡服务。客户端通过使用 DNS 名称，发送 HTTP/TCP 请求到 ELB。ELB 为 EC2 或 ECS 集群提供负载均衡服务。AWS 并没有提供单独的服务注册中心。而是通过 ELB 实现 EC2 实例和 ECS 容器注册的。
    

*   Nginx 不仅可以作为 HTTP 反向代理服务器和负载均衡器，也可以用来作为一个服务发现的负载均衡器。
    
*   Kubernetes 和 Marathon 是在通过集群中每个节点都运行一个代理来实现服务发现的功能的，代理的角色就是 server-side service discovery，客户端通过使用主机的 IP 地址和 Port 向 Proxy 发送请求，Proxy 再将请求转发到集群中的任意一台可用的服务上。
    

*   如图下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WCwkSUvVCA2SKib1o7N8qYNja1uKmOtcYtMbzM3LERJCxW3kseABiaWxA/640?wx_fmt=png)

四、服务治理技术对比  

=============

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2Ws0kOVLPDf59XmbicqdVe7nXkF8FoRTxUx2yabv2L6NWqlnRIu1fBXGA/640?wx_fmt=png)

【注】网上有的的说 Consul 是 ca，但是官网介绍，它其实是 CP 架构的，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WXYQHV4vOa3rrfvWYyz3XzicTR7FrDFjAJNia2Icfb3MrDQ1xu6exAGaw/640?wx_fmt=png)

五、CAP、ACID、BASE 的理论知识  

========================

5.1> CAP
--------

### 5.1.1> 什么是 CAP

*   CAP 原则又称 CAP 定理，指的是在一个分布式系统中：**一致性**、**可用性**、**分区容错性**；三者不可得兼。
    
*   CAP 原则是 **NOSQL 数据库的基石**。
    

*   分布式系统的 CAP 理论的三个特性如下所述：
    

*   **Consistency（一致性）**
    
    在分布式系统中的所有数据备份，**在同一时刻是否同样的值**。（等同于所有节点访问同一份最新的数据副本）
    
*   **Availability（可用性）**
    
    在集群中一部分节点故障后，**集群整体是否还能响应客户端的读写请求**。（对数据更新具备高可用性）
    
*   **Partition tolerance（分区容忍性）**
    
    一个分布式系统里面，节点组成的网络本来应该是连通的。然而可能因为一些故障，使得有些**节点之间不连通**了，整个网络就分成了几块区域。数据就散布在了这些不连通的区域中。这就叫**分区**。当你一个数据项只在一个节点中保存，那么分区出现后，和这个节点不连通的部分就访问不到这个数据了。这时分区就是**无法容忍**的。提高分区容忍性的办法就是**一个数据项复制到多个节点上**，那么出现分区之后，这一数据项就可能分布到各个区里。容忍性就提高了。然而，要把数据复制到多个节点，就会带来**一致性的问题**，就是多个节点上面的数据可能是不一致的。要保证一致，每次写操作就都要等待全部节点写成功，而这等待又会带来可用性的问题。总的来说就是，数据存在的节点越多，分区容忍性越高，但要复制更新的数据就越多，一致性就越难保证。为了保证一致性，更新所有节点数据所需要的时间就越长，可用性就会降低。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W532AAAAhjzEg7fGX7WRxiaiaaqJk7MdUC5StX6ZYRIRedd8PLunwRhuw/640?wx_fmt=png)

### 5.1.2> CAP 理想的运行情况    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2Wh7xZlAv6yJU41fVjpcEZeaokqTJaGViaBkbjQoycRBzrJ0PfgcpZj3w/640?wx_fmt=png)

### 5.1.3> 验证 CAP 理论    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WMYeB4Aib4vRKAPhbaHK1eGKibwv50xc0FV5dWsiaDep0NA2Azt8KFoxeg/640?wx_fmt=png)

### 5.1.4> CAP 特性的取舍

我们分析一下既然可以满足两个，那么舍弃哪一个比较好呢？

*   **满足 CA 舍弃 P**
    
    也就是满足一致性和可用性，**舍弃容错性**。但是这也就意味着你的系统不是分布式的了，因为涉及分布式的想法就是把功能分开，部署到不同的机器上
    
*   **满足 CP 舍弃 A**
    
    也就是满足一致性和容错性，**舍弃可用性**。如果你的系统允许有段时间的访问失效等问题，这个是可以满足的。就好比多个人并发买票，后台网络出现故障，你买的时候系统就崩溃了。
    
*   **满足 AP 舍弃 C**
    
    也就是满足可用性和容错性，**舍弃一致性**。这也就是意味着你的系统在并发访问的时候可能会出现数据不一致的情况。
    

实时证明，**大多数都是牺牲了一致性**。像 12306 还有淘宝网，就好比是你买火车票，本来你看到的是还有一张票，其实在这个时刻已经被买走了，你填好了信息准备买的时候发现系统提示你没票了。这就是牺牲了一致性。

但是不是说牺牲一致性一定是最好的。就好比 mysql 中的事务机制，张三给李四转了 100 块钱，这时候必须保证张三的账户上少了 100，李四的账户多了 100。因此需要数据的一致性，而且什么时候转钱都可以，也需要可用性。但是可以转钱失败是可以允许的。

5.2> ACID  

------------

*   ACID，指**数据库事务**正确执行的四个基本要素的缩写，包含：**原子性（Atomicity）**、**一致性（Consistency）**、**隔离性（Isolation）**、**持久性（Durability）**。
    
*   一个支持事务（Transaction）的数据库，必须要具有这四种特性，否则在事务过程（Transaction processing）当中无法保证数据的正确性，交易过程极可能达不到交易方的要求。
    

5.6> BASE  

------------

### 5.6.1> 理论说明

*   BASE 是 **Basically Available（基本可用）**、**Soft state（软状态）**和 **Eventually consistent（最终一致性）**三个短语的简写，BASE 是**对 CAP 中一致性和可用性权衡的结果**，其来源于对大规模互联网系统分布式实践的结论，是基于 CAP 定理逐步演化而来的。
    
*   BASE 理论是 eBay 架构师提出的。
    
*   **定理来源**
    
    是 **CAP 中**一致性和可用性的权衡结果，它来自大规模互联网分布式系统的总结，是基于 CAP 定理逐步演化而来的。
    
*   **核心思想**
    
    即使**无法做到强一致性**（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统**达到最终一致性**（Eventual consistency）。
    

**原理解释**

*   **基本可用（Basically Available）**
    

*   是指分布式系统在出现故障的时候，**允许损失部分可用性**，即**保证核心功能可用**。
    
*   例如：电商大促时，为了应对访问量激增，部分用户可能会被引导到降级页面，服务层也可能只提供**降级服务**。这就是损失部分可用性的体现。
    

*   **软状态（ Soft State）**
    

*   软状态是指**允许系统数据存在中间状态**，而该中间状态不会影响系统整体可用性。分布式存储中一般一份数据至少会有三个副本，允许不同节点间副本**同步延时**就是软状态的体现。mysql replication 的异步复制也是一种体现。
    

*   **最终一致性（ Eventual Consistency）**
    

*   最终一致性是指系统中的所有数据副本经过一定时间后，最终能够达到一致的状态。**弱一致性和强一致性相反，最终一致性是弱一致性的一种特殊情况**。
    

### 5.6.2> 原理解释

#### a> 基本可用性

*   **损失相应时间**
    
    CAP 可用性的服务相应时间可能是 10ms，而 BASE 基本可用性的相应时间 1-2s，即允许损失部分可用性的（时间上的损失）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WOg5pqdzpnDaV5oPrQvUXvos35holobfIBy6uLOTzDfx3XiaqalTialYw/640?wx_fmt=png)

*   **损失系统功能**
    
    BASE 的基本可用性是允许某个服务出现故障时，采用服务降级等手段保证用户的体验性，即允许损失部分系统功能。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WRpucIVOrxHTwpfJ17p5gFZzzVn5s3HyG1teGANL2NUJ9NV3fYibwIkg/640?wx_fmt=png)

#### b> 软状态

*   我们在之前学习过硬状态，指的就是 ACID 的原子性。
    
*   **硬状态**
    
    只有在订单状态、积分发送成功、仓库出单成功，即三者同时成功的情况才算支付成功。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WXMn1gf6Qe2uibhAlZ79w1JKEWuXKCSeXcUHTdRKZTCgicSVj59PZjanQ/640?wx_fmt=png)

*   **什么是软状态？**
    
    即不要完全符合 ACID 的原子性
    
    例如：先把订单状态改成已支付成功，然后告诉用户已经成功了。剩下在异步发送 mq 消息通知积分服务和仓库服务，即使消费失败，MQ 消息也会重新发送（重试）。![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WAPyJ1IibNPGuW70ZKfiaks0xpzAV0TOk1SibhoHSjd9YL9TnaF6RP1AzA/640?wx_fmt=png)
    

#### c> 最终一致性

*   **强一致性**
    
    指 ACID 的原子性，和硬状态一样。
    
*   **最终一致性**
    
    和强一致性相反，数据不用即时一致。
    
    实现最终一致性：一般是通异步来实现的，失败了就重试。在不影响用户体验的情况下，可以延时，即使失败了采用重试处理。![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WYGkdSPbWbSU7Oicpsicmshad9mRoJBMTZEro1o6sUbVfcbxd9SDyKW9g/640?wx_fmt=png)
    

六、Eureka 实战与原理  

=================

6.1> 搭建服务注册中心（单机）
-----------------

*   通过 @EnableEurekaServer 注解，来开启一个服务注册中心
    

```
package com.muse.springcloud.eurekademo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
@EnableEurekaServer // 开启一个服务注册中心
@SpringBootApplication
public class EurekaDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaDemoApplication.class, args);
    }
}

```

*   增加相应配置 application.properties
    

```
# Eureka服务端应用的端口，默认为8761
server.port=8000
# 设置当前Eureka实例的hostname（http://localhost:8000）
eureka.instance.hostname=localhost
# 暴露给其他Eureka Client的注册地址
eureka.client.service-url.defaultZone=http://localhost:8000/eureka/
# 由于该应用为注册中心，所以设置为false，代表不向注册中心注册自己，默认为true
eureka.client.register-with-eureka=false
# 由于注册中心的职责就是维护服务实例，它并不需要去检索服务，所以也设置为false
eureka.client.fetch-registry=false

```

*   访问 http://localhost:8000   ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WMwicm4Le1UVcomlzVcDvMAQjUZy7kVQCzvhmbTBiaEuKP30icq3CuqUYw/640?wx_fmt=png)
    
*   Eureka 自我保护机制（上图中的红色字体警告）
    

*   服务注册到 Eureka Server 之后，**会****维护一个心跳连接**，告诉 Eureka Server 自己还活着。也就是说，默认每隔一段时间（默认为 **60 秒**）将当前清单中超时（默认为 **90 秒**）没有续约的服务剔除出去。
    
*   但是当网络发生故障的情况下，微服务和 Eureka Server 就无法通讯，这样就很危险，因为微服务本身是健康的，此时不应该注销该服务，那此时，Eureka 的自我保护机制就排上用场了。
    

*   即：在运行期间，会统计心跳失败的比例在 **15 分钟内**是否**低于 85%**，如果出现低于的情况（在单机调试的时候很容易满足，在生产环境上通常是由于网络不稳定导致），Euraka Server 会将当前的实例注册信息保存起来，让这些实例不会过期尽可能的保护这些注册信息。当网络健康的时候，EurekaServer 就会自动退出自我保护机制。
    

*   访问 http://localhost:8000/eureka/apps
    

    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2Ww84rSLDiaDhOPPMhGaNGibvFiaoWn5LK6ibECFGtuZIN01r6hkX7SLg9Gw/640?wx_fmt=png)

6.2> 服务提供者
----------

*   通过 @EnableDiscoveryClient 注解，来激活 DiscoveryClient 实现
    

```
@EnableDiscoveryClient // 激活DiscoveryClient实现，可以不使用改注解
@SpringBootApplication
public class EurekuService1Application {
    public static void main(String[] args) {
        SpringApplication.run(EurekuService1Application.class, args);
    }
}

```

*   增加相应配置 application.properties
    

```
# 端口号
server.port=7000
# 服务名
spring.application.name=eureka-client-producer
# 指定服务注册中心的地址
eureka.client.service-url.defaultZone=http://localhost:8000/eureka/

```

*   请求地址
    

```
@RestController
public class HelloController {
    @Resource
    private Registration registration;
    @Resource
    private DiscoveryClient client;
    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String index() {
        List<ServiceInstance> instances = client.getInstances(registration.getServiceId());
        for (ServiceInstance instance : instances) {
            System.out.println(String.format("/hello host=%s serviceId=%s", instance.getHost(),
                    instance.getServiceId()));
        }
        return "Hello World!";
    }
}

```

*   访问：http://localhost:7000/hello
    

    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WYkAkfoiaI9XH509TJQrnTsUhp3EibQJoUAk7mSQuInXuzAuXAKgCn1Jg/640?wx_fmt=png)

日志输出：

```
/hello host=172.24.134.20 serviceId=EUREKA-CLIENT-PRODUCER

```

*   访问 http://localhost:8000，已经可以看到 EUREKA-CLIENT-PRODUCER 这个服务被注册进去了。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W8wwqH1aa8CLUfm93YgSdTK46hUr0OY7hCUaE7deEhUkQB1qkqdIRyA/640?wx_fmt=png)

*   访问：http://localhost:8000/eureka/apps
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W4tuiaPGy4XKx6mPxbJ70MWCGLIfz2c4Ozwdic5hlLafvQCghtlvvBbWQ/640?wx_fmt=png)

6.3> 服务消费者  

-------------

*   再此之前，我们先启动两个 producer，用于稍后验证负载均衡。修改端口为 7000，启动。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WYmlxsSEK9ddDibOKYUxBWMPghWXTgibnHlFuhUEIOwNSOwPZWAjhs0uA/640?wx_fmt=png)

*   修改端口为 7001，并且勾选 Allow parallel run，才可以在一个 eureka-client-producer 项目中启动两个 producer。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WsHgEP4iaCrPQz85TQXribVK65nVwdicW4dLt1eqzZkmadrWicH0Cbrdjbg/640?wx_fmt=png)

*   启动后，效果如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WdGlYnduYzd0mQF7iamx5RdnHmNcp5ibTGXbe1YtUAgx7jO3MaWpjmBicg/640?wx_fmt=png)

*   通过 @EnableDiscoveryClient 注解，来激活 DiscoveryClient 实现；同时，在该主类中创建 RestTemplate 的 Spring Bean 实例，并通过 @LoadBalanced 注解开启客户端的负载均衡。
    

```
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaClientConsumerApplication {
    /** 创建RestTemplate的Spring Bean实例，并通过@LoadBalanced注解开启客户端的负载均衡 */
    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientConsumerApplication.class, args);
    }
}

```

*   增加相应配置 application.properties
    

```
# 服务名
spring.application.name=eureka-client-consumer
# 端口号
server.port=7002
# 指定服务注册中心的地址
eureka.client.service-url.defaultZone=http://localhost:8000/eureka/

```

*   请求地址
    

```
@RestController
public class ConsumerController {
    @Resource
    private Registration registration;
    @Resource
    private DiscoveryClient client;
    @Resource
    private RestTemplate restTemplate;
    @RequestMapping(value = "/consumer/hello", method = RequestMethod.GET)
    public String index() {
        return restTemplate.getForEntity("http://eureka-client-producer/hello", String.class).getBody();
    }
}

```

*   启动后，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WicJAyWkE0j49oC7OWqFDkuKkhkQo9H22JraupJfuFdzCgu5lbd0g3Ng/640?wx_fmt=png)

*   访问 http://localhost:8000，已经可以看到 EUREKU-CLIENT-PRODUCER（2 个服务）和 EUREKU-CLIENT-CONSUMER（1 个服务）服务都被注册进去了。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WqRVI5IXpww0ggSnthfrNTT97RGUUvia9b6cIxF5lMwLrPQZLCmBnZ6Q/640?wx_fmt=png)

*   第一次访问：http://localhost:7002/consumer/hello 只有 7001 有日志输出：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W8jkMMgOfAeicOpScQS6icYuicKumOTtjyVwIogW8lNY79Tciatu4NAiafeg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WeErebfbasf9SiaprNLroSehThyDZNs4kmzBYISrtWl09bHvIWdSicplg/640?wx_fmt=png)

*   第二次访问：http://localhost:7002/consumer/hello 发现 7000 也有日志输出了：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WgQPHrZDI0Qm0qLRM0yTWUicP30p1VI3AclCkrqF6YLuHuVNTNTibsGQg/640?wx_fmt=png)

【注意】顺序不一定是按照文档顺序输入的，但是会发现，两台服务会有负载均衡的方式来接收每次的请求。  

6.4> 搭建服务注册中心（集群）  

--------------------

*   配置好 host  
    

```
vim /etc/hosts
127.0.0.1 cluster1
127.0.0.1 cluster2

```

*   分别新建两个工程——eureka-cluster1 和 eureka-cluster2。配置文件分别为：**eureka-cluster1 的 application.properties**
    

```
# 同一个集群，应用名称保持一致
spring.application.name=eureka-cluster

server.port=8001

# http://cluster1:8001
eureka.instance.hostname=cluster1

# 注册到cluster2:8002里
eureka.client.service-url.defaultZone=http://cluster2:8002/eureka/

```

*   **eureka-cluster2 的 application-cluster2.properties**
    

```
# 同一个集群，应用名称保持一致
spring.application.name=eureka-cluster

server.port=8002

# http://cluster2:8002
eureka.instance.hostname=cluster2

# 注册到cluster1:8001里
eureka.client.service-url.defaultZone=http://cluster1:8001/eureka/

```

*   启动这两个服务，访问 http://cluster1:8001 和 http://cluster2:8002 ，registered-replicas 已经可以看到彼此了。  
    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WqGDVR1O85I3F4T4H0AJmSW8eXJrPwV9O9leYFmSNgh8026oQHpIMkw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2Wutic5w3g7GGoAdQX2f8piaKKp949LW9pADVhicpfHDFco8iaWSdaRKUkOQ/640?wx_fmt=png)
    
*   同一个 jar 包，启动时读取不同的配置文件
    

```
java -jar eureka-demo-0.0.1-SNAPSHOT.jar --spring.profiles.active=cluster1
java -jar eureka-demo-0.0.1-SNAPSHOT.jar --spring.profiles.active=cluster2

```

七、Eureka 核心功能  

================

7.1> Eureka 提供的功能
-----------------

*   **服务注册功能**
    
    Client 端会**通过 REST 请求**的方式，向 Server 端注册自己的服务信息（包括：ip，port，url 等信息）
    
    Server 端会将该信息存储在**双层 Map** 中，其中，第一层：服务名  第二层：具体服务的实例名（后面源码会涉及到）
    
*   **服务续约功能**
    
    服务注册成功后，Client 端会默认**每隔 30 秒**来通知 Server 端（心跳机制），告诉 Server 端，自己还存活着。
    

```
eureka.instance.lease-renewal-interval-in-seconds=30

```

*   **服务同步功能**
    
    Eureka 集群中，Server 之间通过**相互注册**来进行同步，保证注册服务信息的一致性。**同步临界点**。
    
*   **服务获取功能**
    
    Client 端启动后，会发送一个 REST 请求给 Server 端，然后从 Server 端获取已注册的服务列表，并**缓存 30 秒**到 Client 端本地。同时，为了性能考虑，Server 端也会维护一份**只读 30 秒缓存**的的服务列表。
    
    服务注册：内存——> 读写缓存（**180 秒清除 1 次**）——> 只读缓存
    
    服务获取：只读缓存——> 读写缓存——> 内存
    

```
eureka.client.fetch-registry=true
eureka.client.registry-fetch-interval-seconds=30

```

*   **服务调用功能**
    
    Client 端通过服务列表，来查到远程服务地址，并进行调用。其中，Eureka 有 **Region** 和 **Zone** 的概念，一个 Region 可以包含多个 Zone，在进行服务调用时，优先访问处于同一个 Zone 中的服务提供者。
    
*   **服务下线功能**
    
    当 **Client 需要关闭或重启**的时候，可以发送 REST 请求给 Server 端，告知自己下线的请求，此时 ，服务状态会置为**下线状态（DOWN）**，并把该状态同步给集群中的其他 Server 节点。
    
*   **服务剔除功能**
    
    当 Client 端没有发送下线通知给 Server，但是，由于网络故障等原因导致不能提供服务，那么就会触发服务剔除机制。Eureka Server 在启动的时候，会创建一个定时任务，默认**每隔 60 秒**从当前服务列表中把**超时 90 秒****（默认）**没有续约的服务剔除掉。
    

```
eureka.instance.lease-expiration-duration-in-seconds=90

```

*   **自我保护**
    
    如果由于网络在一端时间内发生了异常，导致所有服务都没有续约，那么为了防止 Server 端把所有服务都剔除了，就有了自我保护机制，即：**当 15 分钟内**，出现了**低于 85% 的续约失败比**，那么则会**触发自我保护机制**。该机制下不会对任务服务进行剔除操作。等网络正常后，再退出自我保护机制。
    

```
eureka.server.enable-self-preservation=true

```

7.2> Eureka 常用的 http rest 接口  

-------------------------------

### 7.2.1> 查看所有服务的注册列表

*   GET http://localhost:8000/eureka/apps
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WoCD2Ia2YzNuVYvnHPAOpwagnPBia5uHXk7x4bRALGr18xtNXiagiaf3Eg/640?wx_fmt=png)

----------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WzFdX9tI5srAhMl1w5PvoXvFeQxAVhbcyThkZGD577rCn9Lic5BkvmxA/640?wx_fmt=png)

### 7.2.2> 查看某一个服务的注册列表

*   请求方式
    

```
Get http://localhost:8000/eureka/apps/SERVICE-NAME
举例：
http://localhost:8000/eureka/apps/EUREKA-CLIENT-CONSUMER

```

*   查看服务名称为：EUREKA-CLIENT-CONSUMER 的服务信息    
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WhL7j62riaq5uwqyU5IFibq0PHSAGKjRuS3KN2kibVPgy1pry1t1ljKH3A/640?wx_fmt=png)

### 7.2.3> 服务下线

*   请求方式
    

```
PUT http://localhost:8000/eureka/apps/SERVICE-NAME/INSTANCE-NAME/status?value=OUT_OF_SERVICE （强制下线）
举例：
http://localhost:8000/eureka/apps/EUREKA-CLIENT-CONSUMER/172.24.134.20:eureka-client-consumer:7002/status?value=DOWN

```

*   查看被下线的服务实例 172.24.134.20:eureka-client-consumer:7002    
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W9skhcaN0fNnUrBEuQGv0GyF7oLxZvb7YHw4icCiaLfV1qoaViaiaFalibfg/640?wx_fmt=png)

### 7.2.4> 服务恢复

*   请求方式
    

```
PUT http://localhost:8000/eureka/apps/SERVICE-NAME/INSTANCE-NAME/status?value=UP
举例：
http://localhost:8000/eureka/apps/EUREKA-CLIENT-CONSUMER/172.24.134.20:eureka-client-consumer:7002/status?value=UP

```

*   查看被上线的服务实例 172.24.134.20:eureka-client-consumer:7002
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WicQla3wW7lqopaXfxH6KUOffWmaPvtaibmQlysYb0DG868fZ6ONTHSYQ/640?wx_fmt=png)

### 7.2.5> 服务剔除  

*   请求方式
    

```
DELETE http://localhost:8000/eureka/apps/SERVICE-NAME/INSTANCE-NAME
举例：
http://localhost:8000/eureka/apps/EUREKA-CLIENT-CONSUMER/172.24.134.20:eureka-client-consumer:7002

```

*   执行服务剔除操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WNkZEj6c6Pne4otJZKch8ibFT7Ub0vLVdav9TJvosylPnEriaVM89g0sQ/640?wx_fmt=png)

*   查看被剔除的服务实例 172.24.134.20:eureka-client-consumer:7002
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W10jriaibBxSIUWN9fRygwibBQr8ApH1iaOpKJa001sHMqBG7f35Q6bs0oQ/640?wx_fmt=png)【注】由于 eureka-client-consumer 这个服务依然正常运行中，所以，过了几秒后，该服务还会成功的注册到 Eureka 中。

八、Eureka Client 源码解析  

=======================

8.1> @EnableEurekaServer
------------------------

*   用于表示开启了 Eureka 服务端的服务。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WTaLmEnSYOevgQ7niajCAKiblX1TWOPjIYYpqia9EOIVa27w0S0kyHqcmg/640?wx_fmt=png)

*   Eureka 服务 Marker 的配置类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WVtnXT2zzyXC9PhOSBzEFlWgtaARoRZ1Umicm6L1kteZWpI8knUvQnyQ/640?wx_fmt=png)

*   我们获得这个 Marker Bean 是用来干什么的呢？它其实类似一个标记，告诉当前 SpringBoot 我要开启 Eureka Server 的服务。为探究原因，我们可以看一下 Spring Cloud Eureka 包下的 META-INF/spring.factories 文件。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WJCR9eWZ6qTOFFVeHnTkApeeruuG6V6ykIkxoEFEQt2opYm9RKUOuQA/640?wx_fmt=png)

----------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W4ZkpHTjfZUfHlqYIE2Fe12uKBFicHqaTjMkvhPruoiaKracHG8I9QlyQ/640?wx_fmt=png)

----------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WCBiagMib6LoIhicmu2j0j5hl1Ow5SLmhviaxNblym0J53vibeoA0GDwFj0A/640?wx_fmt=png)

*   其中，@ConditionalOnBean 的含义就是，只有存在 Marker 实例，才会初始化 EurekaServerAutoConfiguration 这个类。
    

8.2> dashboard path  

----------------------

*   默认我们通过访问 http://localhost:8000 去访问 Eureka 的控制台。它默认是根路径下。源码设置如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WvroYibbPt1w30k7MQAELjhX0IGPxJx0U5M5mroMmQFbgYCwIQX9icLFQ/640?wx_fmt=png)

*   查看自动配置类中的 EurekaDashboardProperties 类。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WQibIBrLF3iccuu4HRmibibCsHY8eKUZ6BMfcpQHOw6VJ3lCU4WpM521zzA/640?wx_fmt=png)

8.3> @EnableDiscoveryClient  

------------------------------

*   该注解可以不要的。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WnAnicNvX44C9rXhViaiaJk9S8fKE84sz4lGicVLoavHicnvJrwhdAZKjlSw/640?wx_fmt=png)

*   为什么可以不要该注解呢？我们来看一下源码：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WFILOHccEicK0ZlO8dxC11aQzBjlFPricsfzSE48Pf0Cib0EaUvibibnjLWg/640?wx_fmt=png)

*   由于 autoRegister 已经默认是 true 了，所以不需要显示使用该注解了。我们再看一下 Eureka-client 的自动配置类：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WrmG3BFq8WfCr0vjP9go6wtZQ0X52kQ5SxjPd8biaV6nfHS3WwmYGm3A/640?wx_fmt=png)

----------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WVmTFUlhllPiaLapMrib9UG6SibcOxP1lA53V6gO5j5JMb6wAvGScsAW4A/640?wx_fmt=png)

----------------------------------  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WxMEibw3iaP7ckOLiayXbkicuuRgtkgJht7tuLDe3plia7z7CsGcBeJ5Kzuw/640?wx_fmt=png)

九、Eureka Server 源码解析  

=======================

9.1> EurekaServerAutoConfiguration
----------------------------------

*   我们先来看一下，EurekaServerAutoConfiguration 里面的一些重要属性和方法。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WQhRe7W3FH5oggMgSE80LBchGdjFicURfnq9vJIK3YnskmZ8ibbHTd63g/640?wx_fmt=png)

----------------------------------  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WrdYvicgG3821oyge5YKptX5bYpC5rFxP8go2V2Fnz5Ffxn2MiaP7HKrQ/640?wx_fmt=png)

*   我们来看一下被引入的类——EurekaServerInitializerConfiguration
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WQyC1qICQh2D5Bf4Fia1EWaxs6J65jtPeicEuTKU8ggLoMfFLYibqW7S8Q/640?wx_fmt=png)

*   EurekaServerInitializerConfiguration 重点内容如下，但是，是怎么被调用了 start 方法呢？那么我们就需要看一下 Spring IOC 的加载过程。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WqSNS8hOGCatdJvYnUEYGjkOnF2icq4J24mp1C8y2kc9nww0v6S7hGxQ/640?wx_fmt=png)

*   我们看一下 AbstractApplicationContext 的 refresh() 方法  
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WO1LA39Ccf49WIDsfvicmPpBEPhALvhdwt1DAVPW8SrMibicrO3JXor4PA/640?wx_fmt=png)

-------------------------  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WBhZQ2uXmggG2d4QxlPBBMjKAelj0dIKzajoLLjNsslBfxGyfDetz6Q/640?wx_fmt=png)

-------------------------  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W3nesicpgdAXj1nmCW1FiaN4kLhYfFfotxQxibl0QkholZia3fgMv92yJew/640?wx_fmt=png)【注】EurekaServerInitializerConfiguration 的 isAutoStartup 方法返回的就是 true。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WbahiaIsrqDBC5kJCuUpShuYMvTufmHKveDKKvT6WucqX84Siaspr5IzQ/640?wx_fmt=png)

-------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2Wxvm2oRGT1y8nwrbBJYU5GYgSm2oTLgAh9JJib3wEDtXj2Mo35Rb14fQ/640?wx_fmt=png)

*   上面讲完调用 EurekaServerInitializerConfiguration 的 start 方法后，我们回过头来看一下 eurekaServerBootstrap.contextInitialized(EurekaServerInitializerConfiguration.this.servletContext); 的代码
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WVDRicQAB49zeK04o7ERweAgumDdVwOS8OdhZjy4e9GB48b7ia3LL2dcg/640?wx_fmt=png)

-------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2W2jHMVgicZJeFbNKRFjKq08jDQYqhx8vEcd8hgiaWO55NSDHB2xtXX8ZQ/640?wx_fmt=png)

-------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WLicvrczfOUvMO8x2UmJV0l3C8xdWHAicurs323x8WJsJGPSQzyFOSxJw/640?wx_fmt=png)

*   我们来看 this.registry.syncUp()
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WPdBUd6rCOrvdL9VNjGP8hiaibAuBt9YwN86R24x7x42JRKia8oRj319Vg/640?wx_fmt=png)

*   我们来看 this.registry.openForTraffic(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WcpgnbkUctSoOqEQO9NPr13Rs5l7EM27PKOQGqQic3YQhncvGrIKwGWQ/640?wx_fmt=png)

-------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WJbfxaxW4MZPejekpxSB6KlO75zj5C2fib5XVlic9pBRKEDEPAH4c6jnQ/640?wx_fmt=png)

-------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WKEwWAO0r3DzduhQ2AzWUwX0Wrr0YibKYgiakvzQAaqDJZ4b7xFdHiaFQQ/640?wx_fmt=png)

-------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WIA4lIBXFn8lqK3bcFS43eiaauKh15LG6pJziaUVBjNR8qryS1zo4OAwA/640?wx_fmt=png)

-------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9pbE5FArLtK6WBCWQloE2WbpCegbDud1KicgQ21yPfkvyEu9cH2GbeUeGpOfjDzl7vcGkZxJSQD5A/640?wx_fmt=png)