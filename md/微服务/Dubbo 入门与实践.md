

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v30WVunQzzDUico7yr9qeHm1RDy3DJ4U6BRTH4Fz073KFSmtHd0GUqGQA/640?wx_fmt=png)

一、概述  

=======

1.1> 什么是 Dubbo
--------------

如官网描述：Dubbo 是一款高性能、轻量级的开源服务框架。

那么要想了解服务框架的含义，我么首先需要介绍一下，什么是 RPC。

1.2> 什么 RPC
-----------

RPC，即：Remote Procedure Call，是指远程过程调用，是一种进程间的通信方式；他是一种技术思想，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不是程序员显示编码这个远程调用的细节。即：开发者无论是调用本地的方法还是远程的方法，编写调用代码的方式基本相同。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v35sdyWgtibpYJLyibg9oV1bibdSqBEqNh0shQzYfo3icQVbARmTjtT3kEUg/640?wx_fmt=png)

*   了解了什么是 RPC 之后，我们就来去 Dubbo 的官网，看一看它都具有哪些特性和功能。官网地址：https://dubbo.apache.org/zh/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3VIrJyvyLOGNYfLX7iac8iaiakWVwhAW4uibQ4WpTOjeB9KOAud9cal0KMw/640?wx_fmt=png)

1.3> Dubbo 提供了六大核心功能  

-----------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3dBP8pDNJ39yiaygoIs6HGGqhUDMiahdwjm1ac3L9ydgbDYKNZO8vwRiaA/640?wx_fmt=png)

*   智能负载均衡
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3sZv15JGN1GplbdfiaKiaZIuMJW5zwH94b8mGTibBdsTiarwGuQAtWF4DIg/640?wx_fmt=png)

*   服务自动注册与发现
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3pghdiaP5RjmXyw45yoXulBZpuyL4fd2QCWDibuSkc1gRosujz8e5r8tQ/640?wx_fmt=png)

*   运行期流量调度
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3VFFGm2QiaZxriadS8p2wQn41iauyJWM9rZfiaESJYZCBCbMbxymIicwuFkw/640?wx_fmt=png)

*   名词解释
    

*   IDL（Interface Description Language）
    

IDL 是用来描述软件组件接口的一种计算机语言。IDL 通过一种中立的方式来描述接口，使得在不同平台上运行的对象和用不同语言编写的程序可以相互通信交流；比如，一个组件用 C++ 写成，另一个组件用 Java 写成。

关于 Protobuf 的语法请参见官方文档：https://developers.google.cn/protocol-buffers/docs/proto3

IDL 通常用于远程调用软件。在这种情况下，一般是由远程客户终端调用不同操作系统上的对象组件，并且这些对象组件可能是由不同计算机语言编写的。IDL 建立起了两个不同操作系统间通信的桥梁。

*   SPI（Service Provider Interface）
    

SPI 是一种服务发现机制。它通过在 ClassPath 路径下的 META-INF/services 文件夹查找文件，自动加载文件里所定义的类。

参考文章：https://www.jianshu.com/p/3a3edbcd8f24

*   Dubbo 的架构
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3se1OBQT7sV9BSJQU7icEriaeymILGN2atxbIAVjNom5HqqMIibLdIYhiaw/640?wx_fmt=png)

二、搭建 admin 客户端  

=================

*   Zookeeper 的安装（略）
    
*   进入 Dubbo 的 github 页面 https://github.com/apache/dubbo
    
*   下载 Dubbo Admin https://github.com/apache/dubbo-admin.git
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3KAbpr8IIXoegHLY9ich1cLzyF78N3SrxgtvR4XBIFmxkG0sibSOuu3Kw/640?wx_fmt=png)

*   安装流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3gBRtHxjVr9GZqljHiapmFOEemhicpQLzPMLhvIsK404v2Jmbv0OsNv3g/640?wx_fmt=png)

*   访问 Dubbo Admin，http://localhost:8080  用户名：root 密码：root
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3CZgbu55hvXKCWrpw2cticWcMChIMMoH8PcolFz1gF8hw6xg0saL6UMw/640?wx_fmt=png)

三、通过 API 方式使用 Dubbo  

======================

3.0> 创建待暴露接口
------------

*   项目结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3icFmrfwYibPib0adrFCZ951tgjIxkndIZgEFtbkHMeaJibAFicJ8dib8b5aQ/640?wx_fmt=png)

*   在 apidubbo 下，创建接口 GreetingsService.java
    

```
public interface GreetingsService {
    String sayHi(String name);
}

```

3.1> 编写 Provider
----------------

*   项目结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3gDc2jicFEmT3N7r2vQLzpgKhqALJcAUu5hziasu8qRicACxp1RI5l3YibQ/640?wx_fmt=png)

*   创建项目 api-dubbo-provider，并在 pom.xml 中加入 dubbo、zk 和 common-dubbo-api 的依赖
    

```
<dependencies>
    <!-- dubbo -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
    </dependency>
    <!-- Dubbo Zookeeper Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <type>pom</type>
    </dependency>
    <!-- Common Dubbo Api -->
    <dependency>
        <artifactId>common-dubbo-api</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>

```

*   创建接口 GreetingsService.java 和实现类 GreetingsServiceImpl.java
    

```
public class GreetingsServiceImpl implements GreetingsService {
    @Override
    public String sayHi(String name) {
        return "hi, " + name;
    }
}

```

*   创建 Provider 端执行主程序 ApiMainProvider.java
    

```
public class ApiMainProvider {
    private static String zookeeperHost = System.getProperty("zookeeper.address", "127.0.0.1");
    public static void main(String[] args) throws Exception {
        ServiceConfig<GreetingsService> service = new ServiceConfig<>();
        service.setApplication(new ApplicationConfig("api-dubbo-provider"));
        service.setRegistry(new RegistryConfig("zookeeper://" + zookeeperHost + ":2181"));
        service.setInterface(GreetingsService.class);
        service.setRef(new GreetingsServiceImpl());
        ProtocolConfig protocol = new ProtocolConfig("dubbo", 20881);
        service.setProtocol(protocol);
        service.export(); // 暴露provider服务（此处可以作为服务的启动的源码分析的入口）
        System.out.println("dubbo service started");
        new CountDownLatch(1).await();
    }
}

```

3.2> 编写 Consumer
----------------

*   项目结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v32Diap8uUfDS3YbU11PibemNHjvn1Gz5YN95niaQxxVlGZFNgldfRWKq1w/640?wx_fmt=png)

*   创建项目 api-dubbo-consumer，并在 pom.xml 中加入 dubbo 和 zk 的依赖
    

```
<dependencies>
    <!-- dubbo -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
    </dependency>
    <!-- Dubbo Zookeeper Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <type>pom</type>
    </dependency>
    <!-- Common Dubbo Api -->
    <dependency>
        <artifactId>common-dubbo-api</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>

```

*   创建 Consumer 端执行主程序 ApiMainConsumer.java
    

```
public class ApiMainConsumer {
    private static String zookeeperHost = System.getProperty("zookeeper.address", "127.0.0.1");
    public static void main(String[] args) throws Exception {
        ReferenceConfig<GreetingsService> reference = new ReferenceConfig<>();
        reference.setApplication(new ApplicationConfig("api-dubbo-consumer"));
        reference.setRegistry(new RegistryConfig("zookeeper://" + zookeeperHost + ":2181"));
        reference.setInterface(GreetingsService.class);
        GreetingsService service = reference.get(); // 获得代理服务（此处可以作为消费端启动的源码分析的入口）
        String message = service.sayHi("dubbo");
        System.out.println(message);
        new CountDownLatch(1).await();
    }
}

```

四、通过 Spring 与 Dubbo 整合  

=========================

https://dubbo.apache.org/zh/docsv2.7/user/quick-start/

4.0> 创建待暴露接口
------------

*   在 springdubbo 下，创建接口 GreetingsService.java
    

```
public interface GreetingsService {
    String sayHi(String name);
    String timeoutMethod();
}

```

4.1> 编写 Provider
----------------

*   项目结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3RXIeDfgcvmV1dpjgOvgzbAlxXxNwVKOqg5LGddt4MoA8iaO4jpC3Z1Q/640?wx_fmt=png)

*   创建项目 spring-dubbo-provider，并在 pom.xml 中加入 dubbo 和 zk 的依赖
    

```
<dependencies>
    <!-- Dubbo -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
    </dependency>
    <!-- Dubbo Zookeeper Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <type>pom</type>
    </dependency>
    <!-- Common Dubbo Api -->
    <dependency>
        <artifactId>common-dubbo-api</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>

```

*   创建接口 GreetingsService.java 和实现类 GreetingsServiceImpl.java
    

```
public class GreetingsServiceImpl implements GreetingsService {
    @Override
    public String sayHi(String name) {
        return "hi, " + name;
    }
}

```

*   创建 Spring 配置文件 provider.xml
    

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo
       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <!-- 提供方应用信息，用于计算依赖关系-->
    <dubbo:application  />
    <!-- 使用zk注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
        <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.muse.service.springdubbo.GreetingsService" ref="greetingsService" />
    <!-- 和本地bean一样实现服务 -->
    <bean />
    <!-- 用dubbo协议在20881端口暴露服务（注：如果采用默认的20880，则dubbo admin启动时，会报端口被占用异常） -->
    <dubbo:protocol  />
</beans>

```

*   创建 Provider 端执行主程序 MainProvider.java
    

```
public class MainProvider {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"provider.xml"});
        context.start();
        System.out.println("dubbo service started");
        System.in.read(); // 按任意键退出
    }
}

```

*   启动后，在 Admin 控制台查看 Provider 的注册服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3tvlRmqpvUTDxicfKmVuINwC42U7gHkuvWbiamBNNtyS8MHaH6xObxSmg/640?wx_fmt=png)

4.2> 编写 Consumer  

-------------------

*   项目结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3GvkIoN7Mjnq2cHdbyBibmvKGqXiaibUH9uVUWvr0eiawIX9r7BAEB8iaC8w/640?wx_fmt=png)

*   创建项目 spring-dubbo-consumer，并在 pom.xml 中加入 dubbo 和 zk 的依赖
    

```
<dependencies>
    <!-- Spring Boot Web Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
    </dependency>
    <!-- Dubbo Zookeeper Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <type>pom</type>
    </dependency>
    <!-- Common Dubbo Api -->
    <dependency>
        <artifactId>common-dubbo-api</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>

```

*   创建 Spring 配置文件 consumer.xml
    

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo
       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application   />
    <!-- 使用zk注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference interface="com.muse.service.springdubbo.GreetingsService" />
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:protocol />
</beans>

```

*   创建 Consumer 端执行主程序 MainProvider.java
    

```
public class MainConsumer {
    public static void main(String[] args) throws Exception {
       ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"consumer.xml"});
       context.start();
       GreetingsService greetingsService = (GreetingsService)context.getBean("greetingsService");// 获取远程服务代理
       String hello = greetingsService.sayHi("muse"); // 执行远程方法
       System.out.println(hello); // 显示调用结果
    }
}

```

五、Dubbo 的相关特性（dubbo:2.7.1）  

=============================

5.1> 属性配置优先级
------------

https://dubbo.apache.org/zh/docsv2.7/user/configuration/properties/

*   属性重写优先级从高到低：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3ibk917Sm5dSHFM86sLR2LcwuxoOI8kNOdSdpV3zcOnaAEgVmBd20WNQ/640?wx_fmt=png)

【解释】  

**JVM -D 参数**：当你部署或者启动应用时，它可以轻易地重写配置，比如，改变 dubbo 协议端口；

**XML**：XML 中的当前配置会重写 dubbo.properties 中的；

**Properties**：默认配置，仅仅作用于以上两者没有配置时；

*   如果在 classpath 下有超过一个 dubbo.properties 文件，比如：两个 jar 包都各自包含了 dubbo.properties，dubbo 将随机选择一个加载，并且打印错误日志。如果 id 没有在 protocol 中配置，将使用 name 作为默认属性。
    

*   配置 jvm 参数（**-Ddubbo.application.name=spring-dubbo-provider-jvm**）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3P01bzqjuXic95iadw9BC8icKa3YSDJB3HmlmLEB23bPndKGF5XGGyhicxQ/640?wx_fmt=png)

*   配置 provider.xml 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3HChuC9M0ZsPae900UNiau60Szd0ZgvuUy0o8JQYiaKNS1duU0oyTFOwg/640?wx_fmt=png)

*   配置 dubbo.properties 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3Th4Qu6mmP5uHlnAE2vRsrFYYd8OXXqHYhlzfTI4JbqdU4JErGhyKhQ/640?wx_fmt=png)

*   首先：启动项目可以看到，jvm 首先生效了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3AThpGMNNw3BbuWyomjCLrE0J63zFU0Zy8xO7MCtuQdwibRVhp5KicRWw/640?wx_fmt=png)

*   其次，删除掉 jvm 参数，再次启动项目，我们发现，xml 配置文件生效了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3r9tKH5VE6gxJgexkianXjibGWpyG485MG0wrw00z9rGXXPKWHduT2caQ/640?wx_fmt=png)

*   最后，注释掉 xml 中应用名称的配置，再次启动项目，我们发现，dubbo.properties 生效了（如果配置文件名称为 application.properties 会报错）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3fmxIGpcN1v8ia5n86sQZm40qu6P2YN7ibj35IpZX8hmzDLj9PKQV1jpg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3QU2urAibBHP5VIPPMgLibB9EmLTOKhZucLGuBN1WtlaHPDUR9qbrTIAQ/640?wx_fmt=png)

5.2> 启动时检查  

-------------

*   Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认 check="true"。可以通过 check="false" 关闭检查，比如，测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动。另外，如果你的 Spring 容器是懒加载的，或者通过 API 编程延迟引用服务，请关闭 check，否则服务临时不可用时，会抛出异常，拿到 null 引用，如果 check="false"，总是会返回引用，当服务恢复时，能自动连上。
    
*   配置方式
    

```
dubbo.reference.com.foo.BarService.check=false  # 关闭某个服务的启动时检查 (没有提供者时报错)：
dubbo.consumer.check=false  # 关闭所有服务的启动时检查 (没有提供者时报错)：
dubbo.registry.check=false # 关闭注册中心启动时检查 (注册订阅失败时报错)：

```

*   如果不启动 Provider，当启动 Consumer 时，回因为找不到 Provider 接口而报错：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3bPmSaiczkPmLPGp1l3f2lmx605UDWkicRsEic20dumAyylleEQ69dcQBQ/640?wx_fmt=png)

### 5.2.1> 针对具体服务进行配置  

*   在需要配置的服务配置中，添加 check=false，再次启动，就不会报错了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3p7zW1hR56dr89D0B7NorsLiba2qx8p7fIbiaNeLWQwvauicO5khB4fCUQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3H2TzEXp0p0scQnY1VkSnq0VbApiaRZ6qZTMpHHtsxaCZrGMplfTHklA/640?wx_fmt=png)

### 5.2.2> 针对所有服务进行配置

*   在配置文件中添加 <dubbo:consumer check="false"/>，再次启动，就不会报错了  
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3U0KyVGrA38icSc2sB6mJAWVicQMXvQnKa5BNKtg3PFKVYlss7srfcj3w/640?wx_fmt=png)

5.3> 请求超时设置  

--------------

*   我们在 Provider 端提供一个超时方法，配置 Sleep 时间为 2 秒钟。启动 Provider
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v33fO3iaIPr0OicC8n8OBKuiacsVC0niaWicKCuniaNANvQmYjiacSFkiaW2qLpg/640?wx_fmt=png)

*   我们在 Consumer 端调用 timeoutMethod() 方法，就会以为超时报错（默认请求超时时间为 1000 毫秒）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3Me9juDDBgxLjvlqXT4ibXRKXSgw4BnXYsNrQ1hvb6Ep38tA0eJJ4ALA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3zNVfGbSR8NGHV230IYBo9j6CLBO0U74dbqd4PrY9Q0TXq5EiakjzLoA/640?wx_fmt=png)

### 5.3.1> 针对全局设置超时时间

*   通过配置 <dubbo:consumer timeout="3000"/>，针对所有的 Provider 都设置超时请求 3000 毫秒
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3ibiaoQtWibw4eppicv9g5ht9NcEpDOCYQOiaTPJZdNUIm6cSt0IS8SicRu5g/640?wx_fmt=png)

### 5.3.2> 针对接口设置超时时间  

*   我们也可以针对该接口，通过在指定的接口配置中设置 timeout="3000"，这样，接口就不会报超时了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3tmn9b5VXdwZQEDgFgcia72twaTVHfiaLOrqph86xU59BfFMphIS8GCLA/640?wx_fmt=png)

### 5.3.3> 针对方法设置超时时间  

*   如果我们还想要更加精细粒度的控制，我们可以在方法上面设置超时时间，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3m1J7YpA2QDGyd6LicH7k5fWWiayqIibTdV6icXC61jvC1DuOwqgcOhibibgw/640?wx_fmt=png)

### 5.3.4> 配置的覆盖关系  

*   配置覆盖规则
    

*   **方法级 > 接口级 > 全局配置**
    
*   如果级别一样，则**消费方 > 提供方**
    

*   其中，服务提供方配置，通过 URL 经由注册中心传递给消费方。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3DIfkicSc1J4bKbKOTjH8rKkO6rPv0jkMicdQepVsk6oXPejxgdzXKOUw/640?wx_fmt=png)

5.4> 重试次数  

------------

*   当某个接口是幂等接口，并且当接口出现异常的时候，需要我们进行重试请求的话，那么我们可以配置重试次数。【注意】这里配置的是重试次数为 2 次，并不包含正常的那一次请求
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v34100vmLRUylTFA3iakdCXku90eUZmiamdVI1jXRDYMKJjTy9wFPE984Q/640?wx_fmt=png)

*   查看 Provider，后台输出了 3 次，其中一次是普通的请求，另外两次，是重试请求
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3blSiaEaFroV95I5EyxQyd7X0aGfunGBGT47waN7TNLYkwXKe09vtR5w/640?wx_fmt=png)

*   那么，当 Provider 服务部署集群的时候，重试也会尝试请求到不同的服务器上。我们开启两个 Provider 服务，Consumer 再次请求 timeoutMethod 方法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3ibBX0Ws05DfbPBEXc0njjpEyqEdicIFzUXMOpr4feFAXSyrYBDm1KpYg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3ia1ZZIKCsnSYtZfqkXQ2jyUlobxicAia8fqlXmy1dOn7YSwvNNEMCT2aA/640?wx_fmt=png)

5.5> 多版本控制  

-------------

*   在 Dubbo 中为同一个服务配置多个版本。当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。
    
*   可以按照以下的步骤进行版本迁移：
    

*   在低压力时间段，先升级一半提供者为新版本
    
*   再将所有消费者升级为新版本
    
*   然后将剩下的一半提供者升级为新版本
    

### 5.5.1> Provider 端

*   编写 MuliVersionService.java 接口和两个版本的实现类 MuliVersionServiceImpl.java 和 MuliVersionServiceV2Impl.java
    

```
public interface MuliVersionService {
    String version();
}
public class MuliVersionServiceImpl implements MuliVersionService {
    @Override
    public String version() {
        return "GreetingsService v1";
    }
}
public class MuliVersionServiceV2Impl implements MuliVersionService {
    @Override
    public String version() {
        return "GreetingsService v2";
    }
}

```

*   配置 provider.xml
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3rGICvRfh1pkicRicfXoA1iaCgSdQuRGibzuAh0ED6u8vnrjgBYt4kibvyOA/640?wx_fmt=png)

### 5.5.2> Consumer 端  

*   在 consumer.xml 配置文件中设置 Consumer 请求的是 0.0.1 版本
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3892JxVuVsVBCubX6faW3Q3pBzlV2dDjw5bpzsicEz5ibrAHhCyicmQdAA/640?wx_fmt=png)

*   在 consumer.xml 配置文件中设置 Consumer 请求的是 0.0.2 版本
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3UFfHCPH4QJVclUnvdBzwPMEviaV2eg9O7Gj4drjf8f4keFI1gs0umMw/640?wx_fmt=png)

*   如果不需要区分版本，可以配置 version="*"
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3LiaUz5ldR7UqbFhjwhJJQAiczgq1sicFxDDYFH3vGjJ6B4xsrLibyWPdVQ/640?wx_fmt=png)

5.6> 本地存根  

------------

*   在 Dubbo 中利用本地存根在客户端执行部分逻辑。
    
*   远程服务后，客户端通常只剩下接口，而实现全在服务器端，但提供方有些时候想在客户端也执行部分逻辑，比如：做 ThreadLocal 缓存，提前验证参数，调用失败后伪造容错数据等等，此时就需要在 API 中带上 Stub，客户端生成 Proxy 实例，会把 Proxy 通过构造函数传给 Stub（Stub 必须有可传入 Proxy 的构造函数），然后把 Stub 暴露给用户，Stub 可以决定要不要去调 Proxy。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3h32DeaSZZbwyLgrUm6ptmeCJoZiaSaNFiczIgECCVavwsaBadNZHqibiaw/640?wx_fmt=png)

### 5.6.1> Consumer 端  

*   实现 GreetingsService 接口，创建 GreetingsServiceStub.java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3cWdlnh8g93fArichgN3e5K9Rv61dVHzn1ibyVUnXppcPibpaZHHHQFCCA/640?wx_fmt=png)

*   修改 consumer.xml 配置文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3ojHCOiaiao28cz9wVBGMO4AOiaibYuXY7zNgP9WWNjkyax50YRwUiah8pgQ/640?wx_fmt=png)

*   调用 sayHi 方法，发现已经是先由 Stub 处理后，再调用了代理接口
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3tb1ViaqzHMejNmM6ibkQp66LPnqYhNs0b2nlwvibGQO6N1QhFIbBVBgWA/640?wx_fmt=png)

六、通过 SpringBoot 与 Dubbo 整合 https://github.com/apache/dubbo  

=============================================================

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3KQjzwg00Au9npWWMIneZJswjjcdb5Owib2Z6zhqSgY2n2ua5kYRgVIg/640?wx_fmt=png)

6.0> 创建待暴露接口  

---------------

*   在 springbootdubbo 下，创建接口 DemoService.java
    

```
public interface DemoService {
    String sayHello(String name);
}

```

6.1> 编写 Provider
----------------

*   项目结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3OicLibTUoh1Wyf2rdEYGglnZhoUqBsDrnrjpCamg4icn2bwAjZU8ewEGg/640?wx_fmt=png)

*   在 pom.xml 中添加 dubbo、zk 和 web 的依赖
    

```
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Dubbo-->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
    </dependency>
    <!-- Dubbo Spring Boot Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
    </dependency>
    <!-- Dubbo Zookeeper Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <type>pom</type>
    </dependency>
    <!-- lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <!-- configuration-processor -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    <!-- Common Dubbo Api -->
    <dependency>
        <artifactId>common-dubbo-api</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>

```

*   编写配置文件 application.properties（可与 provider.xml 中的配置一一对照编写）
    

```
server.port=8083
# Spring boot application
spring.application.name=springboot-dubbo-provider
# Base packages to scan Dubbo Component: @org.apache.dubbo.config.annotation.Service
dubbo.scan.base-packages=com.muse.service
# Dubbo Application
## The default value of dubbo.application.name is ${spring.application.name}
## dubbo.application.name=${spring.application.name}
## Dubbo Registry
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper
# Dubbo Protocol
dubbo.protocol.name=dubbo
dubbo.protocol.port=20883

```

*   编写接口 DemoService.java 和实现类 DemoServiceImpl.java
    

```
import org.apache.dubbo.config.annotation.Service;
import org.springframework.beans.factory.annotation.Value;
import com.muse.service.springbootdubbo.DemoService;
@Service // 切记！此处引用的是dubbo的Service注解
public class DemoServiceImpl implements DemoService {
    /**
     * The default value of ${dubbo.application.name} is ${spring.application.name}
     */
    @Value("${dubbo.application.name}")
    private String serviceName;
    public String sayHello(String name) {
        System.out.println(String.format("[%s] : Hello, %s", serviceName, name));
        return String.format("[%s] : Hello, %s", serviceName, name);
    }
}

```

*   编写 SpringBoot 启动类 ProviderApplication.java
    

```
@EnableDubbo /** 打开Dubbo开关 */
@SpringBootApplication
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}

```

*   启动后，在 Dubbo Admin 控制台可以查看服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3rjMpgVyPvcfI5vJ2bxangGyhZrZVGC8IWeQ0m6j03jXVeMfWIzI0OA/640?wx_fmt=png)

6.2> 编写 Consumer  

-------------------

*   项目结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v34Wn4RPxIqn9z7DKQFNTn3xfY903qP7UIzwvWDapkIGuGC9aibVbSamQ/640?wx_fmt=png)

*   在 pom.xml 中添加 dubbo、zk 和 web 的依赖
    

```
<dependencies>
    <!-- Spring Boot Web Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Dubbo-->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
    </dependency>
    <!-- Dubbo Spring Boot Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
    </dependency>
    <!-- Dubbo Zookeeper Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <type>pom</type>
    </dependency>
    <!-- Common Dubbo Api -->
    <dependency>
        <artifactId>common-dubbo-api</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>

```

*   编写配置文件 application.properties（可与 consumer.xml 中的配置一一对照编写）
    

```
server.port=8088
# Spring boot application
spring.application.name=springboot-dubbo-consumer
# Dubbo Application
## The default value of dubbo.application.name is ${spring.application.name}
## dubbo.application.name=${spring.application.name}
## Dubbo Registry
dubbo.registry.address=zookeeper://127.0.0.1:2181
# Dubbo Protocol
dubbo.protocol.name=dubbo
dubbo.protocol.port=20888
# dubbo.consumer.check=false

```

*   编写 Controller 类 ConsumerController.java
    

```
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import com.muse.service.springbootdubbo.DemoService;
@RestController
public class ConsumerController {
    @Reference
    private DemoService demoService;
    /**
     * http://localhost:8088/hello
     */
    @GetMapping("/hello")
    public String hello() {
        return demoService.sayHello("muse");
    }
}

```

*   编写 SpringBoot 启动类 ConsumerApplication.java
    

```
import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@EnableDubbo /** 打开Dubbo开关 */
@SpringBootApplication
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}

```

七、Dubbo 的高可用性  

================

7.1> 注册中心宕机
-----------

*   当我们启动 Provider 和 Consumer 之后，关闭 Zookeeper
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3D3vzVLicar0DHmPHcNbtNeXXibaEUqb80bOQ3WbuGHpwau9V2prXN7Vg/640?wx_fmt=png)

*   此时，Provider 和 Consumer 后台都开始报连接 Zookeeper 被拒绝
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3kHH9zZOMdhngYicqicpFvKECfFxvOpAduz5tMsWLBz2fNicPhxkAgQh8Q/640?wx_fmt=png)

*   此时，我们请求 Consumer 的接口 http://localhost:8082/hello，发现依然可以访问得通
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3zx3ibSqEVAjIgNyMlibjmGicMhIEAdRZl2UPBTP2apdR8BrOFGJNMbZrg/640?wx_fmt=png)

*   所以，我们可以得出以下结论
    

*   注册中心宕机后，Consumer 依然可以通过本地缓存的服务列表进行查询和通信，但是此时不能再注册新的服务。
    
*   注册中心如果是集群，那么如果任意一台宕机了，将自动切换到另外一台。
    
*   由于 Provider 是无状态的，所以任意一台 Provider 宕机，都不影响使用。除非所有的 Provider 都宕机了，那么 Consumer 则无法调用 Provider 远程方法，会执行无限次重连操作，等待 Provider 恢复。
    

7.2> 负载均衡  

------------

### 7.2.1> 概述

*   在集群负载均衡时，Dubbo 提供了多种均衡策略，**缺省为** **random** **随机调用**
    
*   **Random LoadBalance** **随机**——按权重设置随机概率。
    
    在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。
    
*   **RoundRobin LoadBalance** **轮询**——按公约后的权重设置轮询比率
    
    存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。
    
*   **LeastActive LoadBalance** **最少活跃调用数**——相同活跃数的随机，活跃数指调用前后计数差
    
    使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。
    
*   **ConsistentHash LoadBalance** **一致性 Hash**——相同参数的请求总是发到同一提供者
    
    当某一台 Provider 挂掉时，原本发往该 Provider 的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。缺省只对第一个参数 Hash，如果要修改，请配置 <dubbo:parameter key="hash.arguments" value="0,1" />，缺省用 160 份虚拟节点，如果要修改，请配置 <dubbo:parameter key="hash.nodes" value="320" />
    
*   负载均衡的具体实现，都是通过集成 **AbstractLoadBalance** 类实现的，通过查看该抽象类的实现类，我们就可以看到 Dubbo 支持的所有负载均衡策略。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3bH0TPAd31VAvcNNNfyfEhTiarkCg55hdunpJmAaxRWR6LmbZFSdribvg/640?wx_fmt=png)

### 7.2.2> 配置方式  

*   服务端服务级别
    
    <dubbo:service interface="..." loadbalance="roundrobin" />
    
*   客户端服务级别
    
    <dubbo:reference interface="..." loadbalance="roundrobin" />
    
*   服务端方法级别
    
    <dubbo:service interface="...">    
    
        <dubbo:method />
    
    </dubbo:service>
    
*   客户端方法级别
    
    <dubbo:reference interface="...">    
    
        <dubbo:method />
    
    </dubbo:reference>
    

### 7.2.3> 操作演示——默认负载均衡

*   开启 3 个 Provide
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v348ZMun8l9SiczrhcCjcf6omNNppYQCGyewXE7QJNk8392xoib4s0HnUQ/640?wx_fmt=png)

*   在 Dubbo Admin 控制台上，查询 Provider 服务情况
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3OBjxS2g3Pq2tpChx4AUYOQtOAficaE07I94RezHHrPfic6ZicGuPOXElA/640?wx_fmt=png)

*   针对 Consumer 服务做 6 次请求。我们发现，3 个 Provider 都收到了请求，但是并不是平均的，就是因为默认的负载均衡使用的是 Random 随机的方式。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3eRkfNIteIkXHOA2MN7kS84uD9lGMT84H7h5eViap64pVZ45I9GUUIDA/640?wx_fmt=png)

### 7.2.4> 操作演示——指定轮询方式

*   我们通过查询轮询的实现类，可以看到，配置 roundrobin 即可使用轮询的方式
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3haiclN2z8OkpCIUibI7eLqWrPONjsN7Qj8PzBtYK6icGUF5KibMFHUPGFw/640?wx_fmt=png)

*   我们即可以在 Provider 端配置负载均衡策略，也可以在 Consumer 端进行配置；此例中，我们配置到 Consumer 中，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3icd8jY8kqpxnbJVmjDAV5KdgtWvz8vQxf1wPTvvmcAibkebDv5A256CA/640?wx_fmt=png)

*   重启 Consumer 之后，再次进行 6 次请求，我们发现请求已经均衡的打到了 3 个 Provider 上了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v38YhbgCGicicFcrrW0JicITric7QCiaa7kfoW9SUGLBebibolYqHvPPDmIPZQ/640?wx_fmt=png)

### 7.2.5> 操作演示——配置请求权重

*   我们通过 Dubbo Admin 可以查看到，在默认情况下，3 个 Provider 的权重都是一样的（默认值为 100）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v31u1iatVRQbTb7HUv5Q7eaEES0FEqtKaica5t3HA7dhODW9ibwlHMcyqKg/640?wx_fmt=png)

*   我们尝试将 20883 的权重提升，设置为 300
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3GHib7bhWzTesKeS0Duojotg9vLiccxdqKTgdchF2DFAL2IHib6RNvic9UQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3iciaL3Xf0zZLtO55ibweDj5upIkdqCT29R36znESuepoH3vVbn6B313hg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3vZMBZOiaJ0X9H9XneL3cxaR0icDfNAY6hJRt6MMfND8iaMjCoodYh0ZYQ/640?wx_fmt=png)

*   此时，我们清除 3 个 Provider 的后台日志，再次执行 6 次请求 Consumer，发现权重为 300 的 Provider（port: 20883）接收到了 4 次请求，而另外两个权重为 100 的 Provider，分别各自接收到了 1 次请求。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3Htt0STlLicIodVXr8iaQ7Awo4Ndzubr99O9XQINv16PDDmLS4B1eEdWg/640?wx_fmt=png)

7.3> zookeeper 注册中心  

----------------------

*   provider&consumer 注入 zk 流程说明：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v33a6uGKOswAN69qicBBWHrqXG5mlMukWNsA4viaU6BicRfVt62S80S4xag/640?wx_fmt=png)

【解释】

*   服务提供者启动时: 向 **/dubbo/com.muse.service.springbootdubbo.DemoService/providers** 目录下写入自己的 URL 地址
    
*   服务消费者启动时: 订阅 **/dubbo/com.muse.service.springbootdubbo.DemoService/providers** 目录下的提供者 URL 地址，并向 **/dubbo/com.muse.service.springbootdubbo.DemoService/consumers** 目录下写入自己的 URL 地址
    

*   监控中心启动时: 订阅 **/dubbo/com.muse.service.springbootdubbo.DemoService** 目录下的所有提供者和消费者 URL 地址。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3CVOW29GiaMQVGuQLUBUd23rLm0k9aPUia2ic0HRs6J0Lx5m0r8Niay38lw/640?wx_fmt=png)

*   支持以下功能：
    

*   当提供者出现断电等异常停机时，注册中心能**自动删除**提供者信息。
    
*   当注册中心重启时，能自动恢复注册数据，以及订阅请求。
    
*   当会话过期时，能自动恢复注册数据，以及订阅请求。
    
*   当设置 <dubbo:registry check="false" /> 时，记录失败注册和订阅请求，后台定时重试。
    
*   可通过 <dubbo:registry user /> 设置 zookeeper 登录信息。
    
*   可通过 <dubbo:registry group="dubbo" /> 设置 zookeeper 的根节点，不配置将使用默认的根节点。
    
*   支持 * 号通配符 <dubbo:reference group="*" version="*" />，可订阅服务的所有分组和所有版本的提供者。
    

八、Dubbo 原理  

=============

8.1> 框架设计
---------

### 8.1.1> Dubbo 架构设计图

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3S5HvwWtlkvnvj32BqDymrlGM8oCbRoyW3lq91QD7H7TSgHmJSOcZ7A/640?wx_fmt=png)

【图例说明】  

*   图中**左边淡蓝**背景的为**服务消费方**使用的接口，**右边淡绿色**背景的为**服务提供方**使用的接口，位于中轴线上的为双方都用到的接口。
    
*   图中从下至上分为 10 层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，**Service 和 Config 层为 API，其它各层均为 SPI**。
    
*   图中**绿色小块**的为**扩展接口**，**蓝色小块**为**实现类**，图中只显示用于关联各层的实现类。
    
*   图中**蓝色虚线**为**初始化过程**，即启动时组装链，**红色实线**为**方法调用过程**，即运行时调时链，**紫色三角箭头**为**继承**，可以把子类看作父类的同一个节点，线上的文字为调用的方法。
    

### 8.1.2> 各层说明

*   **config 配置层**
    
    对外配置接口，以 **ReferenceConfig**, **ServiceConfig** 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类
    
*   **proxy 服务代理层**
    
    服务接口透明代理，生成服务的**客户端 Stub** 和**服务器端 Skeleton**, 以 **ServiceProxy** 为中心，扩展接口为 **ProxyFactory**
    
*   **registry 注册中心层**
    
    封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 **RegistryFactory**, **Registry**, **RegistryService**
    
*   **cluster 路由层**
    
    封装多个提供者的路由及负载均衡，并桥接注册中心，以 **Invoker** 为中心，扩展接口为 **Cluster**, **Directory**, **Router**, **LoadBalance**
    
*   **monitor 监控层**
    
    RPC 调用次数和调用时间监控，以 **Statistics** 为中心，扩展接口为 **MonitorFactory**, **Monitor**, **MonitorService**
    
*   **protocol 远程调用层**
    
    封装 RPC 调用，以 **Invocation**, **Result** 为中心，扩展接口为 **Protocol**, **Invoker**, **Exporter**
    
*   **exchange 信息交换层**
    
    封装请求响应模式，同步转异步，以 **Request**, **Response** 为中心，扩展接口为 **Exchanger**, **ExchangeChannel**, **ExchangeClient**, **ExchangeServer**
    
*   **transport 网络传输层**
    
    抽象 mina 和 netty 为统一接口，以 **Message** 为中心，扩展接口为 **Channel**, **Transporter**, **Client**, **Server**, **Codec**
    
*   **serialize 数据序列化层**
    
    可复用的一些工具，扩展接口为 **Serialization**, **ObjectInput**, **ObjectOutput**, **ThreadPool**
    

### 8.1.3> 关系说明

*   在 RPC 中，**Protocol 是核心层**，也就是只要有 **Protocol + Invoker + Exporter** 就可以完成非透明的 RPC 调用，然后在 Invoker 的主过程上 **Filter** 拦截点。
    
*   图中的 Consumer 和 Provider 是抽象概念，只是想让看图者更直观的了解哪些类分属于客户端与服务器端，不用 Client 和 Server 的原因是 Dubbo 在很多场景下都使用 Provider, Consumer, Registry, Monitor 划分逻辑拓扑节点，保持统一概念。
    

*   而 Cluster 是外围概念，所以 **Cluster 的目的是将多个 Invoker 伪装成一个 Invoker**，这样其它人只要关注 Protocol 层 Invoker 即可，加上 Cluster 或者去掉 Cluster 对其它层都不会造成影响，因为**只有一个提供者时，是不需要 Cluster 的**。
    
*   Proxy 层封装了所有接口的透明化代理，而在**其它层都以 Invoker 为中心**，**只有到了暴露给用户使用时，才用 Proxy 将 Invoker 转成接口，或将接口实现转成 Invoker**，也就是去掉 Proxy 层 RPC 是可以 Run 的，只是不那么透明，不那么看起来像调本地服务一样调远程服务。
    

*   而 **Remoting 实现是 Dubbo 协议的实现**，**如果你选择 RMI 协议，整个 Remoting 都不会用上**，Remoting 内部再划为 Transport 传输层和 Exchange 信息交换层，**Transport 层只负责单向消息传输**，是对 Mina, Netty, Grizzly 的抽象，它也可以扩展 UDP 传输，而 **Exchange 层是在传输层之上封装了 Request-Response 语义**。
    
*   Registry 和 Monitor 实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在一起。
    

### 8.1.4> 模块分包

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3sMdibCus2zjTiaYA7H5gf8l41TWEabJnvj4MaCeYg049rQGdBa7XTs0g/640?wx_fmt=png)

【模块说明】

*   **dubbo-common 公共逻辑模块**
    
    包括 Util 类和通用模型
    
*   **dubbo-remoting 远程通讯模块**
    
    相当于 Dubbo 协议的实现，如果 RPC 用 RMI 协议则不需要使用此包
    
*   **dubbo-rpc 远程调用模块**
    
    抽象各种协议，以及动态代理，只包含一对一的调用，不关心集群的管理
    
*   **dubbo-cluster 集群模块**
    
    将多个服务提供方伪装为一个提供方，包括：负载均衡, 容错，路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发
    
*   **dubbo-registry 注册中心模块**
    
    基于注册中心下发地址的集群方式，以及对各种注册中心的抽象
    
*   **dubbo-monitor 监控模块**
    
    统计服务调用次数，调用时间的，调用链跟踪的服务。
    
*   **dubbo-config 配置模块**
    
    是 Dubbo 对外的 API，用户通过 Config 使用 Dubbo，隐藏 Dubbo 所有细节。
    
*   **dubbo-container 容器模块**
    
    是一个 Standalone 的容器，以简单的 Main 加载 Spring 启动，因为服务通常不需要 Tomcat/JBoss 等 Web 容器的特性，没必要用 Web 容器去加载服务。
    
*   整体上按照分层结构进行分包，与分层的不同点在于：
    

*   container 为服务容器，用于部署运行服务，没有在层中画出。
    
*   protocol 层和 proxy 层都放在 rpc 模块中，这两层是 **rpc 的核心**，在不需要集群也就是只有一个提供者时，可以只使用这两层完成 rpc 调用。
    
*   transport 层和 exchange 层都放在 remoting 模块中，为 **rpc 调用的通讯基础**。
    
*   serialize 层放在 common 模块中，以便更大程度复用。
    

### 8.1.5> 依赖关系

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3TANfuxKOpEHCNdSicnMChR3dWfCdK4PtZBicaibGCPrF6EJPsHttbAFkg/640?wx_fmt=png)

【图例说明】

*   图中小方块 Protocol, Cluster, Proxy, Service, Container, Registry, Monitor 代表层或模块，**蓝色模块**的表示**与业务有交互**，**绿色模块**的表示**只对 Dubbo 内部交互**。
    
*   图中背景方块 Consumer, Provider, Registry, Monitor 代表**部署逻辑拓扑节点**。
    
*   图中**蓝色虚线**为初始化时调用，**红色虚线**为运行时异步调用，**红色实线**为运行时同步调用。
    
*   图中只包含 RPC 的层，不包含 Remoting 的层，**Remoting 整体都隐含在 Protocol 中**。
    

### 8.1.6> 领域模型

在 Dubbo 的核心领域模型中：

*   Protocol 是**服务域**，它是 Invoker 暴露和引用的主功能入口，它负责 Invoker 的生命周期管理。
    
*   Invoker 是**实体域**，它是 **Dubbo 的核心模型**，其它模型都向它靠扰，或转换成它，它代表一个可执行体，可向它发起 invoke 调用，它有可能是一个本地的实现，也可能是一个远程的实现，也可能一个集群实现。
    

*   Invocation 是**会话域**，它持有调用过程中的变量，比如方法名，参数等。
    

8.2> 标签解析
---------

*   Spring 是通过 **BeanDefinitionParser**（Bean 的定义解析器）来解析 xml 中配置的标签。我们可以看到，Dubbo 实现了它自己的解析器——**DubboBeanDefinitionParser**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3icFDaPsPO0SIXUCt6xB59KXIYL3PPRU5Z5xSI0w6Bry2AsK0beZjdeA/640?wx_fmt=png)

*   初始化过程是在 DubboNamespaceHandler 类的 init() 方法里实现的，其中 registerBeanDefinitionParser(String elementName, BeanDefinitionParser parser) 方法是 Spring 框架中 NamespaceHandlerSupport 实现的方法，方法的逻辑是将入参保存到 Spring 框架的 Map<String, BeanDefinitionParser> parsers 中。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3oEky2T1VeSsEyliaoScInTYQjOiaJxibtZzxC6k2yajhbIIGXGeJlRWDw/640?wx_fmt=png)

8.3> 服务暴露流程
-----------

*   Dubbo 通过实现监听器，在 Spring 启动时，可以对 Dubbo 的服务进行暴露。如下是 Dubbo 相关的监听器和监听事件。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3xicFWZzfIWofsqCURKGUDCuhnrtuaATUP8QqQXNCU726aDEEQib6F8QA/640?wx_fmt=png)

*   Dubbo 监听如下四个事件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3dia15gOPsb3ian5A1wo8wdhI5tnasvL8eibKKuScLErd7AR28KjibkaHXQ/640?wx_fmt=png)

*   onApplicationContextEvent 方法由 **DubboBootstrapApplicationListener** 和 **DubboLifecycleComponentApplicationListener** 实现。所以，暴露服务的逻辑就在这两个子类中。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3T2niaHJcvuCByZtQWXsUaAibuWPQDA2nmBDx6YzAhO4gCKVSFia2VB4lw/640?wx_fmt=png)

*   我们查看 DubboBootstrapApplicationListener 和 DubboLifecycleComponentApplicationListener 这两个类对 **onApplicationContextEvent** 方法的实现，发现 Dubbo 会再次细分，只监听两个事件 **ContextRefreshedEvent** 和 **ContextClosedEvent**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3iaUXB9KXyJTQoaIKS0kicial3ljAzAxnFI9xbZIfyVuItDSP9TLC1VbIQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3ItZ2FuV6yoGeLAWYEZQp3UIosteRiaiaxbvUVV6Qtt2oNI4qbdokj1xw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3mqQvSWzFiblsVPXYA4dQyGxpxDgFuX2nglZjS7C7vWz8Vbgs4BrQLGA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v30ILH09tlnIveibfD36QwJockicD4HICjCice3RpyuzFyh4tloicRooP3ug/640?wx_fmt=png)

*   展开总设计图左边服务提供方暴露服务的蓝色初始化链，时序图如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3YqfyGeF80Rd3icdj1FUic5zvazLwXLvl06BsuFbHGYFGKasNwnHIgOJg/640?wx_fmt=png)

8.4> 服务引用流程  

--------------

*   展开总设计图右边服务消费方引用服务的蓝色初始化链，时序图如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3RZxEib1D1tExFibBVhOHHYOAGwV5Tmd8LnicMdbJJibamzibRMTLAnXic5ibg/640?wx_fmt=png)

8.5> 服务调用流程  

--------------

*   展开总设计图的红色调用链，如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v34Cwdv13kqqbGA8eCARtSfdTc7wZlJsTrnxCfo4dgw5JqzHe8PQqdLg/640?wx_fmt=png)

九、通过 IDL 使用 dubbo 框架  

=======================

*   下载源码
    
    git clone -b master https://github.com/apache/dubbo-samples.git
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3tZnb70yPny8OopfyHDwibxeZxzpm6ibqFqH0qeEs6sDX1YPFEtpEXK1A/640?wx_fmt=png)

*   在 dubbo-samples 的项目中，包含了各种的 dubbo 调用方式，例如：
    

*   dubbo-samples-api 使用 api 的方式实现 dubbo 的调用
    
*   dubbo-samples-basic 使用传统的 xml 配置方式实现 dubbo 的调用
    
*   dubbo-samples-protobuf 使用 IDL 的方式实现 dubbo 的调用（由于最新的 dubbo 3.x 建议采用 IDL 的方式进行 dubbo 的调用，所以，下面我们会针对该方式进行操作演示）
    

*   进入 demo 目录
    

$ cd dubbo-samples/dubbo-samples-protobuf

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3Nia7BlkVyCWYyq9mfqEqgOmBGfXfUxxvGoKTiaLqo4sWwbD9qVrBK0Yg/640?wx_fmt=png)

*   执行 maven clean package
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3bpOsSM1NBAFlh4icoKfWFnvpkkopVgzib02cUZfU5prciaVP21nrld4AA/640?wx_fmt=png)

*   我们发现在 target 路径下分别生成了 consumer 和 producer 的 jar 包。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3MiacibxwiaTQtjm20AQgoIh6dC475jDhPsqooCcfrdEWodgLdoZaia6ZMQ/640?wx_fmt=png)

*   并且生成了 build 目录，里面有 IDL 生成的 java 类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v308NqjmwdwSziaFqX4sicsychiaHWiaCNFoLXlsd7rz5QH4wqqrsLvQw0Rw/640?wx_fmt=png)

*   将类拷贝到 org.apache.dubbo.demo 路径下，也可以通过运行 jar 包的方式（java -jar [jar 包]）启动 consumer 和 provider
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v3IqFXkfq3bHmZ2RDKOSYfs4NaRQQ6htvy6dTjpC0x4va35icl8GftEhg/640?wx_fmt=png)

*   启动 provider 和 consumer
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9yVECezf4pNnI01nic3a8v38raR54INIsXRUkG1POZ9S8iaqgmKfSsR42TbjzWyiat1IlGHXK5TTEIg/640?wx_fmt=png)