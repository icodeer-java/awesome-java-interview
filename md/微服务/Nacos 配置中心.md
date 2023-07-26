

    SpringCloud Alibaba 的 Nacos 其实包含了两部功能，一个是配置中心，另一个是服务发现。那么本篇文章，就先从它的配置中心功能来说起吧。还是老样子，目录如下所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhqPZ4oDBBBZoKsic4YUXxBBEVApo48rpxTXwHgfprrPUIriaexfFFGWdQ/640?wx_fmt=png)

一、什么是配置中心
---------

### 1.1> 什么是配置

*   应用程序在启动和运行的时候，往往需要读取一些配置信息，配置基本上伴随着应用程序的整个生命周期，比如：数据库连接参数、启动参数等；配置主要有以下几个特点：
    

*   **配置是独立于程序的****只读****变量**
    
    配置对于程序是只读的，程序通过读取配置来改变自己的行为，但是程序不应该去改变配置。
    
*   **配置伴随应用的****整个生命周期**
    
    配置贯穿于应用的整个生命周期，应用在启动时通过读取配置来初始化，在运行时根据配置调整行为。比如：启动时需要读取服务的端口号、系统在运行过程中需要读取定时策略执行定时任务等。
    
*   **配置可以有****多种加载方式**
    
    常见的有程序内部硬编码、配置文件、环境变量、启动参数、基于数据库等。
    
*   **配置****需要治理**
    
    同一份程序在不同的环境（如：开发、测试、生产）、不同的集群（如：不同的数据中心）经常需要有不同的配置，所以需要有完善的环境，集群配置管理。
    

### 1.2> 什么是配置中心

*   在微服务架构中，当系统从一个单体应用，被拆分成分布式系统上一个个服务节点后，配置文件也必须跟着迁移（分割），这样配置就分散了。不仅如此，分散中还伴随着冗余，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh5zVw7BnibhKjNarMJCMh8jMZNmA1ehAq0DicP7GuovkaCbl6E2MjHkhg/640?wx_fmt=png)

*   创建配置中心，将配置从各个应用中剥离出来，**对****配置进行统一管理**，应用自身不需要自己去管理配置。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhk9pIUGbQm7PvvhQbFrB4eW6iaD1ia568X72qdmNKmFvEtJWfCQLIKS3w/640?wx_fmt=png)

*   配置中心的服务流程如下所示：
    

*   1> 用户在配置中心更新配置信息。
    
*   2> 服务群及时得到配置更新通知，从配置中心获取最新配置。
    
    **总体来说，配置中心就是一种统一管理各种应用配置的基础服务组件。**
    

*   在系统架构中，配置中心是整个微服务基础架构体系中的一个组件，它的功能看上去并不起眼，无非就是配置的管理和存取，但它是整个微服务架构中不可或缺的一环。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhGx03e0IteyBaLibktOSmOCJ2veDyE0aMKfpEER97BhaxtLGzosCibqeA/640?wx_fmt=png)

*   总结一下，在传统巨型单体应用纷纷转向细粒度微服务架构的历史进程中，配置中心是微服务化不可缺少的一个系统组件，在这种背景下，中心化的配置服务——配置中心，应运而生；一个合格的配置中心需要满足如下特性：
    

*   配置项**容易读取和修改**
    
*   分布式环境下，应用配置的可**管理性**，即：提供远程管理配置的能力
    
*   支持对配置修改的**监视**以把控风险
    
*   可以查看配置修改的**历史记录**
    
*   不同部署环境下应用配置的**隔离性**
    

二、Nacos 简介
----------

### 2.1> 主流配置中心对比

*   第三方配置中心产品
    

**Disconf**：百度开源的配置管理中心，目前已经不维护了

**Spring Cloud Config**：Spring Cloud 生态组件，可以和 Spring Cloud 体系无缝整合。

**Apollo**：携程开源的配置管理中心，具备规范的权限、流程治理等特性。

**Nacos**：阿里开源的配置中心，也可以做 DNS 和 RPC 的服务发现。

*   由于 Disconf 不再维护，下面对比一下 Spring Cloud Config、Apollo 和 Nacos。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhme2ib9PQfLh10zX045scqOnleT2YoGibsFE4gygiaDNGiaib6waLA7FJZ1A/640?wx_fmt=png)

【说明】  

*   压测环境
    

*   Nacos 和 Apollo 使用同样的数据库（32 核心 128G）
    
*   部署 server 服务的机器（8 核心 16G）磁盘是 100GSSD
    

*   版本
    

*   Spring Cloud Config：2.0.0.M9 版本
    
*   Apollo：1.2.0 release 版本
    

*   Nacos：0.5 版本
    

*   Spring Cloud Config 依赖 Git，使用局限性较大。
    
*   性能方面 Nacos 读写性能最高，Apollo 次之，Spring Cloud Config 依赖 Git 场景不适合开放的大规模自动化运维 API。
    
*   功能方面
    
    Apollo 功能最为完善，Nacos 具有 Apollo 大部分配置管理功能，而 Spring Cloud Config 不带运维管理界面，需要自行开发。
    
*   Nacos 的其他优点
    
    Nacos 整合了注册中心、配置中心；部署和操作相比 Apollo 都要直观简单，因此它简化了架构复杂度，并减轻运维及部署工作。
    
*   综上所述，Nacos 的特点和优势还是比较明显的。那么，下面我们来一起进入 nacos 的世界。
    

### 2.2> Nacos 简介

*   https://nacos.io/zh-cn/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhYtaCn1sSLwWT10ic4ZtwRibyoYibwhUIyfUkyHk7upASVibkVnL8xrvAibw/640?wx_fmt=png)

*   Nacos 是阿里的一个开源产品，它是针对微服务架构中的**服务发现**、**配置管理**、**服务治理**的综合型解决方案。
    
*   官方介绍如下：
    

*   致力于帮助您发现、配置和管理微服务。
    
*   提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。
    
*   帮助您更敏捷和容易地构建、交付和管理微服务平台。
    
*   是构建以 “服务” 为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。
    

### 2.3> Nacos 特性

    Nacos 主要提供以下**四大功能**：

*   **服务发现与服务健康检查**
    
    Nacos 使服务更容易注册，并通过 **DNS** 或 **HTTP** 接口发现其他服务；
    
    Nacos 还提供服务的**实时健康检查**，以防止向不健康的主机或服务实例发送请求。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh21dcTkTmUnqdFtgWPq4D8fNSdl80oJhic4wia2bicZgPqfu9GhOujnia9g/640?wx_fmt=png)

*   **动态配置服务**
    
    动态配置服务运行在所有环境中以集中和动态的方式管理所有服务的配置。
    
    Nacos 消除了在更新配置时重新部署应用程序，这使配置的更改更加高效和灵活。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhZUHafUwBgftuQia6jqIjbQSBmXhzfrlAiaxNkvPuC107hQhLlaIJRJDA/640?wx_fmt=png)

*   **动态 DNS 服务**
    
    Nacos 提供基于 **DNS** **协议**的服务发现能力，旨在**支持异构语言**的服务发现；
    
    支持将注册在 Nacos 上的服务以**域名**的方式暴露端点，让三方应用方便查阅及发现。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhcGHoGZuZzh5hrE1wMSH8e8kaIhB8UlTGb7JYNjZMs1NEM4qOia5QPicA/640?wx_fmt=png)

*   **服务和元数据管理**
    
    Nacos 能让您从微服务平台建设的视角管理数据中心的所有服务及元数据（即：服务相关的一些配置和状态信息）
    
    包括：管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略。
    

三、Nacos 快速入门
------------

### 3.1> 安装 Nacos Server

*   官方文档
    

https://nacos.io/zh-cn/docs/quick-start.html

*   下载地址
    

https://github.com/alibaba/nacos/releases

*   解压
    

```
unzip nacos-server-2.0.2.zip
tar -xvf nacos-server-2.0.2.tar.gz
cd nacos/bin

```

*   启动服务器
    
    启动命令 (standalone 代表着单机模式运行，非集群模式)：
    
    **sh startup.sh -m standalone**
    
*   启动成功后界面如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhaaUBc5NxuvtRzr16BuXPBBWLT8GPNBTicOEhmltDPS6Cc3HJibGpocJw/640?wx_fmt=png)

*   访问 http://127.0.0.1:8848/nacos，打开如下 Nacos 控制台登录页面：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhhJVYcMuHeiaSGVIqFfp48ia2vqH1ibKNb0ByP5X7LDhJ4FAeXohuO3hSQ/640?wx_fmt=png)

默认用户名：**nacos**，默认密码：**nacos**

*   登录后界面
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhaMn2E9D6z9OGYuQrJS1MWc0icsNTdcrs9ChUxicFmf1IduHwawQSkQVA/640?wx_fmt=png)

### 3.2> OPEN API 配置管理测试  

*   **服务注册**
    

*   执行请求：
    

```
curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'

```

*   执行示例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhALtQVxmzHUB7GyjOxiayicKOicYNyXNEVkEpLpMsGd8IXWrSGrpz2Eq4w/640?wx_fmt=png)

*   Nacos【服务管理】-【服务列表】：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhYg6pRZKfauw7RGhTNyVgCey58sEW9NDAkNibickwDotQaom06oJywpicA/640?wx_fmt=png)

*   Nacos【服务列表】-【点击详情】：查询服务详情
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhXmqB16RfqmlWITmUMgQM6osK18pkV2SoKVzqHoUWSquyRWElEw6zJg/640?wx_fmt=png)

*   **服务发现**
    

*   执行请求：
    

```
curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'

```

*   执行示例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDht2AsqyTVlbN2XialltCv7AU1lrR8PMCQluPdhX2iaXuDdqyJicWy4hJnQ/640?wx_fmt=png)

*   **发布配置**
    

*   执行请求：
    

```
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=HelloWorld"

```

*   执行示例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhHcALR9L7UiaLA6omtjrLonb9jdKbeH3UGOv59f6ZaicWFibuHiaCxpQNeA/640?wx_fmt=png)

*   Nacos【配置管理】-【配置列表】：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhFvE574UP3X0ez6dpGd4aDeicbr2rQdqvh3uRhE6ibW0kPF4dY3fUYChQ/640?wx_fmt=png)

*   Nacos【配置列表】-【点击详情】：查询服务详情
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhSXLB95t3oVdcg3ZhXoTlZg7u3fRxrPUzicwdibWvG6JT1ngL4pS3Henw/640?wx_fmt=png)

*   **获取配置**
    

*   执行请求：
    

```
curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"

```

*   执行示例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh0Kl8hW7g5EVia0r3fYXU8TCPp3NX73cp7bLQAp2mr9wACibDhtzwEe1A/640?wx_fmt=png)

*   **关闭服务器**
    

*   执行请求：
    

**sh shutdown.sh** (Linux/Unix/Mac)

*   执行示例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhE0TwWqNzcs8PLaOyOY9OE9iboc6ecYZIJdNtlVohVibyjCTZLyAvGvOA/640?wx_fmt=png)

### 3.3> 配置外部 MySQL 连接  

*   若要 Nacos 使用外部 MySQL 存储配置数据，那么需要进行如下操作：
    

*   1>  安装 MySQL
    
*   2> 创建 nacos_config 数据库，并执行初始化脚本：
    
    ...**/nacos/conf/nacos-mysql.sql**
    
*   3> 修改...**/nacos/conf/****application.properties** 配置文件，增加支持 MySQL 数据源配置（目前只支持 MySQL），添加数据源 url、用户名、密码。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhNUw3rruQzFplaDRRDDwd8ia9ibhdVTctxC9oiaJjJIreZvhVDrFPaKS2g/640?wx_fmt=png)

*   4> 重启 Nacos
    

**sh shutdown.sh**

**sh startup.sh -m standalone**

*   5> 验证是否配置 ok
    
    创建命名空间
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhWDLlvkZUXxka2DOI1C4vibMiaTQIJ2vvKKq05zqzjTpwpiasGn7bRevUw/640?wx_fmt=png)        查询 tenant_info 表中数据

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh127WCNUibavNfdcaUiaFcOk8Qq0PMkmEVEkqmiaHcLmiaLqInNRLuWib0XQ/640?wx_fmt=png)

### 3.4> Nacos 配置入门  

#### 3.4.1> 发布配置

*   新增配置
    

【Nacos】——> 【配置管理】——>【配置列表】——>【右上角➕】

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh4I3nbhjZV22Ph7IF9pqHHr2cKoEVkOIl6jYTcUHiaelHnYmxicpeYaGw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhl8FcEwLy8RMQQlsIQfzq7OmRZddiaHkvUibzueuDj7TCwb3iaUlSNLtsg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhzfR5LKSV0MtibtdaxIIagtFZylotnnwFD85R8eEO8lAyibeGvg5vfeicA/640?wx_fmt=png)

#### 3.4.2> Nacos 客户端获取配置

*   创建 Maven 项目 nacos-demo
    
*   引入 maven 依赖
    

```
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>1.1.1</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>

```

*   编写 NacosDemo 类，获取 Nacos 配置
    

```
public class NacosDemo {
    private final static String SERVER_ADDR = "127.0.0.1:8848";
    private final static String DATE_ID_1 = "nacos_demo_1";
    private final static String DEFAULT_GROUP = "DEFAULT_GROUP";
    private final static Long TIMEOUT_MS = 1000L;
    public static void main(String[] args) {
        try {
            /** 查询默认命名空间（public）的配置信息 */
            ConfigService configService = NacosFactory.createConfigService(SERVER_ADDR);
            String configContent1 = configService.getConfig(DATE_ID_1, DEFAULT_GROUP, TIMEOUT_MS);
            System.out.println("configContent1 = " +  configContent1);
        } catch (NacosException e) {
            e.printStackTrace();
            System.out.println("error Msg=" + e.getErrMsg());
        }
    }
}

```

*   执行结果
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhRxBaN3HlHEu8Y31ricxs4EFu9s3HGbcbjkOU38fKCicFPMVgFxUib9pBQ/640?wx_fmt=png)

四、Nacos 配置管理基础应用  

-------------------

### 4.1> 数据模型

*   对于 Nacos 配置管理，通过 **Namespace**、**Group**、**Date ID** 能够定位到一个配置集。
    
*   Nacos 数据模型如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhJD0xogeeoHOGw4nZZxSmzvxK1Z5BToc8XWbMwvAhOHg0DJco8Rsc2w/640?wx_fmt=png)

*   **命名空间 (Namespace)**
    
    可用于进行不同环境的配置隔离。
    
    例如：
    

*   可以**隔离开发环境**——测试环境和生产环境，因为它们的配置可能各不相同;
    
*   可以**隔离不同的用户**——不同的开发人员使用同一个 nacos 管理各自的配置，可通过 namespace 隔离。不同的命名空间下，可以存在相同名称的配置分组 (Group) 或 配置集
    

*   **配置分组 (Group)**
    
    配置分组是对配置集进行分组。
    
    通过一个有意义的字符串（如 Buy 或 Trade ）来表示。
    
    不同的配置分组下可以有相同的配置集（Data ID）。
    
    当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 DEFAULT_GROUP 。
    
    配置分组的常见场景——**可用于区分不同的**项目**或**应用******。**
    
    例如：
    

*   学生管理系统的配置集可以定义一个 group 为：STUDENT_GROUP。
    

*   **配置集 (Data ID)**
    
    在系统中，一个配置文件通常就是一个配置集。
    
    一个配置集可以包含了系统的各种配置信息。
    
    例如：
    

*   一个配置集可能包含了数据源、线程池、日志级别等配置项。每个配置集都可以定义一个有意义的名称，就是配置集的 ID 即 DataID。
    

*   **配置项**
    
    配置集中包含的一个个配置内容就是配置项。
    
    它代表一个具体的可配置的参数与其值域，通常以 key=value 的形式存在。
    
    例如：
    

*   我们常配置系统的日志输出级别（logLevel=INFO|WARN|ERROR） 就是一个配置项。
    

*   最佳实践
    
    Nacos 抽象定义了 Namespace、Group、Data ID 的概念，具体这几个概念代表什么，取决于我们把它们看成什么。
    
    这里推荐给大家一种用法，如下所示：
    
    **Namespace**：代表不同环境，如开发、测试、生产环境。
    
    **Group**：代表某项目，如 XX 医疗项目、XX 电商项目
    
    **DataId**：每个项目下往往有若干个工程，每个配置集 (DataId) 是一个工程的主配置文件
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhI2wD52wI5xjJUXx8MvQNJW8ibM6atRFvR0cr4n6GjpAtgBV02XEZU3A/640?wx_fmt=png)
    

### 4.2> 命名空间管理

*   不同的 namespace 下，数据是彼此隔离不可见的。
    
*   namespace 的设计是 nacos 基于此做**多环境**和**多租户**数据（配置和服务）隔离的。
    
    **多环境**：online、preonline、test、dev
    
    **多租户**：多个用户共同使用**同一个 Nacos**
    

#### 4.2.1> namespace 隔离设计

*   以**环境**的不同来创建不同的 NameSpace
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhtdMtWeNECia8bHUKLofEXO7xUfsqic3VzdM9yWxTG2KzZohkp7ScNLfw/640?wx_fmt=png)

*   以**多租户**的不同来创建不同的 NameSpace
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhmG4UuTCicIPYiaeBv4NnkmU0srgnBY4m5LwIpxtG2TQWYybZmhgMQicEA/640?wx_fmt=png)

#### 4.2.2> 针对命名空间操作  

*   创建命名空间
    

【命名空间】——>【新建命名空间】

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhUlicichmUbria7peIDbj1ia1Q15XhYUdGeagMWrk7kITOchvUsXblvuaxQ/640?wx_fmt=png)

填写 “命名空间名” 和“描述”即可：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh65og0mwVJXPbpMb58OlurIM2oHCSqGYINWBO9ILmTp0tWZvJvAU6ibg/640?wx_fmt=png)

*   查看命名空间列表
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhf7eD6nMtnoDPfcxR1qrvZic9sJDZ1YYF9ABcGdjiaVUuUz9IUsgDcLicg/640?wx_fmt=png)

*   切换命名空间
    

【配置管理】——>【配置列表】——> 左上角有命名空间，点击可以切换

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhZTDILVXcic2Dn7guqe0y7pGldricjnSZ7uKwzoe3zJSIQwxzI44aQVCw/640?wx_fmt=png)

*   程序读取 muse 命名空间的配置
    

```
public class NacosDemo {
    private final static String SERVER_ADDR = "127.0.0.1:8848";
    private final static String MUSE_NAMESPACE = "e842ab49-1ca0-4a3a-b465-4959dd5b1dc4";
    private final static String DATE_ID_1 = "nacos_demo_1";
    private final static String DEFAULT_GROUP = "DEFAULT_GROUP";
    private final static Long TIMEOUT_MS = 1000L;
    public static void main(String[] args) {
        try {
            /** 查询默muse命名空间的配置信息 */
            Properties properties = new Properties();
            properties.put(PropertyKeyConst.SERVER_ADDR, SERVER_ADDR);
            properties.put(PropertyKeyConst.NAMESPACE, MUSE_NAMESPACE); // 指定命名空间为muse，切记：此处使用NameSpace ID
            configService = NacosFactory.createConfigService(properties);
            String configContent2 = configService.getConfig(DATE_ID_1, DEFAULT_GROUP, TIMEOUT_MS);
            System.out.println("configContent2 = " +  configContent2);
        } catch (NacosException e) {
            e.printStackTrace();
            System.out.println("error Msg=" + e.getErrMsg());
        }
    }
}

```

### 4.3> 配置管理

【配置管理】——>【配置列表】

#### 4.3.1> 添加、修改、删除

*   在 muse 命名空间中添加配置项
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhbYG1UzGoIYIMAkMYelCSjgkdgEjiagb7QMfeCkaeJcbKctWcRsfHVqg/640?wx_fmt=png)

*   在 muse 命名空间中修改配置项
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhU0viaBUYic2iccnoxMemqicicX59OBcPZojQoPx5zf3lB5vwUKgiaaJia6Wyg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhkDfv2W8RzpUoCfATia78RIWhDBgR6TsuZR4aqbeV6VWxdYh5vskbzjw/640?wx_fmt=png)

*   在 muse 命名空间中删除配置项
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhYFfQLYsslCpj6RkeOGiaK3XLavGwVkeEISqVpbNYs91JNXGoK09xuPQ/640?wx_fmt=png)

#### 4.3.2> 导出、导入、克隆

*   在 muse 命名空间中导出配置项
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhUtjYuY7ibdJad41HFItq5KEhTMjuC0ggErJ5VDriaFgA8KicHjGVI4ypQ/640?wx_fmt=png)

导出文件如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh3icgXkQmzAB0EfjOwewsnibJBlPBX7sFX4WgUIWMd1kSDibcIu5FO4NUg/640?wx_fmt=png)

针对文件打压缩包，才能导入

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhor98HvNO1Jygzl9ravibdNm7aUV1bibnV8B37SnPfvwgAa4a4DelCXAw/640?wx_fmt=png)

*   在 dev 命名空间导入配置项
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDheU70Un0UXz5v6TY8rTM8GR9l7OfcKvicwFFBb9a0MTiaLmqwOGBVrgwA/640?wx_fmt=png)

*   在 dev 命名空间克隆配置项
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhm11V5rXOREibARDnicjgh8pEGLdYlS6AqaP4o8jiaDahDXCpCicDVWDpKw/640?wx_fmt=png)

#### 4.3.3> 历史版本查看、回滚  

*   查看历史版本
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhsXE1Y6DKKflzjFqz1lwsv5HJ0An2EGCxKgZZ1ELlYIX83GWEtZwz6g/640?wx_fmt=png)

*   根据某个版本进行回滚操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhic52PuWABOVC3IQ9OObjnArtprxshQdiaECwB97nd1gI1vOeaNyTTGnA/640?wx_fmt=png)

#### 4.3.4> 配置监听

*   Nacos 提供配置订阅能力，同时提供客户端当前配置的 MD5 校验值，以便帮助用户更好的检查配置变更是否推送到 Client 端。
    
*   编写监控代码 NacosDemoListener.java
    

```
public class NacosDemoListener {
    private final static String SERVER_ADDR = "127.0.0.1:8848";
    private final static String DATE_ID_1 = "nacos.cfg.dataId";
    private final static String GROUP_1 = "test";
    public static void main(String[] args) throws Throwable {
        ConfigService configService = NacosFactory.createConfigService(SERVER_ADDR);
        configService.addListener(DATE_ID_1, GROUP_1, new Listener() {
            public Executor getExecutor() {
                return null;
            }
            // 修改namespace="public", data_id="nacos.cfg.dataId", group="test"的配置项之后，该方法会被调用
            public void receiveConfigInfo(String configInfo) {
                System.out.println("addListener = " +  configInfo);
            }
        });
        while (true) {
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

### 4.4> 登录管理

*   通过修改 users 表来配置登录用户
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhgxbgVRia81KtRcub2bzyhOSYyiaD2holibVBxiaicwdDCHTB9mfwEiauicXPA/640?wx_fmt=png)

#### 4.4.1> 添加、修改  

*   密码是被 Bcrypt 来加密的，采用 BCrypt 加密方法在每次生成密码时会加**随机盐**，所以生成密码每次可能不一样。
    
*   添加依赖
    

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>5.1.4.RELEASE</version>
</dependency>

```

*   所以我们编写一个生成 Bcrypt 密码的工具类——PasswordGenericUtils.java
    

```
public class PasswordGenericUtils {
    public static void main(String[] args) {
        System.out.println(new BCryptPasswordEncoder().encode("muse"));
    }
}

```

*   尝试新增一个用户名：muse，密码为：muse
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh7V8dLdGsiavZUxlr4YgoA8ibSk1N8MQBxQWjwF3DX2Do1UgezbQ29GwA/640?wx_fmt=png)

*   尝试使用 muse 进行登录
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhYrU2jvDvmH39q7TmTJjVlhXA1oHW5K3rIU5dB1qDiaXrbGEsQgHXYoA/640?wx_fmt=png)

*   显示登录成功
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhO4oQ7BM6BEq42CvqCCGvKR8VZqticBZHHSACBqPe2lQcT67xoxGbkCw/640?wx_fmt=png)

#### 4.4.2> 配置免登陆（1.2.0 版本被废弃）  

*   由于部分公司自己开发控制台，不希望被 nacos 的安全 filter 拦截。
    
*   因此 nacos 支持定制关闭登录功能找到配置文 ${nacoshome}/conf/application.properties ，把以下红框内容打开即可。
    

```
## spring security config
### turn off security
spring.security.enabled=false
management.security=false
security.basic.enabled=false
nacos.security.ignore.urls=/**
# nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**

```

*   重启 Nacos
    

**sh shutdown.sh**

**sh startup.sh -m standalone**

*   尝试访问 http://127.0.0.1:8848/nacos
    

五、Nacos 配置管理应用于分布式系统
--------------------

### 5.1> 服务架构的演变

略

### 5.2> 基于 Spring Cloud Alibaba 的配置读取

#### 5.2.1> 读取单个 Data ID 的配置信息

*   添加 Nacos 配置信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh3qAzCgTjz3NBib6XPiaws99Cib0bNSLb7yjsQia1ibytwRHrMyWcAuPn4dg/640?wx_fmt=png)

*   创建项目，引入 maven 依赖
    

*   nacos-demo 的父 pom 如下：
    

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.1.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Greenwich.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```

*   nacos-server 的 pom 如下：
    

```
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

```

*   在 resources 目录下创建 bootstrap.yml 文件
    
    因为 nacos 的配置要优先加载，所以放到 bootstrap.yml 中，而不是 application.yml 中
    

```
server:
  port: 9000
# 在Nacos Spring Cloud 中，dataId 的完整格式如下：${prefix}-${spring.profiles.active}.${file-extension}
spring:
  application:
    name: config1 # 配置名 
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848 # 配置中心地址
        file-extension: yaml # 后缀名
        namespace: 32210854-b7f7-40b9-8321-96bb7fb38f07 # 命名空间ID
        group: DEFAULT_GROUP # 指定要获取配置的组

```

*   创建测试类
    

```
@RefreshScope
@RestController
public class NacosServerController {
    @Resource
    private ConfigurableApplicationContext configurableApplicationContext;
    @Value("${student.name:null}")
    private String name;
    /**
     * 如果只是通过@Value获得的配置信息，不会随着Nacos的修改操作而获得最新配置信息。那么，实时获取最新配置信息有两种方式：
     * 方式1：添加@RefreshScope注解。
     * 方式2：通过getProperty的方式，获得最新的配置信息。
     *
     * @return
     */
    @GetMapping("/name")
    public String getName() {
        String name1 = configurableApplicationContext.getEnvironment().getProperty("student.name");
        return String.format(", name, name1);
    }
}

```

*   发送测试请求   http://localhost:9000/name
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhWA1tM9JqgfoHBwibdbH7OaN4qY6VJ70z4pIicCKEw0iaNKA9j11qN0p9w/640?wx_fmt=png)

#### 5.2.2> 读取多个 Data ID 的配置信息

##### **a> 使用 ext-config[n]，配置加载多个 dataId**

*   添加配置项
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhzeL2aaVqDvrWXbmwSBCR19HyHdF8iaC5VJfdgEvrJNBbLp80h2Boibnw/640?wx_fmt=png)

【注】

*   **config2.yml 的配置内容：**
    

config2:

 name: config2's name

*   **config3.yml 的配置内容：**
    

config3:

 name: config3's name

*   **config4.yml 的配置内容：**
    

config4:

 name: config4's name

*   修改在 resources 目录下的 bootstrap.yml 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhfG2Y5n9ZS7faVZeB1n7CHgGXrbP4yiaJvaP3Wtn3LSccpfM4tFdfricQ/640?wx_fmt=png)

*   编写测试类
    

```
@RefreshScope
@RestController
public class NacosServerController {
    @Value("${student.name:null}")
    private String name;
    @Value("${config2.name:null}")
    private String config2ame;
    @Value("${config3.name:null}")
    private String config3ame;
    @Value("${config4.name:null}")
    private String config4ame;
    @GetMapping("/allname")
    public String getAllName() {
        String name1 = configurableApplicationContext.getEnvironment().getProperty("student.name");
        return String.format(", name, config2ame, config3ame,
                config4ame);
    }
}

```

*   发送测试请求 http://localhost:9000/allname
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhGF8Z41gS3Y5ibMPibQdmV5KoNwAVbuEadJNTA3ARtspnLwibs0UDRVajw/640?wx_fmt=png)

*   把所有配置的内容都修改一下，测试结果，是不是只有配置了 refresh: true 才会更新。
    

*   **config1.yml** **的配置内容：**
    

student:

         name: muse4 new

         age: 20

         sex: 1

*   **config2.yml 的配置内容：**
    

config2:

 name: config2's name new

*   **config3.yml 的配置内容：**
    

config3:

 name: config3's name new

*   **config4.yml 的配置内容：**
    

config4:

 name: config4's name new

*   不用重启服务，发送测试请求 http://localhost:9000/allname
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh028L185eDRiaTrEH7tFkzQeZlRFdPArz1ick9F8FHP7UFp8hXfs6SGXQ/640?wx_fmt=png)

【注】结果所示，除去 name 配置的方式，只有配置了 refresh: true 才会更新。

##### **b> 使用 shared-dataids，配置加载多个 dataId**

*   shared-dataids 方式有局限性。即：只能加载默认组——DEFAULT_GROUP 的配置。
    
*   修改在 resources 目录下的 bootstrap.yml 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhFxv1gMOn2sIIqK4q0yiavfEQVravgpMwcCyGudk8eib5RvHSUZ6DeqwQ/640?wx_fmt=png)

*   重启服务，发送测试请求 http://localhost:9000/allname
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhklEOwDag0aabJc2qOtbibyyI4yj75VVjKn0p5tiaoTv864r64tFOE49g/640?wx_fmt=png)

【注】由于 config3 和 config4 都不是 DEFAULT_GROUP，所以加载不到配置，显示 null。

#### 5.2.3> 配置的优先级

*   Spring Cloud Alibaba Nacos Config 目前提供了三种拉取配置的能力：
    

*   1> 通过 spring.cloud.nacos.config.shard-dataids 的方式，拉取多个 Data ID 的配置。
    
*   2> 通过 spring.cloud.nacos.config.ext-config[n].data-id 的方式，拉取多个 Data ID 的配置。其中，n 的值越大，优先级越高。
    
*   3> 通过内部相关规则（应用名、扩展名）自动生成相关的 Data ID 配置
    

以上三种优先级关系是：3 > 2 > 1

六、Nacos 集群  

-------------

### 6.1> 部署 Nacos 集群

前提条件说明

*   最初使用 2.x 版本的 Nacos，发现启动三个 nacos 时，总会有一个节点出现 JVM 地址被占用的情况。然后采用了 1.3.1 版本启动没问题。所以，以下集群部署实验，皆为 Nacos 的 1.3.1 版本。
    
*   但是 1.x 版本默认不支持 MySQL8.x 版本，所以，需要我们手工将这个版本的 mysql 的 jdbc 的驱动 jar 包放到 plugins/mysql 下，这样就可以支持 8.x 版本的 MySQL 了。
    

*   拷贝 3 份 Nacos（请配置 3 个或 3 个以上节点，用于选举）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhqoPMZqicgoaqbeDZZh8ZpjEHIbsxic6n12FfeQOGAW8udVapItKs7QibQ/640?wx_fmt=png)

*   修改配置文件——application.properties
    
    由于是单机演示，所以需要修改 Nacos 的 conf 目录下 application.properties 中 server.port，防止端口冲突
    
    如果服务器有多个 ip，也需要指定具体的 ip 地址。如：nacos.inetutils.ip-address=127.0.0.1
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhIDewjF4WUibWwkqqNibQnWysuIuTDGVQiaMsy8fa8GGuaXoQQRUUXL9icA/640?wx_fmt=png)

*   配置外部 MySQL 连接——application.properties
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh35AoE2Ej7SUx8I11tTBc3ibV0s3xqR0HGlhUZ63ue76umgqBVM2Lh7Q/640?wx_fmt=png)

*   修改集群配置文件——cluster.conf
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhiaePGqt2uWjyG4qzz7QMafMWezAeRESqyw7LJiaqicdg9K5Bg6H5aVlDg/640?wx_fmt=png)

在所有 Nacos 目录下的 conf 目录下，有文件 cluster.conf.example，将其命名为 cluster.conf，并添加如下内容：

```
# ip:port
127.0.0.1:8860
127.0.0.1:8861
127.0.0.1:8862

```

*   启动三个节点 bin> startup.sh
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhiaeADTaVcWIibNpOtt5awsXvjyMEOfg6WSWicxWD2hOUI1oKYjogAHwqA/640?wx_fmt=png)

*   访问三个节点的控制台界面
    

*   http://127.0.0.1:8860/nacos/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhPBbJ09v81Eiammz3ImUd9W2CjgAn3rTCMNNOVhxR5VNibLbibiclwVIf4A/640?wx_fmt=png)

*   http://127.0.0.1:8861/nacos/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhicUxQ9icNzbZibqliaKjLXdjrGW44ODc6HBh1gAmpqnSUV7uojonph80Ig/640?wx_fmt=png)

*   http://127.0.0.1:8862/nacos/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhSibZ6RVompwNyX6ANiaVaE8ZaWibDusD3GRDwpG654CgKw3Jy3ex43atQ/640?wx_fmt=png)

*   查看主从信息（点击【节点元数据】）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhV1TCwpp8eK34hHWf7e66Hqsnud3ich0ibkSFPibbHA5PJE8lZCE8hEQhg/640?wx_fmt=png)

### 6.2> 配置客户端  

*   配置集群访问——bootstrap.yml
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhwn5LMDgibFuyHUicDY7mT4AZ7D9DIhZiayqBpmKSNHMic57yqhfcWVbOdw/640?wx_fmt=png)

*   启动服务，访问 http://localhost:9000/allname
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDh260AUGgUdApDunWYo73gyL4frfVMIGSBJObyd7UZoSROiaKF7mEzibtg/640?wx_fmt=png)

*   我们尝试把 Leader（port:8860）停掉，看集群发生什么情况
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhZIbmHRt5L7WsuyKnqMMXGkxficdoTyibuoLRbyxa3rmSIb83uR6hTgOQ/640?wx_fmt=png)

此时，8860 的 Nacos 控制台无法访问：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhQ3jMXQrGSVm0UCEwYfhp84Fmm12YzOLHYwIJ1hViaI4gGMibgacDVloA/640?wx_fmt=png)

访问 8861 的控制台，可以看到，8861 被选举为了 Leader，8860 节点状态为 DOWN。  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhJdXSZHnpk29uM8w5cDttDib9ou28fdouxuQya8ty2pV8dVQvwLD3aHw/640?wx_fmt=png)

*   不用重启服务，再次访问 http://localhost:9000/allname，如下所示，对服务没有任何影响。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhlxTvJ4J2uOwCWG56YNeKa2PZvqZosiabuyykHICDiaP3ohYGWicOLEb2w/640?wx_fmt=png)

### 6.3> 集群部署架构图  

*   以下内容，拷贝自官网介绍：https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html
    
*   开源的时候推荐用户把所有服务列表放到一个 vip 下面，然后挂到一个域名下面
    

*   **直连 ip 模式**
    

机器挂则需要修改 ip 才可以使用。eg：http://ip1:port/openAPI

*   **挂载 SLB 模式**
    

(内网 SLB，不可暴露到公网，以免带来安全风险)，直连 SLB 即可，下面挂 server 真实 ip，可读性不好。eg：http://SLB:port/openAPI

*   **域名 + SLB 模式**
    

(内网 SLB，不可暴露到公网，以免带来安全风险)，可读性好，而且换 ip 方便，推荐模式。eg：http://nacos.com:port/openAPI

*   架构图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhQs5tpcgMzPDHjVv9ZY6BVjPCewWUVZVkqqhOJ6bvsbHsnibGgzp4pKA/640?wx_fmt=png)

*   数据库集群（主备）配置——修改 application.properties 文件内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibaybBJJ1OKKfRmZFcLUNDhhDBZ48ugnJdficQ4Tswc1WnsdGf19yrYTkBKlnkkzrNJJM8Kspjxw2Q/640?wx_fmt=png)