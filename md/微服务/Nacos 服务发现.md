

    几个星期前曾经分享过 Nacos 在配置中心部分的内容，今天来分享它的另一部分内容，即：服务发现。废话不多说，来看一下今天这篇小文章的目录结构吧：  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuoUwmKicreKgxx2RytTIs2ZeTHZgJe6UyM7XtrmO5LFoYI5hn4GNq4ciccKR4hBPfj6QDqzD03Rg/640?wx_fmt=png)

一、概述  

-------

### 1.1> 从单体服务到微服务

#### 1.1.1> 单体服务（all in one）

*   一个项目中包含了所有的服务。通常来说，如果一个 war 包 / jar 包里面包含一个应用的所有功能，则我们称这种架构为单体架构。即：所有服务都在 service 模块中。
    
*   如下图所示：    
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuoUwmKicreKgxx2RytTIsCkPT8U4iaxXM0waicO3hyibolQX2hT6xTSCqIrTn91lSHTemRxEkQJNCg/640?wx_fmt=png)

*   优点：
    

*   1> 架构简单，清晰。服务之间调用方便，快捷。
    
*   2> 服务部署简单。部署量少。运维量很低。
    

*   缺点：
    

*   1> 随着项目进展时间变长，整个项目的**代码复杂度越来越高**，很容易一个小改动，导致很大的系统问题。
    
*   2> 由于代码量很大，**编译和部署会越来越慢**，甚至 20~30 分钟都很正常。
    
*   3> 由于有的功能属于大量运算的 **CPU 密集型**模块，有的是大量读写磁盘的 **IO 密集型**模块。但是由于融合在了一个项目中，无法针对单个功能模块进行扩展。那么只能 CPU 和内存和磁盘都要提升，资源投入很大。
    
*   4> 如果我们想要针对已有项目改变技术选型，那么就需要针对整个项目进行修改，工作量将会巨大。
    

1.1.2> 集群级垂直化  

*   随着用户量越来越大，服务器负载也随之增高，用户需求导致对产品的需求也会剧增。那么这一个 war 包中的代码量也会越来越大，面临着每一次代码修改（无论小修改还是大迭代），都要把整个商城的代码重新测试和部署，而且部署时间也会非常的漫长。针对这种情况，我们进行如下优化：
    

*   1> 通过**横向增加服务器**，把单台机器变成多台机器的集群。
    
*   2> **按照业务的垂直领域进行拆分**，减少业务的耦合度，以及降低单个 war 包带来的伸缩性困难的问题。
    

*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4wK44fF1vkrw6kUL6VIZdJXQPWAribxcC1akn5hFR1Sd0iczzcYjwCA2Q/640?wx_fmt=png)

#### 1.1.3> SOA  
  

*   核心目标是把一些**通用的**、会被多个上层服务调用的**共享业务****提取成独****立的基础服务**，这些被提取出来的共享服务想对来说比较独立，并且可以重用。所以在 SOA 中，服务是最核心的抽象手段，业务被划分为一些粗粒度的业务服务和业务流程。
    
*   SOA 主要解决的问题是：
    

*   1> **信息孤****岛。**
    
*   2> **共享业务的重用。**
    

*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4SAwvl6EuooFPBEzyg7e8CVjuiaFW9LaicZ3bOicOI4JbhGiaK7ocoD6ygQ/640?wx_fmt=png)

#### 1.1.4> 微服务  

*   那么被 SOA 拆分出来的服务是否也需要以业务功能为维度来进行拆分和独立部署，以降低业务的耦合及提升容错性？微服务就是这样一种解决方案。
    
*   我们可以**把 SOA 看成微服务的超集**，也就是**多个微服务可以组成一个 SOA 服务**。
    
*   伴随着服务颗粒化的细化，会导致原本 10 个服务可能拆分成了 100 个微服务，一旦服务规模扩大，就意味着服务的构建、发布、运维的复杂度也会成倍增加。
    
*   如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY48tas5VG2DpnpuVWfyMKIsCqd4aGaf4wuarH4QWdo4qWUic7CVMpkpDg/640?wx_fmt=png)

*   优点：
    

*   1> 每个服务足够小，足够内聚，**专注于一个业务功能点**提供服务。代码更容易理解。
    
*   2> 有**代码修改**或**部署上线**，只会影响对应的微服务，而不会是整个服务。
    
*   3> 可针对服务是**计算型**还是 **IO 型**进行针对性的**硬件升级**。
    
*   4> 可以针对某些**高吞吐服务**进行硬件升级或者服务横向扩容，而不是对所有服务都升级，**节约投入成本**。
    

*   缺点：
    

*   1> 极大的**增加了运维工作量**，以前几个 war 包，现在可能需要部署几百个。
    
*   2> 微服务之间的互相调用，会**增加通讯成本**。
    
*   3> 分布式事务问题会引出**数据一致性**的问题。
    
*   4> 服务增多，如果管控成百上千的服务。如何准确并快速**定位问题**。
    

#### 1.1.5> SOA 与微服务的区别  

*   SOA 关注的是**服务的重用性**及解**决信息孤岛问题**。
    
*   微服务关注的是**解耦**，虽然解耦和可重用性从特定的角度来看是一样的，但本质上是有区别的；
    

*   **解耦**：降低业务之间的耦合度。
    
*   **重用性**：关注的是服务的复用。
    

*   微服务会更多地关注在 DevOps 的持续交付上，因为服务粒度细化之后使得开发运维变得更加重要，因此微服务与容器化技术的结合更加紧密。
    

### 1.2> 服务发现介绍  

*   在传统的系统部署中，服务运行在一个**固定的已知的 IP 和端口上**，如果一个服务需要调用另一个服务，那么可以通过地址直接调用。但是，在虚拟化或者容器化的环境中，**服务实例的启动和销毁是很频繁的**，那么**服务地址也是在动态变化**的。这种情况下，就需要服务发现机制了。服务发现的方式有如下两种：
    
*   **客户端****——服务发现 （client-side service discovery）**
    

*   客户端通过查询服务注册中心，获取可用服务的实际网络地址（IP&PORT）。然后通过负载均衡算法来选择一个可用的服务实例，并将请求发送至该服务。
    

*   在服务启动的时候，向服务注册中心注册服务；在服务停止的时候，向服务注册中心注销服务。服务注册的一个典型实现方式就是通过 **heartbeat 机制（心跳机制）**定时刷新。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY421sGLEwzIWOHBaWBQNq9RNWJsG3GNk0FuY7wl1KwjN5vVkIsh2sQkA/640?wx_fmt=png)

*   **服务端****——服务发现 （server-side service discovery）**
    

*   客户端向 load balancer 上发送请求。load balancer 查询服务注册中心，找到可用的服务，然后转发请求到该服务上。和客户端发现一样，服务都要到注册中心进行服务的注册和销毁。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4v0Ovzc4yVicsV0F0MmUPEaaCMzjIMovUome2Xol8iaTtTviauy8Ex3J1A/640?wx_fmt=png)

*   服务发现调用流程
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4pMtiaPDHicEI5AxL9gco2dhCXpUKuxIhIrglF1BSxMJ1AjN2xV43fgVA/640?wx_fmt=png)

### 1.3> 服务发现技术对比  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4bBibsiatHZjHBy03HQt7r32hz5RcaEhE0ZVSWCDMvT88ahVaG2Pr3TcQ/640?wx_fmt=png)

### 1.4> Nacos 简介  

*   https://nacos.io/zh-cn/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY453fqq7jEGe5ibo12Adh4eEMXQB3t6atvJGGWbINkLNek3G4Ub6kG8kA/640?wx_fmt=png)

*   Nacos 是阿里的一个开源产品，它是针对微服务架构中的**服务发现**、**配置管理**、**服务治理**的综合型解决方案。
    
*   官方介绍如下：
    
    致力于帮助您发现、配置和管理微服务。
    
    提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。
    
    帮助您更敏捷和容易地构建、交付和管理微服务平台。
    
    是构建以 “服务” 为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。
    

### 1.5> Nacos 特性  

Nacos 主要提供以下**四大功能**：

*   **服务发现与服务健康检查**
    
    Nacos 使服务更容易注册，并通过 **DNS** 或 **HTTP** 接口发现其他服务；
    
    Nacos 还提供服务的**实时健康检查**，以防止向不健康的主机或服务实例发送请求。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY446UW1CfOucujdrT7icwxmicJMiaIbgvfqFsmc1jV5uYtONV8b6APiaPuWA/640?wx_fmt=png)

*   **动态配置服务**
    
    动态配置服务运行在所有环境中以集中和动态的方式管理所有服务的配置。
    
    Nacos 消除了在更新配置时重新部署应用程序，这使配置的更改更加高效和灵活。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4l4SA7PW5j4ufY5DDMdxOtd5j8Hhxx1cboVichb7RTcJibV3WDyI0LCbg/640?wx_fmt=png)

*   **动态 DNS 服务**
    
    Nacos 提供基于 **DNS** **协议**的服务发现能力，旨在**支持异构语言**的服务发现；
    
    支持将注册在 Nacos 上的服务以**域名**的方式暴露端点，让三方应用方便查阅及发现。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4cPwK9Y6pn7zoYykSJibC500JnRyItEBmrY102GWrAyK1awcEiazpiatOw/640?wx_fmt=png)

*   **服务和元数据管理**
    
    Nacos 能让您从微服务平台建设的视角管理数据中心的所有服务及元数据（即：服务相关的一些配置和状态信息）
    
    包括：管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略。
    

二、快速入门  

---------

### 2.1> 服务协作流程

*   Spring Cloud 常见集成方案
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4ok1CWoeZ0zkFEjEHpqZu56VnvB7eDXqKUwmNibv9PoIZbayMOzNC49g/640?wx_fmt=png)【注】

*   Ribbon：基于客户端的负载均衡。
    
*   Feign：可以帮我们更快捷、优雅地调用 HTTP API。将 HTTP 报文请求方式伪装为简单的 java 接口调用方式。
    

### 2.2> 搭建 Nacos 服务端  

请参见【配置中心 Nacos】中 Nacos 的安装步骤

三、服务发现基础应用  

-------------

### 3.1> Nacos 架构图

*   官方给出的架构图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4nLx6eFPfyvNbXouGPzdWQdYiauibR3bJ4qJMmB9C0EI4xIeVL0zEYbbQ/640?wx_fmt=png)

【注】  

*   **Provider APP ：**服务提供者
    
*   **Consumer APP ：**服务消费者
    
*   **Name Server：**通过 VIP（Vritual IP）或者 DNS 的方式实现 Nacos 高可用集群的服务路由。
    
*   **Nacos Server ：**Nacos 服务提供者，里面包含：
    

*   Open API：功能访问入口。
    
*   Config Service：配置服务模块。在服务或者应用运行过程中，提供动态配置或者元数据以及配置管理的服务提供者。
    

*   Naming Service：名字服务模块。提供分布式系统中所有对象 (Object)、实体(Entity) 的“名字”到关联的元数据之间的映射管理服务，服务发现和 DNS 就是名字服务的 2 大场景。
    
*   Consistency Protocol：一致性协议，用来实现 Nacos 集群节点的数据同步。使用 Raft 算法（使用类似算法的中间件还有 Etcd、Redis 哨兵选举等）。
    

*   Nacos Console：Nacos 的控制台。
    

### 3.2> 前期准备  

#### 3.2.1> 添加父类 pom 依赖

```
<properties>
    <java.version>1.8</java.version>
    <spring.boot>2.1.3.RELEASE</spring.boot>
    <spring.cloud>Greenwich.RELEASE</spring.cloud>
    <spring.cloud.alibaba>2.1.0.RELEASE</spring.cloud.alibaba>
</properties>
<dependencyManagement>
    <dependencies>
        <!-- 引入Spring Cloud Alibaba依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring.cloud.alibaba}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- 引入Spring Cloud依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring.cloud}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- 引入Spring Boot依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring.boot}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```

#### 3.2.2> 项目结构

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4ERW9pUkmR6lpFlU4Giac8ZZOIIQz2LXbVzUcOcLHvdZgXiazicqn9ouYA/640?wx_fmt=png)

### 3.3> 基于 Feign+Nacos 的服务调用  

*   Feign 和 OpenFeign 两者的区别：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4wYmc55ZSuNfFQwW1bqXhrdaURZmeqDmCDGDWxGPcQhnk8Cra1PLDRQ/640?wx_fmt=png)

#### 3.3.0> 整体项目  

*   **nacos-provider**
    

*   项目主 pom
    

*   添加 Lombok、Spring Cloud OpenFeign、Spring Cloud Alibaba Nacos Discovery、Spring Boot Web 依赖
    

*   **nacos-provider-web**
    

*   pom.xml 添加 Spring Boot Web、Nacos Discovery、OpenFeign 的依赖
    
*   application.yml 添加 Nacos 配置
    
*   NacosProviderApplication 类上添加 @EnableDiscoveryClient 和 @EnableFeignClients 注解
    
*   ProviderController 添加 hello() 方法
    

*   **nacos-provider-api**
    

*   无
    

*   **nacos-provider-application**
    

*   无
    

*   **nacos-consumer**
    

*   项目主 pom
    

*   添加 Lombok、Spring Cloud OpenFeign、Spring Cloud Alibaba Nacos Discovery、Spring Boot Web 依赖
    

*   **nacos-****consumer****-web**
    

*   application.yml 添加 Nacos 配置
    
*   ProviderServicepom.xml 添加 Spring Boot Web、Nacos Discovery、OpenFeign 的依赖
    
*   在 feign.service 包中添加 Feign 的接口
    
*   NacosConsumerApplication 类上添加 @EnableDiscoveryClient 和 @EnableFeignClients 注解
    
*   ConsumerController 添加 hello() 方法，引入 ProviderService 并调用 hello() 方法
    

*   **nacos-****consumer****-api**
    

*   无
    

*   **nacos-****consumer****-application**
    

*   无
    

#### 3.3.1> 架构图

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4SsAZQhicItTa2ibkOtqqqJNk42c01ribUpBZ3DAuhDxjmiaJEKZal5cH2A/640?wx_fmt=png)

#### 3.3.2> nacos-provider 相关代码  

*   **项目的主 pom**
    

```
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring Cloud Alibaba Nacos Discovery -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!-- Spring Cloud -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>

```

【注】添加 **Lombok**、**Spring Boot Web**、**Spring Cloud OpenFeign**、**Spring Cloud Alibaba Nacos Discovery** 的依赖。

*   **nacos-provider-web**
    

pom

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>

```

application.yml

```
server:
  port: 7000
spring:
  application:
    name: nacos-provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

```

【注】添加 **nacos.discovery** 的配置

NacosProviderApplication

```
@EnableDiscoveryClient
@EnableFeignClients
@SpringBootApplication
public class NacosProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication.class, args);
    }
}

```

【注】添加 **@EnableDiscoveryClient** 和 **@EnableFeignClients** 注解

ProviderController

```
@Slf4j
@RestController
@RequestMapping("/provider")
public class ProviderController {
    /**
     * http://localhost:7000/provider/hello
     */
    @GetMapping("/hello")
    public String hello() {
        log.info("provider hello");
        return "provider hello";
    }
}

```

#### 3.3.3> nacos-consumer 相关代码

*   **项目的主 pom**
    

```
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring Cloud Alibaba Nacos Discovery -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!-- Spring Cloud -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>

```

【注】添加 **Lombok**、**Spring Boot Web**、**Spring Cloud OpenFeign**、**Spring Cloud Alibaba Nacos Discovery** 的依赖。

*   **nacos-consumer-web**
    

pom

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>

```

application.yml

```
server:
  port: 7002
spring:
  application:
    name: nacos-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

```

【注】添加 **nacos.discovery** 的配置

NacosConsumerApplication

```
@EnableDiscoveryClient
@EnableFeignClients
@SpringBootApplication
public class NacosConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }
}

```

ProviderService

```
@FeignClient(name = "nacos-provider")
public interface ProviderService {
    @GetMapping("/provider/hello")
    String hello();
}

```

ConsumerController

```
@Slf4j
@RestController
@RequestMapping("/consumer")
public class ConsumerController {
    @Resource
    private ProviderService providerService;
    /**
     * 基于Feign调用
     * http://localhost:7002/consumer/hello
     */
    @GetMapping("/hello")
    public String hello() {
        log.info("Feign invoke!");
        return providerService.hello();
    }
}

```

请求 http://localhost:7002/consumer/hello

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuoUwmKicreKgxx2RytTIscQwicib4ibpcibmePF8KupdcYVnPiab9Jcrv92OIk9rEXLnHW89WNudNWBw/640?wx_fmt=png)

### 3.4 > 基于 Dubbo+Nacos 的服务调用

#### 3.4.0> 整体项目

*   **nacos-provider**
    

*   项目主 pom
    

*   添加 dubbo 依赖
    

*   **nacos-provider-web**
    

*   application.yml 添加 dubbo 配置
    
*   pom.xml 添加 nacos-provider-application 依赖（用于 dubbo 扫描该 module）
    

*   **nacos-provider-api**
    

*   接口 OrderService
    

*   **nacos-provider-application**
    

*   pom.xml 添加 dubbo 依赖和 nacos-provider-api 依赖
    
*   实现类 OrderServiceImpl
    

*   **nacos-consumer**
    

*   项目主 pom
    

*   添加 dubbo 依赖
    

*   **nacos-****consumer****-web**
    

*   application.yml 添加 dubbo 配置
    
*   pom（添加 nacos-provider-api 依赖）
    
*   ConsumerController 添加 getOrder() 方法
    

*   **nacos-****consumer****-api**
    

*   无
    

*   **nacos-****consumer****-application**
    

*   无
    

#### 3.4.1> 架构图

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4hRvhzES8iaJabibbL6FE5wwr4HoL1J1bh1flr7u1qBjOxltG1d5duu2g/640?wx_fmt=png)

#### 3.4.2> nacos-provider 相关代码  

*   **项目的主 pom**
    

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-dubbo</artifactId>
</dependency>

```

【注】添加 dubbo 依赖

*   **nacos-provider-web**
    

application.yml

```
dubbo:
  scan:
    base-packages: com.muse.nacos.provider # Dubbo服务扫描基准包
  protocol:
    name: dubbo # dubbo 协议
    port: 21880 # dubbo 协议端口 （与server.port一样，需要修改）
  registry:
    address: nacos://127.0.0.1:8848 # Nacos注册地址
  cloud:
    subscribed-services: nacos-provider

```

【注】添加 dubbo 配置。

pom 依赖。添加这个 module 的依赖，才能扫描到该 module 下使用了 dubbo 的 @Service 注解的类

```
<dependency>
    <groupId>com.muse.nacos.provider</groupId>
    <artifactId>nacos-provider-application</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>

```

*   **nacos-provider-api 添加**
    

OrderService 接口

```
public interface OrderService {
    /**
     * 获得订单信息
     *
     * @return
     */
    String getOrder();
}

```

*   **nacos-provider-application**
    

pom 依赖

```
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-dubbo</artifactId>
    </dependency>
    <dependency>
        <groupId>com.muse.nacos.provider</groupId>
        <artifactId>nacos-provider-api</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
</dependencies>

```

添加实现类 OrderServiceImpl

```
@Slf4j
@Service // org.apache.dubbo.config.annotation.Service
public class OrderServiceImpl implements OrderService {
    public String getOrder() {
        log.info("Order Info");
        return "Order Info";
    }
}

```

#### 3.4.3> nacos-consumer 相关代码

*   **项目的主 pom**
    

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-dubbo</artifactId>
</dependency>

```

【注】添加 dubbo 依赖

*   **nacos-consumer-web**
    

application.yml

```
dubbo:
  scan:
    base-packages: com.muse.nacos.consumer
  cloud:
    subscribed-services: nacos-provider

```

pom.xml

```
<dependency>
    <groupId>com.muse.nacos.provider</groupId>
    <artifactId>nacos-provider-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>

```

ConsumerController

```
@Slf4j
@RestController
@RequestMapping("/consumer")
public class ConsumerController {
    @Reference // org.apache.dubbo.config.annotation.Reference
    private OrderService orderService;
    /**
     * 基于Dubbo调用
     * http://localhost:7002/consumer/getOrder
     */
    @GetMapping("/getOrder")
    public String getOrder() {
        log.info("getOrder invoke!");
        return orderService.getOrder();
    }
}

```

请求 http://localhost:7002/consumer/getOrder

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuoUwmKicreKgxx2RytTIs3OShY5H14NCrdribGAnYeZI7hcSicbQgC9TwBuyjmxwwjkT4lVUws9zw/640?wx_fmt=png)

四、Nacos 源码解读  

---------------

### 4.1> Spring Cloud 完成服务注册的时机

#### 4.1.1> 通用服务注册标准

*   在 spring-cloud-commons 包中有一个类 org.springframework.cloud.client.serviceregistry.**ServiceRegistry**，它是 **Spring Cloud** **提供的服务注册的标准**。集成到 Spring Cloud 中实现服务注册的组件，都会实现该接口。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4XpzJHyt7AX4uBLf6VWqoPcse0SyiaEE8cwLm9jq8YexxibRfUia4ticvvA/640?wx_fmt=png)

*   Nacos 中针对这个接口有一个实现类是 com.alibaba.cloud.nacos.registry.**NacosServiceRegistry**。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4E9Dr08pyNQgZiawBxEJxeJxMPXWPdIPsN3dwhS9DEP6j5QwiampP7HibQ/640?wx_fmt=png)

#### 4.1.2> Spring Cloud 集成 Nacos 的实现过程  

*   在 spring-cloud-common 包的 META-INF/spring.factories 中包含自动装配的配置信息，其中 **AutoServiceRegistrationAutoConfiguration** 就是服务注册相关的配置类。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4acN3bw3xwTJHnlKTv7TXU3YzR1zM9b4h9KhGiaJjaCGA3ULcRQI5gow/640?wx_fmt=png)

*   在 AutoServiceRegistrationAutoConfiguration 配置类中，可以看到注入了一个 **AutoServiceRegistration** 实例。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4ka4caASxuPQGvUpjbRa0BF4jFtRqTIdFOhGXMjnr7QC7uuYicPBkmng/640?wx_fmt=png)

*   可以看出，**AbstractAutoServiceRegistration** 抽象类实现了 AutoServiceRegistration 接口，并且最重要的是 **NacosAutoServiceRegistration** 继承了 AbstractAutoServiceRegistration。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4vcKKKjuhO9sEP68KN07IibzuUdyYHxzWvY38eRVicS6ByqY4Dq5mVEuw/640?wx_fmt=png)

*   其中，从上面类图可见，AbstractAutoServiceRegistration 实现了接口 **ApplicationListener**。ApplicationListener 类的作用就是提供一种事件监听机制，它的方法 onApplicationEvent(E event) 的作用就是监听某一个指定事件。而 AbstractAutoServiceRegistration 实现该方法，并且监听 WebServerInitializedEvent 事件（当 Webserver 初始化完成之后），调用 this.bind(event) 方法。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4icV1JoSI8oCJ2PFsqaSxOKWEWw5YpBYeGtEAnXhmCIX1gg0JIq9W5xA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4mR82UicatRDghlYcAd7s93kKHYFwMoRmXUGsXhu0JVQ70sWnp6Lu8ibw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4fHrZvvREiaKyoSkRjlExibdriaIbwDibhFB61JFhMpMUibAN1skVOhk1bBQ/640?wx_fmt=png)

#### 4.1.3> Dubbo 集成 Nacos 的实现过程

*   因为要简化标题，所以准确的说，介绍的应该是 Spring Cloud Alibaba Dubbo 集成 Nacos 的实现过程。
    
*   spring-cloud-alibaba-dubbo
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4fpMg1icph6YPWGoa12QyoQ8mfI50nu9WVgjgwns1ECg1KrsuQkI01Sw/640?wx_fmt=png)

*   DubboServiceRegistrationNonWebApplicationAutoConfiguration
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY422GtLdibGn2JaJWLib9JHDHiaf09JLibLnJsROibwdy7MEN9uMIp7juNriaw/640?wx_fmt=png)

### 4.2> NacosServiceRegistry 的实现  

*   register(...) 方法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4KPYQEJzjMBR5DVmHsz0LM7Eg1A1Xo6JzYZQu2cCszccJxIXusZWecA/640?wx_fmt=png)

*   registerInstance(...) 方法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4NngRiaEK8JnEUIntNHiaL917KJwt9BKbWjkJyicISwJT5E7XiaLfJEOyfA/640?wx_fmt=png)

【注】serverProxy.registerService(...) 方法在 4.3 节介绍。

*   addBeatInfo(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4ga7ICoo0EiaSwKfA5K43LYOChibEnkFwmztf9mQwcHt3U44PgGFHQaug/640?wx_fmt=png)

*   BeatTask.run(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4mdia0Ts91p8NibkamJyXIiaPHGTxGMz0lHn8CvPT43DuBgF74FEYrZwQA/640?wx_fmt=png)

### 4.3> 服务注册原理

*   serverProxy.registerService(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY41VVlyBNHWMXmqyKP0SdqBrM0Ye2QDuicB5YVx5ZPGhoCFAOsQjia5uDA/640?wx_fmt=png)

*   Nacos 提供了 SDK 及 Open API 的形式来实现服务注册。这两种形式的本质是一样的，SDK 方式只是提供了一种访问的封装，在底层仍然是基于 HTTP 协议完成请求的。
    
*   InstanceController.register(...)  对应源码项目为：nacos-1.3.0
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4ZOj7GqSYiaWeIgsXlQw1d97TtbBib9tQfhfGu5bBoxvp08t5FewDoVuw/640?wx_fmt=png)

*   registerInstance(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4GmrbmOybyQH57hPLyfVkch62zIhTiadcq48ZzDvoWJ9Icj1eQT3Z7Ew/640?wx_fmt=png)

*   createEmptyService(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4wd6gCnmOpGs0ibgf8b4dTglyWia5eCh5yrjMyicosqxiaNyiaS2C0yzzNzg/640?wx_fmt=png)

*   putServiceAndInit(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4xQMPvcxwaE2OxYj49Hd0l6U2CRYELE8bRo92Pe9IvWwHt9ODFTLSuw/640?wx_fmt=png)

*   service.init() 心跳检测机制
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4JXDn4ThoHicytTmrXJJuCQy2x3YdwBuSlo8TuhKGKnf7QOiatOTl7PRA/640?wx_fmt=png)

*   pushService(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4ibyqRZXz3Ivx6oB2rzNqice9bMyR02DgupzuBuQCkOic6YGrF4EAPZWWw/640?wx_fmt=png)

*   简单总结一下服务注册的完整过程：
    

*   Nacos 客户端通过 **Open API** 的形式发送服务注册请求。
    
*   Nacos 服务端收到请求后，做以下三件事：
    

*   构建一个 **Service 对象**保存到 **ConcurrentHashMap** 集合中。
    
*   使用**定时任务**对当前服务下的所有实例建立**心跳检测机制**。
    
*   基于**数据一致性协议**将服务数据进行同步。
    

### 4.4> 服务提供者地址查询原理  

*   InstanceController.list(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4V8ZianhfiaRSLfhTWpwqBBX1NtDqUB4nRP4kFY2YqpftNntk9Mr5GSkw/640?wx_fmt=png)

*   doSrvIPXT(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4oUT6G5gqsYUnia2JehIzVFfYneSrH60r3GfQEWLF68MLVuxmXWAN8GQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4TRnH0Bgex4wrCSSegmxav71wGGDYibicED4rCVBpCHpc5D1hS3iaIC3zw/640?wx_fmt=png)

*   总结一下上面关于 doSrvIPXT 方法的主要逻辑
    

*   根据 namespaceId、serviceName 获得 **Service 实例**。
    
*   从 Service 实例中基于 srvIPs 得到**所有服务提供者的实例信息**。
    
*   遍历**组装 JSON** 字符串并返回。
    

### 4.5> 服务地址动态感知原理  

*   原理图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY47vAwmhxK8fnmDFc5I0bHSYbwoCNQumqD99P5rjWHKmrL9icUzwVicjVw/640?wx_fmt=png)

【解释】  

Nacos 客户端有一个 **HostReactor** 类，它的功能是实现服务的动态更新，基本原理如下：

*   客户端发起事件订阅后，在 HostReactor 中有一个 UpdateTask 线程，每 10s 发送一次 Pull 请求，获得服务端最新的地址列表。
    
*   对于服务端，它和服务提供者的实例之间维持了心跳检测，一旦服务提供者出现异常，则会发送一个 Push 消息给 Nacos 客户端，也就是服务消费者。
    
*   服务消费者收到请求之后，使用 HostReactor 中提供的 processServiceJSON 解析消息，并更新本地服务地址列表。
    

*   HostReactor 的建立
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4NtcqB5yuoetEKBa3j0SntNFc0tc28r7jN7Jw2jib7YS8hYhHrnELKbg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4dsEOrdbvAzm9ufHqm9LFovp2nt8masCxibVlfc8SFNS823xhh8KDnRw/640?wx_fmt=png)

在 init 里会有三个任务：

*   1> FailoverReactor.SwitchRefresher, 默认每 5 秒检测是否开启故障转移，若是开启，则把文件数据读入 serviceMap。
    
*   2> FailoverReactor.DiskFileWriter, 默认一天把服务信息写入本地。
    
*   3> 建立 10 秒后调用 DiskFileWriter#run，检测本地缓存文件，若是没有则建立缓存文件。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4tWKctuCho3mRQ5XHugAASZHG0weExLuOm7VozkQQibLAibZjVLW0HQ9g/640?wx_fmt=png)

*   UpdateTask
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY44ibYRkM8qsEyCurovCPuK8NQVj4EfGOyRTibbn6W48LbnGf4fqtr2WqQ/640?wx_fmt=png)

*   服务获取和刷新的流程图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibFuupAOxupzvgWx9UZTVY4Sk5F0gyr7icWoGX2DtKwibqI8jaAe05CCouaDoVyKKE1WmyvWSNTJibvg/640?wx_fmt=png)