

    今天我们针对 SpringCloud 中服务之间的通讯方式全面的聊一聊。如下是本篇文章的大纲：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiavxpNgLrHo0ECGQiaqwXMrkIiaiba5kAJibZsNF5MHo8nxP8lK6XtYoOmXw/640?wx_fmt=png)

第一部分：RestTemplate  

====================

1> 服务通信概述
---------

*   服务通信产生的背景
    

*   由于随着系统的演变，从单体服务转变为微服务，那么将单体应用围绕业务进行了服务的拆分，拆分出来每一个服务独立运行、独立部署，且运行在自己计算机进程中。那么基于分布式服务直接的交互问题，就产生了服务通信的概念。
    

*   首先，我们先了解一下 OSI 的七层模型
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiatHgHiaa9VUlJeRlUvghXVmWoJjjkjgibkdUfnNNkY2Wlnm7UtS0qz9KQ/640?wx_fmt=png)

*   应用层
    
    OSI 参考模型中最靠近用户的一层，是为计算机用户提供应用接口，也为用户直接提供各种网络服务。我们常见应用层的网络服务协议有：HTTP，HTTPS，FTP，POP3、SMTP 等。
    
*   表示层
    
    表示层提供各种用于应用层数据的编码和转换功能, 确保一个系统的应用层发送的数据能被另一个系统的应用层识别。如果必要，该层可提供一种标准表示形式，用于将计算机内部的多种数据格式转换成通信中采用的标准表示形式。数据压缩和加密也是表示层可提供的转换功能之一。
    
*   会话层
    
    会话层就是负责建立、管理和终止表示层实体之间的通信会话。该层的通信由不同设备中的应用程序之间的服务请求和响应组成。
    
*   传输层
    
    传输层建立了主机端到端的链接，传输层的作用是为上层协议提供端到端的可靠和透明的数据传输服务，包括处理差错控制和流量控制等问题。该层向高层屏蔽了下层数据通信的细节，使高层用户看到的只是在两个传输实体间的一条主机到主机的、可由用户控制和设定的、可靠的数据通路。我们通常说的，RPC，TCP，UDP 就是在这一层。端口号既是这里的 “端”。
    
*   网络层
    
    本层通过 IP 寻址来建立两个节点之间的连接，为远端的传输层送来的分组，选择合适的路由和交换节点，正确无误地按照地址传送给目的端的运输层。就是通常说的 IP 层。这一层就是我们经常说的 IP 协议层。IP 协议是 Internet 的基础。
    
*   数据链路层
    
    将比特组合成字节，再将字节组合成帧，使用链路层地址 (以太网使用 MAC 地址) 来访问介质，并进行差错检测。数据链路层又分为 2 个子层：逻辑链路控制子层（LLC）和媒体访问控制子层（MAC）。MAC 子层处理 CSMA/CD 算法、数据出错校验、成帧等；LLC 子层定义了一些字段使上次协议能共享数据链路层。 在实际使用中，LLC 子层并非必需的。
    
*   物理层
    
    实际最终信号的传输是通过物理层实现的。通过物理介质传输比特流。规定了电平、速度和电缆针脚。常用设备有（各种物理设备）集线器、中继器、调制解调器、网线、双绞线、同轴电缆。这些都是物理层的传输介质。
    
*   在每一层都工作着不同的设备
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaN7LdOOJQ88nY9BY23uQTicRbOKIYErJyorUicUjMDoQZTAEYoKRrozAw/640?wx_fmt=png)

*   下面我们用一个生活中的例子来说明一下 OSI 七层模型的工作方式
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia619E2WC8BDcKic4seKOsCMbTQdq6ibpttOrNQ7ibEQyF8gk6XnjN24zUw/640?wx_fmt=png)

*   微服务通信的两套解决方案：HTTP 的 **Rest 方式**和 **RPC 方式**。
    
*   HTTP 是基于 OSI 第七层应用层，采用传输 Json 的方式进行通信。传输速度没有 RPC 快，但是可以与编程语言解耦。
    
*   RPC 是基于 OSI 第四层传输层，传输二进制数据流。传输速度比 HTTP 快，但是耦合度高，通信两端必须使用同一种编程语言。
    
*   所以，针对以上对比，SpringCloud 一直推荐采用基于 HTTP 的通信方式进行服务间的通信调用。
    
*   为了发起 Http 请求，Spring 框架提供 RestTemplate 对象，来负责发送 Http 请求。
    

2> 实例演示  

----------

*   首先：创建两个项目 ribbon-producer 和 ribbon-consumer，分别引入 springboot 的 web 依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaKIpbZ8O0H0hiaNGJ0RttiaDsQaWc3Libnwe8OUIicrPEQYQiaqkBI0sQ1gw/640?wx_fmt=png)

*   其次：配置两个项目的 application.yml 文件（producer：7000，consumer：7002  ）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaibEMI9mOngdIj7lsbRMnc2YxTpLRiadz4bVKZBM7JxJt3JticSdKh137w/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiayASfhzudDwEZicbOuJmUtQnMKYXVSibPSelh4DExFOtQtwn4V7ico6TbQ/640?wx_fmt=png)

*   第三：创建两个项目的 Controller
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiaictc0FoydPmUwKz0Xc2mF4uf22D1xeNpbHpxQYxr5zzOnpIVGpuCicSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EianpFy3FFS9pI6EQ34wxX6Mb6bkSdenXerOZSmpdvQAUY7h5smbzpFwg/640?wx_fmt=png)  

*   第四：也可以声明 RestTemplate 为 Bean
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaSse2R3MqQCV6dnHibia4tKDDpbopS1GNTusm05uVRS690DJvkUSPlmQg/640?wx_fmt=png)

*   第五：改造 ComsumerController，使用 Template 的 Bean
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiavfufjs0mHVfTTSF6CyVJykkBlVGdK0KpsiaLPj7Vibz8L7ScCGy3libUQ/640?wx_fmt=png)

3> 存在的问题  

-----------

*   由于在请求的 **url 中写死了 ip 和端口调用**，所以无法实现请求的负载均衡，并且微服务一般都会部署在云环境上，ip 和端口会随着上线、重启等操作产生变化，所以，写死 ip 和端口也不是明智的选择。那么针对这种情况，引出下面的 Spring Cloud 负载均衡组件——RIbbon。
    

第二部分：Ribbon  

==============

1> 概述
-----

*   Spring Cloud Ribbon 是一个基于 **HTTP** 和 **TCP** 的客户端**负载均衡**工具，它基于 Netflix Ribbon 实现，通过 Spring Cloud 的封装，可以让我们轻松地将面向服务的 Rest 模板请求自动转换成客户端负载均衡的服务调用。
    
*   Ribbon **只具有负载均衡的能力**，**并不具有发送请求的能力**。所以，需要配合服务通信组件，如：RestTemplate
    

2> 在微服务中的 Ribbon  

-------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia0UGEYyE3nHlDQkMS3qlFyawolH95KB72DeSJMStTs4ZWcJjiaQHaV4w/640?wx_fmt=png)【解释】

*   Ribbon 实现了从注册中心获取服务列表的能力。
    
*   然后通过获取到的服务列表，采用负载均衡算法（Ribbon 默认采用的是轮训方式），利用通信框架（RestTemplate 或 Feign 等）进行服务调用。
    

3> 使用 Ribbon+RestTemplate 实现服务间请求调用  

--------------------------------------

*   有三种方式可以实现基于服务名 + 负载均衡的服务间通信方式。分别是：
    

*   **DiscoveryClient + 自实现负载均衡 + RestTemplate**
    
*   **LoadBalanceClient + RestTemplate**
    
*   **@LoadBalanced + RestTemplate**
    

*   首先，启动 Nacos，两个项目都加入 Nacos 的依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiasUM8aN0kPsUEpHfhaB6wTe9CAoOh9AEOqBgmdL3aRf3jicwk8PRCGtg/640?wx_fmt=png)

*   由于 **Nacos 已经集成了 Ribbon**，所以，我们就不需要额外引入 Ribbon 了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia72dcm5jWa168bAZ8bz3PvPzianRAFgIVmDibDbxxF6mCmbcfJbHr5EGg/640?wx_fmt=png)

*   其次，在 consumer 和 producer 的 application.yml 配置文件中加入 Nacos 的配置信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiabXoibklBjX8AbXeEBHx00gNIcHPyrhHBD6UJcttVBlCwrZ7F6BUM6Bw/640?wx_fmt=png)

### 3.1> DiscoveryClient  

*   优点：客户获得指定服务名称下的所有服务实例。
    

*   缺点：**没有负载均衡**，需要通过获取服务列表，来编程实现负载均衡。
    

*   实现方式**（【注意】不能加 @LoadBalanced，否则请求失败）**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaMgA6obr5xia8DqUibQVnVic6zL8ZyKQcIrpNpIGKy84cww7lVFUUBIVNA/640?wx_fmt=png)

*   DiscoveryClient 的实现类是 **NacosDiscoveryClient**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaWr9eibW5a4jXyKch7vBICyIG9MRaMdvBhiakl6UbqSbdGOClHon5Z3Ng/640?wx_fmt=png)

### 3.2> LoadBalanceClient  

*   优点：**自带负载均衡算法**，不需要自己实现了。
    

*   缺点：使用时需要每**次先根据服务 id 获取一个负载均衡机器**，然后再通过 RestTemplate 调用服务。
    
*   实现方式**（【注意】不能加 @LoadBalanced，否则请求失败）**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiakyhZmNDccCWHWdydR5HJUDhCUVu2rmwxfUIF0axIOJO4wia59b7Jibqw/640?wx_fmt=png)

*   LoadBalanceClient 的实现类是 RibbonLoadBalancerClient
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia6pNMX7H12Xx8fcdZlVUKiao4uXgejt1ptWEibMstSgkoZVwCby9Mbwiaw/640?wx_fmt=png)

### 3.3> @LoadBalanced  

*   基于 LoadBalanceClient，提供了使用更方便的 **@LoadBalanced** 注解。
    
*   优点：使得 RestTemplate 实例具有了负载均衡的能力，不需要特殊调用 loadBalanceClient 实例的 choose 方法获得负载均衡计算出的实例了。使用方式，跟普通 RestTemplate 一样。非常简单。
    
*   缺点：依然还需要指定请求的 uri 和返回值类型。调用依然没有基于 rpc 方式简洁和直观。
    
*   修饰范围：**方法上**。
    
*   作用：让 restTemplate 具有 ribbon 负载均衡特性。使用更简单。
    
*   实现方式
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaWzvp7NRY3VLflpePzQr0EoicZjwpR8sriantgdXgeqoOwjIjtSDuhy3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaR0tVyHugQPHXl61PBeL6IwpdSH4tMBlh0DguiajHiclWicibKU49HR4ibfA/640?wx_fmt=png)  

4> Ribbon 实现负载均衡原理  

---------------------

### 4.1> 负载均衡相关源码解析

*   源码分析入口。因为 @LoadBalanced 注解是基于 LoadBalanceClient 的。所以。我们从红框这块代码分析入手：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiagJkoU7EhfkicEeEWssibTh4sDYZGlgeOcJ4ib4ISLic1u0uBxMhN5YibwNg/640?wx_fmt=png)

*   RibbonLoadBalancerClient.choose(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiatE3w3qib9Ax4RAWdqzE6RNU0phAkhE6y0SRB1ugaF0wNzxicia6k6kVmw/640?wx_fmt=png)

*   RibbonLoadBalancerClient.getServer(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaJKvsOwdRC9FPhH23YzutLk1RJonylnptRyyde8QeFlkn8XfHzTjrwQ/640?wx_fmt=png)

*   BaseLoadBalancer.chooseServer(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaqApktZiajT1b9iaWsApEXaMzYACrZnUhqa5EiabGgGvEPzkXCXcm8mmaQ/640?wx_fmt=png)

*   默认负载均衡算法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaTnqrNtZLcWfVFicODpAwKOib0f0glwOLxCEqFybpCY8bDwnFzxvmicpQw/640?wx_fmt=png)

### 4.2> 负载均衡策略  

*   负载均衡规则的类关系图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaCJDPIlnZXE3ic2RyKrKZsCImM78wVVnIibbzHhe8lePkUJ00jCDjI6gA/640?wx_fmt=png)

*   每个类具体负责功能
    

*   **RoundRobinRule  轮询策略**
    
    按顺序循环选择服务实例
    

*   **RandomRule  随机策略**
    
    随机选择服务实例
    

*   **AvaliabilityFilteringRule  可用过滤策略**
    
    会先过滤由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务。然后对剩余的服务列表按照轮询策略进行访问。
    

*   **WeightedResponseTimeRule  响应时间加权策略**（对于不同配置或负载的服务，请求偏向于打到负载小或性能高的服务上）
    
    根据平均响应的时间计算所有服务的权重，响应时间越快服务权重越大，也就越大概率被选中，刚启动时如果统计信息不足，则使用 RoundRobinRule 策略，等统计信息足够了，会切换到本策略。
    

*   **RetryRule  重试策略**（会使客户对于服务列表中不可用服务的调用无感，因为会 retry 到别的服务）
    
    先按照 RoundRobinRule 的策略获取服务，如果获取失败，则在制定时间内进行重试，获取可用服务。
    

*   **BestAvailableRule  最低并发策略**
    
    会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。
    

### 4.3> 自定义负载均衡策略  

*   在调用方，即：ribbon-consumer 工程的 application.yml 文件中，加入指定负载均衡策略的配置，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia4SxsNlhM2VW5lBmYGYN6W4oNl3Uryfl3Ov9OKLibyJkVFdQmOlibpg1w/640?wx_fmt=png)

【解释】  

*   ribbon-producer：指定的是服务提供方 spring.application.name 指定的值。
    
*   RandomRule：采用的是随机策略。即：随机请求某个服务实例。
    

*   进行测试，请求 http://localhost:7002/consumer/loadBalancerAnnotation，已经是随机访问服务了，不是默认的轮询方式了。结果如下所示：
    

```
provider hello! port:7001
provider hello! port:7001
provider hello! port:7000
provider hello! port:7001
provider hello! port:7001
provider hello! port:7000
provider hello! port:7000

```

第三部分：OpenFeign  

=================

1> 概述
-----

*   由于 Netflix 的 Feign 组件进入的维护阶段，所以 Spring Cloud 团队开始吸收开源的 Feign 组件的简化，封装开发了 OpenFeign 组件，而**使用上，OpenFeign 与 Feign 是一样的**。
    
*   OpenFeign 属于 Spring Cloud 自己的组件。
    
*   官网地址：https://cloud.spring.io/spring-cloud-openfeign/reference/html/
    
*   OpenFeign 是一个声明式的伪 Http 客户端（即：封装了 Http 请求，底层还是使用的 RestTemplate 发送的 http 请求），它使得编写 Http 客户端变得更简单。只需要创建一个接口并加入相关注解。它具有可插拔的注解特性（可以使用 SpringMVC 注解），可以使用 Feign 注解和 JAX-RS 注解。它将支持可插拔的编码器和解码器。**默认继承了 Ribbon 和实现了负载均衡。并且 SpringCloud 团队为 Feign 添加了 Spring MVC 注解的支持。**
    
*   到目前为止，如果 A 服务和 B 服务要实现 Http 方式的调用。有如下三种方式：
    

*   1> RestTemplate + 自实现的负载均衡策略
    
*   2> RestTemplate + Ribbon
    
*   3> OpenFeign
    

2> 实现 OpenFeign 的请求调用  

------------------------

### 2.1> 创建两个项目 openfeign-producer 和 openfeign-consumer

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaHPbUz74nFCxIXYkaYlLPf49UtvKqem2V0BRLkiaWO98hjGicmnkePLMA/640?wx_fmt=png)

### 2.2> 引入服务注册中心 Nacos 的依赖  

*   openfeign-producer 项目和 openfeign-consumer 项目的 pom 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaDgfpUoOpFT5aSIxWITuGlTMicPiaxf3DFoESNicoNumpGc6pYc2D3CPYA/640?wx_fmt=png)

### 2.3> 加入 Nacos 的配置信息  

*   openfeign-producer 项目的配置文件 application.yml
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia1WxmucWbj4YsD3OjeJMJOREMmB9m7Tawan2llcEC3s8vyQGNwMUQYA/640?wx_fmt=png)

*   openfeign-consumer 项目的配置文件 application.yml
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiav6l8F4WxvtmPpQYGOmZxUeue9B5ddAR9HQuI3sNX2ojypRNibjOCK1Q/640?wx_fmt=png)

### 2.4> 开启服务发现和 FeignClient  

*   openfeign-producer 项目的 OpenFeignProducerApplication.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaOpfrxUFADu5Bw1Faq1FjkiaGwXZ2tMibdXsy9e1u8D8OibUGcLIdVemwA/640?wx_fmt=png)

*   openfeign-consumer 项目的 OpenFeignConsumerApplication.java
    
    同上，略。
    

### 2.5> 创建 ProducerController 类

*   openfeign-producer 项目的 ProducerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaUaCdLicanLzeAu350ibK4icWmZJ4k6zK9WJtqwndFLfmhwC3PpArI58vA/640?wx_fmt=png)

### 2.6> 在 openfeign-consumer 项目中创建 OpenFeign 接口  

*   openfeign-consumer 项目的 ProviderService.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiatwrrgDwZEfuSyFHXJxGP8DYr6LF92a7sbakaibd5pG00ibRxbdWIla5w/640?wx_fmt=png)【注意】

*   ProviderService 接口中定义的方法，方法名称可以不与目标调用方法相同，但是返回值、请求路径必须相同。
    

### 2.7> 创建 ConsumerController 类

*   openfeign-consumer 项目的 ConsumerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaSSNHBn5nLsnf2812hKNYfE784ImYkKSy8pVbsyblFYkmnPOfxuhOmQ/640?wx_fmt=png)

### 2.8> 进行测试  

*   请求 http://localhost:8002/consumer/hello
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaPGvIEfSBuDSGy65LLSgMfF2qBJnefIQkGh9fMe7iceChibc6FnwOkf6A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia6kJ0ibiaksk9AxwxAGbetibZ3edwG8wBOubMevqIbARoxjT1FJVwK5aVA/640?wx_fmt=png)  

3> OpenFeign 的参数传递  

---------------------

### 3.1> 传递基本类型参数

#### 3.1.1> queryString 方式传递参数

*   请求参数中必须要加入 **@RequestParam**。否则如果传入多个参数的话，会报如下错误。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaRhZC3PO49rDPdftY0bicGkxUdBZ5l1hpIAvxn6q664Nf6kDsbiaYY92w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaPNqtzkAITibNBGXibic7qjZqiaxDiaibQnicXz6iayFuib4tW8DicddPGlAtSAvw/640?wx_fmt=png)  

*   openfeign-producer 项目的 ProducerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiav7sNFN1pYXHxrApk6BXB9Sz6x7MQnlbdmIfvrr41qGC5HicpjOJFZ6g/640?wx_fmt=png)

*   openfeign-consumer 项目的 ProviderService.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaNPlG7IfZWvbl6Bjkzqr0GApefd9CEy8bPGnLBtYZ5gHiarw8z2ouQlw/640?wx_fmt=png)

*   openfeign-consumer 项目的 ConsumerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia2ict9qSdUGbblofz2z2Hsh24wkknxGGhsemCkvCic23oMTvQAJB1VdNg/640?wx_fmt=png)

#### 3.1.2> 路径方式传递参数  

*   采用 **@PathVariable** 注解的方式
    
*   openfeign-producer 项目的 ProducerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiaa7ELbqxtJb9Ow7UHOcIjA8nNQzzgMiaHibyvAzd6m6qNJKfXcn3ibB9wg/640?wx_fmt=png)

*   openfeign-consumer 项目的 ProviderService.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaU6bfRiaxiacVHuZib4Pyyr3bLssP3zbhDhAq72rMicOYyw0tJFaUIWj6Xw/640?wx_fmt=png)

*   openfeign-consumer 项目的 ConsumerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiabL3x5pUJ0AB6ZmOcPC4EuictlPQXlIjSFnialMibP4ahhxRqlXMkNptTQ/640?wx_fmt=png)

### 3.2> 传递对象类型参数  

*   使用 **@PostMapping** 和 **@RequestBody** 的注解方式
    
*   openfeign-producer 和 openfeign-consumer 都增加 User 实体类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia6ib6JabKrkia2ZTBOfgCHYgfWkZ8T5GCfwTz1wo5AVR3obqgbLnkBiaWw/640?wx_fmt=png)

*   openfeign-producer 项目的 ProducerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaLIQMABJI5hPqb6zXh4BkjolHJevH3aYpTmkvxB9qzjxouGQM2O1DMA/640?wx_fmt=png)

*   openfeign-consumer 项目的 ProviderService.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiasMf2CibvfaIMNsVlyaJFhVSIf6V5cu2pepmibZk5amFSiabUNacU4vDDg/640?wx_fmt=png)

*   openfeign-consumer 项目的 ConsumerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaziaHFJoTMueAnmTsxfpcBJ14MKtFNwcB9N54zZVaYVhzLRibJxEiaL3hw/640?wx_fmt=png)

### 3.3> 传递数组类型参数  

*   使用 **@GetMapping** 和 **@RequestParam** 的注解方式传递数组类型
    
*   openfeign-producer 项目的 ProducerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia7quzViaQIwfXXicviaFRJpWVs63dAA7m4icm97rRke42GAkjFniajOS0mWQ/640?wx_fmt=png)

*   openfeign-consumer 项目的 ProviderService.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaUmwxgJFJNHP5r9nyK7LOV9fKkzl06wFEPr7AsITvIJjFsaK4g27PHw/640?wx_fmt=png)

*   openfeign-consumer 项目的 ConsumerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaRZP7tO9nLwiacdicKUUgnzaSAuvBAUbdKEgmQVtfWic3wELhb343fjNPw/640?wx_fmt=png)

### 3.4> 传递集合类型参数

*   与数组调用方式一样的。
    
*   openfeign-producer 项目的 ProducerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiagv2DiaRAVuSPldC6pLicp9iab0WEavreQTicnicGiaqTBQS1r1XygRhkfSaw/640?wx_fmt=png)

*   openfeign-consumer 项目的 ProviderService.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiat2G81BzbmB91h5gYNSpomJJ2icDQpe0eHkpqHpfnazVdSneZ2EGEcIg/640?wx_fmt=png)

*   openfeign-consumer 项目的 ConsumerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiajp7zK5SMr2ZbbO7ku2kibzevJLWOuJhroNcib3Fp2HkIIV7Pdww1BvSw/640?wx_fmt=png)

4> OpenFeign 的返回值处理  

----------------------

### 4.1> 使用 OpenFeign 调用服务，并返回对象

*   以对象类型返回即可
    

*   openfeign-producer 项目的 ProducerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiazRr9asox8ELvLYHpll4KxbQZuFYEMmhbXCicJUmbZzAVrRqsIMad5Eg/640?wx_fmt=png)

*   openfeign-consumer 项目的 ProviderService.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaQAr1aLqJHogXTd5kKI4e5zLibcaGiapdvBzU1HdicrviceBuuSVdbtROrQ/640?wx_fmt=png)

*   openfeign-consumer 项目的 ConsumerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaicdImDLxzB3w6zWcu2ziawlpm8OTL4K0s6SqfuZmktmlsHb0Vlq8VJ3g/640?wx_fmt=png)

5> OpenFeign 的细节  

-------------------

### 5.1> OpenFeign 默认超时处理

*   调用服务时，**默认是 1 秒超时**，如果服务提供方的服务 1 秒内没有响应，则会抛异常。
    
*   如何修改**指定服务**的超时配置；
    

*   【application.properties 文件】
    
    feign.client.config.[**服务提供者的服务名**].connectTimeout=5000
    
    feign.client.config.[**服务提供者的服务名**].readTimeout=5000
    

*   【application.yaml 文件】
    
    feign: 
    
      client:
    
        config:
    
          [服务提供者的服务名]:
    
            connectTimeout: 5000
    
            readTimeout: 5000
    

*   针对**所有微服务**，修改超时设置
    

*   【application.properties 文件】
    
    feign.client.config.**default**.connectTimeout=5000
    
    feign.client.config.**default**.readTimeout=5000
    

*   openfeign-producer 项目的 ProducerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiazLxg7mKkeUzzB7tLO2b9AdahUDJrOlz2PPaFs2811gLWKibWeFRM73Q/640?wx_fmt=png)

*   openfeign-consumer 项目的 ProviderService.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaNyMtBktAibuxJAvZPkKg5DYv2LTBey2lFAEkdkEticytLQ3URJfWDLOA/640?wx_fmt=png)

*   openfeign-consumer 项目的 ConsumerController.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaSG8d03QEicXNUJXmqOamZynYxRlG98tjplDmZWz5G8Fiby9zZ0mOSaKQ/640?wx_fmt=png)

*   测试请求 http://localhost:8002/consumer/timeoutDemo，结果如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eiad5DhQThZpW5hU2Fnhfmc3pjaA9WzDoR56sYM3MGCZGo8mg1382aW6g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaBTRerxrFtn9CutB7nHaQf7jDbtibZf0HmdR1z35oRHFuQAjrMTTlg8Q/640?wx_fmt=png)

*   修改 application.yml 文件，将默认超时时间修改为 5 秒
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2Eia6mtL3rYK5CK7LHT0FFB6g6BxY7GMJ9sNX6JibrGYAwbNOPpk1Km1sRQ/640?wx_fmt=png)

*   再次执行 http://localhost:8002/consumer/timeoutDemo，结果如下，没有报超时异常：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9RYu56eJCo5Lb2ek18q2EiaH8ZCeEHfyYibkJrer1LKM2q8tVd66ZCO9K6yD43fIGPflv3l6MlzIDw/640?wx_fmt=png)