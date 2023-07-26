

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg4RHJhVzSzej04W0dzQPpzKjQtK5lExdyPFF295gDd12J1oJXJEgeWA/640?wx_fmt=png)

一、Sleuth  

===========

1.1> 概述
-------

*   在业务发展前期，我们的项目一般都是比较小的或者是单体架构的，那么针对服务出现的问题，我们可以直接去服务器的 log 目录下去看对应的日志文件来排查系统问题，但是，随着业务的逐步发展，单体变为微服务，整个服务的数量将会剧增，并且服务与服务直接的调用也会越来越复杂，那么我们此时再一个个的登录对应的服务器去看服务日志，显然是不现实的。那么这个时候，对于每个请求，全**链路调用跟踪**就变得越来越重要了，通过实现对请求调用的跟踪可以帮助我们快速发现错误根源以及监控分析每条请求链路上的性能瓶颈。
    
*   Spring Cloud Sleuth 就提供了一套完整的解决方案。下面我们将详细的介绍如何使用 Sleuth 来为微服务框架提供一套微服务的服务调用跟踪能力。
    

1.2> 项目演示
---------

### 1.2.1> 普通日志输出

*   创建两个项目 simple-demo-1 和 simple-demo-2，来实现如下的调用关系：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgrc0ZpNy8BnVrYOpiaojkLHuiajLl1rfLyrvlUQ9icCHG5Z7HAThlyB0Zg/640?wx_fmt=png)

*   首先，启动 **Eureka Server**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgRrXaMw1iajPbjR5epIgJibsapiagttTCG5pomBT4OnSZU9NHLIs6VUyFA/640?wx_fmt=png)

*   创建两个项目 simple-demo-1 和 simple-demo-2，并且都加入如下的 **maven 引用**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg54P0VdCIzMveJyXUDWiaatTXVMWbG2qWXPX7wA7sW8U505z7fibcBZyA/640?wx_fmt=png)

*   为 simple-demo-1 添加 RestTemplate 的 bean
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgYkSQaHvOInP2O1sw9wcNpibXsAtxJQvpxLibUFmm65qWl6t1YicOrBVgg/640?wx_fmt=png)

*   创建 **SpringDemo1Controller.java**，提供 / order 接口，里面通过 restTemplate 发送 HTTP 请求
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg3FNI31WAIPMYSSbHyI8lpk5ofEmmr2QiaeXkvItTia1E8U7Cia418XJfQ/640?wx_fmt=png)

*   创建 **SpringDemo2Controller.java**，提供 / payment 接口用于接收请求，并输出日志
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgzWPOvtalP3MichzXwBF3XG7fXgqcctGby5ugkSfVWnoroBviam5KdT2Q/640?wx_fmt=png)

*   为 simple-demo-1 创建 **application.properties** 配置文件，并指定 Eureka 配置中心地址
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgPzbxaBA9M5wl3z0NKSq6dnLKoibf23Ce2ZbXjkTbau8iaYs1kQfYQHCw/640?wx_fmt=png)

*   为 simple-demo-2 创建 **application.properties** 配置文件，并指定 Eureka 配置中心地址
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg7bBtLaibVPo2fgVyAiaHtfFNib2G7vpvs6BNkpvJ0BbOfVmUGMtju3x2w/640?wx_fmt=png)

*   请求 http://localhost:9000/order，会有如下日志输出：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgUPjFib1yqoklcxiaVIRicNExX1LRWvHSKQIFxicCMmIMRJAzDMF0Xibt5RQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgBvufuoxHE0HZ867RmB4cajw6bibYxqg5b8ygx88x44iagsqQwdVtcChw/640?wx_fmt=png)  

### 1.2.2> 引入 Sleuth 的日志输出  

*   我们只需要在这两个项目中加入 sleuth 的 Maven 依赖，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg5Gib9YYSzduAV0cKkxpDxf96y9TO4v2ib5XGVEbHzibFZuiaS0hAwpW0Xg/640?wx_fmt=png)

*   请求 http://localhost:9000/order，我们会发现，日志信息中新增了一些内容（如红框所示）， 会有如下日志输出：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgxERJs6LPGZU8ZkUicLPcxJ3Ae7bEkMDiag9VwibagcKOWw8ztkvp1icuMA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg3iaHA7jd5uiaSPGq5ojk0wNcERibS8patiaTg3ALx7ktg2pXkUgfCwjl4Q/640?wx_fmt=png)  

【解释】INFO [simple-demo-2,ddfe378c0a8ec7cc,d4f2e63ad9bc890b,true]

*   第一个值：**simple-demo-2**，它记录了应用的名称，也就是 application.properties 配置文件中的 spring.application.name 参数配置的属性值。
    
*   第二个值：**ddfe378c0a8ec7cc**，Spring Cloud Sleuth 生成的一个 ID，称为 Trace ID，它用来标识一条请求链路。一条请求链路中包含一个 Trace ID，多个 Span ID。
    
*   第三个值：**d4f2e63ad9bc890b**，Spring Cloud Sleuth 生成的另外一个 ID，称为 Span ID，它表示一个基本的工作单元，比如发送一个 HTTP 请求。
    
*   第四个值：**true**，表示是否要将该信息输出到 Zipkin 等服务中来收集和展示。
    

*   上面的四个值中，**Trace ID 和 Span ID 是 Spring Cloud Sleuth 实现分布式服务跟踪的核心**。在一次服务请求链路的调用过程中，会保持并传递同一个 Trace ID，从而将整个分布于不同微服务进程中的请求跟踪信息串联起来。例如上面的例子，由于是一次前端请求输出的整个日志链路，所以两个日志内容中的 Trace ID 是相同的。
    

1.3> 跟踪原理  

------------

*   **Trace ID**
    

*   为了实现请求跟踪，当请求发送到分布式系统的入口端点时，只需要服务跟踪框架为该请求创建一个**唯一的跟踪标识**，同时在分布式系统内部流转的时候，框架始终保持传递该唯一标识，直到返回给请求方为止。这个唯一标识就是 Trace ID。
    
*   通过 Trace ID 的记录，我们就能将所有请求过程的日志关联起来。
    

*   **Span ID**
    

*   为了统计各个**处理单元的时间**延迟，当请求到达各个服务组件时，或是处理逻辑到达某个状态时，也通过一个唯一标识来标记它的开始、具体过程以及结束，该标记就是 Span ID。对于每个 Span 来说，它必须有开始和结束两个节点，通过记录开始 Span 和结束 Span 的时间戳，就能统计出该 Span 的时间延迟。
    

*   在上面例子中，当请求要发送到 simple-demo-2 之前，Sleuth 会在该请求的 **Header 中**增加实现跟踪需要的重要信息，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgibpa7CLAPazgOhvnyYouwPQHsnrqryqcDD56ic1uymwWNHRZu32FcYtA/640?wx_fmt=png)

【解释】  

*   **X-B3-TraceId**：一条**请求链路**（Trace）的唯一标识，必需的值。
    
*   **X-B3-SpanId**：一个**工作单元**（Span）的唯一标识，必需的值。
    
*   **X-B3-ParentSpanId**：标识当前工作单元所属的**上一个工作单元**，Root Span（请求链路的第一个工作单元）的该值为空。
    
*   **X-B3-Sampled**：是否被**抽样输出**的标志。1：表示需要被输出；0：表示不需要被输出。
    
*   **X-Span-Name**：工作单元的**名称**。
    

*   接收请求后，日志输出如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgj5s7UbeBNLP54YdbUYVZnlqjaYsPzH0icZicx77nTplUDkvCav8aP2AQ/640?wx_fmt=png)

1.3> 抽样收集  

------------

*   理论上来说，我们收集的跟踪信息越多就可以越好地反映出系统的实际运行情况，并且给出更精准的预警和分析。但是在高并发的分布式系统运行时，大量的请求调用会产生海量的跟踪日志信息，如果收集过多的跟踪信息，如果收集过多的跟踪信息将会对整个分布式系统的**性能造成一定的影响**，同时保存大量的日志信息也需要不少的存储开销。所以，在 Sleuth 中采用了抽象收集的方式来为跟踪信息**打上收集标记**，也就是我们在上面的**第 4 个布尔类型的值**，它代表了该信息是否要被后续的跟踪信息收集器获取和存储。
    
*   如果我们没有配置抽样收集，多次请求 http://localhost:9000/order，我们看到后台日志信息都是 true
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgKMhmf0T7bRJPqgjAbXwCN9Pb8bvd8eHOEThRD65uFEiaMaU9UH53xdw/640?wx_fmt=png)

*   我们可以在 **application.properties** 配置文件中配置 **spring.sleuth.sampler.probability=0.1**，即：代表收集 10% 的请求跟踪信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgrdVw9r7DrDeLlQeMBKH6lve0pHib3cxMg9icrVSbloLTh70rA6QCqHGA/640?wx_fmt=png)

*   重启服务后，多次请求 http://localhost:9000/order，我们看到后台日志信息大部分都是 false 了，只有少数是 true
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgziauIpdNJbSHqmlQviaBr83Niaad7icbXq02j3oAiaFea9mpBtmbgS6RkMw/640?wx_fmt=png)

【解释】

*   如果配置了 spring.sleuth.sampler.probability=0.1 并且重启了服务后，多次请求后依然全都是 true，此时我们需要去 target 目录下看一下 application.properties 中是否是最新的配置信息。如果不是的话，则需要执行 Maven 的 clean 和 package 指令，重新编译打包一下服务即可。
    

二、Elastic Stack  

==================

2.1> 概述
-------

*   通过上面我们引入 Sleuth，已经实现了在各个微服务的日志信息中添加了跟踪信息的功能。但是，由于**日志文件都离散地存储**在各个服务实例的文件系统之上，仅仅通过查看日志文件来分析我们的请求链路依然非常的麻烦，所以我们还需要一些工具来帮助**集中收集**、**存储**和**搜索**这些跟踪信息。比如我们下面要介绍的 Elastic Stack。https://www.elastic.co/cn/what-is/elk-stack
    
*   关于 Elastic Stack 其实会比较生疏一些，但是提到 ELK 大家就比较耳熟能详了。那么，我们先介绍一下 ELK。“ELK” 是三个开源项目的首字母缩写，这三个项目分别是：**Elasticsearch**、**Logstash** 和 **Kibana**。
    

*   Elasticsearch 是一个搜索和分析引擎。
    
*   Logstash 是服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如 Elasticsearch 等 “存储库” 中。
    
*   Kibana 则可以让用户在 Elasticsearch 中使用图形和图表对数据进行可视化。Elastic Stack 是 ELK Stack 的更新换代产品。
    

*   那什么是 Elastic Stack 呢？一切都起源于 Elasticsearch…
    

    这个开源的分布式搜索引擎基于 JSON 开发而来，具有 RESTful 风格。它使用简单，可缩放规模，十分灵活，因此受到用户的热烈好评，而且如大家所知，围绕这一产品还形成了一家专门致力于搜索的公司。

*   引入 Logstash 和 Kibana，产品更强大
    

    Elasticsearch 的核心是搜索引擎，所以用户开始将其用于日志用例，并希望能够轻松地对日志进行采集和可视化。有鉴于此，我们引入了强大的采集管道 Logstash 和灵活的可视化工具 Kibana。

*   然后我们向 ELK 中加入了 Beats
    

    “我只想对某个文件进行 tail 操作，” 用户表示。我们用心倾听。在 2015 年，我们向 ELK Stack 中加入了一系列轻量型的单一功能数据采集器，并把它们叫做 Beats。

*   那么，ELK 需要怎么变化呢？
    

    ELK 这个名称又要变了，的确如此。把它叫做 BELK？BLEK？ELKB？当时的确有过继续沿用首字母缩写的想法。然而，对于扩展速度如此之快的堆栈而言，一直采用首字母缩写的确不是长久之计。

*   就这样，Elastic Stack 这个名字应运而生了
    

    和用户一直以来熟知并喜爱的开源产品一模一样，只是集成程度更高了，功能更加强大了，入门也更加容易了，而且可以带来无限可能。

总结：Elastic Stack 就是 ELK Stack，但是更加灵活，可以帮助人们出色完成各项事务。

*   下面我们来整体的看一下 Elastic Stack 技术栈的应用方式
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgiavrOFXMKloZUqSUjVibW6aZbXHgeM5xLWjHKkIHL5PSY6621FsHMzog/640?wx_fmt=png)

2.2> 前期准备  

------------

### 2.2.1> 安装 & 启动 ZK

*   首先进入 Zookeeper 官网 https://zookeeper.apache.org/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgppFdGd8nkvugzXSiciaBSibM2KCXLeSOiam3qPzR0uZ6BOxib5M4zYHzA3A/640?wx_fmt=png)

*   下载最新版本 ZooKeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgkPsaxzibaVMHMr0slSY2JNOgsnD1n6FHHwyMzBlYI5qc2pibupxDSia4w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg0yTjky0RPySNSib6pLiamGJ35a0cQUeB2XSQsJuPHePwVqSibDDbqalsQ/640?wx_fmt=png)

*   解压到本地路径，进入 ZooKeeper 的 conf 目录下，复制 zoo_sample.cfg 配置文件，命名为 **zoo.cfg**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgnSC2wuIbQpk0srgGeApOzFsFhia6Go6pSa1AQ2zLIKShx8CFBVJoD4g/640?wx_fmt=png)

*   **zoo.cfg** 配置文件中各配置项的含义如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg18ibKGktP5O4w3KO2G0fNUEHqBghxauQvrIMiaiagWs1sgkKSKI2gkibWg/640?wx_fmt=png)

*   启动 zookeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgnnqBkuzNBO7lYgfQByIRGDuY8IzopHaibyOVRnAsh4YwHniaaDKcrtLg/640?wx_fmt=png)

*   查看 zookeeper 的运行状态
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgJEKIQlgOzOtO7xiajGRjhictAbecF8nfHgdB7cEwnojvlpdOtbbuM4RA/640?wx_fmt=png)

### 2.2.2> 安装 & 启动 Kafka

*   首先进入 Kafka 官网下载 kafka http://kafka.apache.org/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKguBYcqLkQZb2Fu46TuDyKdb1XIkZNbibx0Bwj0iaYphO4Pib4y5bOIyibGg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgSuW7VEgmgZABKs6D5DWic5SrF4h5xdOvqTibeUm3fYg5UK6PF7Joz5qw/640?wx_fmt=png)

*   解压，然后进入 config 目录下，编辑 **server.properties** 配置文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgSPZ2shdbLAZCDXeOwoLV5fepOZSRmKjFb8WuHU0tIqJLyKkkz4iaibvg/640?wx_fmt=png)

*   server.properties 配置项解析
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgDEDP0mGtH64hq9iaCO7icfUuIb2T21m0EAfRrAlia7t0JUcYyrXSThXIg/640?wx_fmt=png)

【解释】我们需要关注如下几个配置内容  

*   **broker 的序号**
    
    # The id of the broker. This must be set to a unique integer for each broker.
    

broker.id=0

*   **当前 kafka 的监听地址**
    
    listeners=PLAINTEXT://localhost:9092
    

*   **日志的存储路径**
    
    log.dirs=/Users/muse/kafka_2.13-3.0.0/kafka-logs
    

*   **zookeeper 的服务地址**
    
    zookeeper.connect=localhost:2181
    

*   修改完响应配置后，启动 Kafka，并通过查看 kafka 的进程来判断是否启动起来了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgvTicYWGA4Oot4dh7pNKzBKPFcgb3GNb9zrFWiclgKUwucqU9A6tNfRWw/640?wx_fmt=png)

2.3> Filebeat  

----------------

*   从官网中下载 Filebeat，并解压压缩包。官网地址：https://www.elastic.co/cn/downloads/beats/filebeat
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKga2mZYRm6LEkkntgFW55a2No8u0nB3Jc5LA0gtWuwESc2noDNey9L9A/640?wx_fmt=png)

*   在 **/Users/muse/Lesson/ServerContext/elastic-stack** 目录下创建 msg.log 文件，里面添加日志内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgJpTib9rHZJLBeaTL8gyOVzPV2E2lrRVPVj4mibI1jeDaBxZawWicSkljA/640?wx_fmt=png)

*   编辑 filebeat 的配置文件 **filebeat.yml**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgc97ybhZpo0bXl9dKhibPQx2L7ibbMfia5ZOwib9nqjLxCvM6ffL3XVriawg/640?wx_fmt=png)

*   配置 Filebeat 的 **input** 信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgREPWLiakagPUNTAX8XhfwuJ9wA4YDj1QfT9UaVyF1SPrAxGbMlPnJWQ/640?wx_fmt=png)

*   配置 Filebeat 的 **output** 信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgtROyI7niaqztD8af2AW3TyTibxywFxjsz78uy2drPtbOYempoxg9Oxgw/640?wx_fmt=png)

*   开启 filebeat，**./filebeat -e -c filebeat.yml**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgIF2CpO6ic0g0XwaKCcnj7cvP3mGtzjdqn8sezBGXIptQ9yWjvl4k0WQ/640?wx_fmt=png)

*   创建主题 Topic=filebeat（如果没有这个 Topic，则会在 filebeat 启动时自动去创建这个 Topic）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgnjFerMJfxetHj625pq7BNrfW5Pliba10P6MpEbqSiabiacGSdynjnS9UA/640?wx_fmt=png)

【解释】

*   创建主题：
    
    **kafka-topics.sh --bootstrap-server localhost:9092 --create --topic filebeat --partitions 1 --replication-factor 1**
    

*   查看主题：
    
    **kafka-topics.sh --list --bootstrap-server localhost:9092**
    

*   打开 kafka 的消费端：**./kafka-console-consumer.sh --topic filebeat --bootstrap-server localhost:9092 --from-beginning**，我们可以接收到由 filebeat 发送过来的日志信息。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgfUMaVc1w2PWY23icRvuCzqs7uyVqyfeYUJHjL7cUwIEdfVjqBCzpicYg/640?wx_fmt=png)

2.4> Elasticsearch  

---------------------

*   不需要安装，解压运行即可。官网地址：https://www.elastic.co/cn/elasticsearch/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg82hgrCfrkbeMtJZInD69skrqsBOR9R71sUrTtbTFT6ibLcTHia9Oy33g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgI6TUMM3CNmP2r4Zg8VzhOXltneHPnO9WrhuzEKbiciaeo3DSrcnyB2Ug/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgbh7kCwWHtemyqMicjK22J3ccHhib2nlU6SMM7eVtianJ9Qe9tupYU1RLg/640?wx_fmt=png)

*   运行 **bin/elasticsearch**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg39yUTMj6C93FuibQbtKW41WialSS24fibpUhPYHMMGnGX6ctFTgM8aNhQ/640?wx_fmt=png)

*   访问 http://localhost:9200/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgLLwHibHjulm029iaRsvW1wzmwaFO8aWg5f1q8mX5zfazZ3cbPtS1utTg/640?wx_fmt=png)

2.5> 安装 Node.js  

------------------

*   下载 Node.js 并安装，官网 https://nodejs.org/en/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgwbG0x43Tz8cibEVTr1p9fN1RrYAZMwbpe2gwdBxIqGGHq52pyibOwbicA/640?wx_fmt=png)

*   通过 node -v 和 npm -v 查看是否成功
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgvKWAA6MgaSmtevSyu5KURvAXKkLKnZMDO7Cw8VvKnXgAkyU5bO4KIQ/640?wx_fmt=png)

*   npm install 安装依赖出现 PhantomJS not found on PATH PhantomJS not found on PATH Downloading https://github
    

*   1> 手工下载 https://github.com/Medium/phantomjs/releases/download/v2.1.1/phantomjs-2.1.1-windows.zip
    
*   2> 将下载下来的 phantomjs-2.1.1-macosx.zip 放到 / var/folders/12/hwj74fs53f79l1mcj909s8wh0000gn/T/phantomjs 路径下
    

2.6> 安装 Elasticsearch-head  

-----------------------------

*   安装 elasticsearch-head 需要有 npm 支持
    
*   安装指令
    
    git clone git://github.com/mobz/elasticsearch-head.git
    
    cd elasticsearch-head
    
    npm install
    
    npm run start
    
    open http://localhost:9100/
    
*   官网截图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg5zmwLKa0LZuhp4sNOvNKfF5uqUdQCG2Htx8fcq6GwdEHNXSONDhuSA/640?wx_fmt=png)

*   启动 elasticsearch-head
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgGnx3XTL9Nibe7gyiaibTOpkF0PSZ8REY7oGNmsyLl5FzAhaEtqPHCRgiag/640?wx_fmt=png)

*   elasticsearch-head 界面如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgtFoCjuVyv2NHuXROgQKJbR8ZJiawK0JXQChZ915EdW3rgRMvwYSf8Fw/640?wx_fmt=png)

*   但是点击连接，没有反应，我们来看一下请求响应，发现是**由于 CORS 跨域问题**，导致了连接不上 ES（因为 head 是 9200 端口，而 es 是 9100 端口）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg35g3iaibw7ZHC5MlYcT95gR3KUz5C9uIScIxcRpAzTxy5BMWVVr51brA/640?wx_fmt=png)

*   解决跨域问题。修改 es 的配置文件——**elasticsearch.yml**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg2qmX9u1RUQ7kXfhcIpOEBDiagWLZBwuYEL1A4icL3FnKPJfCM4sxIdCQ/640?wx_fmt=png)

*   在配置文件 elasticsearch.yml 的末尾加上如下配置：
    
    http.cors.enabled: true
    
    http.cors.allow-origin: "*"
    
*   重启 es，发现我们已经可以通过 head 连接到 es 了
    

2.7> Kibana  

--------------

*   下载 Kibana，不需要安装，解压运行即可。注意 **Kibana 的版本定要与 ES 版本一致** https://www.elastic.co/cn/start
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgrN9iacXAqryucouibLOHJsJqB2fEicyX1H9fkKY6JXzzXcG1OmD5EqLTw/640?wx_fmt=png)

*   下载之后解压，执行如下语句，开启 kibana。bin/kibana
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgqYmq9Y0u1kTTvc4Yo8rcXicjsXAM03XXw4jOoOYZmhLETCibXVI9KiavQ/640?wx_fmt=png)

*   访问 http://localhost:5601
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgkq6X078JyFOvh6tMbIZCs96lhWriaZfiasibAhgDviapcXO4uMibKcmZcOw/640?wx_fmt=png)

*   修改为中文显示，打开 kibana-7.15.1/config/**kibana.yml** 配置文件，按照提示修改，然后重启 kibana 即可。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgGic8Ppz5LsaFrGl4F8XX6XVy0JCSVOj7R1YatG4lLKTzxaBmb5mW9XQ/640?wx_fmt=png)

*   访问 http://localhost:5601，我们发现，已经变为中文了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgNjeIX5XbMobfmf40ABr2ticfo8BN5lyFyANVAic6YWGJZLDA89uTr6zw/640?wx_fmt=png)

*   GET /logstash-2022.04.27/_search
    

2.8> Logstash  

----------------

*   从官网中下载 Filebeat，并解压压缩包。官网地址：https://www.elastic.co/cn/downloads/logstash
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgVZ32d3apH0uUunTvYtObJsbKavc1VW1snFdoeGic1HCiaKkibyXRl0e1w/640?wx_fmt=png)

*   编辑 logstash 的配置文件 **logstash.conf**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg0NwaVRftIryERmfCTboiaKg5tZPXw832jsYnGPdSCpVupMibiaNxlpPfQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgZ0w3Vk2L2d8SweFGz1fvdlCkeTwz4eEZVz1umgZ8w9zTWpaz9KZhNA/640?wx_fmt=png)

*   启动 logstash，**./logstash -f /Users/muse/Lesson/ServerContext/elastic-stack/logstash-8.1.3/config/logstash.conf**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg4J5Wy8oOFtqj8JTq2qom6tadtWP0AlqKric3Pa7icWYibzz1aDRlpE5icQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgyQUkGF5RlDw8ibrolpibEBVgXypzzOexRgpgzFkELjnDX3D2qM9Vo1Vw/640?wx_fmt=png)  

*   向日志文件 msg.log 中添加日志信息 “4444444444”
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgHO42X7bicIgicb8oWk9esnc3bOVkCOOaickZfs9RnFuNfwEGQPiaGDYuBg/640?wx_fmt=png)

*   通过 Kafka 的消费端我们可以获得从 Filebeat 中获得的消息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgKjiaWpz5be1s80BuRvDZPdYhTNUsI7IGJ3WMaFiaXHFTaicsJHGFTcLCQ/640?wx_fmt=png)

*   查看 ES-head 页面，查看索引 “logstash-%{+YYYY.MM.dd}” 中的文档数据
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgjgiboqXCMZnkiafUWoSO1eibMEDqZxaiajI1NOhFJpCM41wSYDhAyXlL5w/640?wx_fmt=png)

*   查询 Kibana 页面
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKggicgOl8j1NEMicrzE5nnnSWQkDTZXYKtKR2paYr9xvcfoYaZkUGz6CQQ/640?wx_fmt=png)

2.9> 配置日志输出  

--------------

*   添加 Lombok 和 slf4j 的依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg6Dhq6cOCZ2mCt1YSrCTs9qGZCIaZl6iaylvBS8RXmvyWQHaPdskHOHg/640?wx_fmt=png)

*   application.properties
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgh4M98ZmED7Vr6OKia216Yrd18MKOYsUx7vfZUAxtp9ouIb4o6dReKqg/640?wx_fmt=png)

*   logback-spring.xml
    

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- scan:当此属性设置为true时，配置文档如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文档是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。
                 当scan为true时，此属性生效。默认的时间间隔为1分钟。-->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。-->
<configuration  scan="true" scanPeriod="10 seconds">
    <contextName>logback-spring</contextName>
    <!-- name的值是变量的名称，value的值是变量定义的值。通过定义的值会被插入到logger上下文中。定义后，可以使“${}”来使用变量。-->
    <property  />
    <!--0. 日志格式和颜色渲染 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <!-- 彩色日志格式 -->
    <property ${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <!--1. 输出到控制台-->
    <appender >
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter>
            <level>debug</level>
        </filter>
        <encoder>
            <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <!--2. 输出到文档-->
    <!-- 2.1 level为 DEBUG 日志，时间滚动输出  -->
    <appender >
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${logging.path}/debug.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy>
            <!-- 日志归档 -->
            <fileNamePattern>${logging.path}/debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy>
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录debug级别的 -->
        <filter>
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 2.2 level为 INFO 日志，时间滚动输出  -->
    <appender >
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${logging.path}/info.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy>
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${logging.path}/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy>
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录info级别的 -->
        <filter>
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 2.3 level为 WARN 日志，时间滚动输出  -->
    <appender >
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${logging.path}/warn.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy>
            <fileNamePattern>${logging.path}/warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy>
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录warn级别的 -->
        <filter>
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 2.4 level为 ERROR 日志，时间滚动输出  -->
    <appender >
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${logging.path}/error.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy>
            <fileNamePattern>${logging.path}/error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy>
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录ERROR级别的 -->
        <filter>
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 4. 最终的策略 -->
    <!-- 4.1 开发环境:打印控制台-->
    <springProfile >
        <logger name="此位置填写自己的项目中全mapper\dao路径，用于在开发环境时将sql语句打印在控制台" level="debug"/><!-- 修改此处扫描包名 -->
    </springProfile>
    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="DEBUG_FILE" />
        <appender-ref ref="INFO_FILE" />
        <appender-ref ref="WARN_FILE" />
        <appender-ref ref="ERROR_FILE" />
    </root>
    <!-- 4.2 生产环境:输出到文档 -->
    <springProfile >
        <root level="info">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="WARN_FILE" />
        </root>
    </springProfile>
</configuration>

```

*   请求 order 接口，此时会有日志输出
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgMQ85TYmiaxn0CJicNBlpLZQm5p4aB0TJeUxJbGfWp7XtBQ8aBH7jbVTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgMDSnicpnzBHBLhoSHdAFYIib6KIu29ucgwickibTkFsx7gicFPae4nbAaDw/640?wx_fmt=png)

*   查询 ES 里面，可以看到已经被采集进来了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgafK4qRmBosv5gwKdOqSL1tCCDzDFJuZQj6yUqcx0bdfda38h7HVVAA/640?wx_fmt=png)

三、Zipkin  

===========

3.1> 概述
-------

*   虽然通过 ELK 提供的收集、存储、搜索等强大功能，我们对跟踪信息的管理和使用已经变得非常便利。但是，在 ELK 平台中的数据分析维度**缺少对请求链路中各阶段时间延迟的关注**，很多时候我们追溯请求链路的一个原因是为了找出整个调用链路中出现延迟过高的瓶颈，或为了实现对分布式系统做延迟监控等与时间消耗相关的需求，这时候类似 ELK 这样的日志分析系统就显得有些发力了。对于这样的问题，我们就可以引入 Zipkin 来轻松解决。
    

*   Zipkin 的基础架构
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgKrew7ApcQQ89csYKI6Yr0nNAcWBqBLpVib2uyOEK482nP1Mbpz0ZKdQ/640?wx_fmt=png)

【解释】  

*   **Collector**（**收集器组件**）：主要处理从外部系统发送过来的跟踪信息，将这些信息转换为 Zipkin 内部处理的 Span 格式，以支持后续的存储、分析、展示等功能。
    
*   **Storage**（**存储组件**）：它主要处理收集器接收到的跟踪信息，默认会将这些信息存储在内存中。我们也可以修改此存储策略，通过使用其他存储组件将跟踪信息存储到数据库中。
    
*   **RESTful API**（**API 组件**）：主要用来提供外部访问接口。比如给客户端展示跟踪信息，或是外接系统访问以实现监控等。
    
*   **Web UI**（**UI 组件**）：基于 API 组件实现的上层应用。通过 UI 组件，用户可以方便而又直观地查询和分析跟踪信息。
    

3.2> 项目演示  

------------

*   官网下载 Zipkin 的 jar 包
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg6x66pdoauWDzRFPhCVL63ib0uLIuHHJicSpUlQfuFaTxOH5WSKeicCibKQ/640?wx_fmt=png)

*   启动 Zipkin，**java -jar zipkin-server-2.23.16-exec.jar**，访问 http://127.0.0.1:9411/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgMcl8GXyzM6dLb6drNLdElEKPVVlYhTiaNDcB2nviacVfIJAFhMswvCwA/640?wx_fmt=png)

*   分别为 simple-demo-1 和 simple-demo-2 引入 Zipkin 和加入 Zipkin 的配置
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgXNG854Hvnsbiad8Xj8jkicQ6XibbLfWul89pppP5IqyygkicudWnT6aO9w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKguibua9WCEAZYse5U9ibqeugjpIVSgOcCCC9WtY9JWIIxGaqPGVOlz7SQ/640?wx_fmt=png)  

*   启动 simple-demo-1 和 simple-demo-2，然后请求 http://localhost:9000/order
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgiaZribqURKHddfeZx69MDoLdiaWVslSLsCCIRUzGoN0XPV6ZUQ2jBbEtA/640?wx_fmt=png)

*   单击 “找到一个痕迹”，就可以查询出刚才在日志中出现的跟踪信息了，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgRFA0hZKKIxDvZs1Zjg2zeRA7BGHsbV2tJsp9tyKfLf4ibNkHmibNlUTg/640?wx_fmt=png)

*   单击 “SHOW” 按钮，还可以得到 Sleuth 跟踪到的详细信息，其中包括我们关注的请求时间耗时等，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgSDYttLiaA8oRb7tPWktgnHSazZooic05BQBpibbwwT06EbpjwTrdGISQw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKg5X6sHGnialYFtEywqniavfw7PsGf3lL9Fmch4icGgtG9uXV4IpYnXbaCw/640?wx_fmt=png)

*   点击依赖，我们可以看到本次日志的依赖关系，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgYOXR72GKx1F9Cuyiajcic9LzjPrTVJ3yCOkofOia7G2SbLdNWQBkWrEjQ/640?wx_fmt=png)

3.3> 接入 MySQL  

----------------

*   根据下载的 zipkin 版本，去 github 上知道对应的初始化 mysql 脚本。我们这里的 zipkin 是 2.23.16 版本。https://github.com/openzipkin/zipkin/tree/2.23.16/zipkin-storage/mysql-v1/src/main/resources
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgtSq2JichSGF0qDKbGPfNVUS8tK8hURMEhmhf8MBJ3oqK3IQtiaFvYn9Q/640?wx_fmt=png)

*   执行如下语句，启动 zipkin：
    
    **java -jar zipkin-server-2.23.16-exec.jar --STORAGE_TYPE=mysql --MYSQL_HOST=127.0.0.1 --MYSQL_TCP_PORT=3306 --MYSQL_DB=zipkin --MYSQL_USER=root --MYSQL_PASS=root**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgj36aV05ttq70j8gMgSAbZn5VbgoI4zYLzIXAdKNsiaTdl6wGdfnO2GQ/640?wx_fmt=png)

*   发送请求：http://localhost:9000/order
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgfCwWjTMj5M2QPUKaLUcWLCYXKFyGjTWxIeSUG5yOasia2JITw2OsZRA/640?wx_fmt=png)

*   查看 mysql 中信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgZ8ZDRzbXI85BoI4IBJYDHzEMdDhnCY2ibYAFN8JWxxI5qxiasswBIzdw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9QXgn7qs7xTQ35dEqsUKKgPkz3KpuC4ze6X7QGEOEBFMxPjSuUibNlS2Np2R7IATcwbEeJqvEqy8Q/640?wx_fmt=png)