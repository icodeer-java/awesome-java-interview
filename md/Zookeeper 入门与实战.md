> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247484932&idx=1&sn=b5c05fdf8b6eae437bf15b535e956ad9&chksm=e91144f9de66cdefe63b6cebf27311a868e935461c70249b372da97a4a1ed7f3d95e8ceacd91&scene=178&cur_album_id=2162186087471906816#rd)

    在分布式场景中，服务发现、分布式锁、节点状态监听等都会频繁的看到 Zookeeper 的身影，那么它同样在微服务中也扮演着重要的角色。那么，今天我们来慢慢的解开 Zookeeper 神秘的面纱。下面是本篇文章的目录  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEmm0yiaHuyf2Xe1376Kcdy7RW0nVRGxXVYGQxSrzxGI9RFqQZeEvPuvg/640?wx_fmt=png)

    那我们闲话少叙，开始今天的正篇吧~

一、Zookeeper 简介  

=================

1.1> 什么是 Zookeeper
------------------

*   Zookeeper 是一种分布式协调服务，用于管理大型主机。在分布式环境中协调和管理服务是一种复杂的过程，Zookeeper 通过简单的架构和 API 解决了这个问题。Zookeeper 运行开发人员专注于核心应用程序逻辑，而不必担心应用程序的分布式特性。
    

1.2> Zookeeper 的应用场景
--------------------

*   分布式协调组件
    
    在分布式系统中，需要有 Zookeeper 作为分布式协调组件，来协调分布式系统中的状态。
    
*   分布式锁
    
    Zookeeper 在实现分布式锁上面，可以做到强一致性。
    
*   无状态化的实现
    
    可以将一些具有无状态的信息存储到 Zookeeper 中，分布式服务直接去 Zookeeper 中获取相关信息。
    

二、搭建 Zookeeper  

=================

2.1> 安装 Zookeeper
-----------------

*   首先进入 Zookeeper 官网 https://zookeeper.apache.org/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE8icHwKZU1vR2cT3MLwsgicxg6II3Ez61m5icSVKAic00N4eYdtK0pSc0tw/640?wx_fmt=png)

*   下载最新版本 ZooKeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEVbhgZB0cyNwibaz4Qs5HqNNHcHN6GOQNMvSte5HbftqZWsuwc9YAxUw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQELUrZT77ibLO82GnomLfIwTE8tSPicJ18mbCLW136glLjeLSu3ficvRibkQ/640?wx_fmt=png)

*   解压到本地路径，进入 ZooKeeper 的 conf 目录下，复制 zoo_sample.cfg 配置文件，命名为 **zoo.cfg**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEkqGYWxV8mw5d4HfKdpDsiaG2REqPLIiaDyYZLeo0tdY6SSPqZY0dxmwg/640?wx_fmt=png)

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

2.2> 服务操作命令  

--------------

*   启动 Zookeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEajGqZF1bkWB5Lo64Nlf2l4JfMt2m3I0yhw9S9soVRgWHqCTJ9PtibXA/640?wx_fmt=png)

*   查看 Zookeeper 的运行状态
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQExyMlsP8vE9UibjRo6kN95anyoCcvbQ42s1W91kzxJ8pNXzlBhIROyfA/640?wx_fmt=png)

*   关闭 Zookeeper 服务器
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE0Rx8aZhv4BA7SpZkuyicHMBG2aPibgnmOsqfOmsWQH8I8cPib4G8cqc7Q/640?wx_fmt=png)

*   Zookeeper 客户端连接
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEzObJzYP6pc5ozFVKnM5RSIHZTkL9icHtAlHLhmhdBBB5agJTU8mlN4g/640?wx_fmt=png)

三、数据结构模型  

===========

3.1> Zookeeper 的数据存储结构
----------------------

*   zk 中的数据是保存在节点（znode）上的，多个 znode 之间够成一棵树形的目录结构。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEkDXK3dhck2xBTkDYemheOFfmRGmd5P0eaT3NINsVhlZrPvd2uY2RYw/640?wx_fmt=png)

【解释】

*   树是由节点所组成，Zookeeper 的数据存储也同样是基于节点，这种节点就做 **znode**；
    
*   但是，不同于树的节点，Znode 的引用方式是路径引用，类似于文件路径: /a/b
    
*   这种的层级结构，让每一个 Znode 节点拥有唯一的路径，就像命名空间一样，对不同信息做出清晰的隔离。
    

3.2> znode 的结构  

-----------------

*   znode 包含下面四个部分：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQERJzcX8UpjWcbvwQc9FCIvw0viaicIcUWmIDHFOUD0F0rz3NB9VlK8u1A/640?wx_fmt=png)

3.3> znode 的类型  

-----------------

### 3.3.1> 持久节点

*   创建出的节点，在会话结束后依然存在。保存数据。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEOwRY8w9GnqpM1oWb6YVEFzEuYhJK4uPmKyAMcRhsSqZiad1nd4P9VbA/640?wx_fmt=png)

### 3.3.2> 持久序号节点

*   兼具持久节点的特征
    
*   创建出的节点，根据先后顺序，会在节点之后带上一个数值，越后执行，这个数值越大。适合于分布式锁的应用场景（单调递增）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEncNH5eIpKylbD2nal4AibHSCH2tWWx8ibicgDyvawMhIFNapNFN8839rA/640?wx_fmt=png)

### 3.3.3> 临时节点

*   创建一个临时节点后，如果创建节点的会话结束，该节点会被自动的删除。通过这个特性，zk 可以实现服务的注册与发现。
    
*   临时节点通过心跳机制，告诉 zk 服务器自己还存活着。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQExEQCXm3YkR1OENyKicK6pxAIemKB0r1hzArppcdZzjz4aDLcpjXibUaA/640?wx_fmt=png)

*   操作示例：
    

*   步骤 1：我们开启一个【客户端 A】，创建一个临时节点
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQELyiae7H6b04IRPbh6bHnyyO7rYpzao4ibHpFZXCwaFyakzib5SsbawaaA/640?wx_fmt=png)

*   步骤 2：我们再开启一个【客户端 B】，用于观察 / test 目录下的所有节点
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEEtNjmib1yWelPButqLcDlynyVhSgzdy1M1FdzunQD4K32x4NLx9449A/640?wx_fmt=png)

*   步骤 3：关闭【客户端 A】，过几秒种后，我们再查询 / test 目录下的所有节点，已经没有 tempNode 的节点了。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEXPTicwu3pvSapicDdYdaQ4Gg1DFXuPnhbnGIuiaiaUdHHLakFTbWEZc9jw/640?wx_fmt=png)

### 3.3.4>  临时序号节点

*   相当于临时节点 + 序号节点
    
*   演示操作：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEuib4ln5UPgVVnhW2icDnmcunpp2bkRkGhmdFLCMNzM9oZmcSZLYiaxgnQ/640?wx_fmt=png)

### 3.3.5> Container 容器节点  

*   是在 3.5.3 版本新增的节点。当我们创建完 Container 容器节点后，如果该节点下没有任何子节点，那么 60 秒后，该容器节点就会被 zk 删除。
    
*   演示操作：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEa57oiaZ2kq4JDfZs2l1fGZMEuocMBw2lXRReogu4QfyTAK45cwHWK6g/640?wx_fmt=png)

### 3.3.6 > TTL 节点

*   可以指定节点的到期时间，到期后会被 zk 删除
    
    需要通过系统配置 **zookeeper.extendedTypesEnabled=true** 开启
    
*   官网介绍
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEMb3Z20qMRRGMkmfBh0xMApEWcqE0Fo3KyHveKiccZ8QtT5HcF4Mib3XA/640?wx_fmt=png)

*   操作演示：
    
    在 zoo.cfg 配置文件中，加入 extendedTypesEnabled=true。
    
*   重启 Zookeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEQ6y00SfibQf8VWqxe6QjogUTfmvr7gy6CSS94bL8duqkm4icI7Q3syicQ/640?wx_fmt=png)

3.4> 数据持久化机制
------------

*   Zookeeper 的数据是在内存中运行的，它提供了两种持久化机制：
    

*   **事务日志**
    
    Zookeeper 把执行的命令以日志的形式保存在 dataLogDir 指定的路径中的文件里，如果没有指定 dataLogDir，则按照 dataDir 指定的路径。
    
*   **数据快照**
    
    Zookeeper 会在一定的时间间隔内做一次内存数据的快照，把这段时间的内存数据保存到快照文件中。
    

*   Zookeeper 通过上面的两种形式的持久化，在恢复时先恢复快照文件中的数据到内存中，再用日志文件中的数据做增量恢复，这样可以加快恢复速度。
    
*   查看事务日志和数据快照文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEBp9HKeZTuynrKXbvibrmkdOosC9xKbWWA2RhRPwN6hMqV3A8VGdJFiaA/640?wx_fmt=png)

四、zkCli 操作演示  

===============

*   官方文档
    
    https://zookeeper.apache.org/doc/r3.7.0/zookeeperCLI.html
    

4.1> 创建节点
---------

*   创建节点并对其赋值
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEe5EyNr8TdkReLx8u11usWGkdiaopghZNmsxEj6LGib3rkj2Kqu2rAKrA/640?wx_fmt=png)

*   关于创建多种类型节点，请参照【3.3 znode 的类型】
    

4.2> 查询节点
---------

*   查询 test 节点下的子节点（但是只能查看下一级）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEgnu6cfJwSGfspiaicpryJjOkscCsnQNcoRKAusRlMEVicFFjS09G83amg/640?wx_fmt=png)

*   查询 test 节点下所有子节点
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEzyUzhkDbWBpy2iaYneRDAy603sqZ1E4L8qhZ1j5IqKvmgc9SWrgyKOg/640?wx_fmt=png)

*   查询节点的状态信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEoFoU0ShzI2euHBAqHeibBvNI5mF6EBn6H9Lth50f9ft1uNBUh7L6D6w/640?wx_fmt=png)

【解释】

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE0JSUxccs1s2tyggwSmSxIeaB9jIiczXR9eJ1ulbDQolhiayxBTd65mUw/640?wx_fmt=png)

4.3> 删除节点
---------

*   删除节点
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEoMA5jtDkP8RhPVVE4CMibVyQOnhcJCqRFUkcS6xzaJWIySYKnSjOBfA/640?wx_fmt=png)

【解释】

*   由于 / test/b 节点没有子节点，所以可以直接 delete 掉。
    
*   由于 / test/a 节点下面还有子节点 / test/a/b，那么是需要执行 deleteall 来执行删除操作的。
    

*   乐观锁删除         
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEnKstSIHFiar6ufKvZNKNB9Mbs4zx0u1zgrFwRpwuyAAvcTJAfax5CJQ/640?wx_fmt=png)

【解释】

    执行删除操作时，通过 **-v** 来指定待删除的 dataVersion 版本，如果符合，则可以执行删除操作；否则，不可以被删除。

4.4> 权限设置
---------

*   ACL 权限
    
    定义了什么样的用户能够操作这个节点，且能够进行怎样的操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEJjcMIvGBu62aMglGulKB8u91dt8A9ZGsKibiaKLPXcvtq6l0rian6B2uw/640?wx_fmt=png)

*   为当前会话添加权限账号和权限密码
    
    addauth digest **[用户名]**:**[密码]**
    
*   创建节点并设置权限
    
    create **[节点] [节点 value]** auth:**[用户名]**:**[密码]**:**[ACL 命令]**
    
*   操作演示
    

*   首先，我们开启 SessionA，创建用户权限
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEBtHgy6gVmZHkf0SKSG5rXziayI8PIzLFpuYqpaD8l2L4qMTfFtOwvSw/640?wx_fmt=png)

*   其次，针对权限用户 muse，创建 muse 的权限节点，并且可以正常读取 authNode 节点内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEH9AJw4E9fpqJ8uJG88T8PQEMbUiaBiaLNxHtibmQhMT5wRkaibUiasSFqgg/640?wx_fmt=png)

*   然后，我们开启 SessionB，执行读取 authNode 节点内容操作，发现没有读取的权限
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEYick7ZyeDwQXgRw5M8xTq2yZ2JUib3I3ZmAro8niaiaPHyibEfmiaDhF81vg/640?wx_fmt=png)

*   我们为 SessionB 添加 muse 的用户权限，再次执行读取 authNode 节点内容操作，发现可以读取
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEKJzMaEgaKSJlX0GacxO11JicohoR9RWMFC4Ftpb0tTsLKEAk5c3m07g/640?wx_fmt=png)

五、Curator  

============

*   官网介绍 https://curator.apache.org/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQErzkU5DLn3LEgQCt1REEgY5dyPjR7JHWvJPucMLS4ub3bFOz6SwrjcA/640?wx_fmt=png)

5.1> Curator 介绍
---------------

*   Curator 是 Netflix 公司开源的一套 Zookeeper 客户端框架，Curator 是对 Zookeeper 支持最好的客户端框架。Curator 封装了大部分 Zookeeper 的功能，比如：Leader 选举、分布式锁等等，极大的减轻了开发者在使用 Zookeeper 时的底层细节开发工作。
    

5.2> 引入 Curator 依赖
------------------

*   依赖如下所示
    

```
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.7.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>5.2.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>

```

*   查找相应版本
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEJftgNWEPwBicOmGUWNGKI72ibmFoNJgH3nRQmdgekttHibQrJ1h3PiaWMw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEjWUxx62rToibF52YL7U1vRpolXL5PfrRSUTFWSJRJfqQ1nnDlzLOhOA/640?wx_fmt=png)

5.3> 操作演示
---------

*   application.yml
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEWzgdOj0oREuunn38Iq1VEs8L0ibzgibSFk0Q4RQTYUnxDQh5Q6eubkwQ/640?wx_fmt=png)

*   ZKConfiguration.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEgWIj58KYSsaxwTx9zFPoOtzh8H9SZvRD7lYYhicCJeua60X2x5g7v3g/640?wx_fmt=png)

*   CuratorConfig.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQECJfpXpQqnvrVk4w0bLJNBoibrrB3mgibUpRTdViakACM7jXlKyvRCHJyw/640?wx_fmt=png)

*   ZookeeperDemoApplicationTests
    

```
@Slf4j
@SpringBootTest
class ZookeeperDemoApplicationTests {
    private final static String NODE_NAME = "/curator-node";
    private final static String EPHEMERAL_NODE_NAME = "/curator-ephemeral-node-";
    private final static String PARENT_NODE_NAME = "/animal/dog/whiteDog";
    private final static byte[] VALUE_BYTES = "muse".getBytes();
    private final static byte[] NEW_VALUE_BYTES = "muse-new".getBytes();
    @Resource
    private CuratorFramework curatorFramework;
    /**
     * 创建永久节点
     */
    @Test
    void createNode() throws Throwable {
        String path = curatorFramework.create().forPath(NODE_NAME, VALUE_BYTES);
        log.info("createNode success! path={}", path);
    }
    /**
     * 创建临时节点
     */
    @Test
    void createEphemeralNode() throws Throwable {
        String path = curatorFramework.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).
                forPath(EPHEMERAL_NODE_NAME, VALUE_BYTES);
        log.info("createEphemeralNode success! path={}", path);
        Thread.sleep(5000); // 线程睡眠5秒钟，这个时间可以查询到临时节点，方法执行完毕，临时节点就不存在了
    }
    /**
     * 如果父节点不存在，则连带着创建父类节点
     */
    @Test
    void createWithParent() throws Throwable {
        String path = curatorFramework.create().creatingParentsIfNeeded().forPath(PARENT_NODE_NAME, VALUE_BYTES);
        log.info("createWithParent()={}", path);
    }
    /**
     * 获取节点的值
     */
    @Test
    void getData() throws Throwable {
        byte[] valueByte = curatorFramework.getData().forPath(NODE_NAME);
        log.info("getData()={}", new String(valueByte));
    }
    /**
     * 修改节点的值
     */
    @Test
    void setData() throws Throwable {
        curatorFramework.setData().forPath(NODE_NAME, NEW_VALUE_BYTES);
    }
    /**
     * 删除节点
     */
    @Test
    void deleteData() throws Throwable {
        curatorFramework.delete().guaranteed().deletingChildrenIfNeeded().forPath(NODE_NAME);
    }
}

```

六、Zookeeper 实现分布式锁  

=====================

6.1> 锁的类型
---------

*   读锁
    
    并发的时候，多个线程都可以去执行读操作，彼此不会阻塞。
    
    加读锁成功的前提是：没有对其待访问的资源加写锁。
    
*   写锁
    
    并发时如果多个线程都要去获得写锁，那么只有一条、程可以获得写锁， 彼此会发生阻塞。
    
    加写锁成功的前提是：没有对其待访问的资源加任和锁（无论是写锁 or 读锁）。
    

6.2> 如何利用 zk 实现 Read 锁
----------------------

*   首先，在 / lock 路径下创建**临时序号节点 / lock/READ-**，该节点就代表将要获取的 Read 锁节点。
    
*   其次：获取 / lock 下的子节点，并按照临时节点的顺序号排序。
    

*   最后：检查此读锁之前**是否有 Write 锁**，若有先注册对该读锁的**前一个写锁**的监听，然后阻塞该读锁的获取。
    
*   若监听到该读锁前一个写锁已释放，则该 Read 锁打开阻塞。
    

6.3> 如何利用 zk 实现 Write 锁
-----------------------

*   首先，在 / lock 路径下创建**临时序号节点 / lock/WRITE-**，该节点就代表将要获取的 Write 锁节点。
    
*   其次：获取 / lock 下的子节点，并按照临时节点的顺序号排序。
    

*   最后：检查此写锁之前**是否有锁（无论是 Write 锁还是 Read 锁）**，若有，则先注册对该写锁**前一个锁**的监听，然后阻塞该写锁获取。
    
*   若监听到该写锁前一个锁已释放，则该 Write 锁打开阻塞。
    

6.4> 读写锁操作演示
------------

*   两个线程同时要获得 Read Lock【001 和 002 加 Read 锁成功】
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEx8NztKH85vSscKrN6mHjkxQgrj9U6vjjeaKe2Ulmr3Amllz8n5V5UQ/640?wx_fmt=png)

【解释】_其中，黄色星星表示获取锁成功！_

*   一个线程尝试获得 Write Lock，发现前一个节点存在锁，则监听前一个节点【003 加 Write 锁失败】
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEFUqe9HIRFDXkaZfMia0emMVwGZdSLVOLYOORx7P6UxmUnhBVCxPKtLg/640?wx_fmt=png)

*   一个线程尝试要获得 Read Lock，发现前面存在存在 Write Lock 的节点，则监听离它最近的 Write Lock 节点【004 加 Read 锁失败】
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEYRicewcFqDjv5Zmb1yKSGwaSlSkpgEDfJljlpIrl74Libc0lmMibm7ubg/640?wx_fmt=png)

*   一个线程尝试要获得 Read Lock，发现前面存在存在 Write Lock 的节点，则监听离它最近的 Write Lock 节点【005 加锁 READ 锁失败】
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEhRYlY1w9HbibqWwsMKtBE3unO2WEvibqn8293Onk1YLPDqVIWoA4uPMQ/640?wx_fmt=png)

*   001 和 002 的业务执行完毕，释放 Read 锁，那么由于 003 监听了 002，所以 002 释放之后，它就获得了 Write 锁【003 加 Write 锁成功】
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE36LlnheQ8VGOZ2laKLibd4x91I6pQ6EHPib5gJUMWg20jIe1R9zYBMaA/640?wx_fmt=png)

*   003 业务执行完毕，释放 Write 锁，由于 004 和 005 都监听了 003，所以 003 释放之后，就获得了 Read 锁【004 和 005 加 Read 锁成功】
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEicicZVTGyuvMsPic5wJGhhc1iaDvhgpeqWT1cf87ib1F8LhCH8dO9vRCwiag/640?wx_fmt=png)

6.5> 使用 Curator 提供的读写锁
----------------------

### 6.5.1> 实现代码

*   引入 Maven 依赖
    

```
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>5.2.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>

```

*   读锁代码实现
    

```
/**
 * 读锁
 */
@Test
void getReadLock() throws Throwable {
    InterProcessReadWriteLock rwlock = new InterProcessReadWriteLock(curatorFramework, "/lock");
    InterProcessMutex readLock = rwlock.readLock();
    log.info("等待获取Read锁...");
    readLock.acquire(); // 获取读锁
    log.info("等待获取Read锁成功！开始执行业务代码...");
    for (int i = 0; i< 100 ; i++) {
        Thread.sleep( 1000);
        log.info("worker_{} done!", i);
    }
    readLock.release(); // 释放锁
    log.info("业务代码执行完毕！释放Read锁成功！");
}

```

*   写锁代码实现
    

```
/**
 * 写锁
 */
@Test
void getWriteLock() throws Throwable {
    InterProcessReadWriteLock rwlock = new InterProcessReadWriteLock(curatorFramework, "/lock");
    InterProcessMutex writeLock = rwlock.writeLock();
    log.info("等待获取Write锁...");
    writeLock.acquire();
    log.info("等待获取Write锁成功！开始执行业务代码...");
    for (int i = 0; i< 100 ; i++) {
        Thread.sleep( 1000);
        log.info("worker_{} done!", i);
    }
    writeLock.release(); // 释放锁
    log.info("业务代码执行完毕！释放Write锁成功！");
}

```

### 6.5.2> 演示结果

*   我们运行的顺序是根据【线程 1】getReadLock() ——>【线程 2】getReadLock() ——>【线程 3】getWriteLock() ——>【线程 4】getReadLock() 依次开启的
    
*   执行结果如下所示：
    

*   步骤 1：那么首先我们可以【线程 1】和【线程 2】都是获取 Read 锁，所以没有被阻塞，都获得了 Read 锁。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEWxInRZfjBQYJ1ia0RUe7SiakmVibtILnM0bpCVbNFcDtZVvQgPLeuYUCQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEZ9R74GlP2LgMWFEwALbz3sPZELlKgeaThqS8mGhiauibzG6MrLrM5ptQ/640?wx_fmt=png)

*   步骤 2：由于【线程 3】要获取 Write 锁，所以被阻塞了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE919ba0T0ibScrFdcmwhF4evzGe8VUJticyVfuicyvjvyQMhKP6WOJsVgA/640?wx_fmt=png)

*   步骤 3：由于【线程 4】要获取 Read 锁，它排在了【线程 3】这个 Write 锁的后面，所以也被阻塞了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEkGRyvIia8ialDjxcicDaia5HAXucWbzcODHTcPlqpp3WgTvbtl3DGxRdOA/640?wx_fmt=png)

*   步骤 4：【线程 1】和【线程 2】都执行完毕，释放了 Read 锁
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEhm43YXg7njfW0UJGNhu5LSRuPNlvpJdjylxusMU9WaagaZHib9Dv3qQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQERrY9CeetWvuUKtYichZBvrqXDZyMTxetrGkh8IBty7WPI1skvoKGQmg/640?wx_fmt=png)

*   步骤 5：【线程 3】获得了 Write 锁，开始执行业务代码
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE5ZRC82WtyYPic7JaS4d49ibGIqjicFcRYE8ptVdIWWPYjcEpRRkVeOSAg/640?wx_fmt=png)

*   步骤 6：由于【线程 3】执行完毕，释放了 Write 锁，所以【线程 4】获得了 Read 锁，开始执行业务代码
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE8J7TmGI7aU8iaiaCWumuicrSKEo1tRPFOiajdqdbC6bfqt1C31mELyqXqw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEKngskibom4y0rwIs1my1XOBbn4Yn3agnOAM8yoVXwpWrKpQeVkrf5Yg/640?wx_fmt=png)

*   我们再来看一下 Zookeeper 中，针对这 4 个并发的线程，如何创建临时序号节点的
    

*   开启四个线程后，我们发现，创建了如下 5 个节点
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQExXeUl3mJgYic9icD4HYOBBIhs9Bzjak1oFe6trhjP3VYCrlIlHJTrUqQ/640?wx_fmt=png)

*   当线程 1 和线程 2 执行完毕解锁后，它们锁创建的临时序号节点也自动被删除掉了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE1NFz7picguyhL9YZs0HkZIdsHjZicHCXTwGpTWTQtysic3WJ3Zq7sIB3w/640?wx_fmt=png)

*   随着线程 3 执行完毕后，解锁的同时，临时序号节点也被删除了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEfILbhEx1UP7Tp1LrFd7lhM73kYmvewSlveVDIgOcMwAqLwS4hP1CjA/640?wx_fmt=png)

*   随着最后一个线程 4 执行完毕，/lock 节点下没有任何子节点了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEUQWT0Py2gez286gcvialDtfibs4dZL5ibxNoZXNTjxic0HwfQL69QPS9gw/640?wx_fmt=png)

*   并且，过几秒后，/lock 节点也自动删除掉了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEumHfFqhcfk7UQ9GFOSY8fDzlWc8zicxq7H6UmF9haGrsh2LS3Yg0FmQ/640?wx_fmt=png)

*   通过 Curator 读写锁组件创建的节点中，/lock 是持久节点，而它下面的子节点，则是临时序号节点。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQExc2CNQhXJayzibn54gc8XyJBYaBra1gD9ZzicFTibLnr23ELb6JNJycmQ/640?wx_fmt=png)

七、Watch 机制  

=============

7.1> 概述
-------

*   我们可以把 Watch 理解成是注册在特性 Znode 上的触发器。
    
*   当被 Watch 的这个 Znode 发生了变化（即：create、delete、setData 方法）时，将会触发 Znode 上注册的对应监听事件，请求 Watch 的客户端会接收到异步通知。
    

*   客户端使用了 NIO 通信模式监听服务的调用。
    

7.2> zkCli 客户端使用 watch  

-------------------------

### 7.2.1> 监听节点内容变化

*   监听**节点内容**的变化，我们可以使用 **get -w [节点]**
    
*   操作演示：
    

*   首先，开启 SessionA，创建一个 watchNode 节点，然后对这个节点进行 watch 监听
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEZdmvvrUMvUcJTgNy2oWLNFmjgWItArHVdic1ib32KeONZuuZibaSJ4fiaw/640?wx_fmt=png)

*   然后，开启 SessionB，对 watchNode 进行修改操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEaRAaSmpbqYyvzUTLLjmEtIp1uhlRt4pPHInGoMFL4DUGvhwBMK9ThA/640?wx_fmt=png)

*   这时，我们发现 SessionA 已经有监听反馈了。调用 get 指令，获得了 watchNode 节点最新的值 “muse”
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEzzGtayc2wBcvyBLhLjFJvEia3RH4MTYLSuAc5pQ0npOtVrx2g16altw/640?wx_fmt=png)

【注意】

    此时如果 SessionB 再次修改 watchNode 节点的话，SessionA 则无法接收到 Watch 监听事件反馈。如果我们还想继续去对这个节点进行监听的话，那么就不要通过 get /watchNode 的方式去获取 value，而是采用 **get -w /watchNode** 的方式即可。

### 7.2.2> 监听下一级子目录变化

*   监听节点目录的变化，我们可以使用 **ls -w [节点]**
    
*   操作演示：
    

*   首先，开启 SessionA，对节点 watchNode 进行监听
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEzqYRNSia2wy49HGsAuJT9g4puL0oeDNs6BPkiaYBpZxLoRgHBWdNAsSQ/640?wx_fmt=png)

*   再开启 SessionB，创建节点 / watchNode/sub1
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEyMwqL4qnOhTTafTWKf98cEGZEm8Blw31Q7IHAJNZUfqkMgSJmV4NIg/640?wx_fmt=png)

*   我们发现 SessionA 接收到了监听消息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEucwOyOficXUUtINCJ4HgwnNHHg8Bic0aPCrDu9G8vs8Ny0YPYb1WHncQ/640?wx_fmt=png)

*   此时在 SessionB 中，创建节点 / watchNode/sub1/sub2
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE8Z835iaW3JlHLwgjWgDicgCzYw6AHDz8T9TH4eyiaVOekhhHJUKwlyjvw/640?wx_fmt=png)

*   我们发现 SessionA 没有收到监听消息，_如果想要监听这种变化怎么办呢？_我们来看下小节的介绍。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQElOEiaHAMpDCLiaeDFOjhHVwiafjJ136MjwYhP6XdMN1ibA2BHib4akPUHJg/640?wx_fmt=png)

### 7.2.3> 监听所有级别子目录变化

*   监听所有级别子目录变化，我么可以使用 **ls -w -R [节点]**
    
*   操作演示：
    

*   首先，开启 SessionA，对节点 watchNode 进行监听
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEWPunUu4ibAiaymBNt1Aibja9AG4RibgJteDbdK65RxIfgj0QpibruW4nXYw/640?wx_fmt=png)

*   再开启 SessionB，创建节点 / watchNode/sub1/sub3
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEpbiaP1icRDJltc5Yt8xopvlGvg9vB9ZicuxGUdRWksoTQX9jKBtIS4oxg/640?wx_fmt=png)

*   我们发现 SessionA 接收到了监听消息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEelubhGfkhr37VicRUFdbs32hZFN2oicwKtgjJwH87Q88w63SMgJia4hrQ/640?wx_fmt=png)

*   此时我们通过 SessionB，再创建更深入的一层子集
    
    /watchNode/sub1/sub3/sub4，看看是否还能被监听到
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEt4LDNOSr3u3DHfZtGEuAmeKRx34D7hcjjNTT1ysQWBRwLiaXDUtDAEg/640?wx_fmt=png)

*   我们发现 SessionA 同样也是可以接收到监听消息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEiaRTibIbGFfZbN57o47UNvjOPbGfYgTqAsRdUG6qa5TXV4bzC7SMWDIg/640?wx_fmt=png)

7.3> 基于 Curator 使用 Watch  

---------------------------

*   我们可以通过调用 CuratorCache 和 CuratorCacheListener.builder() 的 for 方法，获取我们想要的相关监听内容。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQErf86erpDnBiaosShowNBbzvUGw9bhibPh0j9nJxSBSShaotP1EOH8GZA/640?wx_fmt=png)

*   我们来演示 forAll 和 forNodeCache，代码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE022xshiaB9RMDlGx8JaqCLr04ZrA49Dicab61Z8YPZcc5PvvRTX2OMBA/640?wx_fmt=png)

*   我们开启 zk 客户端，执行创建子节点和修改节点内容操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEG1Vvn6lY8h7OnqAd4zuETzTjx4Olvv4WHVIleyicxyQWO6Okib8OcbGw/640?wx_fmt=png)

*   我们来看程序对监听的抓取日志
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEmnQxV34EV0LRKfGamrsRsNP5rTkqicIus2iaFYh6b6IiakhclvHQODQSw/640?wx_fmt=png)

*   forAll 的详细日志如下所示，此处是修改节点内容的日志内容
    

```
2021-12-05 18:13:39.528  INFO 12720 --- [NotifyService-0] c.m.z.ZookeeperDemoApplicationTests      : 
-----forAll-----/curator-watch-node node is changed！
type="NODE_CHANGED" 
oldDate={
          "path": "/curator-watch-node",
          "stat": {
            "czxid": 23,
            "mzxid": 27,
            "ctime": 1638698845680,
            "mtime": 1638698968271,
            "version": 1,
            "cversion": 1,
            "aversion": 0,
            "ephemeralOwner": 0,
            "dataLength": 4,
            "numChildren": 1,
            "pzxid": 28
          },
          "data": [
            109,
            117,
            115,
            101
          ]
        }   
date={
      "path": "/curator-watch-node",
      "stat": {
        "czxid": 23,
        "mzxid": 37,
        "ctime": 1638698845680,
        "mtime": 1638699219525,
        "version": 2,
        "cversion": 1,
        "aversion": 0,
        "ephemeralOwner": 0,
        "dataLength": 4,
        "numChildren": 1,
        "pzxid": 28
      },
      "data": [
        109,
        117,
        115,
        101
      ]
    }

```

八、Zookeeper 集群搭建  

===================

8.1> Zookeeper 集群中的角色介绍
-----------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEjjueuQoJEC8uZtefRUPqlcxC5aURL8j330rQyQwYFu92QO2mCgGCpQ/640?wx_fmt=png)

8.2> 集群的搭建
----------

*   步骤一：创建四个数据存储目录
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEe75icsNuZ299DvNp5R6owN6BdIPHbRu9ZV323m0qryEQ5D4o2nyicrTQ/640?wx_fmt=png)

*   步骤二：分别在 zk1~zk4 目录中创建 myid 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEqBZWe5F9sojVpYvra5zI1J9ytAgyBGfCC4ZXh8kJVURL7Mu5qYPREg/640?wx_fmt=png)

*   步骤三：创建四个 zoo.cfg 配置文件，分别：
    
    zoo1.conf，zoo2.conf，zoo3.conf，zoo4.conf
    

```
# zookeeper数据存储目录及日志保存目录(zk1~zk4)
dataDir=/Users/muse/apache-zookeeper-3.7.0-bin/dataDir/zk1
# 对客户端提供的端口号(2181~2184)
clientPort=2181
# 2001为集群通信端口；3001为集群选举接口；observer表示不参与集群选举(此处zk中集群节点配置都一样)
server.1=127.0.0.1:2001:3001
server.2=127.0.0.1:2002:3002
server.3=127.0.0.1:2003:3003
server.4=127.0.0.1:2004:3004:observer

```

*   步骤四：启动四台 Zookeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEYO26DrlUz8r3yA8uowdS34GA53MyWCPxicEpP9r3ribsQptxWRZsnk2Q/640?wx_fmt=png)

*   步骤五：查看四台 Zookeeper 的角色
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEm0dib4ppZJ97QQWrSzfPT6Nhzic8vBn6G7fdVBfw95Hg2sP90ye7Tcrg/640?wx_fmt=png)

*   连接集群
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEJETwEt8VH5IKcMPybL9iavtTjiaczUA0uc3SEbsGoO4PF2rvrlJyeBUw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQElEKDqRl4xp3g5RMlibBtpib3niaS1aDxf1cPiacHA1yqzOZneOGtTfEzyg/640?wx_fmt=png)

九、Leader 选举与主从同步  

===================

*   Zookeeper 作为非常重要的分布式协调组件，需要进行集群部署，集群中会以一主多从的形式进行部署。为了保证数据的一致性，使用了 ZAB（Zookeeper Atomic Broadcase）协议，这个协议解决了 Zookeeper 的崩溃恢复和主从数据同步的问题。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE5EjlpX4y2KTmZiapEA4icQviaFyBMYSMseOmhCr0UpP2ortoAgFnib5Yiaw/640?wx_fmt=png)

*   ZAB 协议定义了如下四种节点状态
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEicgTd7mqOTJUricE15oxcicAHdTp3Mnc20LoPEhd9cgU8ouNGthtuquEQ/640?wx_fmt=png)

9.1> Zookeeper 的 Leader 选举
--------------------------

*   Leader 的选举过程如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQE74yzqiaj6nZPrBHiaziahuq26Mffwj1X8K8JoNbu145gq77KiaicmjPNaQQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEM4FIHtwkdGT8xRjHGjnVrPwiaRd8NIvYySDTaiaWbCPWuk6R05MVGJ9w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEtic4dYr2cssGoDoomeiajQn3TS1CGtwPy6pXU1JMkwiceojDcyywHWpRg/640?wx_fmt=png)

*   Leader 选举出来之后，会周期性不断的向 Follower 发送心跳 （ping 命令，没有内容的 socket）。
    
*   当 Leader 崩溃后，Follower 发现 socket 通道已经关闭，那么 Follower 就会从 Following 状态进入到 Looking 状态，然后重新开始进行 Leader 的选举，在 Leader 选举的这个过程中，zk 集群不能堆外提供服务。
    

9.2> Zookeeper 的数据同步
--------------------

*   如果客户端连接了 Leader 节点，则直接将数据写入到主节点；如果客户端连接到了 Follower 节点，那么 Follower 节点会将数据转发给 Leader 节点，Leader 节点再将注解写入到本节点中。
    

*   数据同步流程如下图所示（以客户端连接主节点为例）：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9F3K0wjKeDmCd2m6XBEiaQEzOtm7qvAR2Oud2A9mcefJ5ZjuQ8aQbIIwjONL0aHcdA6VFwfjkVetA/640?wx_fmt=png)

9.3> Zookeeper 中的 NIO 与 BIO 的应用
-------------------------------

*   NIO
    

*   用于被客户端连接的 2181 端口，使用的是 NIO 模式与客户端连接。
    
*   客户端开启 Watch 时，也使用 NIO，等待 Zookeeper 服务器的回调。
    

*   BIO
    

*   集群在选举时，多个节点之间的投票通道端口，使用 BIO 进行通信。