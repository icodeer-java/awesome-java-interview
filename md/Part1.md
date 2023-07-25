> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486233&idx=1&sn=fda7d410bedf6b56457757225a4d6bfa&chksm=e91149e4de66c0f2acd4a95cba907f43dbbd0700c8f70ccd9fc0cc7f57048683732cf79fc77b&scene=178&cur_album_id=2208187391738249217#rd)

    本篇是 SpringBoot 系列文章的第一篇，后面陆续会更新。本篇主要是介绍 SpringBoot 的相关特性，知识点比较基础入门。浏览起来应该会比较畅快淋漓。下面是本文的大纲：  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5Lt5Xic0CGenadaicfOhKJGibNUlmm8ibHUy50fKdGg47QfMQJY0lQ9W0kQ/640?wx_fmt=png)

一、SpringBoot 概述  

==================

1.1>Spring 能做什么？
----------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5WpSiaXFpgSKDNldsULL07udtP5VJZmLCMC2kOBoMMLTjJQdiaSo2xA0g/640?wx_fmt=png)

### 1.1.1> 微服务 Microservices  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5TMdHScqXjQQbqhograWkerosYLVWogzO0sO8j5N4z6czzMmSQt297A/640?wx_fmt=png)

*   简介：
    
    微服务架构是 “新常态”。 构建小型的、自包含的、随时可以运行的应用程序可以为您的代码带来极大的灵活性和弹性。 Spring Boot 的许多专门构建的功能使在生产中大规模构建和运行微服务变得容易。 别忘了，如果没有 Spring Cloud（简化管理并提高容错能力），任何微服务架构都是不完整的。
    
*   什么是微服务？
    
    微服务是一种现代软件方法，其中应用程序代码以小的、可管理的、独立于其他部分的方式交付。
    
*   为什么要构建微服务？
    
    它们的小规模和相对隔离可以带来许多额外的好处，例如：更容易维护、提高生产力、更大的容错能力、更好的业务一致性等等。
    
*   服务的演变：
    
    单体——> 系统的横纵拆分——>SOA——> 微服务
    

### 1.1.2> 响应式编程 Reactive

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5SvicWRTPqyH81DV9BckPaLGPaL3mxx7yxLpu5ibib5c0Nnl0w2tgRSSMA/640?wx_fmt=png)

*   简介：
    
    响应式系统具有某些特性，使其成为低延迟、高吞吐量工作负载的理想选择。 Project Reactor 和 Spring 产品组合协同工作，使开发人员能够构建具有响应性、弹性、伸缩性和消息驱动的企业级反应式系统。
    
*   什么是响应式处理？
    
    响应式处理是一种范例，它使开发人员能够构建可以处理背压（流控制）的非阻塞、异步应用程序。
    
*   为什么要使用响应式处理？
    
    响应式系统更好地利用现代处理器。 此外，在响应式编程中包含背压可确保解耦组件之间具有更好的弹性。
    
*   什么是背压？
    
    Backpressure 其实是一种现象：在数据流从上游生产者向下游消费者传输的过程中，上游生产速度大于下游消费速度，导致下游的 Buffer 溢出，这种现象就叫做 Backpressure 出现。
    
*   基于异步非阻塞方式，可以通过构建异步数据流。这个数据流可以通过占用少量的服务器资源，来构建一个高可用的应用。
    

### 1.1.3> 云开发 Spring Cloud

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5fSNARknyow5ysNTKRja435MJWZppibFB2mVWrEvOat7J6jlxILkj6Lw/640?wx_fmt=png)

*   简介
    
    开发分布式系统可能具有挑战性。复杂性从应用层转移到网络层，需要服务之间进行更多的交互。使您的代码成为 “云原生” 意味着处理 12 个因素（https://12factor.net）的问题，例如外部配置、无状态、日志记录和连接到支持服务。Spring Cloud 项目套件包含使应用程序在云中运行所需的许多服务。
    

### 1.1.4> Web 应用开发 Web apps

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5icibPnlv6t56o7GUs8iatmVdXmqp50X6uPtU7wodsgDlndYtNeian9ssiaA/640?wx_fmt=png)

*   简介
    
    Spring 使构建 Web 应用程序变得快速而轻松。 通过删除与 Web 开发相关的大部分样板代码和配置，您可以获得一个现代 Web 编程模型，该模型简化了服务器端 HTML 应用程序、REST API 和双向、基于事件的系统的开发。即：Spring MVC。
    

### 1.1.5> 无服务器

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5AzD99YXGpgEa3AGjLwKOq5bzcoxuz0E4PKsqSib9elkSs6aRR44gBJg/640?wx_fmt=png)

*   简介
    
    无服务器应用程序利用现代云计算功能和抽象，让您专注于逻辑而不是基础设施。 在无服务器环境中，您可以专注于编写应用程序代码，而底层平台负责扩展、运行时、资源分配、安全性和其他 “服务器” 细节。
    
*   什么是无服务器？
    
    无服务器工作负载是 “事件驱动的工作负载，与通常由服务器基础设施处理的方面无关。” “运行多少实例”和 “使用什么操作系统” 等问题都由功能即服务平台（或 FaaS）管理，让开发人员可以自由地专注于业务逻辑。
    
*   无服务器特性？
    
    无服务器应用程序具有许多特定特征，包括：带触发器的事件驱动代码执行、平台处理所有的启动、停止和扩展工作、可扩展至零、闲置时成本低至零、无国籍
    
*   无服务器
    
    即：函数式服务 Serverless，可以将函数式服务上传到云平台，按量计费、实时计费。
    

### 1.1.6> 事件驱动 Event Driver

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5cqWCfiaRBVO290n8dLnFxvnzWZaQNr8icpiadHz5ianibaHNfLenrJNQxhA/640?wx_fmt=png)

*   简介
    
    事件驱动的系统反映了现代企业的实际运作方式——每天都在发生数以千计的小变化。 Spring 能够处理事件并使开发人员能够围绕它们构建应用程序，这意味着您的应用程序将与您的业务保持同步。 Spring 有许多事件驱动的选项可供选择，从集成和流式传输一直到云功能和数据流。
    
*   事件驱动的微服务
    
    当与微服务结合时，事件流提供了令人兴奋的机会——事件驱动架构就是一个常见的例子。 Spring 简化了事件的产生、处理和消费，提供了几个有用的抽象。
    
*   流数据
    
    流数据表示事件的持续流。 一个例子可能是股票代码。 每次股票价格变化时，都会创建一个新事件。 之所以称为 “流数据”，是因为有数千个此类事件会产生持续的数据流。
    
*   一体化
    
    任何事件驱动系统的基石都是消息处理。 连接消息平台、路由消息、转换消息、处理消息。 使用 Spring，您可以快速解决这些集成挑战。
    

### 1.1.7> 批处理任务 Batch

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5GWY75hmgQMF742UgvmF2csyZHFTzc5Dtia61eYvSfbVzoSGuWn47xQg/640?wx_fmt=png)

*   简介
    
    批处理有效处理大量数据的能力使其成为许多用例的理想选择。 Spring Batch 对行业标准处理模式的实现让您可以在 JVM 上构建健壮的批处理作业。 从 Spring 产品组合中添加 Spring Boot 和其他组件可让您构建任务关键型批处理应用程序。
    
*   什么是批处理？
    
    批处理是以不需要外部交互或中断的方式处理有限数量的数据。
    
*   为什么要构建批处理？
    
    批处理是处理大量数据的一种极其有效的方式。根据 SLA（ervice-Level Agreement：服务等级协议） 安排和优先处理工作的能力让您可以分配资源以获得最佳利用。
    

1.2> SpringBoot2 给我们带来了什么  

----------------------------

*   SpringBoot 用来便捷整合 Spring 生态圈的各种技术。
    

*   由于 Spring5 是由 JDK8 实现的，由于 lambda 表达式和默认接口实现等 JDK 新的特性，并且 Spring5 开始支持响应式编程，所以 Spring5 的变化很大。
    
*   而 SpingBoot2 底层使用的是 spring5，所以与 SpringBoot1 差异很大。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5xzyAiaKSgrF3OCorog4TPUC9tkw2GzrqblQAmq76UiaYBKXicT0shlM9g/640?wx_fmt=png)

1.3> SpringBoot 优点  

---------------------

*   官网截图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic545Fic1F3ibyict43xXTawBaV070fr5c6HLiabbviamvMicLwuzMGqSJxJmPQ/640?wx_fmt=png)

*   创建独立（stand-alone）的 Spring 应用
    
    SpringBoot 就可以创建独立的 Spring 应用，它比用原生的 SpringFramework 开发的应用更简单，配置更少。
    
*   内嵌（embed）web 服务器
    
    以前开发 web 应用，会把项目打成 war 包，然后外部 Tomcat 运行项目。
    
*   提供可选的（optinionated）start 依赖，简化构建配置
    
    启动器 start，可以引入该场景下所有的包依赖，并且多个 jar 包对应的版本也帮我们选择好了。
    
*   自动配置 Spring 以及第三方功能
    
    以前开发 Spring 项目，有很多常规配置需要配置，并且引入其他技术时，都伴随着大量的配置需要手动执行。那么，有了自动配置后，这些都不需要自己去配置了，我们建立好 SpringBoot 项目后，可以直接面向业务代码开发，而不必被大量配置所困扰了。
    
*   提供生产级别（production-ready）的特性，例如指标、运行状况检查和外部化配置
    
    SpringBoot 自带了生产级别的指标和运行状况检查，可以帮助我们了解服务运行的最新状况。并且，当我们需要修改某些配置的时候，也不需要直接在项目源码上进行修改了，可以通过外部化配置，就可以将修改生效。
    
*   完全不需要代码生成，也不需要 XML 配置
    
    SpringBoot 是整合 Spring 生态圈技术栈的一站式框架
    
    SpringBoot 是简化 Spring 技术栈的快速开发脚手架
    
    SpringBoot 迭代快，变化快。切封装很深，内部原理复杂，不容易精通。
    

二、第一个 SpringBoot 项目  

======================

2.1> 纯手工创建
----------

*   具体步骤，参照官网文档：
    
    https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.first-application
    
*   创建 Maven 项目，添加父依赖
    

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.4</version></parent>

```

*   添加 Spring WEB 依赖
    

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

```

*   创建应用类
    

```
@RestController
@EnableAutoConfiguration
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }
}

```

*   启动项目
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5mb5NIxdBiaalwTS9jJIdq4vUZ2NLgGyjobFEnnheVue28kQuJtTCatQ/640?wx_fmt=png)

*   测试请求 http://localhost:8080
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5sicbFpic2CRZDQTXficcPGrx4vibbXG1oiaE4Dufc9IPYiblmrZkInmClfPw/640?wx_fmt=png)

2.2> 依赖脚手架创建
------------

*   创建 springboot 项目 https://start.spring.io ，引入 Spring Web 的依赖。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5x08Cxy4SwKgW30BAy1vaDba0r6RaHmsoAl4qeuyvQVmcruH3BNLuIg/640?wx_fmt=png)

*   下载项目，引入 idea
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic50dCudziag3ZtICiab7QPSYA4U0Nmom00hePkEXLuSgv4Tw87tJU2pWxA/640?wx_fmt=png)

*   创建 SpringbootDemoApplication
    

```
@SpringBootApplication
public class SpringbootDemoApplication {
    public static void main(String[] args) {
        // 返回配置的应用上下文
        ConfigurableApplicationContext context = SpringApplication.
            run(SpringbootDemoApplication.class, args);
    }
}

```

*   创建 DemoController
    

```
/**
 * 可以根据实际情况，选择不同的注解方式。
* Spring并没有选择一刀切的方式，而是保留了老的注解使用能力。
* 对老项目的改造是非常友好的。*/
// @Controller + @ResponseBody
@RestController
public class DemoController {
    // @RequestMapping(, method=RequestMethod.GET)
    @GetMapping("hello")
    public String hello() {
        return "Hello World!";
    }
}

```

*   启动 SpringBoot
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5LaOJOkI4SG0z7sYzftNXUmrld3hUSZhPalLxoejHLBDJBhqbAO0P7g/640?wx_fmt=png)

*   请求测试
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic57xEge4ribBicUQRibUibZDcQHSI9NJmfic4LvB8Dv7IaZlzkYSZ98kS0f5Q/640?wx_fmt=png)

三、关键特性介绍  

===========

3.1> spring-boot-starter-parent 的依赖管理
-------------------------------------

*   父项目（**spring-boot-starter-parent**）做依赖管理，子项目就不需要写版本号了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5Nb0VK4aSKyVmcXucKdIO1bWd8qcu1Obor4kWaXfSJCNNBIWvjlpfQA/640?wx_fmt=png)

*   **spring-boot-starter-parent** 的父 pom 为 **spring-boot-dependencies**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5I62VrcAdxibHQ4BZVkPMtnoNjmkOPziajAShFOlAPuY3RmTMLPg2paicA/640?wx_fmt=png)

*   **spring-boot-dependencies.xml** 里面包含了开发中常用的版本集合
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5Kmu9B6mWvxXgEicmgnJiayVeicfUEW9qBwEO94icmiaP4lwa11EBZAw6aiag/640?wx_fmt=png)

*   如果我们不想使用默认的版本，那么我们可以自定义版本，引入 dependency 即可。如果引入默认版本，可以使用如下方式：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5b3DmLHb4ms2L67f8IrJaaQIAt8xcFCQvdLCNsoPTWC3lyqzrnjlA9Q/640?wx_fmt=png)

*   查询想引入其他版本（比如：5.16.1），添加指定版本的依赖，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5LAiayQiaS33eXM5x6jOibDZQKXbNMlN1TJT90InYibvSehXevCWJdqY0dQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5XWtJjsgLicZnzvgU4Rib0eoeklovqVIf3A3TZnoCKZKSyxuHbg8bJ4Dw/640?wx_fmt=png)

3.2> Starter  

---------------

*   我们引入什么场景的 starter，那么就会将一整套场景的 jar 包都引入进来，我们也不需要关注多 jar 包直接的版本号是否兼容彼此，这块工作 spring 已经帮我们做好了。
    
*   SpringBoot 提供的 Starter 有哪些？
    
    https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters；分为三类 Starter，分别为：**application starters**，**production starters** 和 **technical starters**
    
*   查看官方文档中的介绍
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5Fh5iaJdxWbJwyBdAodmNQVvib5t0FkqBX9Nw23Spa6fiaNAn5M6pnA5bw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic59sfAwQ0m7NeV6XibpzC3StLyrWHUHibrEce9y8v4WqP52k0hrrTRQKOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5CYQOxpXahVbia3WibODicqd0SSJ7uBAzKvoPFb3SKVrWKoHYvA1q1Eicbw/640?wx_fmt=png)

*   所有场景启动器最底层的依赖
    

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.5.0</version>
  <scope>compile</scope> 
</dependency>

```

*   我们项目中，只引入了 spring-boot-starter-web，让我们来看一下依赖关系：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5XjbPF4hbzwvYdCwf921jhib81mRtNbp0T4uwU9blflosakktrQVLcwQ/640?wx_fmt=png)

3.3> 自动配置  

------------

### 3.1> spring-boot-autoconfigure

*   SpringBoot 所有的自动配置功能都在 **spring-boot-autoconfigure** 包里面。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5nnuoVoYEicleQge7O8wKzC7kHiaOCVqWNFBBgZCHabQXibExC2188krBQ/640?wx_fmt=png)

*   包下面包含了各种自动配置类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic587JHYglJibvVzIJFODrterT9PgL9eOFyuiaTJwQl3vwk2OL8ep52FUVA/640?wx_fmt=png)

### 3.2> spring-boot-starter-web

*   当我们引入了 spring-boot-starter-web 的 Starter 之后，那么会自动的引入如下一系列 Starter。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5mtoXQ2D5btyicwjibczmiciadiarFKR9SCLuJqllMaia3O5Kbqicicev5t0FicQ/640?wx_fmt=png)

*   SpringBoot 帮我们配置好了所有 web 开发的常见配置 bean。查看注册到 IOC 的 bean 名称。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5nuxvwQFUq16dqeicWibrVibn6JAC0AQerTTaH0jgMbb3eW9qslm829fOQ/640?wx_fmt=png)

### 3.3> 默认包扫描路径  

*   主程序 MyApplication.java **所在的包**及其**下面的所有子包**里面的组件都会被默认扫描。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5g6tSFV0ZOWs3GxrzW6GY2EMy7xmf6Ku0SCSTE43AnH3J1M12dhvIPQ/640?wx_fmt=png)

*   在主程序上一级路径创建 Controller。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic51bbUjZrNcGLXHvMy3P0hmSwxHicFgufRp2fsPjD3xaNAuPfpchph6CA/640?wx_fmt=png)

*   启动后，请求 404。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5qQf9NF6nia5GWVQyUg5AFxlG9iaRaHQCTCZRJ70myXmBYL1qvibhJ3s8w/640?wx_fmt=png)

*   如果想要被扫描到，可以指定 **@SpringBootApplication(scanBasePackages = "com.muse")**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5JzfwLOREyFiatdQfM3rF8xutXfCR03iayJibYO77ibV9JYYEibju2k2VRlw/640?wx_fmt=png)

*   或者指定 **@ComponentScan("com.muse")**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5bjoTedEPwjaJuQIn8tFktly1HJF0W03NEBX6ERVG4EbiaEmQEsKNGEw/640?wx_fmt=png)

*   启动后，请求正常。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5bXV9IFpmibttBwC9nL1g62icqdXXFfx64wqfTRYbrK2S7Aa7VFPbT7XQ/640?wx_fmt=png)

四、配置信息  

=========

4.1> SpringBoot 支持配置类型简介
------------------------

*   Spring 的配置文件分为以下两种：
    

*   application.properties
    
*   application.yaml 或 application.yml
    

*   以及两个配置方式简单对比。（采用服务已有配置）。官方文档 7.2.5 Working with YAML。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5ksJxBa3NCDm4RNIkEQaJ15icZc36SAHYV2TI5GiakXWMiaibqXKtj8ib2BA/640?wx_fmt=png)

*   它们的加载优先级是 **properties>yaml>yml**，也就是 properties 里配置的内容会覆盖另外两个的配置
    
*   至于原因可以在 **spring-boot-starter-parent** 里找到，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic50icZSauVb49yNXNDiaibVKG1vl8doibTzIgIZxhib6nqQ6yTbSWerYV9oxA/640?wx_fmt=png)

4.2> 默认配置值和配置对应处理类  

---------------------

*   SpringBoot 为我们的配置也提供了默认的配置值。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5OhRd9XcUcicaU1Mpib6bmwcB10YLAHEgEADvT1kuedhB3WECvGKjeqIg/640?wx_fmt=png)

*   SpringBoot 的配置文件 **application.properties**
    

点击对应的配置值，可以跳转到对应的 **xxxProperties.java** 类中

配置文件的值最终会绑定每个类上，这个类会在容器中创建对象。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5seUsl5DqKuXiaRTq4Pfa6dVKNHicujia9VAoc1UrmfuYJKibq3WJcYFO3A/640?wx_fmt=png)

*   ServerProperties.java 里使用了 **@ConfigurationProperties** 注解
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5QsdkH32iarDaiauZt76ZWnevHA4VnkjzSyiby6YcJbox6c4WWXGkukUmw/640?wx_fmt=png)

*   ServletWebServerFactoryAutoConfiguration.java 里使用了 **@EnableConfigurationProperties** 注解
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5SJZCZiakGUCDHae3zgFzibng62aS2yR6mm66bb2A6QmlpM0ghDnOuL9Q/640?wx_fmt=png)

4.3> JavaBean 绑定配置文件  

-----------------------

### 4.3.1> @ConfigurationProperties + @Component

*   当我们想要**自定义配置信息**，并且可以**将配置信息自动注入**到某个 bean，方便我们日后获取的时候，我们可以利用该注释进行操作。如下所示：
    
*   **application.properties**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5brQ5LrPyYmlovQtcEOXKgOLUEqsHicGAB3IsMo0Nn1hIhqFLu0UibBaw/640?wx_fmt=png)

*   **Muse.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5xRicKt4QReHEzEbeJX70lvDY3YKdiaJRDB8O4PiayemibMticGkuic25yFnA/640?wx_fmt=png)

*   **SpringbootDemoApplication.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5MqGXA9nYBQD24oYZic2vqk2grU1kOicO4sibLiafqG17rYLyzHLYHJsryw/640?wx_fmt=png)

### 4.3.2> @ConfigurationProperties + @EnableConfigurationProperties

*   **@EnableConfigurationProperties 要写在配置类上**
    
*   利用 @EnableConfigurationProperties 注释开启 Muse 配置绑定功能，并把这个 Muse 组件自动的注入到容器中
    

*   **DemoConfig.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic55Ie4OEj8hvyPicWibxZ01lmUUBIm39BrtZUWs6bGdMWcOkbQ3NsBcVXQ/640?wx_fmt=png)

*   **Muse.java 文件**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5ROFDiaOXsREYiaPIiaEdhqODMmnduEic3J1Q6Y09icOjicMRMZx6ib9kF4ZEw/640?wx_fmt=png)

*   **SpringbootDemoApplication.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5V7KOUMWjn477Lbicg1cFWZ2xRbRYicENaicR70IMdId03iaT1LC21JBNYg/640?wx_fmt=png)

### 4.3.3> 添加配置提示

*   但是，在 yml 中添加配置项的时候，却没有提示。那么如何处理呢？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic58yWQEibicTbA39voiawfRkMncAiboIiaFWm2ibD6Oa6s51XOd8YibFPCzR3eQ/640?wx_fmt=png)

*   加入 **spring-boot-configuration-processor** 依赖
    
*   并配置打包时，不要将该依赖打包进来
    

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <excludes>
          <exclude>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
          </exclude>
        </excludes>
      </configuration>
    </plugin>
  </plugins>
</build>

```

*   刷新 Maven
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5iaYyMHMcSFnlDfXy3Ym3icfsECdJC6pCV1F4kUzZXNCaT7icAdxp6xBlA/640?wx_fmt=png)

*   重启项目即可，如果重启也不生效，可以执行 Build——>Rebuild Project
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5NX3dmATWeHUHZfJCenz5JOWSSR1icuXLylk9mjPgmwAstB6JvgbNTBQ/640?wx_fmt=png)

4.4> yaml 的用法  

----------------

### 4.4.1> 简介

*   YAML 是 "YAML Ain't a Markup Language"（YAML 不是一种标记语言）的递归缩写。
    
*   在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言），但为了强调这种语言以数据做为中心，而不是以标记语言为重点，而用反向缩略语重命名。
    

*   非常适合用来做以**数据为中心**的配置文件
    

### 4.4.2> 基本语法

*   key: value  （**kv 之间有空格**）
    
*   **大小写敏感**
    

*   使用缩进表示层级关系
    
*   缩进**不允许使用 tab**，只允许使用空格，默认两个空格
    

*   **缩进的空格数量不重要**，只要相同层级的元素左对齐即可
    
*   **#表示注释**
    

*   **字符串无需加引号**，如果要加，‘’和 “” 表示字符串内容，会被认为是 转义 / 不转义
    

### 4.4.3> 数据类型

#### a> 字面量

*   单个的、不可再分的值。如：date、boolean、string、number、null  
    
*   写法如下：
    
    k: v
    

#### b> 对象

*   键值对的集合。如：map、hash、object
    
*   写法如下：
    
    k: {k1: v1,k2: v2,k3: v3} 或  
    
    k:
    
      k1: v1
    
      k2: v2
    
      k3: v3
    

##### c> 集合、数组

*   一组按次序排列的值。如：set、array、list、queue
    
*   写法如下：
    
    k: [v1,v2,v3] 或  
    
    k:
    
      - v1
    
      - v2
    
      - v3
    

### 4.4.4> 实战一下

*   创建 Student 实体类  
    

```
@ConfigurationProperties(prefix = "student")
@Component
@Data
@ToString
public class StudentStudent {
    private String name;
    private Boolean sex;
    private Date birth;
    private Integer age;
    private Address address;
    private String[] interests;
    private List<String> friends;
    private Map<String, Double> score;
    private Set<Double> weightHis;
    private Map<String, List<Address>> allFriendsAddress;
}

```

*   创建 Address 实体类  
    

```
@ToString
@Data
public class Address {
    private String street;
    private String city;
}

```

*   创建 application.yml  
    

```
student:
  name: muse
  sex: true
  birth: 2021/06/05
  age: 20
  address:
    street: 王府井大街
    city: 北京市
  interests: [看书，写字，玩电子游戏]
  friends:
    - 李雷
    - 韩梅梅
  score: {math: 100,english: 80}
  weightHis:
    - 176
    - 199
    - 160
    - 179
  allFriendsAddress:
    bestFriendAddress:
      - {street: 中关村大街,city: 北京市}
      - street: 上海某大街
        city: 上海市
    worstFriendAddress:
      - {street: 深圳某大街,city: 深圳市}
      - {street: 成都某大街,city: 成都市}

```

五、注解介绍  

=========

5.1> 常用组件注解
-----------

### @Configuration

*   定义配置类，之前的 Spring 配置都是写在 xml 配置文件里的。在新的 Spring 版本中，建议首要选择把配置写在配置类中。
    

### @ComponentScan

*   定义扫描路径。
    

### @Bean

*   默认**方法名**就是就是 **bean 的 id**，返回类型就是方法返回的类型。也可以 **@Bean("ak47Gun")**，指定 bean 的名称。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5P0njmLpFuUmYe3jjkzITprvnFwMNtwlcgmM7a0AyRAPnMeYNPlI8Uw/640?wx_fmt=png)

*   配置类里面使用 @Bean 标注在方法上给容器注册组件，默认也是**单实例**的；并且配置类本身也是组件。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5at9guDFTWa6hNGtBOKcO97cl2Hk8USKS5S0oraa3dtXfmOlKtFAEDQ/640?wx_fmt=png)

*   **proxyBeanMethods**——> 代理 bean 的方法
    

*   **Full（proxyBeanMethods = true）**
    

每次从 DemoConfig 中调用 soldier() 或 gun() 获得的都是**单例对象**。

配置类组件之间**有依赖**关系，方法会被调用得到之前单实例组件，用 Full 模式。

*   **Lite（proxyBeanMethods = false）**
    

每次从 DemoConfig 中调用 soldier() 或 gun() 获得的都是**非单例对象**。

配置类组件之间**无依赖**关系，用 Lite 模式加速容器启动过程，减少判断。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5VkRramLVxI46wiaDax63EQkRQBsp4OfW7khojDwUMdWloHUPCSWqrcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5CReE4boyGnTLEGpzg2hTB0o5ibKLWM4xS2rwBtlOHdhzAXjIPbAibdVg/640?wx_fmt=png)

### @Import  

*   给容器中自动创建出注解中指定类型的组件，默认组件的名字就是全类名。如下所示，通过 @Import 注解，Spring 会通过调用 Gun 和 Soldier 这两个类的无参构造方法，来创建这两个类型的 bean 实例。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic54mIjfJab83QrmbWq1fFDn4EKv6j7K5zHDEz8IhgtA02za8JLAWgseQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5a4g4jvU5Tu3ewcs7xrIic4mgk3uXDicaUpc54ByGaZicRE9DZ13Zicdbag/640?wx_fmt=png)

### @Conditional

*   SpringBoot 自动加载的时候，会对应很多自动配置类。如果我们都加载到 IOC 的话，不仅仅很多是我们这个场景下使用不到的，而且也会造成整个启动过程异常缓慢。那我们如何只拥有在某场景下，我们所需要的 Bean 呢？
    

那么这时候，@Conditional 就可以大展伸手了。

*   条件装配：满足 Conditional 指定的条件时，才向 IOC 容器中注入组件。Spring 提供了很多条件装配注解，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5k31IyoQ9tTPBNic4H4PFzajyqP132pNHyibvsHSRakUvDqBQqibbdVrTQ/640?wx_fmt=png)

*   @ConditionalOnBean，当存在某个 bean 的时候，才进行被注释的 bean 初始化。
    
*   注意！！！ **gun() 方法一定要在 soldier() 方法上面**，否则该注解无法识别。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5WBIG4LgzYHRcwPbqnRgc34YJb5nebfIuQIbHzBEH0xx0Yssick5WlZg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5m2VEXfmfwDibk5ziaUrQO3iabiaWkI0ctJx6tEbJ53Zfib2j79iarn8enjmg/640?wx_fmt=png)

*   注解如果放在 soldier()方法上，说明如果 IOC 容器中不存在名称为 “gun” 这个 bean，那么就不会向 IOC 中初始化 soldier 这个 bean。
    
*   注解如果放在类上，则说明如果 IOC 容器中不存在名称为 “gun” 这个 bean，那么这个配置类里面所有的 @Bean 都不会向 IOC 中注入。
    

### @ImportResource

*   针对于旧的项目，由于依然采用 xml 配置的方式，那么迁移为 SpringBoot 项目的时候，是要写大量的 @Bean 的代码的，如果 xml 文件中编写了 100 个 bean，那么我们就需要编写 100 个 @Bean，那么这样的工作量实在太大了。但是，幸好 Spring 给我们提供了导入注解 @ImportResource，通过它，我们指定对应的 xml 文件，Spring 就可以把 xml 中配置的 Bean 都加载到 IOC 中，而不用我们一个个的手写 @Bean 了。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5icicSImt0oxXB2icOZb2oEmjSlLO9Ar68eqrssaZGERWKpqzlib3GQloAw/640?wx_fmt=png)

*   如果引用多个，可以使用 **@ImportResource({"classpath:oldbean.xml","classpath:a.xml"})**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic58n0oKaDke4WwMzDJCEgw0O1vv5rYG1evWgdBMqO8E8TyccVVcTCBTg/640?wx_fmt=png)

六、自动配置原理  

===========

*   我们先来看一下 @SpringBootApplication 注解，它 **@SpringBootConfiguration**+**@ComponentScan**+**@EnableAutoConfiguration**
    

这三个注解组合的。

```
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

```

6.1> @SpringBootApplication 自动配置解析  

-------------------------------------

### 6.1.1> @SpringBootConfiguration

*   我们先看一下其中的第一个注解——@SpringBootConfiguration，它只是表示被该注解标注的类是一个配置类。没什么多说的。
    

```
@Configuration
@Indexed
public @interface SpringBootConfiguration {

```

### 6.1.2> @ComponentScan  

*   我看再看第二个注解——@ComponentScan，它的作用是标记组件扫描，指定扫描路径。也没啥多说的。
    

```
@Repeatable(ComponentScans.class)
public @interface ComponentScan {

```

### 6.1.3> @EnableAutoConfiguration  

*   我们再看最后一个注解——@EnableAutoConfiguration，它包含两个注解。**@AutoConfigurationPackage** + **@Import**
    

```
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

```

#### a> @AutoConfigurationPackage  

*   其中的第一个注解——@AutoConfigurationPackage，利用 Registrar 给容器中导入一系列组件。如下所示：
    

```
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

```

*   我们来看一下 AutoConfigurationPackages.Registrar 类的实现
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5rVFlFFcfOcpke1022iarOMqfsBxSOCRT9Mo0GVOMluhicMHsSetCCPYw/640?wx_fmt=png)

即：将 SpringbootDemoApplication 所在的 package 及其子包都注册到 IOC 中。指定了默认的包加载规则。

#### b> @Import(AutoConfigurationImportSelector.class)  

*   获取自动配置实体类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5Kx8xUhicqrgheHlRQjwkgSBN2icFEovX0GdvDz2ic4P14x0S7NYgWJw7Q/640?wx_fmt=png)

*   获取需要导入的配置
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5iaicQO4pck6uTIDyomJCtibRhIXTicBBlGBBDllicibib58HMe5fLZTo6DqAQ/640?wx_fmt=png)

*   通过 SpringFactoriesLoader，加载配置
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5ceDOks2NycT0ncpIrmkkK6vQjMIpgu5YdpD06JnAmYtQHzIgpibATcw/640?wx_fmt=png)

*   加载项目种所有依赖包下 META-INF/spring.factories 路径下的文件配置
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic50K7vMRS3OfGswd7uXjXceibuGcVcCibQ26JMQic1mJghGwbnK55oseibLw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic59506JjiceJrVNcJTF9hAeiaVX6ry6cgEQVkKrLM58gSNglhMu8aZYhhg/640?wx_fmt=png)

*   查询所有引入包中，具有 / META-INF/spring.factories 的配置文件，并进行加载。本例子中，只涉及 3 个包具有 spring.factories 文件，分别为 **spring-boot-2.5.1.jar**、**spring-boot-autoconfigure-2.5.1.jar** 和 **spring-beans-5.3.8.jar**。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5NXf72zIawLrb4ssdUal4Rut39GElEbMJUiaY6yNG34SZWzeKjiawRuYQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5lPrVER0kgyeD6ThbEr07XoQjo9ESn8ibicAiaVBPhSYiaoGvOqV9b9IYcw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5D1DZvaVyE0duIUUctpwbTDQ0Fo4JTsXIXTbdXrkh1M1G70Elre4iazg/640?wx_fmt=png)

6.2> 按需开启自动配置项  

-----------------

### 6.2.1> 按需加载的方式

*   如上所述，虽然加载了 spring.factories 配置文件中所有的自动配置类，但是，**并不是全都加载到 IOC 中**。而是采用**按需加载（即：@ConditionOnXXX）**的方式进行加载。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5CVlBd5aLrEia0wHBruqicI9u5bmgTibGZlAhIib4KzaK0zygKS9qeia7jwg/640?wx_fmt=png)

### 6.2.2> 场景 1：容错兼容

*   以文件上传解析器 MultipartResolver 为例，很多同学在创建解析器的时候，由于不熟悉编码方式，并没有将 bean 的名称按规则配置为 “multipartResolver”，从而造成解析器使用异常。那么，针对于这种情况，SpringBoot 利用如下方式，实现了容错兼容。
    
*   如下所示：
    
    DispatcherServletAutoConfiguration.multipartResolver(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5iczrwFsWb61fA8q6ySBiaA2I30J0BTic6cfVkDhQt333yEyxVks4l00Jw/640?wx_fmt=png)

### 6.2.3> 场景 2：用户配置优先  

*   SpringBoot 会提供所有默认组件的实现方式，但是如果用户自己定义了，则以用户配置为优先，不使用默认配置组件。
    
*   如下所示：
    
    WebMvcAutoConfiguration.defaultViewResolver()
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5RNghtoWYFlXpPj0ZbkSDquOeaFibwC8k1lUlaCckGNsaFiasEicf1z4bg/640?wx_fmt=png)

### 6.2.4> 场景 3：外部配置项修改组件行为  

*   针对 Spring 中的每个组件，一般都会有对应的外部配置项（application.properties）可以修改其行为。
    
*   如下所示：
    
    WebMvcAutoConfiguration.defaultViewResolver()
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic5lMun9DA3q03ibB0DbTokW5NmTYcbULribw2VJwFankqM6MhoauJdAkTw/640?wx_fmt=png)

### 6.2.5> 场景 4：查看自动配置情况

*   我们通过配置 **debug=true**，可以在后台输出中查看到自动配置的具体情况。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8I2JetgOkFnG3jGEHogNic57uxgIzSBEibtvjaKYGezbySzs7ZTBsq63BdRRZt7FDGO372wZsGAiasg/640?wx_fmt=png)