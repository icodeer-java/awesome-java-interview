> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247488170&idx=1&sn=59acd974d08f0600e04dca238adac66a&chksm=e9115057de66d94158c7729e371c5510f7d566447c872f2ac3e2744964f94776a2b8e6594863&scene=178&cur_album_id=2173835491853336577#rd)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqTnDKicBqEZD9kmiaQbSQ7ia09RocQsJd4YniaIibsQvf1Cja0iaXpepkPeTw/640?wx_fmt=png)

一、Spring Cloud Config  

========================

1.1> 概述
-------

*   Spring Cloud Config 用来为分布式系统中的基础设施和微服务应用提供集中化的外部配置支持。它分为**服务端**和**客户端**两个部分。
    
*   服务端——**spring-cloud-config-server**
    
    它作为分布式配置中心，默认通过配置 Git 地址，来连接配置仓库并为客户端提供配置信息。
    
*   客户端——**spring-cloud-config-client**
    
    通过它来创建客户端，通过指定配置中心来管理应用资源与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。
    
*   由于 Spring Cloud Config 实现的配置中心默认采用 Git 来存储配置信息，所以使用 Spring Cloud Config 构建的配置服务器，天然就支持对微服务应用配置信息的版本管理。
    

1.2> 实战操作  

------------

### 1.2.1> 构建 Server 端

*   首先，引入 Config 端 maven 依赖，即：**spring-cloud-config-server**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqLqKP6a41nj51SamWpSjA48C3XQpGFEPfg8kdDlkicibYK2kIYBcu2lWg/640?wx_fmt=png)

*   通过 **@EnableConfigServer** 注解开启 Config Server 的功能
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqOy5wdhLJbl3qick3qeCZYdibiaQPvoqu9ia9vbpqaX3WjT3LqOKClKGKxw/640?wx_fmt=png)

*   在 **application.properties** 中添加配置信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqte1ORHEmicEsuZ5yDwqGANVjIicJ7tyr6THgA8IycvOpNhHEMRyXiaNVw/640?wx_fmt=png)

【解释】

*   **git.uri**：配置 Git 仓库位置。
    
*   **git.search-paths**：配置仓库路径下的相对搜索位置，可以配置多个。
    
*   **git.username**：访问 Git 仓库的用户名。
    
*   **git.password**：访问 Git 仓库的密码。
    

*   在 Gitee 上创建配置文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqzxQT2ujeObRKCf06T3c8ia92uQTJopicWzaxfPnUATRRXqvPMdKwURTw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqfRo0Xicb0ZjVzLaZTqhc190BR9RQ8F85Bz1YXn6RJBhWM2nLt3Ov3VQ/640?wx_fmt=png)

*   启动 SpringBoot 服务，并执行获取配置信息的请求
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqvjyP3GjIZ06epgyqzu7CUAOicL1tIibefLXTeNQMhHBQkcqQCJTic3UicA/640?wx_fmt=png)

【解释】  

*   访问配置信息的 URL 与配置文件的映射关系如下所示：
    

*   /{application}/{profile}[/{label}]
    
*   /{application}-{profile}.yml
    
*   /{label}/{application}-{profile}.yml
    
*   /{application}-{profile}.properties
    
*   /{label}/{application}-{profile}.properties
    

*   **label**：Git 的分支名。例如：config-muse
    
*   **application**：配置的应用名。例如：order
    
*   **profile**：配置的环境名。例如：dev
    

*   我们从控制台的输出也可以看到，Config Server 从 Git 中获得配置信息后（git clone），会**复制一份到本地系统**中，然后读取这些内容并返回给微服务应用进行加载，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqYLUE0jEhPMKrqfG8QR0n8Kb6jHSDvaEggQiaGhgnmqtayglEaFvkFhw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqw9ZXpLWBR9ryuexa8OQuc2JcjTWT5T6xniaYoIIlHNCniafSOHYQu7sQ/640?wx_fmt=png)  

### 1.2.2> 构建 Client 端

*   在依赖中，加入 web 和 Config Client 端依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq231jiaRMYLXNVLAEiaooKF73dlmzxp4FuekDAPf7tJwArLjbyS3nJgJA/640?wx_fmt=png)

【解释】  

*   在 Spring Boot 2.4 能够直接在 application.properties 或 application.yml 文件中使用新的 spring.config.import 属性。当使用配置中心时，由于 SpringCloud 2020.* 以后的版本默认禁用了 bootstrap，导致读取配置文件时读取不到该属性。解决这个问题的办法，就是在 maven 中加入 spring-cloud-starter-bootstrap 依赖。
    

*   在 **bootstrap.properties** 中添加配置信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq96ibXvWH5GHTDEjIAXjTOBNVMxic3BPAiaNpriaBPW2znTWIKctyZ8bYwQ/640?wx_fmt=png)

【解释】  

*   此配置文件的名称一定是 **bootstrap.properties**，因为只有这样，config-server 中的配置信息才能被正确的加载。
    

*   创建 ConfigClientController.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqgDAvx5JuMXha4FDXgkO5qxjJYia3NOVSgibaTZadaOZibWtNrSknxcFcQ/640?wx_fmt=png)

【解释】  

*   此处可以使用 **@Value** 注解来获取配置信息，也可以使用 **Environment** 来获取配置信息
    

*   启动 config-client 微服务，针对 / mysql 和 / mysqlEnv 这两个接口做请求测试
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqQx1ENZlZMgozlahxmYMkria20bpM56Rhp5oU7ice4UDBFa6T8icDPWObA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqibH3fPUYJiaG1TtrKzmibmrbdoQ7rqSh6l2xktG4sQCqReg0ew0PMs3dA/640?wx_fmt=png)

1.3> 特性说明  

------------

### 1.3.1> 基础架构

*   基础架构图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqHwRcxicUIrMjxI7ByNkjC2cyaRdOKB2dBDBcYZtACZWozpJceBPK0dQ/640?wx_fmt=png)

【执行流程】  

*   1> Config Client 端启动后，会根据 bootstrap.properties 配置文件中所配置的 application、profile、label，向 Config Server 请求获取配置信息。
    
*   2> Config Server 接到 Client 端的请求后，根据配置文件中的 Git 配置信息，连接 Git 仓库，查找 Client 端需要的配置信息。
    
*   3> 找到配置信息之后，通过 git clone 将配置信息下载到 Server 端的文件系统中。
    
*   4> Config Server 创建 Spring 的 ApplicationContext 实例，并从 Git 本地仓库中加载配置文件，最后将这些配置内容读取出来并返回给客户端应用。
    
*   5> Client 端在获得外部配置文件后加载到客户端的 AplicationContext 实例，该配置内容的优先级高于客户端 Jar 包内部的配置内容，所以在 Jar 包中重复的内容将不在被加载。
    

### 1.3.2> 配置 Git 本地库  

*   在 Git 的本地库中，创建 4 个 order 的配置文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqrJNe3YZhovico8E8OzwcrdAj0OdBEbjCswyhY3CwO54qicVD6Sia5baSA/640?wx_fmt=png)

*   然后修改配置文件为：git.uri=file://[git 本地库文件地址]
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqQSibYuZuMX0eMaAHlIibK9ibKSqMDx5YiaROlIxpxdb6AQIwohIMuoiaLBg/640?wx_fmt=png)

*   请求测试是否 ok
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqNgtCUic4WSNJBlyYZ4oUebL6c5eMQKVCE3bf7D3Yia2a9E88xowso7pQ/640?wx_fmt=png)

*   查看 Server 端后台打印日志
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq5MwFic0YCSXy79QuAj81ZMxgFQ67ibbKLUDYfD6oVlYe3icTDhBqvuAlw/640?wx_fmt=png)

### 1.3.3> Server 端连接的安全配置  

*   由于配置中心存储的配置一般来说都是比较敏感的，所以我们最好是对其连接的时候进行安全性的连接校验。实现起来很简单，只需要在 Server 端引入 **Spring Security**，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqZlAp8vttfOtQuFL83118Tm0Ar8QJtK7CnicIicVcImoRNkldaVXxcTMQ/640?wx_fmt=png)

*   为 Server 端添加用户名和密码
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqSPw4pY7tBHzyLwx2EmRPlCD4aicj3PAwzDHjA0yBuyKWnvia83PSS5EA/640?wx_fmt=png)

*   为 Client 端添加用户名和密码
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDql9vdOgHC0PzURjZDaN3dv7YLIPqdSCyXERLiaZlBNLy3995AMLQYaRQ/640?wx_fmt=png)

### 1.3.4> 高可用配置

*   只需要在多个 Server 端**配置相同的 Git 仓库**，这样所有的配置内容就通过统一的共享文件系统来维护。而 Client 端请求的时候，只需要将多个 Server 的上层配置负载均衡设备即可。由于 Server 也是微服务，所以，也可以通过将多个 Server 都注册到 Eureka，然后 Client 端通过服务名来进行请求。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqVgicso8Jr6Y2GDkmkwZm6UtPYHM8hAmibk2qvibiaoTII8VZ4vbJXY0tag/640?wx_fmt=png)

*   启动 Eureka Server
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqEuaPgGoqgIpp3WiaNx170ibiaczffE3tKVZjvmicAk9gXvE1SEAqD2Qv6A/640?wx_fmt=png)

#### a> 配置服务端  

*   在 pom 中添加 Eureka 客户端的 Maven 依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqJ8S6Nia62xJFbxof4csg3Ixa1wxPtRTkBwib2nnZf2ZNsgM51kkwk4Ig/640?wx_fmt=png)

*   在 application.properties 中配置服务注册中心
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq5l0P9dPdgM5juibaWvuKrjsxt3UhvDvH6Sm970L1AIZ1D0GvmRKvEUg/640?wx_fmt=png)

*   在主类中添加 @EnableDiscoveryClient 注解
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq3icoBBNQwWkCfaUtV4QFgdK5xM14n1BPeujsO8PaKg5exU7yVqFzb7A/640?wx_fmt=png)

*   启动 config-server 服务，可以在 Eureka 上面看到启动后的服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDql6qic7fovFad5QRocCuKZr8sc7nWHARGk3uWtxOibVnMW0icz8LAzZaVw/640?wx_fmt=png)

#### b> 配置客户端

*   在 pom 中添加 Eureka 客户端的 Maven 依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqBxw0y7gKoY3ViaMsvURbsnTn5wj6ibJbWSFjX4jpeomO6Avw701ddXOA/640?wx_fmt=png)

*   添加 Eureka 注册中心的配置
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqeqRBib53Gnewo58fFZkgzdkDy9OBFXHvw7YQib86iaMnPiblibENn1EG19g/640?wx_fmt=png)

*   在主类中添加 @EnableDiscoveryClient 注解
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqm4kJj4qf4cDtgM8BppSxusKux9BFnbYTgpoPapuJlYmibbicI3YfpDaQ/640?wx_fmt=png)

*   启动 config-client 服务，可以在 Eureka 上面看到启动后的服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqCDcOTccdd7lliclVzubiaickZC9fkO59bayEibImvs5Y0UdCRVWMGSjsiaQ/640?wx_fmt=png)

*   发起请求测试
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq2e1iatGCoR0EEGbGAh49OuSQxvkUxPRcycfYKY5IHiaKeGLVqicXBo5ibw/640?wx_fmt=png)

### 1.3.5> 配置对 Config Server 的快速失败检测  

*   不启动 Config Server，直接启动 Client 端应用。默认会预先加载很多其他信息，然后再开始连接 Config Server 进行属性的注入。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqSbJicNRO4o1mrW0uFFQDdm2fY4vpvpqkia10oKBQqAecGltibyuDrhfow/640?wx_fmt=png)

*   可以通过在 bootstrap.properties 中配置 **spring.cloud.config.fail-fast=true** 来打开快速检测。打开后，启动 Client 端，可以看到后台很快抛出了异常信息。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqgLiaic8tpqz4m3Ps8j2tFlbzyXk1Diaw7iaLpgcupU7XzBekpewcAHrtFA/640?wx_fmt=png)

### 1.3.6> 动态刷新配置  

*   引入 **Actuator** 的 Maven 依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqXpKSDDdQsAB1M1Nxzc901rn0hsBqZr09ZpDQicPj5HPcRlkbA3nryjA/640?wx_fmt=png)

*   在 Client 端的配置文件 bootstrap.properties 中**添加 actuator 配置**信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqqFjUKvL7Q7DyRM3Xj6jdnicerKrmrFFq3ZoCOsrKy9N1VQ7iclPiaW0GA/640?wx_fmt=png)

*   首先尝试请求 / mysql
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqRsfzwPgK0ickuQ6LKLh19FG6FiclQJPwoU8rCK7szHREv45xtEt2jTKw/640?wx_fmt=png)

*   在 Git 上修改配置内容，将 jdbc 前面加个 1，此时再次请求 / mysql，返回的内容依然是原先的配置内容。由于 actuator 监控模块，它包含了 / refresh 接口的实现 ，所以我们请求一下 **/actuator/refresh** 接口，将实现客户端应用配置信息的重新获取与刷新。返回值中已经提示，mysql 有改变。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqbAZUiaB7hLQejKQQib9Cleyxsic0Pucz0RiaGxicTlIw53icy81XVZ6l4VyA/640?wx_fmt=png)

*   随后，我们再次调用 / mysql 请求，发现返回的配置信息中，已经包含了最新的内容，即：jdbc 前面已经有 “1”。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq3WhOC5gTJbV2OoYvDQ1z7JgxI9avYJ7zYtaEE3wXHzWt46HYhqNesw/640?wx_fmt=png)

1.4> 总结  

----------

*   通过上面对 Spring Config 的介绍，大家应该是对它有了一定的了解。并且，在动态刷新配置的介绍中，我们也能猜想到该功能应该是同 **Git 的 Web Hook 功能**有关联的。也就是说，当 Git 中配置的内容有变化时，就针对配置了 actuator 并且发送了 / refresh 请求的客户端实现配置信息的实时更新。
    
*   但是，由于 / refresh 刷新操作**只是通知某个服务实例**去获取最新配置，而**不是刷新所有**的服务实例。所以，随着微服务中服务越来越多，如果还是采用这种 / refresh 刷新来获取最新配置信息的方式，显然是不合适且工作量浩大的。那么针对于这种情况，我们就可以使用 Spring Cloud Bus 来实现以消息总线的方式进行配置变更的通知，并完成集群上批量配置更新的操作。
    

二、Spring Cloud Bus  

=====================

2.1> 概述
-------

*   什么叫做消息总线
    

在微服务架构中，构建公用的消息主题并由其他微服务去订阅和消费，从而起到广播通知的作用，那么我们就称之为消息总线。

我们可以通过 Spring Cloud Bus 非常便捷的搭建消息总线，比如可以与 Spring Cloud Config 进行结合，实现配置更新的全局广播。

*   在当前的 Spring Cloud Bus 中，**仅支持 RabbitMQ 和 Kafka**，如果我们使用的是本机的 MQ，那么我们甚至都不需要做任何配置，只需要引用 Bus 的 Maven 依赖就可以了。下面实战部分，我们将以 Kafka 为例。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqahULVbZznpjhBp1MUcK76vsruZnFibTlhaNefzuOIaa6icI1athdTHfg/640?wx_fmt=png)

2.2> 实战操作（实现 Config 的全局刷新）  

-----------------------------

### 2.2.1> 前期准备工作

*   安装 & 启动 Zookeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqGxmOccIe359VYKWq1cT07YkmB7g73B5acemGRgYOwrD4MlFKYCKiafg/640?wx_fmt=png)

*   安装 & 启动 Kafka
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqzMyldmLsNria26w674ibPrDdaJWQ2nHhMj4ZQDBzuYs85EFLjlmmjbRA/640?wx_fmt=png)

*   启动 Eureka 注册中心
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqLoh0Z2ibA3vAfEfMfb67DiaZGaib6moH056FXNh8IQbR1owcY3yYcUjUA/640?wx_fmt=png)

### 2.2.2> confg-server-bus 服务端集成

*   首先，我们引入 **spring-cloud-starter-bus-kafka** 的 Maven 依赖，由于需要刷新端点，所以也需要**依赖 actuator**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqhrMnUjibTc44JxyR4XVcGVicwogWHiaUvLy6dJ5UhX9KqTcibc8wibnOpaQ/640?wx_fmt=png)

*   在配置文件 application.properties 中增加 **config**、**Eureka**、**actuator** 配置信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq57I31h38as9ia0YReOJ1dibjF8mj8LvoRgbiarjM7780PunaYLgyNAIVg/640?wx_fmt=png)

*   在应用类上开启 Eureka 客户端（@EnableDiscoveryClient）和 Config Server 端（@EnableConfigServer）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqvYkuG7xLQTlWicGonDSibOqaibb13EsGricKy2RS6A10o1QzUxooSiaGTyQ/640?wx_fmt=png)

*   启动 config-server-bus 服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqvNSc89icaes12oWWAblFr6A5DAZC5P5QhNlsUpnr6ovJiat07GH7ENuw/640?wx_fmt=png)

*   启动服务之后，在控制台中会输出 Kafka 的连接信息，其中 “Resetting the last seen epoch of partition springCloudBus-0” 显示消息总线采用的是 **springCloudBus** 这个 Topic。
    

```
INFO 4193 --- [pool-7-thread-1] org.apache.kafka.clients.Metadata        : [Consumer clientId=consumer-anonymous.4a8521bd-2d93-4ee8-a7be-6842d8b8a27c-4, groupId=anonymous.4a8521bd-2d93-4ee8-a7be-6842d8b8a27c] Resetting the last seen epoch of partition springCloudBus-0 to 0 since the associated topicId changed from null to z9pN7q99TVq64Bisub2Ofw

```

*   我们可以查看 Kafka 下 Topic 为 springCloudBus 的 log 文件，该文件中保存着消息数据
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqSOACLZE51Glia9NJdqUkFNeFpjsFETlRGzW7Qib0y7TIfHxSj36skBKA/640?wx_fmt=png)

*   登录 Eureka 控制台 http://localhost:8000/，查看 Server 端服务是否注册成功
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqPFQDRpydE8u8BTMJ2kvWAhbFUmM4Ie6B14JNdhbbVTgZy4wgiaxU9pQ/640?wx_fmt=png)

### 2.2.3> confg-client-bus 客户端集成  

*   首先，我们引入 bus kafka 的 Maven 依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqyqydFtLg5pbXs8j8ibtERqeicF5EPA1zTokoicibddaTqmlM3kmT0d6Prw/640?wx_fmt=png)

*   在配置文件 **bootstrap.properties** 中增加 config 和 Eureka 的配置信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq0aib9rjHQAr41OibFia5dRy3qLjSeTSNp6EfFamTns5TKP2ulmNAvxnYg/640?wx_fmt=png)

*   在应用类上开启 Eureka 客户端（**@EnableDiscoveryClient**）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqTwU4ib1BLSkXES0JVUVA5TfhH2JiaInQwNHib3Ta7byVfYDhDNRjDnmHA/640?wx_fmt=png)

*   启动两个 Client 端，分别为 9001 和 9002
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqhYaNb8zEuVIou8VYkiax6vxgD3gOOV7oHib1uiapwACW7MibicCMm9Ehyuw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq5hRzIv8HhnDwcRP8E1bT8YVDHaeuZfbun9otK6DfUiaeOHickYpe5UfA/640?wx_fmt=png)

### 2.2.4> 通过 Bus 进行配置的全局广播更新

*   首先，我们先查询配置信息，获得配置信息为：“jdbc:mysql...”
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqCiamQ9q8A1SwCBSRUVh8jG5RHnkSSlAwjebicftm7shQjm5fDsD76q9g/640?wx_fmt=png)

*   我们在 Gitee 上修改配置信息，修改为：“1111jdbc:mysql...”
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqicsyZHU3BXRlPwicgMY3mnFbBheEJxB5dStl9VRvJKWRjiaJhkG1TTOGQ/640?wx_fmt=png)

*   此时，再次请求获取配置信息接口，发现返回的配置信息还是旧的信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqOatFHZfCeiaibibHbPI0odWkjaTqiax4nxnN3TwZrZ3xmohHjzyD5yBovg/640?wx_fmt=png)

*   为了能够全局更新配置，我们请求 9000 这台 Config Server 的微服务接口，请求路径为 **/actuator/busrefresh**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqahzDupSLzEjGASWibBeTP46DCIyuG82xH1Y1ABBpXvTnmV0bdm1O48A/640?wx_fmt=png)

*   刷新完毕后，我们分别查看 9001 和 9002，发现已经获取到了最新的配置信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqsf6VLWoticcFPjMmqPtLuBibia9S9u13oI7ILFX8VY4EB8POBibwkHV1nQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq72sVlWQVpcf7GTenqF7CUc6ITuUhq0J6hdo10cfcgqxxzeL0b31Iow/640?wx_fmt=png)

### 2.2.5> 触发配置刷新的两种方式

*   通过向某一个 **Config Server 端**发送刷新配置请求，然后该客户端将消息发送给消息总线，然后其他的服务端获取最新配置信息。但是，如果 ServiceA 宕机或者服务迁移了，则会影响到我们刷新操作了。为了解决这个问题，我们建议将请求发送给 Config Server 端。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqhX3Nq9hJTcS7DzL3jyTFP5PBlkgKSr9XkrwWQ5QNIHduhHZ4rNia4aw/640?wx_fmt=png)

*   下面的图就是我们采取将配置刷新请求发送到 **Config Server 端**的配置方案。针对该方案，我们需要将 Config Server 中引入 Spring Cloud Bus，即：将配置服务端也加入到消息总线中来。通过这种方式，即使 ServiceA~C 有任何宕机或迁移，都不会影响到我们对配置的刷新操作。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqSZQ5MDSQXmdexAhwAuxSialYRtvQtu6WVsibqfdqLWmOsqKxKqo2HKKg/640?wx_fmt=png)

2.3> 原理及源码解析  

### 2.3.1> 消息通知解析

*   从上面的例子中，我们知道了消息是通过 Kafka 为 “**springCloudBus**” 的 Topic 发送的。所以，我们可以开启对这个 Topic 的消费端。当执行修改完配置信息后，执行 **/actuator/busrefresh** 请求，我们就会从 Kafka 中获得如下消息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqnuOic2wy1TVdZrdiaeEib6paLDZWnmhaDochYkyib6gryYLib6TNC1ZTNGA/640?wx_fmt=png)

*   把从 Kafka 中获得的消息 Json 格式化，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqH9XGcHjy03yVhibusyFpdk5CISWiaFVgYVQoQm4VicQSzqqrRN7KXrekw/640?wx_fmt=png)

【解释】下面，我们来详细理解消息中的信息内容：  

*   **type**：消息的事件类型。包含：
    
    **RefreshRemoteApplicationEvent**（用来刷新配置的事件）**AckRemoteApplicationEvent**（响应消息己经正确接收的告知消息事件）
    
*   **timestamp**：消息的时间戳
    
*   **originService**：消息的**来源**服务实例
    
*   **id**：消息的唯一标识
    
*   **destinationService**：消息的**目标**服务实例。上面示例中的 **“**”** 代表了总线上的**所有服务实例**。如果想要指定服务或是实例，只需要通过使用 **/actuator/busrefresh/order:9002** 请求，就可以仅仅刷新 order:9002 服务上获取的最新配置信息。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqZFTictVmh3vTzUjQTrL1KzMzCRKCCIcP9q6bUUJl38qz5PVbwsic9A3g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqDpLLTtrZ12gUZ05uXuzia9xoPrhcCP23gGq7az6hpFnPGTRhzicchFQg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq2G0nA8LJTicNzAGmkib4zicrhugAO33TgflfEee8PCMpfFfIayyh1AvVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqVT6yIicG1Msu8hmciaUaK3jZvacR0FgNKg4vW4wQc45t3QfCRMw5yqKA/640?wx_fmt=png)

*   上面的消息内容是 RefreshRemoteApplicationEvent 和 AckRemoteApplicationEvent 类型**共有的**，下面几个属性是 **AckRemoteApplicationEvent 独有的**，分别表示如下含义：
    

*   **ackId**：**Ack 消息对应的消息来源**。我们可以看到第一条 AckRemoteApplicationEvent 的 ackId 对应了 RefreshRemoteApplicationEvent 的 id，说明这条 Ack 是告知该 RefreshRemoteApplicationEvent 事件的消息已给被收到了。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqDPTqLicRiboQ45GkZboeW4DDFicicRDiaEFvpia3IkOaGHYd5eVNt3I3glog/640?wx_fmt=png)

*   **ackDestinationService**：**Ack 消息的目标服务实例**。可以看到这里使用的是 “**”，所以消息总线上所有的实例都会收到该 Ack 消息。
    
*   **event**：**Ack 消息的来源事件**。可以看到上例中的两个 Ack 均来源于刷新配置的 RefreshRemoteApplicationEvent 事件，我们在测试的时候由于启动了两个 config-client，所以有两个实例接收到了配置刷新事件，同时它们都会返回一个 Ack 消息。由于 ackDestinationService 为 “**”，所以两个 config-client 都会收到对 RefreshRemoteApplicationEvent 事件的 Ack 消息。
    

### 2.3.2> 事件驱动模型  

*   Spring 的事件驱动类型中包含三个基本概念，分别为：
    
    **事件——ApplicationEvent**
    
    **事件监听者——ApplicationListener**
    
    **事件发布者**——**ApplicationEventPublisher** 和 **ApplicationEventMulticaster**
    

#### 2.3.2.1> ApplicationEvent 事件  

*   当我们需要自定义事件的时候，只需要继承 **ApplicationEvent**，比如：RemoteApplicationEvent 和 RefreshRemotecApplicationEvent 等。
    
*   通过上面我们对消息内容的分析，我们了解到消息的事件类型为 RemoteApplicationEvent 的子类：
    
    **RefreshRemoteApplicationEvent**
    
    **AckRemoteApplicationEvent**
    
*   那么我们对于源码的解析，就从这两个类开始分析。如下是这两个类的事件关系类图。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqNl2bPtMcficpFl4ib5nNqdUHdwIibutAfg5kpPmEYwr08fyQEpib4uGTJw/640?wx_fmt=png) 

*   **ApplicationEvent** 的继承关系如下所示，其中包含两个属性，分别为 **timestamp** 和 **source**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqb7S9XUIaT7uaOqIj4zGiayNRs1OHhmA8fXJ2P1euLoWHibpXmcnBJKYQ/640?wx_fmt=png)

【解释】  

*   ‍source：表示源事件对象。
    

*   timestamp：用于存储事件发生的时间戳。‍
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqicXfUtQ3k5rtmVJia4VAAb8q0OoOq8reibpTSaRur821qEt7AtN1bqCiag/640?wx_fmt=png)

*   我们先来看一下抽象类 **RemoteApplicationEvent**，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqXMwC7zkvb83pt4vFd5uQ9yeEDLMo6YBPteKUrwZXdib7qibtBhqicBu7Q/640?wx_fmt=png)

【解释】  

*   **@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, property = "type")** 当进行序列化时，会**使用子类的名称作为 type 属性的值**。比如："type": "RefreshRemoteApplicationEvent"；
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq6QichEvWvxVGEGtcibKV0PI34e0gHHpZCSvU2P79taiby0yEPuegV9PcQ/640?wx_fmt=png)

*   **@JsonIgnoreProperties("source")**
    
    序列化的时候**忽略 source 属性**，source 是 ApplicationEvent 的父类 EventObject 的属性，用来定义事件的发生源。
    

*   RefreshRemoteApplicationEvent 事件类是**用于远程刷新应用的配置信息**。它**只是继承了 RemoteApplicationEvent，并没有其他内容**。所以，我们上面看到的关于 "type": "RefreshRemoteApplicationEvent" 的 json 内容中，与 RemoteApplicationEvent 中包含的属性完全一致。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqnnUE8mBHr3IPdrBWzyP3c0VcUlsT7gqYAhagicveRoL3X9FueAAc27A/640?wx_fmt=png)

*   AckRemoteApplicationEvent 事件类**用于告知某个事件消息已经被接收**，通过该消息我们可以监控各个事件消息的响应。由于 Ack 消息是有一个事件源头的，而每个事件都必须继承 RemoteApplicationEvent 抽象类，所以 event 也就被限制为一定是 RemoteApplicationEvent 的子类了。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqHo8RA02ibsj4yiaKQyXJetAlib4kTN52ibibKgWwpGpxXicOibuQ1aUdnDicNA/640?wx_fmt=png)

#### 2.3.2.2> ApplicationListener 事件监听者  

*   类关系图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq5BibTHr8UQaKG5c9FL8XZf4a9VVKC1SGOdFU92xPPe3I6icBicxegn0sQ/640?wx_fmt=png)

*   ApplicationListener 继承自接口 EventListener，而 **EventListener 是一个的空接口**，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqtKGibgUfpU1QTXt8Foo1e4AfXU6mvTjovPK4wYVcNc3Q7LgYLtr88Bw/640?wx_fmt=png)

*   ApplicationListener 的 **onApplicationEvent(E event) 方法**对于入参是有限制的，即：要求入参类型是 ApplicationEvent 的子类，也就是说，**只针对 ApplicationEvent 的子类进行监听和处理**。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqvBENUL64uEqicrbCsU8fbjfmeFCnqDlqTibb7qs7Is5icY4weaCK34y0A/640?wx_fmt=png)

*   我们先看一下 RefreshListener
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqSMs48tH6iczrTUIpibsuDEQVdUpM5iaUbKV2jS6agteWXnibMibUdy6icLxg/640?wx_fmt=png)

【解释】  

*   从 onApplicationEvent 的入参可以看出，它监听的是 RefreshRemoteApplicationEvent 事件类，并且通过 refresh 方法进行配置属性的刷新。
    

*   我们在看一下 EnvironmentChangeListener 的实现
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq489LK8DS670hxfLlniaoIye1LcibvlTQkgacpoqHBsyum9aPXhMIxBxA/640?wx_fmt=png)

【解释】  

*   它针对 EnvironmentChangeRemoteApplicationEvent 事件的监听类，在处理类中，可以看到它从 EnvironmentChangeRemoteApplicationEvent 中获取了之前提到的事件中定义的 Map 对象，然后通过遍历来更新 EnvironmentManager 中的属性内容。
    

#### 2.3.2.3> ApplicationEventPublisher&ApplicationEventMulticaster 事件发布者  

*   **ApplicationEventPublisher** 接口定义了**发布事件**的方法，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq0OyOrfBlvVT9cicZ7FVfVDfrReLy2fjAOJHcL2dUBO2WfIFdX1n8rxQ/640?wx_fmt=png)

*   **ApplicationEventMulticaster** 接口定义了**新增 & 删除 & 事件广播的方法**，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq0jM2s7Rics6m8biaTv2kbJmNPrvLLQibLybO3RfgDGriaPiaJk5FeVzTyaQ/640?wx_fmt=png)

*   ApplicationEventPublisher 中的 publishEvent 方法是由 **AbstractApplicationContext** 来实现的，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqFEeJqWnPjo0Aib8bmkuHX7ibxVNZicBt7I4sXRBXricel761odhh0xyccQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqCc1HmMQv2dKYh2AiaOcPpKXhc1evnjjXpH5V0iczEZd3kbIQRc5pQictg/640?wx_fmt=png)  

*   SimpleApplicationEventMulticaster 通过 getApplicationListeners 方法获得 ApplicationListener 集合，并对其进行遍历调用监听器的 **onApplicationEvent()** 方法来对具体事件做出处理操作。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqicibI1icU0HJibovJ2sHFuwKiagiaRqhtOuz5TynFnBicWTre1o0BX4ktibibyQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqc2aPp752nZ408ZJJom7St7cehu5ujl2TcZHvnciayQXRvF1b9uYgSXw/640?wx_fmt=png)  

*   自动配置类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq5PjJISpdIgGvcBJ2ZTmcuBkTg4s5OWFyEibMia8I2CB9B27dzYxaCvKQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqxXIdkXmpd5KHiajaH1rdOs8K80JOa3HtPBf6dhIBGS7JVvclqZNbdQw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqFNLjYibpoQ1QiarVawODs9raAGm7LhQBWvgrLgHJJGXqmVl7C3EG9ic4Q/640?wx_fmt=png)  

### 2.3.3> 控制端点 Endpoint  

*   AbstractBusEndpoint 作为控制端点的抽象类，包含了两个子类，分别为：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqxFsUsB4CdlSPia5bJFiaYevXyS6okSKETSuVeIe3Q0KvAU2E2OticHOicw/640?wx_fmt=png)

*   类图关系如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqLL8WUo8vBcGj7uPHnI4HoibK29mkNcVcLe0jhVlS9xSyOqS7hwCB60g/640?wx_fmt=png)

三、Spring Cloud Stream  

========================

3.1> 概述
-------

*   消息中间件是我们平时在企业级开发中经常使用的中间件，它具有**缓存**、**解耦**、**削峰**等功能，但是市面上消息中间件很多，比如 Kafka，RabbitMQ，RocketMQ，ActiveMQ 等等，每个种类的 MQ 在使用上都是不尽相同的，那么如果我们想要将中间件进行更换，也会面临着要重现实现对接中间件的代码。那么，Spring Cloud Stream 的诞生，解决了这部分的内容，不过有一点大家需要注意的就是，它现在只支持 **Kafka** 和 **RabbitMQ**，那么它还有那么重要吗？如果我们是使用 kafka 或者 RabbitMQ 的话，它不仅仅可以极大的简化我们使用这两种 MQ，而且如果要将两种 MQ 进行切换的话，也非常的便捷。下面我们就来了解一下 Spring Cloud Stream。
    
*   Spring Cloud Stream 是用来为微服务应用构建消息驱动能力的框架，它本质上就是整合了 **Spring Boot** 和 **Spring Integration**，实现了一套轻量级的消息驱动的微服务框架。
    

3.2> 简单例子入门  

--------------

*   引入 Stream Kafka 的 Maven 依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqtdT2cKvxSMBXxqetZkX673XvtSHuCxPnWs7layt5cIlGR8picdb2ib9Q/640?wx_fmt=png)

*   创建用于接收来自 Kafka 消息的消费者 **SinkReceiver**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqpHiaJTDGbia6FOM5NQvVyTY3WV0dVXCM5IW3zoj0qiaKhCcFBmvILkePw/640?wx_fmt=png)

*   启动 Spring Boot 应用后，通过 Kafka 客户端 kafka-console-producer.sh 向 topic 为 “input” 的主题上发送消息。**./kafka-console-producer.sh --topic input --bootstrap-server localhost:9092**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq4iaLVRSpI55RUX5w1U1VhD8LMicYiaFibtdbRrZxUFDM9UD5Ctb6uEyib5w/640?wx_fmt=png)

*   我们可以在控制台看到消息已经接收到了，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqu0wd4th7doiayDpHI87pWw2Jl74QBz0FvENYzJXIHmyErWtr7j8fyMA/640?wx_fmt=png)

*   查看所有 topic 主题列表——**./kafka-topics.sh --list --bootstrap-server localhost:9092**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqetaCpaRxqvKWjg5YXiaRcUkmOIcpuJbicdicEej7A2GnT0rjQ7bJMoHDg/640?wx_fmt=png)

3.3> 核心概念说明  

--------------

### 3.3.1> @EnableBinding

*   该注解用来指定一个或多个定义了 **@Input** 或 **@Output** 注解的接口，以此实现对消息通道（Channel 的绑定）。上面例子中的 **@EnableBinding(Sink.class)** 绑定了 Sink 接口，该接口是 Spring Cloud Stream 中默认实现的对输入消息通过绑定的定义。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqGuaU9OGa4EvWnbXjuLiaYYFqzbGlM5vgMtuqTm7LYb63yzG5qNXrJfw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDquKbzrh5uOGcjhnkjddmgmpgD3wYHicDHcDUTZGMiaRbCC5NPCkYEypdA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqJxrEH8PytVbqezoq2duABG6494LEmDTmRzoeGcicWicdUo2Q5XB4SUnw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqaDC8g3RpR2JO6E45fK73QNmUIc3bTpkv1UAZ2WkTNgPvuHFGbDB0BQ/640?wx_fmt=png)  

【解释】

*   从上面的源码中，我们可以看到，Sink 和 Source 中分别通过 @Input 和 @Output 注解定义了输入通道和输出通道，而 Processor 通过继承 Source 和 Sink 的方式同时定义了一个输入通道和一个输出通道。
    
*   其中 @Input 和 @Output 注解还有一个 value 属性，该属性可以用来设置消息通道的名称，这里的 Sink 和 Source 中指定的消息通道名称分别为 input 和 output。如果我们直接使用这两个注解而没有指定具体的 value 值，将默认使用**方法名**作为消息通道的名称。
    

### 3.3.2> @StreamListener  

*   该注解主要是定义在方法上，作用是将被修饰的方法注册为消息中间件上数据流的**事件监听器**，注解中的属性值对应了监听的消息通道名。在上面的例子中，我们通过 @StreamListener(Sink.INPUT) 注解将 receive 方法注册为 input 消息通道的**监听处理器**，所以当 kafka 发送消息的时候，receive 方法会做出对应的响应动作。
    

### 3.3.3> Spring Cloud Stream 应用模型  

*   Spring Cloud Stream 构建的应用程序与消息中间件之间是通过**绑定器 Binder** 相关联的，绑定器对于应用程序而言起到了隔离作用，它使得不同消息中间件的实现细节对应用程序来说是透明的。如下图所示，在应用程序和 Binder 之间定义了两条输入通过和三条输出通道来传递消息，而**绑定器则是作为这些通道和消息中间件之间的桥梁进行通信**。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqlHv6ibdPPvFibgID3LLrJhVCWhyv4qwRRRmv0dFD53nI51MMgiaol7m1g/640?wx_fmt=png)

3.4> 注入绑定接口  

--------------

*   在完成了消息通道绑定的定义之后，Spring Cloud Stream 会为其创建具体的实例，而开发者只需要通过注入的方式来获取这些实例并直接使用即可。就像下面的例子，我们创建一个 SinkSender 接口，并通过 @EnableBinding 来执行绑定操作，那么 SinkSender 实例便会注入到 IOC 中，我们可以直接通过 @Resource 或 @Autowired 来使用 SinkSender 实例发送消息。
    
*   构建 SinkSender 接口
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq7Zd6r094A7omLibKrDyibH04Uq3EJWicvxKZd2e6PCUY9DMK9bQ3NwKdw/640?wx_fmt=png)

*   绑定 SinkSender 接口
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq8XTg67Cc55QjAqxZEUNKIXYcBHzUU5JSdic0Kgc7Q3jzlibocB5QuQiaA/640?wx_fmt=png)

*   通过 SinkSender 实例发送消息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqE1g7mCHHXsv5d45a5XiaeiaAzxxvfg55Fss0NJmFWSuZouSgh1wGDD3w/640?wx_fmt=png)

*   执行 http://localhost:8080/sendMsg?msg=aaa 请求，可以在控制台看到 aaa 这个消息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqsAqbhldObdmAbQel3CrGlw4xIdq1tNk23eNnoK88hYobz6oFAiaia00A/640?wx_fmt=png)

3.5> 注入消息通道  

--------------

*   由于 Spring Cloud Stream 会根据绑定接口中的 @Input 和 @Output 注解来创建消息通道实例，所以我们也可以通过直接注入的方式来使用消息通道对象。我们也可以通过 **@Resource("xxx")** 或者 **@Autowired@Qualifier("xxx")** 来指定要使用的注入实例。如下所示：
    
*   创建 MultiSinkSender 接口
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqZamNCsmMJz0IY1l4hdf24asGCd74uHl9LdMhibNLJsZWXQLqYicjohVQ/640?wx_fmt=png)

*   在 SinkReceiver 中绑定 MultiSinkSender 接口，并监听 “output1” 和“output2”这两个消息通道
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq2bO3OAA42ia19BuJH2YmggeAJ098Kicf5UXv6mcFa53qDgLqDIicEzwMg/640?wx_fmt=png)

*   通过指定 name，来确定使用哪个消息通道。在下面例子中，通过入参 mothod，确定使用 output1Sender 还是 output2Sender
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq4xtv80P0TbIGzl3ug1tgewWa67l96jwhCjQjITdWFR2K6ibLJJtHYfA/640?wx_fmt=png)

*   请求：http://localhost:8080/multiSendMsg?msg=aaa&method=1，我们使用的是 output1Sender 实例
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqicfoGP5W0mUFqmHAuAH1qWyjfaeDVj4icf9ec4wK0ic1ofdPLtIaVMzcg/640?wx_fmt=png)

*   请求：http://localhost:8080/multiSendMsg?msg=aaa&method=2，我们使用的是 output2Sender 实例
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqUKKjAuFee65SIR4anJ3vsnrySEMSpMMvbfGqOdPPxXroGXkZmxeyLQ/640?wx_fmt=png)

3.6> Spring Integration 原生支持  

-------------------------------

*   创建待绑定接口 IntegrationProcessor，并指定 topic 为 integration
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqA5KXdXwcmn75t5ibSprTP4ibypVrp5icsMjE8dMw7wymAVdEIEZXic1Zbw/640?wx_fmt=png)

*   编写接收方 SinkIntegrationReceiver，使用原生的 **@ServiceActivator** 注解替换为 @StreamListener，实现对 IntegrationProcessor.TOPIC 通道的监听处理
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqarsl0LEyCibrQP4xhTPOlCUqa2VMiaRBHpYq7V3GjEJRVDjeQdS2Q0Qw/640?wx_fmt=png)

*   编写发送方 SinkIntegrationSender，其中 **@InboundChannelAdapter** 注解定义了该方法是对 IntegrationProcessor.TOPIC 通道的输出绑定，同时使用 poller 参数将该方法设置为轮询执行，它会以 2 秒的频率向 IntegrationProcessor.TOPIC 通道输出当前时间
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqzvt835uTttR8vEBNibgafKnDI68He59UxjkHV5v1Zaic0uTrib6aC5c3w/640?wx_fmt=png)

*   启动服务，后台会有如下输出：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq0ic0logazV870cl3Gk126zB2oD6q1dJTvnlNbH4TCqZricb1L7Cfib0Ew/640?wx_fmt=png)

3.7> 消费组  

-----------

*   通常在生产环境中，我们的每个服务都不会以单节点的方式运行，当同一个服务启动多个实例的时候，这些实例会绑定到同一个消息通道的目标主题上。默认情况下，当生产者发出一条消息到绑定通道上，这条消息会产生多个副本被每个消费者实例接收和处理。但是，在有些业务场景下，我们希望**生产者产生的消息只被其中一个实例消费**，这个时候就需要为这些消费者设置消费组来实现这样的功能。
    

### 3.7.1> 生产者

*   生产者通过配置 **spring.cloud.stream.bindings.output.destination** 指定输入通道对应的主题名为 greetings，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqY5eXDHAzyrT6GBA2WsichxVqwOhCicj8L7pd8eApYgBic3xmhKpE0iaC9A/640?wx_fmt=png)

*   发送消息类 ConsumerGroupSender 如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqA93vOWufUeTDfIK0t3pq56AlpfYx8eXQxL4r3QSnicghP3vicRMMMH8A/640?wx_fmt=png)

### 3.7.2> 消费者  

*   消费者通过配置 **spring.cloud.stream.bindings.input.destination** 指定输入通道对应的主题名为 greetings；通过配置 **spring.cloud.stream.bindings.input.group** 指定消费组名称，启动两个服务，server.port 分别为 8081 和 8082，但是都配置相同的消费组名称，比如下面都配置消费组为 ConsumerGroup-A，，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq6hvSJ4LmsM9pWVkQqjOnwx55TT29WZp4xFQ8LEtQt6z3f3wGOoTAfw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqN5TujGkaiccKTSCNBOWfKYbadSMALo00S3oByltY5fNDYJZwYTmuY1w/640?wx_fmt=png)  

*   启动 1 个生产者和 2 个消费者，我们发现，生产者发送的消息只能由其中 1 个消费者（8081）进行消费，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDqEuf51vJhTjAm2age4IRIokq4WBbGe50lciaTxibceLicvzThdfX7OGUKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibMKHI7OU44WaZ3m2ZjVxDq0rtHS8KXmlpib9DZjPjRPhAkTKdjGv2vBu7L2xOYCGTU31p5evWFWMA/640?wx_fmt=png)