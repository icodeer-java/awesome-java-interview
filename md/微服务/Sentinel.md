

    本篇文章介绍的是 SpringCloud Alibaba 技术栈中针对熔断限流的解决方案——Sentinel，本篇文章的大纲如下所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoRN4MD1qdDVcYMW8gBRpgHzEalT2Fv3PS2wptfn0Sia1x0a5aM2YGs7Q/640?wx_fmt=png)

一、概念介绍  

=========

*   当有**大量请求突然涌入**进来，远远超出系统可以处理的并发数。就会造成服务器宕机且服务不可用。为了保障服务仍然能够稳定运行，就需要系统具有**限流**、**熔断**、**降级**等能力。
    

1.1> 什么是服务雪崩  

---------------

*   一个服务失败，**导致整条链路的服务都失败**的情形，我们称之为服务雪崩。
    
*   服务熔断和服务降级就可以视为解决服务雪崩的手段。
    
*   我们可以把它理解为多条有交集的**高速公路路口**。当 ServiceD 这个路口拥堵配对的时候，也会影响到 ServiceG 和 ServiceF。随着时间推移。ServiceG 和 ServiceF 也开始发生拥堵了，那么会影响到 ServiceA 和 ServiceB。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoykWpYv1xiax15WwcLXbLw64G4NhPlHueQdDqZekHPrKfcLvRcC79fHA/640?wx_fmt=png)

1.2> 什么是限流  

-------------

*   限流的主要目的是通过**限制并发访问数**或者限制一个时间窗口内允许处理的**请求数量**来保护系统，一旦达到限制流量，则对当前请求进行处理采取对应的**拒绝策略**。如：跳转错误页面、进行排队、服务降级等。
    
*   比如：系统可以处理 1 万的并发，但是这一时刻并发数是 2 万，那么限流机制就会**保证 1 万的用户**是正常使用的。
    
*   限流的方式有哪些：
    
    1> 在 Nginx 层添加**限流模块**，限制平均访问速度。
    
    2> 通过设置**数据库连接池**或**线程池的大小**来限制总的并发数量。
    
    3> 通过 Guava 提供的 **Ratelimiter 限制接口**的访问速度。
    
    4> TCP 通信协议中的**流量整形**。
    

1.3> 什么是熔断（框架级别的熔断器，相当于保险丝）  

------------------------------

*   服务熔断是指当某个服务提供者无法正常为服务调用者提供服务时，比如请求超时、服务异常等，为了**防止整个系统出现雪崩效应**，**暂时将出现故障的接口隔离出来**，断绝与外部接口的联系，当触发熔断之后，后续一段时间内该服务调用者的请求都会直接失败，直到目标服务恢复正常。
    
*   熔断其实是一个**框架级别的处理**，这套熔断机制的设计，基本上采用的就是**断路器模式**。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPosWOWvIdFMud8BpgkBaa1mqLVuDUUibvXvcOibX6w5PZp07reGOlKosaQ/640?wx_fmt=png)

1.4> 什么是降级（业务级别的手工可配置开关，相当于 06 年笔记本玩魔兽争霸）  

--------------------------------------------

*   当下游的服务因为某种原因响应过慢，下游服务主动停掉一些不太重要的业务，释放出服务器资源，增加响应速度！
    
*   当下游的服务因为某种原因不可用，上游主动调用本地的一些降级逻辑，避免卡顿，迅速返回给用户！
    
*   服务降级大多是属于一种**业务级别的处理**。比如：开关降级。做法很简单，做个开关，然后将开关放配置中心。在配置中心更改开关，决定哪些服务进行降级。
    
*   自己梳理出核心业务流程和非核心业务流程。然后在非核心业务流程上加上开关，一旦发现系统扛不住，关掉开关，结束这些次要流程。
    

1.5> 服务熔断降级的几种常见方案  

---------------------

*   **平均响应时间**
    
    比如 1 秒内持续进入 5 个请求，对应时刻的**平均响应时间均超过阈值**，那么接下来在一个固定的时间窗口内，对这个方法的访问都会自动熔断。
    
*   **异常比例**
    
    当某个方法每秒调用所获得的**异常总数的比例超过设定的阈值**时，该资源会自动进入降级状态，也就是在接下来的一个固定时间窗口中，对这个方法的调用都会自动返回。
    
*   **异常数量**
    
    和异常比例类似，当某个方法在指定时间窗口内获得的**异常数量超过阈值**时，会触发熔断。
    

二、常用的限流算法  

============

2.1> 计数器算法
----------

*   在指定周期内累加访问次数，当访问次数达到设定的阈值时，触发限流策略，当进入下一个时间周期时进行访问次数的清零。
    
*   如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoMObiah6cQPSwZA5ZwjXzrSjfO1LJ3rD8aFnY0VWWuawg32GAibkZ37Hg/640?wx_fmt=png)

*   临界问题，如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoufUf6u1mriaiaf7bGqDExSB9P1CibZ12GGpL3VwLIkmFfnXyRcD7pKsng/640?wx_fmt=png)

2.2> 滑动窗口算法  

--------------

*   滑动窗口为固定窗口的改良版，解决了固定窗口在窗口切换时会受到两倍于阈值数量的请求，滑动窗口在固定窗口的基础上，**将一个窗口分为若干个等份的小窗口**，每个小窗口对应不同的时间点，拥有独立的计数器，当请求的时间点大于当前窗口的最大时间点时，则将窗口向前平移一个小窗口（将第一个小窗口的数据舍弃，第二个小窗口变成第一个小窗口，当前请求放在最后一个小窗口），整个窗口的所有请求数相加不能大于阈值。
    
*   **Sentinel** 就是采用滑动窗口算法来实现限流的。
    
*   如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoXib1uMBH5woszHXAVWXNQCxcNK7fOcvMuBbjxq1mSvXoU0a09vm0HmA/640?wx_fmt=png)【注】

*   1> 把 3 秒钟划分为 3 个小窗，每个小窗限制请求不能超过 50 秒。
    
*   2> 比如我们设置，3 秒内不能超过 150 个请求，那么这个窗口就可以容纳 3 个小窗，并且随着时间推移，往前滑动。每次请求过来后，都要统计滑动窗口内所有小窗的请求总量。
    

2.3> 令牌桶限流算法（控制令牌生成速度，取的速度不控制）  

---------------------------------

*   令牌桶是**网络流量整形**（Traffic Shaping）和**速率限制**（Rate Limiting）中最常使用的一种算法。
    
*   对于每一个请求，都需要从令牌桶中获得一个令牌；如果没有获得令牌，则需要触发限流策略。
    
*   系统会以**恒定速度**（r tokens/sec）往固定容量的令牌桶中**放入令牌**。
    
*   **令牌桶有固定的大小**，如果令牌桶被填满，则会**丢弃令牌**。
    
*   会存在三种情况：
    

*   **请求速度 > 令牌生成速度**
    
    当令牌被取空后，会被限流
    
*   **请求速度 == 令牌生成速度**
    
    流量处于平稳状态
    
*   **请求速度 < 令牌生成速度**
    
    请求可被正常处理，桶满则丢弃令牌
    

*   如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoLqCPPuSAiaOiaCgUvqetljIjibNV9GZib49XOdbCX5Qk5uicibqiaVcrtnIOA/640?wx_fmt=png)

2.4> 漏桶限流算法（控制水滴流出速度，不控制水滴产生速度）  

----------------------------------

*   主要的作用：
    
    1> 控制数据注入网络的速度。
    
    2> 平滑网络上的突发流量。
    
*   漏桶限流算法的核心就是：不管上面的水流速度有多块，漏桶**水滴的流出速度始终保持不变**。
    
*   **消息中间件**就采用的漏桶限流的思想。
    
*   如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPobNVFicMBpgbELsYQn5BHoMRCNDVpTicNtz3OZpPo9oHuwFsM6yg8CmFg/640?wx_fmt=png)

三、Sentinel 概述和基本使用  

=====================

3.1> Sentinel 概述
----------------

*   https://github.com/alibaba/Sentinel/wiki / 介绍
    
*   它是面向分布式服务架构的轻量级流量控制组件，主要以**流量**为切入点，从**限流**、**流量整形**、**服务降级**、**系统负载保护**等多个唯度来帮助我们保障微服务的稳定性。
    
*   Sentinel 的特性有如下：
    
    适用场景丰富。
    
    提供实时监控的功能包括查看单机秒级数据，设置 500 台以下规模的集群汇总运行情况。
    
    只需引入 Maven 依赖并进行简单的配置，即可快速与 Spring Cloud、Dubbo、gRPC 等进行整合。
    
*   组成部分
    

*   **Java 核心库**
    
    不依赖任何框架 / 库，能够运行于所有 Java 运行时环境，同时对 Dubbo、SpringCloud 等框架也有较好的支持。
    
*   **Dashboard 控制台**
    
    基于 SpringBoot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。
    

*   主要特征
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPorCOQvQSgxicqxpfd5HspBg2tEAiaMpYRsu0TOa7LgmicCtUZVpuWcXkqw/640?wx_fmt=png)

*   开源生态
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoNwwuzF8VXloallDXNQGYfAib2rkSUhkSCNP4CicJ1cmxbISWftnteAiaA/640?wx_fmt=png)

3.2> 部署 Sentinel Dashboard  

-----------------------------

*   下载 Sentinel 源码或者直接下载 release 版本 jar 包
    
    源码：https://github.com/alibaba/Sentinel
    
    jar 包：https://github.com/alibaba/Sentinel/releases
    
*   命令启动 Sentinel Dashboard
    
    java **-Dserver.port**=8000 **-Dcsp.sentinel.dashboard.server**=localhost:8000 **-Dproject.name**=sentinel-dashboard **-jar** sentinel-dashboard-1.7.1.jar
    

**【命令参数解释】**

*   **-Dserver.port**
    
    指定 Sentienl 控制台的访问端口，默认是 8080。
    
*   **-Dcsp.sentinel.dashboard.server**
    
    指定 Sentinel Dashboard 控制台的 IP 地址和端口，这里进行设置的目的是把自己的限流数据暴露到控制平台。
    
*   **-Dproject.name**
    
    设置项目名称。
    
*   **-jar**
    
    指定启动的 jar 包
    

*   启动日志如下
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPog6kBzbmPZrEVeZBMyhK3byxatnUGDvhOJFlia3EjtGYpMvmCBvXhe9Q/640?wx_fmt=png)

*   访问 http://localhost:8000，从 Sentinel 1.6.0 开始，加入了登录功能。默认用户名 & 密码都是 **sentinel**。控制台界面如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoaObm9rb6qF0Mo0Wcafice6EJIzZMe7Xs7LYKTG0ekkxGpI9psiauJzBA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPonE0LGGYVNL6hw4Z2zoJx3fVgSnQCTWw9gXB1Oe3icUboSgN4SfsS42Q/640?wx_fmt=png)  

3.3> 实战 1：编码方式接入 Sentinel  

----------------------------

### 3.3.0> 准备工作

*   接入流程
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoHOcGpJ91J9byWodhhFLxstYHAl3B0bDC9U0f50gIV6qolGxzq7Mmjg/640?wx_fmt=png)

*   创建一个项目，名为 sentinel-simple-demo
    
*   项目结构
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoWzjQLTQL7jbsnCjDZFQuGfarPGMlJXia3kIVSTKWo90PsxKaqaYDxMA/640?wx_fmt=png)

### 3.3.1> **引入** Sentinel **依赖**  

*   【注意】从 Sentinel 1.5.0 开始仅支持 JDK1.7 或者以上版本。Sentinel 1.5.0 之前的版本最低支持 JDK1.6。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPojecaNia1PAUc7iaa1gY7at0ibjib79C3GeKywFKKoibJic5oEiaDibAG4DueuA/640?wx_fmt=png)

### 3.3.2> **定义日常业务代码**  

*   **BusinessService.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPop8zl7U7gXzabVp21rlWVdXuqaia8ecibwquD7Rc4jCmHibD5bEoOx4a6g/640?wx_fmt=png)

### 3.3.3> **定义限流规则和熔断规则**  

*   **RulesUtils.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoLstxpORKu4zQxXeg4h2mZxQdlUVAjx7NZfoPDYibN6ZOodqic9LDquXA/640?wx_fmt=png)

### 3.3.4> **定义基于 SphU 方式的限流 & 熔断测试类**

*   接下来，我们把需要控制流量的代码用 Sentinel API `SphU.entry("process")` 和 `entry.exit()` 包围起来即可（SphO 也是一样的）。当然，也支持注解形式——`@SentinelResource`，注解方式后续我们会介绍到。
    
*   资源名称（例如：“process”）可以定义**方法名称**、**接口名称**或者**其他的唯一标识**。
    
*   当使用`SphU.entry`时，如果资源被限流 / 熔断后，会抛出一个 BlockException，然后在捕获异常后进行限流的逻辑处理。
    

*   例子：SentinelSphUSimpleDemo.java
    

*   当使用`SphO.entry`时，如果资源被限流 / 熔断后，会返回 false。然后在 else 判断后进行限流的逻辑处理。
    

*   例子：SentinelSphOSimpleDemo.java
    

*   需要注意，当使用`SphO.entry`时，需要资源使用完之后调用`SphO.exit()`，否则会导致调用链记录抛出 ErrorEntryFreeException 异常。
    

*   **通过 SphU 实现限流规则**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoicBibZqTdQrLMurh4FEV8lr67yhWQuNTS2oOUsMuTpTOwDKPshbQoShg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoH8hTGmklBeYe1J5dwF3ZuSeGsBBz23Bdice2oiaWAPxB2opSW7x8tJYg/640?wx_fmt=png)

*   运行之后，我们可以在日志 `**~/logs/csp/${appName}-metrics.log.xxx**` 里看到下面的输出:  
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPowoZlmudoU4mrOalslbv6oWLuHeKZ7lVcLPU1R8ZokeR79YonPpqFgw/640?wx_fmt=png)【注释】

日志格式为：MetricNode.java

*   **timestamp**——限流统计的时间戳
    
*   **yyyy-MM-dd HH:mm:ss**——限流统计的日期时间
    
*   **resource**——资源名称
    
*   **passQps**——代表通过的请求；
    
*   **blockQps**——代表被阻止的请求；
    
*   **successQps**——代表成功执行完成的请求个数；
    
*   **exceptionQps**——代表用户自定义的异常；
    
*   **rt**——代表平均响应时长；
    
*   **occupiedPassQps**——代表优先通过的请求；
    
*   **concurrency**——代表并发量；
    
*   **classification**——代表资源类型；
    

*   **通过 SphU 实现熔断规则**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoRqAZ6b0nubNC9NhGccps48Gl7GhcdpOUxOR1x3aXXibIGtur5h6CmxA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoXVMGVyj885W6gLunzGbrhg6fkYAHqs3BiactL7Jz7KpE1IjaE4iaTqiaw/640?wx_fmt=png)  

### 3.3.5> **定义基于 SphO 方式的限流 & 熔断测试类**

*   **通过 SphO 实现限流规则**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoOakgjNrnHpLsk8rHlmONaHTRiaKVUNl25Acdm9Hkz9p0ND2VPOickia3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPomWyazgc2xIWL5gVtUfIuGn6T9sMb24dUGEckED1ldpTabxcS8eCicfA/640?wx_fmt=png)

*   运行之后，我们可以在日志 `**~/logs/csp/${appName}-metrics.log.xxx**` 里看到下面的输出:  
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoBOvdK0bLoeNUGU8QsHOLPbBxS25BYAAQMuHo32xmdPMVCibqU8KZp1g/640?wx_fmt=png)

*   **通过 SphO 实现熔断规则**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo50HKzBdLcvFRmfx9sFVBSvAXGNEiaibXjwdtBYBZqSPnPNmwtIlasrCw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoZh1LSpB4iayvf5AEdFIzP3z4nj5VUBNKyt6ZWYP7Mxvv6stIGXvFHzw/640?wx_fmt=png)  

3.4> 实战 2：注解方式接入 Sentinel  

----------------------------

### 3.4.1> 前提介绍

*   在第三步，定义资源时，也可以使用 **@SentinelResource** 注解的方式，方便的进行资源管理。项目目录如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPouDYjb3l5D1HPCfSI0rwk9AgY7h2CUqpfFCrjxCDQaPlmr3qvZUom0w/640?wx_fmt=png)

### 3.4.2> 引入 sentinel-annotation-aspectj 和 SpringMVC 依赖  

*   **sentinel-simple-demo 的 pom**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoL7JHnXdjAT5fRJ5j27xDYJKQOD21SVbIvOgnSB4icQEndguf994Z1hw/640?wx_fmt=png)

### 3.4.3> 编写 Configuration 配置类，创建 SentinelResourceAspect 的 Bean 实例  

*   **SentinelAspectConfiguration.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoBjXicDTHJ6J6ecbYYEOy4GwKvQib08SqPdN58FLQ22ibiaZxl7OhBZXGLQ/640?wx_fmt=png)

### 3.4.4> 创建服务类  

*   SentinelWithAnnotationService.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo5ibZcOHAOE96HXLUGciajghSqBWeS87vGzHWSrr0ibaORwvIb0HAH14TQ/640?wx_fmt=png)

*   SentinelWithAnnotationServiceImpl.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPotqWLMmSaBIXFia1fcqrFZXHqq8KePq5KEpeslsEftZpFAcwWlgf6J8A/640?wx_fmt=png)

### 3.4.5> 创建外部异常类  

*   **ExceptionUtil.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoHibKcwOGS9njCnQlzzuGOJkM96DzYdQOqEiajjKlOK7UoXcPgsB2vpWA/640?wx_fmt=png)

### 3.4.6> 创建 Controller 测试类  

*   **SentinelSimpleDemoController.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPojw3va1naYkYNxWOeY7pQIU7LJyLhKJkAu7Xkm3jG8EqT37uZicZTWUw/640?wx_fmt=png)

### 3.4.7> @SentinelResource 注解包含属性介绍  

*   注意：注解方式埋点**不支持 private 方法**。
    
*   @SentinelResource 用于**定义资源**，并提供可选的 **blockHandler** 异常处理和 **fallback** 配置项。
    
*   注解包含以下属性：
    

*   **value**
    
    资源名称，必需项（不能为空）
    
*   **entryType**
    
    entry 类型，可选项（默认为 EntryType.OUT)
    
*   **blockHandler** / **blockHandlerClass**
    
    blockHandler 对应处理 BlockException 的**函数名称**。
    
    blockHandler 函数访问范围需要是 **public**，**返回类型需要与原方法相匹配**，**参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 BlockException**。
    
    blockHandler 函数默认需要和原方法在同一个类中。若希望**使用其他类的函数**，则可以**指定** **blockHandlerClass** 为对应的类的 Class 对象，注意对应的**函数必需为** **static** **函数**，否则无法解析。
    
*   **fallback** / **fallbackClass**
    
    fallback 函数名称，用于在**抛出异常**的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除 exceptionsToIgnore 里面排除掉的异常类型）进行处理。
    
*   fallback 函数签名和位置要求：
    
    返回值类型必须与原函数返回值类型一致；
    
    方法参数列表需要和原函数一致，或者可以额外多一个 **Throwable** 类型的参数用于接收对应的异常。
    
    **fallback 函数默认需要和原方法在同一个类中**。若希望**使用其他类的函数**，则可以**指定 fallbackClass** 为对应的类的 Class 对象，注意对应的**函数必需为 static 函数**，否则无法解析。
    
*   **defaultFallback**（since 1.6.0）：（可选项）
    
    默认的 fallback 函数名称，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。**若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。**
    
*   defaultFallback 函数签名要求：
    
    返回值类型必须与原函数返回值类型一致；
    
    方法**参数列表需要为空**，或者**可以额外多一个 Throwable 类型的参数**用于接收对应的异常。
    
    **defaultFallback 函数默认需要和原方法在同一个类中**。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的**函数必需为 static 函数**，否则无法解析。
    
*   **exceptionsToIgnore**（since 1.6.0）
    
    用于**指定哪些异常被排除掉**，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。
    

3.5> 限流规则详解  

--------------

### 3.5.1> FlowRule 参数详解

*   RulesUtils.initFlowRules()
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoZIcp8aGS8uf8UbcEwibGKZdJtO836ObyLOJLyDJTkiaDicVEf0zT4jmTQ/640?wx_fmt=png)

【参数方法补充说明】  

*   **rule.setGrade(...) —— 限流阈值类型**
    

*   **并发线程数——RuleConstant.FLOW_GRADE_THREAD=0**
    
    并发线程数限流用来保护业务线程不被耗尽。如果超出阈值，新的请求就会被拒绝。比如：A 服务调用 B 服务，而 B 服务发生**不稳定**或**响应延迟**，造成 A 服务吞吐下降，由于 A 服务中线程阻塞后未释放，造成占用很多线程，最终导致线程池耗尽。
    
*   **QPS——RuleConstant.FLOW_GRADE_QPS=1**
    
    Queries Per Second 即：每秒查询数。也就是每台服务器每秒钟可以响应的查询次数。
    

*   **rule.setStrategy(...) —— 调用关系限流策略**
    

*   **调用方限流——RuleConstant.STRATEGY_DIRECT**
    
*   **根据调用链路入口限流——****RuleConstant.STRATEGY_CHAIN**
    
*   **关联流量控制——RuleConstant.STRATEGY_RELATE**
    

*   **rule.setControlBehavior(...) —— QPS 流量流控行为**
    

*   **直接拒绝**
    
    默认流量控制方式，超出阈值时，直接抛出 **FlowException** 异常。
    
*   **Warm_Up**
    
    是一种冷启动（预热）方式。当流量突然增大时，可能会瞬间把系统压垮。那么我们希望这种请求是**逐步递增**的，并且在一个预期时间之后，达到允许处理请求的最大值。如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoSRl5QichJQRvPLkcvsZLYujPlspAjbGyM13uaa7nC3ZkU6R9v6BlZeg/640?wx_fmt=png)

*   **匀速排队**
    
    它会使请求以匀速的方式通过。**类似于漏桶限流算法**的那种方式。如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoXPAgiat37DphDkGns2YjWicbgibx9ggibNS3JvBrZyL4JqwyRSIxLVlPKQ/640?wx_fmt=png)

### 3.5.2> DegradeRule 参数详解  

*   RulesUtils.initFlowDegradeRules()
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoic3NIPxCibPaLbqgVlHxvqqoHxxbJmPvwo5jPzCUHUFwIkpFlN1TUG0A/640?wx_fmt=png)

【参数方法补充说明】  

*   **rule.setGrade(...) —— 熔断策略**
    

*   **平均响应时间——RuleConstant.DEGRADE_GRADE_RT=0**
    
    如果每秒资源数 >=minRequestAmount（默认值 5），对应的平均响应时间都超过了阈值（count，单位为毫秒），那么在接下来的时间窗口（timeWindow，单位为秒）内，对这个方法的调用都会自动熔断，抛出 **DegradeException**。Sentinel 默认统计的 **RT 上限是 4900ms**，如果超出此阈值都会算作 4900ms，如果需要修改，则通过启动参数 **-Dcsp.sentinel.statistic.max.rt=xxx** 来配置。
    

*   **异常比例——RuleConstant.DEGRADE_GRADE_EXCEPTION_RATIO=1**
    
    如果每秒资源数 >=minRequestAmount（默认值 5），并且**每秒的【异常总数 / 总通过量 】超过阈值 count**，取值范围为 [0.0, 1.0]，代表 0%~100%，则资源将进入降级状态。同样，在接下来的 timeWindow 之内，对这个方法的调用都会自动触发熔断。
    

*   **异常数——RuleConstant.DEGRADE_GRADE_EXCEPTION_COUNT=2**
    
    当资源最近一分钟的异常数目超过阈值 count 之后，就会触发熔断。需要注意的是，如果 timeWindow 小于 60s，则结束熔断状态后仍然可能进入熔断状态。
    

四、Sentinel 的集成  

=================

4.1> 实战 3：集成 Spring Cloud Sentinel
----------------------------------

### 4.1.0> 准备工作

*   创建一个项目，名为 **sentinel-nacos-demo**
    
*   项目目录如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoW71QV7k8P6gL3OEhy6Ziaibl9QTIicXqicb8unNSZ2HUmFAEcUu55CIsPg/640?wx_fmt=png)

### 4.1.1> 只采用编码方式  

*   引入 Spring Cloud Sentinel 和 SpringMCV 的依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoXIG17rW4KlyWI98ur8NicKXicu1obwPN7YxCFa6BDrCXMDPEkcDQ9icbQ/640?wx_fmt=png)

*   创建一个 REST 接口，并通过 @SentinelResource 配置限流保护资源。**SentinelNacosDemoController.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo9syKWtWd2VMopFShQYyolPgyPNUcic218pSQSQrX5YPt05Ed2lvTdew/640?wx_fmt=png)

*   手动配置流控规则，**实现 InitFunc 接口**，完成流控配置。**FlowRuleStrategy.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo5AicHR7yKTVPReeC8ePSdROAV9UNDMic6X8lwZOc85dvLc62N8AF9zjA/640?wx_fmt=png)

*   配置自动加载文件。
    
    **resources/META-INF/services/com.alibaba.csp.sentinel.init.InitFunc**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoRRibMO9anodu4L6xP4HAq0XK9r1hzB6VBBIv2ic9wuicRiboussug1HaDw/640?wx_fmt=png)

*   启动服务，访问：http://localhost:8001/foo1
    

*   1 秒内的第 1 次访问：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoUfiaBt6OSgtNl6IEPDdlUVL4LJvAicWLFuahJ7OYYSqzj1QJ2wibveu9Q/640?wx_fmt=png)

*   1 秒内的第 N 次访问：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoVM9UQXu5MS4wAp1sJ736KPnfTdwkdQyt2QJ2PoiaP2HuqI0McngWCcQ/640?wx_fmt=png)

### 4.1.2> Sentinel Dashboard 方式  

*   启动 Sentinel Dashboard
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo9VGPETzT2LuYoNLfCIU32TZuNn1S2eUpleI8y4R4VPF7H4DeicPVibhw/640?wx_fmt=png)

*   在 **application.xml** 中添加如下配置
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPogRiazXicicuafWmmMaGnbS8icAau0ANlfym9FTeVOY5fzNFBa1QXZBNzag/640?wx_fmt=png)

*   创建一个普通的 REST 接口。**SentinelNacosDemoController.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo02CklASSmUxAUaick0r5f3Pvltib6vrWCaN6eCKvVdGuqzzagicMVKmbQ/640?wx_fmt=png)

*   启动服务后，访问 **http://localhost:8001/dashboard**，这样 Sentinel Dashboard 才会采集到该条 url。此时由于没有配置限流策略，所以无论访问多少次，都会输出 “-----dashboard-----”
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPotwzjGOG3v6VWrC1NsZm55T6lwMDFCico4bg3e5FhcxKKo7xa95PJL3Q/640?wx_fmt=png)

*   访问 http://localhost:8000，登录 Sentinel Dashboard。点击 “簇点链路”，设置流控。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPos75pbnsYhO8LzA0FyD3soBibY1QvmLlxvBYmRuTf9rkIu2zZRZgsHLg/640?wx_fmt=png)

*   配置流控规则
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo5Isw4z9yov4swUm2aBPco8Uf6I08jBgY5Ar6rKoQicZZuIYpr0gpM7A/640?wx_fmt=png)

*   配置完流控规则后，多次请求会出现下面限流提示。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoy89D6pGIOrHib2OoA35J8UQDEEGSYEOBWgicrSoOCic169uibmVNAF9uMA/640?wx_fmt=png)

### 4.1.3> 自定义 URL 限流异常  

*   首先：通过实现 **UrlBlockHandler** 接口，来**重写 blocked 方法**。该实现**只能一个实现类**。否则会报错：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPos56mfktgm5u90Jm8A71nvqiabaiaGxPBY6n3pu4tsiaY3icRaAYmqUprCQ/640?wx_fmt=png)

*   **MuseBlockHandler.java。**也可以负责一份 MuseBlockHandler1.java，验证上面的异常。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoKzvPp6Y8JuPXWtZGV0G4roe3JXMhJmU9HGoy5bjz6DxaPYTibByV4yA/640?wx_fmt=png)

*   配置流控规
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo5q1gvKOkTq8cle4lHZHTqF72NzXvUapRv7BBemwphfhX8neHUnwsmg/640?wx_fmt=png)

*   请求 http://localhost:8080/dashboard
    

*   1 秒内的第 1 次请求
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoRljhhibmKKls7NZaLCS8iabbcPz4nGbEE6PTOFHFBPH0LoDD32LjhYNw/640?wx_fmt=png)

*   1 秒内的第 N 次请求
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo8EXQXiaibicO8b5KhVoP4jo5RUXFcRLpfMnpFvM8YVCMM3AlzfkON6IUw/640?wx_fmt=png)

### 4.1.4> URL 资源清洗  

*   在默认情况下 Sentinel 会**把所有的 URL 当作资源来进行控制**。比如：/urlCleaner/1 和 / urlCleaner/2 被当作两个资源。如果需要聚合资源，则需要进行 url 资源清洗。
    
*   首先，新 REST 接口 **SentinelNacosDemoController.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoHHp6ymsrRibMtv9XC6rSRJDCNpl1aJSz4eaHWib1rsbWA34C1UT3wh6w/640?wx_fmt=png)

*   其次，实现 **MuseUrlCleaner.java** 接口，重写 clean 方法。**将 / urlCleaner/1 和 / urlCleaner/2 都作为资源：/urlCleaner/***
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPovzPY65icXFespajXOrJwYkdZ6kzichjgZXVXicho5U5HibloGEicYBocn3w/640?wx_fmt=png)

*   执行 http://localhost:8001/urlCleaner/1
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo342DXv57s2OUZSWvFuQVsKKkZATdp73KltVpUPXicb5uJFCq2iaDiaXnA/640?wx_fmt=png)

*   打开 Sentinel Dashboard。查看簇点链路，并设置流控。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPokRQU9Etxyd4h6iaaUvSgYP9Dy3RGick9O3Oic9OJ4FxFCrianWPJReQ3FA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoYt7d9a0bXA82IBKAOb7akCHBicviaN9l4vuicCdVpAOOFhdCULOh33jUQ/640?wx_fmt=png)  

*   执行 http://localhost:8001/urlCleaner/1 之后再执行执行 http://localhost:8001/urlCleaner/2 ，那么会展示出限流警告
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoMFtWzRQC5tmEPibmFppZNdKWvToZEj9JGUGGicAkicVVrS3ONypW5pViaw/640?wx_fmt=png)

4.2>  实战 4：集成 Nacos  

----------------------

### 4.2.1> Nacos 的安装

*   由于 Sentinel Dashboard 所配置的流控规则，都**保存在内存中**，一旦应用重启，这些规则都会被清除。为了解决这个问题，Sentinel 提供了动态数据源支持，目前支持 Consul、ZooKeeper、Redis、Nacos、Apollo、etcd 等数据源扩展。下面以 Nacos 为例：
    
*   Nacos 官网
    
    https://nacos.io/zh-cn/docs/quick-start.html
    
*   下载安装包
    
    https://github.com/alibaba/nacos/releases
    
*   解压安装包
    
    unzip nacos-server-$version.zip
    
    或者
    
    tar -xvf nacos-server-$version.tar.gzcd nacos/bin
    
*   启动 nacos
    
    sh startup.sh -m standalone
    
*   如果您使用的是 ubuntu 系统，或者运行脚本报错提示 [[符号找不到，可尝试如下运行：
    
    bash startup.sh -m standalone
    
*   打开控制台
    
    http://localhost:8848/nacos
    
    用户名 & 密码为：**nacos** & **nacos**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoKHq0VyGkzL9Jvu29ILHIVzDYynI1Z5icGGTIqGoOeKZ0Er0sJ7roZpQ/640?wx_fmt=png)

*   流程图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoe3h7jtYhpGPBnicm3MPRoF76cKUkOOpU2ciasian9F4Yo0jcwZP86gQhQ/640?wx_fmt=png)

### 4.2.2> Sentinel 集成 Nacos  

*   首先：添加 Nacos 数据源的依赖包
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPodKzJqdkviaWKunsSmJrIPP1Z0Td6n1SYyBH6HVIJdbfYD9Ctic9HUX4g/640?wx_fmt=png)

*   其次：在 **application.yml** 中添加数据源配置
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo9HGrrA4hibSsx0xMdBKUMgiaklico10G3uC9XmBqJ73zObqzf8cm4Y8Rw/640?wx_fmt=png)

*   第三：在 **SentinelNacosDemoController.java** 类中添加新的接口 / withnacos 请求，**并重启服务**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo6OIEFJrkMz9V0xv3AicRU8IVzRhqEBIyw3KqdDyYfUtBqg3oZf6TfmA/640?wx_fmt=png)

*   第四：在 nacos 中创建 data-id 为 **sentinel-nacos-demo-sentinel-flow** 的限流规则，**此处的 data-id 与 application.yml 中配置的 data-id 一致**。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoGic12xWWHUzDSEmI5WD5egF1I1s9GUOXysZJNzctr7UnI1XE0X5TILg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoLDm9NHTQC44yLv05XIV2VrI2MkFgXiazvbIicCGK2vXic7SVXbic4rsyYg/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoFDCcG9eGImib2KgYrdXWEVbEF3YwbuX43pQ3wYJjLSxiawGtcLBXWmPw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoFDCcG9eGImib2KgYrdXWEVbEF3YwbuX43pQ3wYJjLSxiawGtcLBXWmPw/640?wx_fmt=png)  

*   第五：在 Sentinel Dashboard 中，发现在【流控规则】中，已经加载了 nacos 中配置的规则。（只要 Nacos 中配置发布了，Sentinel 就可以在流控规则中看到这条配置，如果看不到，可以尝试在 Nacos 中修改这个配置点击发布，再修改回来，再次发布
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoFpaH4IWzMR3uuEcuyrvIZPC77hxjdfMPKA0oKUsokU4hVfUKltYj5A/640?wx_fmt=png)

*   请求 http://localhost:8001/withnacos
    

*   1 秒内第 1 次请求
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoHSSxicgjSNCI49bDhPpvRbwyUwwo4woFQ8N040iaylj2wONXPZMpy9ZQ/640?wx_fmt=png)

*   1 秒内第 N 次请求
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPonqQlYcK5ibp4Hph0bd17iaAtopicpqPetadrVEhZ8SJj38CPxOwGNpSWw/640?wx_fmt=png)

五、工作原理  

=========

5.1> Sentinel 的工作原理
-------------------

*   Sentinel 的核心分为三部分：**工作流程**、**数据结构**和**限流算法**。
    

*   整体架构图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoTINudDk3z6OaK8J8fOLUcibOvz7BImmvTp4h8mWBcEnL9micIxMxqickA/640?wx_fmt=png)

*   调用链——Slot Chain
    
    调用链是 Sentinel 的工作主流程，由各个 Slot 插槽组成，将不同的 Slot 按照顺序串在一起（责任链模式），从而将不同的功能（限流、降级、系统保护）组合在一起。Sentinel 中各个 Slot 承担了不同的职责，每个模块更聚焦于实现某个功能。在 Sentinel 中，所有的资源都对应一个**资源名称**（resourceName），每次访问该资源都会创建一个 Entry 对象，并会同时创建一系列**功能槽**（Slot Chain） ，这些槽会组成一个**责任链**，每个槽负责不同的职责。
    
*   功能槽——Slot    
    

*   **NodeSelectorSlot**
    
    负责**收集资源的调用路径**，以**树状结构**存储调用栈，用于根据调用路径来限流降级。
    
*   **ClusterBuilderSlot**
    
    负责创建以**资源名**维度统计的 ClusterNode，以及创建每个 ClusterNode 下按调用来源 origin 划分的 StatisticNode。
    

*   **StatisticSlot**
    
    统计不同维度的请求数、通过数、限流数、线程数等 runtime 信息，这些数据存储在 **DefaultNode**、**OriginNode** 和 **ClusterNode** 中。
    

*   **ParamFlowSLot**
    
    控制热点参数限流。
    
*   **SystemSlot**
    
    控制总的入口流量，限制条件一次是总 QPS、总线程数、RT 阈值、操作系统当前 load1、操作系统当前 CPU 利用率。
    
*   **AuthoritySlot**
    
    权限控制，支持黑名单和白名单两种策略。
    
*   **FlowSlot**
    
    根据限流规则和各个 Node 中的统计数据进行限流判断。
    
*   **DegradeSlot**
    
    根据熔断规则和各个 Node 中的统计数据进行服务降级。
    
*   **LogSlot**
    
    在出现限流、熔断、系统保护时负责记录日志。
    

5.2> Spring Clound Sentinel 的工作原理  

------------------------------------

*   查看自动配置文件 **META-INF/spring.factories** 文件。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoQn4BNUlwroTAZ8xC096AOWxjbYgZRlx5YveXlm1Nvm8SMmGVTr4kicA/640?wx_fmt=png)

*   这里 EnableAutoConfiguration 自动装配了 5 个配置类
    

*   **SentinelWebAutoConfiguration** 是对 Web Servlet 环境的支持。
    
*   **SentinelWebFluxAutoConfiguration** 是对 Spring WebFlux 的支持。
    
*   **SentinelEndpointAutoConfiguration** 暴露 Endpoint 信息。
    
*   **SentinelFeignAutoConfiguration** 用于适配 Feign 组件。
    
*   **SentinelAutoConfiguration** 支持对 RestTemplate 的服务调用使用 Sentinel 进行保护。
    

*   **SentinelWebAutoConfiguration**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoib7Gic1aia653IdgdVHzDoZk3ZODUWibOkEpp5J3pInrR3SJFPcUQ6ZHrQ/640?wx_fmt=png)

*   **CommonFilter**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPo3Dpyubfg4KHDbKAq0hJScKz8ibq79ic9bBvia0TyicuGicm8icwyJqKGGR0w/640?wx_fmt=png)因此，对于 Web Servlet 环境，只是通过 Filter 的方式将所有请求自动设置为 Sentinel 的资源，从而达到限流的目的。

六、Sentinel 核心源码分析  

====================

6.1> Sphu.entry 源码分析
--------------------

*   整体流程图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoZ0GOtRSLtxJIuaRBmEfPFMpYXZicf5BSicYBy6lgZmSCYiaYzBu0ibZY3A/640?wx_fmt=png)

*   **CtSph.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoZaRXSCBJaoibib1DNeSYn9nkXQMg4RJaNg5rySZt7kphH2TCicWdNSLQA/640?wx_fmt=png)

*   限流——FlowSlot & StatisticSlot
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPojMmTjdBD7Z8IQliaffXxlMjJYL06EsGQUQibyibelmicQcvyd2QS2eoXBg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPolmXhTvuqV2ibC5oPODF7G6HOdzXUOgLGXb9C8FJ57icLdX9XReqsEgPw/640?wx_fmt=png)  

*   FlowSlot.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPodeJyJF45wuOXuJbEbVskmlib4zOG5JNTT4uVRPVKgOHSrlpmZaPQWpQ/640?wx_fmt=png)

*   FlowRuleManager.getFlowRuleMap();
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPotrwQczRYazszvicviavJxefLy3jz8TYWVWNz23VUiauia6GAiaiaEAG7NdbQ/640?wx_fmt=png)

*   FlowRuleChecker.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoqmvF8OczHOMfprbln2GRSrByIWdvP5lxHzcEWFWOAAI0O31dmsEmTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoBZekLzNkr2FgGVGbAUichCYKiarMVWibWbHMqye0OrhQkYclibgk1hz9Tw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoyWSTHiaAEHrfWPGk1EKcVYWnbJbhUBVn4OFurMNZNNGswacRR0TdAUg/640?wx_fmt=png)

*   上面场景的应用
    
    假设我们对接口 XXXService 配置限流 1000QPS，这 3 种场景分别如下：
    

*   第 1 种，目的是优先保障重要来源流量
    
    我们需要区分调用来源，将限流规则细化。对 A 应用配置 500QPS，对 B 应用配置 200QPS，此时会产生两条规则：A 应用请求的流量限制在 500，B 应用请求的流量限制在 200。
    
*   第 2 种，没有特别重要来源的流量
    
    我们不想区分调用来源，所有入口调用 XXXService 共享一个规则，所有 client 加起来总流量只能通过 1000QPS。
    
*   第 3 种，配合第一种场景使用。
    
    在长尾应用多的情况下，不想对每个应用进行设置，没有具体设置的应用都将命中。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibiak9K7XY9Rq7AWsnV5OSPoatleazH1kvibribGErl1Orx4nL4jYOSeB6Jslq1zEYiaTwjoa4sic5G60A/640?wx_fmt=png)

*   最后，在 passLocalCheck 方法中，通过 rule.getRater() 获得流控行为，实现不同的处理策略，如下所示：
    

*   DefaultController：直接拒绝。
    
*   RateLimiterController：均匀排队
    
*   WarmUpController：冷启动（预热）
    
*   WarmUpRateLimiterController：匀速 + 冷启动