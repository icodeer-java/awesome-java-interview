> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486062&idx=1&sn=ff7c575533ee8dc3e4aa7a725ce58e7d&chksm=e9114893de66c1851b156907839ef83cbf05fba3e95ee16fc948215a8f5e5290c5e74b5b491c&scene=178&cur_album_id=2173835491853336577#rd)

    这篇文章主要介绍 SpringCloud Gateway，它是 Spring 官方团队研发的 API 网关技术，它的目的是取代 Zuul 成为微服务提供一个简单高效的 API 网关。下面是目录概要：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCT4UsMduf5icyUHJIYmzb7yRSCLO1ibU1icLAvuBHq370ocLc3fCYx8TpQ/640?wx_fmt=png)

一、网关的作用  

----------

### 1.1> 网关作用说明

#### 1.1.1> 无网关的微服务调用

*   无网关的图例    
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCmDQYvz9iaWiaVL17IA4JqTe0GboicDs1Xy5XmqXNMn91fUNnYejQYrSkw/640?wx_fmt=png)

*   无网关存在的问题有哪些？
    

*   1> 客户端需要发起多次请求，增加了**客户端处理的复杂性**。
    
*   2> 服务的鉴权（用户、功能）会**分布在每个**微服务中处理，客户端对于每个服务的提供者都需要**重复鉴权**。  
    
*   3> 在后端的微服务架构中，可能不同的服务采用的**协议不同**，比如：HTTP、RPC 等。客户端如果需要调用多个服务，**需要对不同协议进行适配**。
    

#### 1.1.2> 有网关的微服务调用

*   有网关的图例
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCnnlPbxkvUj4ls8iaINzHu8gUV9Pzeico78jXw2c3Vf3jOcDJQasYtWLg/640?wx_fmt=png)

*   网关可以解决上面的问题，整体来看，网关有点类似于**门面**，所有的外部请求都会先经过网关这层。
    
*   API 网关层可以**把后端的多个服务进行整合**，然后提供唯一的业务接口，客户端只需要调用这个接口即可完成数据的获取及展示。在网关中会再消费后端的多个 微服务，**进行统一的整合，给客户端返回唯一的响应**。
    
*   网关可以提供如下功能：
    

*   针对所有请求进行登陆**统一鉴权（登录态）**、**限流**、**缓存**、**日志（用户打点）**。
    
*   可以根据不同的请求路径 pattern，来进行请求的鉴权、转发、和拒绝。
    
*   **协议转化**。针对后段多种不同的协议，在网关层统一处理后以 HTTP 对外提供服务。
    
*   提供**统一的错误码**。
    
*   请求转发，并且可以基于网关实现**内网与外网的隔离**。
    

### 1.2> 网关的应用  

#### 1.2.1> 鉴权认证

*   图例如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCk1MicbZBLRavK03uzdibo2BIfMEHcUWqicFibuAwbtAjoJA0KhmXGbLAkQ/640?wx_fmt=png)

*   客户端身份认证：
    
    主要用于判断当前用户**是否为合法用户**，一般的做法是使用**账号**和**密码**进行验证。对于一些复杂的认证场景，会采用**加密算法**来实现，比如公钥和私钥。
    
*   访问权限控制：
    
    身份认证和访问权限一般是相互联系的。当身份认证通过后，就需要判断该用户**是否有权限访问该资源**，或者该用户的访问**权限是否被限制**了。
    

#### 1.2.2> 灰度发布（金丝雀发布）

*   由于互联网公司迭代速度快，所以在高频率的迭代模式下，往往会伴随着一些风险，比如：
    

*   新发布的**代码出现兼容性**问题。
    
*   新的功能发布后，**用户是否能够接受**，如果不能，会造成用户流失。
    

*   代码中**存在隐藏的 Bug**，导致线上故障。
    

*   为了规避如上问题，对于大版本改动，一般采用灰度发布（又名：金丝雀发布）的方式来**实现平滑过度**。
    
*   所以灰度发布，就是先把新的功能开发给一小部分用户使用，比如 **A/B Test** 就是一种灰度发布方式，即：一部分用户继续使用 A 功能，另外一小部分用户使用新的 B 功能，通过对使用 B 功能的用户进行满意度调查，以及新功能代码的性能和稳定性进行评测，**逐步放大该新版本的投放，直到全量或者回滚该版本**。
    
*   由于网关是所有客户端请求的入口，因此在网关层可以通过灰度规则进行部分流量的路由，从而实现灰度发布。如下图所示：网关对请求进行拦截之后，会根据**分流引擎配置**的分流规则进行请求的路由。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaC3TUxdfhdQmR6rQYU6rQKynGnjSv7gA8NBwgoLCLfOaCLwRBZ29IuAQ/640?wx_fmt=png)

二、网关技术选型对比  

-------------

网关的本质其实分为两点：

*   请求的**转发路由**
    
    接收客户端的所有请求，并将请求转发到后端的微服务中。
    
*   **过滤器**
    
    拦截所有的请求，来实现一系列横切工作。比如：鉴权、限流。
    
*   常见的网关实现方案有很多，比如：OpenResty、Zuul、Gateway、Orange、Kong、Tyk 等。
    

### 2.1> OpenResty

*   什么是 OpenResty
    
    它实际上是由 **Nginx** 与 **Lua** 集成的一个高性能 Web 应用服务器，内部集成了大量 Lua 库和第三方模块。
    
    简单来说，本质就是**将 Lua 嵌入到 Nginx 中**，在每个 Nginx 的进程中都嵌入了一个 **LuaJIT 虚拟机**来执行 Lua 脚本。
    
    在接受客户端的请求时，同样可以拦截请求进行前置和后置处理，它可以在不同的阶段来**挂载 Lua 脚本实现不同阶段的自定义行为**。
    
*   如下图所示，OpenResty 实现网关功能的核心就是在这 **11 个步骤**中挂载 Lua 脚本来实现功能的扩展的。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCugxNEc3a85EGvzl8KgdmKIoOREADvll9VphrsuLZ6ibTVtcRJPllG2A/640?wx_fmt=png)

*   11 个指令的详细说明
    

*   **1>** **init_by_lua**
    
    当 Nginx Master 进程**加载 Nginx 配置文件**时会运行这段 Lua 脚本。
    
*   **2>** **init_worker_by_lua**
    
    每个 Nginx worker 进程启动时会执行的 Lua 脚本，可以用来做**健康检查**。
    
*   **3>** **ssl_certificate_by_lua**
    
    当 Nginx 开始**对下游进行 SSL（HTTPS）握手连接**时，该指令执行用户 Lua 脚本。
    
*   **4>** **set_by_lua**
    
    设置一个变量。
    
*   **5>** **rewrite_by_lua**
    
    在 **rewrite 阶段**执行 Lua 脚本。
    
*   **6>** **access_by_lua**
    
    在**访问阶段**调用 Lua 脚本。
    
*   **7>** **content_by_lua**
    
    通过 Lua 脚本**生成 content 输出给 Http 响应**。
    
*   **8>** **balancer_by_lua**
    
    实现**动态负载均衡**，如果不走 content_by_lua，则走 proxy_pass，在通过 upstream 进行转发。
    
*   **9>** **header_filter_by_lua**
    
    通过 Lua 来设置 **Header** 或者 **Cookie**。
    
*   **10>** **body_filter_by_lua**
    
    对响应数据进行**过滤**。
    
*   **11>** **log_by_lua**
    
    在 **Log 阶段**执行的脚本。
    

### 2.2> Zuul

*   Zuul 是 Netflix 开源的微服务网关，它的主要功能是**路由转发**和**过滤**。
    
*   Zuul 的**核心**由一系列**过滤器组成**，它定义了 **4 种**标准类型的**过滤器**，这些会对应请求的整个生命周期。
    

*   Zuul 请求过滤如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCf4MTop7vehOXRTMYadtBorHTsXGOZIAp9gKL5BrWLROFKYA3xicic02Q/640?wx_fmt=png)

*   **Pre Filters**
    
    前置过滤器，请求被路由之前调用，可以用于处理**鉴权**、**限流**等。
    
*   **Routing Filters**
    
    路由过滤器，将**请求路由**到后端的微服务。
    
*   **Post Filters**
    
    后置过滤器，路由过滤器中远程调用结束后被执行。可以用于做**统计**、**监控**、**日志**等。
    
*   **Error Filters**
    
    错误过滤器，任意一个过滤器**出现异常**或者远程服务**调用超时**都会被调用。
    

### 2.3> Gateway

*   Spring Cloud Gateway 是 **Spring 官方团队研发**的 API 网关技术，它的目的是**取代 Zuul** 成为微服务提供一个简单高效的 API 网关。
    
*   Zuul 1.x 采用的是传统的 **thread per connection** 方式来处理请求，也就是针对**每一个请求**，会为这个请求专门分配**一个线程**来进行处理，直到这个请求完成之后才会释放线程，一旦后台服务器响应较慢，就会使得该线程被阻塞，所以它的性能不是很好。
    

*   Zuul 本身存在的一些性能问题不适合于高并发的场景，虽然后来 Netflix 决定开发高性能版 Zuul 2.x，但是 Zuul 2.x 的发布时间一直不确定。虽然 Zuul 2.x 后来已经发布并且开源了，但是 Spring Cloud 并没有打算集成进来。
    
*   Spring Cloud Gateway 是依赖于 **Spring Boot 2.0**、**Spring WebFlux** 和 **Project Reactor** 等技术开发的网关，它不仅提供了统一的路由请求的方式，还基于过滤链的方式提供了网关最基本的功能。
    

三、Gateway 的简单例子  

------------------

*   **实战例子：**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaC9qlIh6B4S3iadzPtN4QOBCQ1SY75zbVLxE23sOZh4kdEUu6pqFQYcRQ/640?wx_fmt=png)

*   **gateway-service   [port:8080]**
    

添加 spring-boot-starter-web 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

```

创建 HelloController 类

```
@RestController
public class HelloController {
    @GetMapping("/say")
    public String say() throws Throwable {
        System.out.println("[spring-cloud-gateway-service]:say Hello!");
        return "[spring-cloud-gateway-service]:say Hello!";
    }
}

```

*   **gateway-sample** **[port:8088]**
    

添加 spring Cloud Gateway 依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>

```

在 application.yml 文件中添加 Gateway 的路由配置

```
server:
  port: 8088
spring:
  cloud:
    gateway:
      routes:
        - id: path_route
          uri: http://localhost:8080 #访问地址
          predicates:
            - Path=/gateway/** #路径匹配
          filters:
            - StripPrefix=1 #跳过前缀

```

【详细说明】

*   **id**
    
    自定义**路由 ID**，保持唯一。
    
*   **uri**
    
    目标**服务地址**，支持普通 URI 及 lb:// 应用注册服务名称。
    
*   **predicates**
    
    **路由条件**，根据匹配的结果决定是否执行该请求路由。
    
*   **filters**
    
    **过滤规则**，包含 pre 和 post 过滤。其中 StripPrefix=1，表示 Gateway 根据该配置的值去掉 URL 路径中的部分前缀（这里去掉一个前缀，即：在转发的目标 URL 中去掉 gateway）。
    

*   启动应用，在控制台可以获得如下信息，可以看到，它并没有依赖 Tomcat，而是用 NettyWebServer 来启动一个服务监听。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCW5Vpfojg7q1Hl6HgeZbMNJetgNYvkB0Wd4h7zLIhXTe8n7KUocMR6w/640?wx_fmt=png)

*   请求
    
    http://localhost:8088/gateway/say
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCicmhjmVrib0lZHiceibSXz8wfuXC7icE3KXZsqSx4g3QfDFe4jxw0RDrb3Q/640?wx_fmt=png)

四、Gateway 的原理分析  

------------------

*   Spring Cloud Gateway 的请求处理过程
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCd81FYfQ2x6E5q8wnCucM8rlXa2GNZIjEHMKXrRdVImO7FKlUA3DZ5Q/640?wx_fmt=png)

【详细说明】

*   **Route**——路由（包含 **Predicate** 和 **Filter**）
    
    它是网关的基本组件，由**自定义路由** **ID**、**目标 URI**、**Predicate 集合**、**Filter 集合**组成。
    
*   **Predicate**——谓语（第五节将会介绍 Predicate 的实现）
    
    它是 Java8 中引入的函数式接口，提供了**断言**的功能。它可以匹配 HTTP 请求中的任何内容。如果 Predicate 的聚合判断结果为 true，则意味着该请求会被当前 Router 进行转发。
    
*   **Filter**——过滤器（第六节将会介绍 Filter 的实现，第七节将会介绍自定义 Filter）
    
    为请求提供前置和后置的过滤。
    

*   Spring Cloud Gateway 启动时**基于 Netty Server** 监听一个指定的端口（该端口可以通过 **server.port 属性**自定义）。
    
*   当客户端发送一个请求到网关时，网关会根据一系列 **Predicate 的匹配结果**来决定**访问哪个 Route 路由。**
    

*   然后根据**过滤器链**进行请求的处理。
    
*   过滤器链可以在请求发送到后端服务器之前和之后执行，也就是首先执行 **Pre 过滤器链**，然后将请求转发到后端服务器，最后执行 **Post 过滤器链**。
    

五、Route Predicate Factories  

------------------------------

*   **Predicate 用于条件过滤、请求参数的校验**
    

```
package java.util.function;
import java.util.Objects;
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef) ? Objects::isNull : object -> targetRef.equals(object);
    }
}

```

*   **HTTP 请求的属性对应的 Predicate**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCo2Y7TLHfyBXNN7NSBgypy308INbfW9boXWGhTGFYxPAtqpTZh2dMFQ/640?wx_fmt=png)

### 5.1> 指定时间规则匹配路由

*   **ZonedDateTime**
    
    BeforeRoutePredicateFactory——请求在指定日期之前
    
    AfterRoutePredicateFactory——请求在指定日期之后
    
    BetweenRoutePredicateFactory——请求在指定的两个日期之间
    

#### 5.1.1> BeforeRoutePredicateFactory（请求在指定日期之前）

*   application.yml
    

```
#【指定时间规则匹配路由】
#在2022-01-01 00:00:00之前访问的请求，都会转到http://localhost:8080，即：http://localhost:8088/say会转发到http://localhost:8080/say
spring:
  cloud:
    gateway:
      routes:
        - id: before_route
          uri: http://localhost:8080
          predicates:
            - Before=2022-01-01T00:00:00.000+08:00[Asia/Shanghai]
server:
  port: 8088

```

*   请求 http://localhost:8088/say，转发 8080 服务请求成功！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCGC0ic1L1Aqwr2EUnUvbwcZOVBhTQaJrVjUr36R9Lr87j1iaXv050fzUg/640?wx_fmt=png)

*   如果修改配置 Before 的日期为 2021 年并且重启服务，请求 http://localhost:8088/say 会报错 404。无法转发到 8080 的服务上。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCQVslLrdJv7L0S9Q3L9vibC2bnibwCdEr3phPDtd4ibSnNyZ37MLSAoZ2A/640?wx_fmt=png)

#### 5.1.2> AfterRoutePredicateFactory（请求在指定日期之后）

*   application.yml
    

```
#【指定时间规则匹配路由】
#在2021-01-01 00:00:00之后访问的请求，都会转到http://localhost:8080，即：http://localhost:8088/say会转发到#http://localhost:8080/say
spring:
  cloud:
    gateway:
      routes:
        - id: after_route
          uri: http://localhost:8080
          predicates:
            - After=2021-01-01T00:00:00.000+08:00
server:
  port: 8088

```

*   请求 http://localhost:8088/say，转发 8080 服务请求成功！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCicxcpScFVVVBLmpBOoZArxawk3KyNFl9yibPKSjWMvHmFYwhvWSfBtWA/640?wx_fmt=png)

*   如果修改配置 After 的日期为 2022 年并且重启服务，请求 http://localhost:8088/say 会报错 404。无法转发到 8080 的服务上。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaC9kowQkA2oYicCERFfZ43uzUOy7Aiact5hf7kE8xialic7SP1llNqErBLsA/640?wx_fmt=png)

### 5.2> CookieRoutePredicateFactory（判断请求中携带的 Cookie 是否匹配配置的规则）  

*   application.yml
    

```
#【指定Cookie规则匹配路由】
#请求配置localhost的Cookie(chocolate=mic;) 请求http://localhost:8088/say会转发到http://localhost:8080/say
spring:
  cloud:
    gateway:
      routes:
        - id: cookie_route
          uri: http://localhost:8080
          predicates:
            - Cookie=chocolate, mic
server:
  port: 8088

```

*   请求 http://localhost:8088/say，转发 8080 服务请求失败，报错 404！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCwOhILoLO0DyDWwBHcRuXN5LIBWLol0qTRC1XZg38ia1VxCyAacXiaNYA/640?wx_fmt=png)

*   配置 Cookies
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCpO9pbYpQxm0ciaLIzbGUTBiagHADnPrq48DE7AAib7sdJHcHTGdAE87gg/640?wx_fmt=png)

*   再次请求 http://localhost:8088/say，转发 8080 服务请求成功！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCs7dPjibPeE0NaU08ibSW2zfYwTzicNUvtJmgOwgXSY203fU7EkvqyCtMA/640?wx_fmt=png)

*   如何设置多个 Cookie？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCnUf0LS61hialRMjuH9qUehucibwMj6gIcu6vCM6icYhOcpTVm9dvNMbibg/640?wx_fmt=png)针对与多个 Cookie，那么也需要在请求中设置多个 Cookie，否则请求会无法转发。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCgm1hXQ9F2d6vP02gvwMSqDlSlavBPWHibt1ia3ibVDKeicJBnpwaXiapzMA/640?wx_fmt=png)

### 5.3> HeaderRoutePredicateFactory（判断 Header 头信息是否匹配配置的规则）  

*   判断请求中 Header 头消息对应的 name 和 value 与 Predicate 配置的值是否匹配，value 也是正则匹配形式。
    
*   application.yml
    

```
#【指定Header规则匹配路由】
#请求配置Header(key=X-Request-Id value=1) 请求http://localhost:8088/say会转发到http://localhost:8080/say
spring:
  cloud:
    gateway:
      routes:
        - id: header_route
          uri: http://localhost:8080
          predicates:
            - Header=X-Request-Id, \d+
server:
  port: 8088

```

*   请求 http://localhost:8088/say，转发 8080 服务请求失败，报错 404！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCt0ehrD1DqY27ibsMtMxsbnuZ9sT72hj5U4MicdRGQVjAnAk8zyaAggFQ/640?wx_fmt=png)

*   添加 Headers 内容，再次请求 http://localhost:8088/say，转发 8080 服务请求成功！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaC7B5kA3wNVydnMojpFj95TrEPqryTbv9IDWsickUenUemTgkvDLgMqJg/640?wx_fmt=png)

### 5.4> HostRoutePredicateFactory（判断 Host 信息是否匹配配置的规则）  

*   HTTP 请求会携带一个 Host 字段，这个字段表示请求的服务器网址。HostRoutePredicateFactory 规则就是匹配请求中的 Host 字段进行路由。
    
*   application.yml
    

```
#【指定Host规则匹配路由】
#请求配置Header(Host:www.muse.com) 请求http://localhost:8088/say会转发到http://localhost:8080/say
spring:
  cloud:
    gateway:
      routes:
        - id: host_route
          uri: http://localhost:8080
          predicates:
            - Host=**.muse.com
server:
  port: 8088

```

*   请求 http://localhost:8088/say，转发 8080 服务请求失败，报错 404！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCDgMPhIA0LpTg4e9DQibcYdibJfphVD2iaeOpGvc5hsVcybiaOyE2L7tK2A/640?wx_fmt=png)

*   添加 Headers 内容，再次请求 http://localhost:8088/say，转发 8080 服务请求成功！
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaC8LusdGxhhyicyiaBwMneWXPHLm6iav7rLcbMo7VaEvb4gKpP92Y4O6FAA/640?wx_fmt=png)

### 5.5> MethodRoutePredicateFactory（判断 Method 属性是否匹配配置的规则）  

*   application.yml
    

```
#【请求方法匹配路由】
# 请求POST http://localhost:8088/shout会转发http://localhost:8080/shout。请求GET http://localhost:8088/say 会报"404 Not Found"
spring:
  cloud:
    gateway:
      routes:
        - id: method_route
          uri: http://localhost:8080 #访问地址
          predicates:
            - Method=POST
server:
  port: 8088

```

*   请求 GET http://localhost:8088/say 会报 "404 Not Found"
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCy8GCq6YAPGdSs1UrAVJlAA2evmyQbl0ibibq3pyuLP3e8U3ERMibFiajsg/640?wx_fmt=png)

*   请求 POST  http://localhost:8088/shout 转发 8080 服务成功。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCAS0mSBD3RAQJ2nlLC4VGmhgTx3uIMlkHmNv8VYISHuJCaUq8X34YUw/640?wx_fmt=png)

### 5.6> PathRoutePredicateFactory（根据请求路径匹配 Predicate 配置来进行请求路由。）  

*   application.yml
    

```
#【请求路径匹配路由】
# 请求POST http://localhost:8088/gateway/say会转发http://localhost:8080/say。
spring:
  cloud:
    gateway:
      routes:
        - id: path_routeHeaderRoutePredicateFactory
          uri: http://localhost:8080 #访问地址
          predicates:
            - Path=/gateway/** #路径匹配
          filters:
            - StripPrefix=1 #跳过前缀

```

*   请求  http://localhost:8088/gateway/say 转发 8080 服务成功。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCEunykLWcM73SiaanmywWHRSdwDg6pSQZcr4Uic5tAibd5uo6VKRySa7uw/640?wx_fmt=png)

六、Gateway Filter Factories  

-----------------------------

*   Filter 的类型：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCI5qib02kNEdRqJnTk7u7GC1iayx8vvIarpzgXJYqlUFPBVr17RPZiccEw/640?wx_fmt=png)

*   **Pre 类型过滤器**
    
    在请求转发到后端微服务之前执行，可以做鉴权、限流等操作。
    
*   **Post 类型过滤器**
    
    在请求执行完之后，结果返回给客户端之前执行。
    

*   Filter 的实现方式
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCMicufycb19FicZfppMsgAlhQzEU7rzTxV8FFJe2aJcyTGqJvfKFqAIvA/640?wx_fmt=png)

*   **GatewayFilter**
    
    只会应用到单个路由或者一个分组的路由上。
    
*   **GlobalFilter**
    
    会应用到所有的路由上。
    

### 6.1> GatewayFilter  

#### 6.1.1> AddRequestParameterGatewayFilterFactory（对所有匹配的 request 请求中添加一个查询参数）

*   application.yml
    

```
#【添加request查询参数过滤器】
#会对所有请求增加teacher=muse这个参数
spring:
  cloud:
    gateway:
      routes:
        - id: add_request_parameter_route
          uri: http://localhost:8080 #访问地址
          predicates:
            - Path=/**
          filters:
            - AddRequestParameter=teacher, muse

```

*   HelloController.java
    

```
@RestController
public class HelloController {
    @GetMapping("/sayParam")
    public String sayParam(HttpServletRequest request, HttpServletResponse response) {
        Enumeration enumeration = request.getParameterNames();
        StringBuffer sb = new StringBuffer();
        if (enumeration.hasMoreElements()) {
            String name = String.valueOf(enumeration.nextElement());
            String value = request.getParameter(name);
            sb.append(" name=" + name);
            sb.append(" value=" + value);
            sb.append(";");
        }
        System.out.println("[spring-cloud-gateway-service]:sayParam Hello! requestParam:" + sb);
        return "[spring-cloud-gateway-service]:sayParam Hello! requestParam:" + sb;
    }
}

```

*   请求 http://localhost:8088/sayParam 转发 8080 服务成功
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCJaIpXMudh9SkIy1ffV20lQkGOErSfU1eAHI5zDvunXnJHt2tNMU6pA/640?wx_fmt=png)

#### 6.1.2> AddResponseHeaderGatewayFilterFactory（在返回结果给客户端之前，往 **Header 中添加相应的数据**）

*   application.yml
    

```
#【添加response响应Header参数过滤器】
#返回结果给客户端之前，在Header中添加相应的数据
spring:
  cloud:
    gateway:
      routes:
        - id: add_response_header_route
          uri: http://localhost:8080 #访问地址
          predicates:
            - Path=/**
          filters:
            - AddResponseHeader=X-Response-Teacher, Muse

```

*   HelloController.java
    

```
@RestController
public class HelloController {
    @GetMapping("/say")
    public String say() throws Throwable {
        System.out.println("[spring-cloud-gateway-service]:say Hello!");
        return "[spring-cloud-gateway-service]:say Hello!";
    }
}

```

*   请求 http://localhost:8088/say 转发 8080 服务成功。并且 Headers 里添加了响应的数据。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCHhvYM0IibBWBm0g31PGx2NSLVUgVBDn6oLdTruvmngg1llmZwSrJXIA/640?wx_fmt=png)

#### 6.1.3> RequestRateLimiterGatewayFilterFactory（配置请求限流过滤器）

*   该过滤器会对访问到当前网关的所有请求执行限流过滤，如果被限流，默认情况下会响应 HTTP 429-Too Many Requests。
    
*   RequestRateLimiterGatewayFilterFactory 默认提供了 **RedisRateLimiter** 的限流实现，它采用**令牌桶算法**来实现限流功能。
    
*   Redis 的限流器基于 Stripe 实现，它需要引入下面这个依赖：
    

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>

```

*   application.yml
    

```
#【添加限流过滤器】
spring:
  cloud:
    gateway:
      routes:
        - id: request_ratelimiter_route
          uri: http://localhost:8080 #访问地址
          predicates:
            - Path=/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 1 #令牌桶中令牌的填充速度，代表允许每秒执行的请求数
                redis-rate-limiter.burstCapacity: 1 #令牌桶的容量，也就是令牌桶最多能够容纳的令牌数。表示每秒用户最大能够执行的请求数量
#当我们需要限流功能时，需要提供对应的redis，默认是127.0.0.1:6379，当然，我们也可以在application.yml文件中指定ip和port
redis:
  host: 127.0.0.1
  port: 6379

```

*   HelloController.java
    

```
@RestController
public class HelloController {
    @GetMapping("/say")
    public String say() throws Throwable {
        System.out.println("[spring-cloud-gateway-service]:say Hello!");
        return "[spring-cloud-gateway-service]:say Hello!";
    }
}

```

*   配置 redis 后台开启
    
    redis.conf 中 daemonize 配置为 yes
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCxFoXflibOldVic4WgcZTeZu6LKF7q17PPMn6ibrZn9MthmOiaia92Cb1dBg/640?wx_fmt=png)

*   启动 redis
    
    ./redis-server ../redis.conf
    
*   请求 http://localhost:8088/say 转发 8080 服务成功。但是，如果 1 秒内多次请求，则会被执行限流。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaC6O8g4oQNkcGlyjeDSjnUD7lsg6g77rRibQumRUzf9x2wFded8MVuXrg/640?wx_fmt=png)

#### 6.1.4> RetryGatewayFilterFactory（请求重试过滤器）

*   application.yml
    

```
#【添加请求重试过滤器】
spring:
  cloud:
    gateway:
      routes:
        - id: retry_route
          uri: http://localhost:8080
          predicates:
            - Path=/example/**
          filters:
            - name: Retry
              args:
                retries: 3 #请求重试次数，默认值是3
                status: 500 #HTTP请求返回的状态码，针对指定状态码进行重试。
            - StripPrefix=1

```

*   HelloController.java
    

```
@RestController
public class HelloController {
    @GetMapping("/retryRoute")
    public String error() throws Throwable {
        System.out.println("------------------HelloController.retryRoute!------------------");
        throw new RuntimeException();
    }
}

```

*   请求 http://localhost:8088/example/retryRoute ，会报 500 错误。查看 gateway-service 服务日志里有【1 次正常请求 + 3 次重试请求】共 4 次请求。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaC2BTOef0SutbUX1m5jibygF4ibHCGZ4QP5kfu7ib3faPOwEkVXvBMgLWQQ/640?wx_fmt=png)

### 6.2> GlobalFilter  

*   GlobalFilter 和 GatewayFilter 的作用是相同的，只是 GlobalFilter 针对**所有****的路由**配置生效。
    
*   全局过滤器的执行顺序是：
    

*   1> 当 Gateway 接受到请求时，**FilteringWebHandler** 处理器会将所有的 GlobalFilter 实例及所有路由上配置的 **GatewayFilter** 实例添加到一条**过滤器链**中
    
*   2> 所有过滤器都会按照 **@Order 注解**所指定的数字大小进行排序。
    

*   Gateway 内置的全局过滤器又很多，比如：
    
    **GatewayMetricsFilter**——提供监控指标
    
    **LoadBalancerClientFilter**——整合 Ribbon 针对下游服务实现负载均衡
    
    **ForwardRoutingFilter**——用于本地 forward，请求不转发到下游服务器
    
    **NettyRoutingFilter**——使用 Netty 的 HttpClient 转发 HTTP、HTTPS 请求
    

#### 6.2.1> GatewayMetricsFilter

*   GatewayMetricsFilter 是网关指标过滤器，这个过滤器会添加 name=gateway.requests 的 timer metrics。
    
*   添加 Spring Boot Actuator 依赖（Spring Boot Actuator 就是一款可以帮助你监控系统数据的框架, 其可以监控很多的系统数据，它有对应用系统的自省和监控的集成功能，可以查看应用配置的详细信息）
    

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

```

*   application.yml 中开启监控管理 Endpoint，并将所有断点都暴露出来。
    

```
#【网关指标过滤器——GatewayMetricsFilter】
management:
  endpoint:
    gateway:
      enabled: true
  endpoints:
    web:
      exposure:
        include: "*"
#【请求路径匹配路由】
# 请求POST http://localhost:8088/gateway/say会转发http://localhost:8080/say。
spring:
  cloud:
    gateway:
      routes:
        - id: path_routeHeaderRoutePredicateFactory
          uri: http://localhost:8080 #访问地址
          predicates:
            - Path=/gateway/** #路径匹配
          filters:
            - StripPrefix=1 #跳过前缀

```

*   **请求网关接口，才会有 gateway.requests 的内容**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCm5ujicfnzhoFz7Bj2bTkeea7LNaxOdRR0uej3cIM3bXaxQtNDYQ2hxg/640?wx_fmt=png)

*   访问 http://localhost:8088/actuator/metrics/gateway.requests ，将会获得如下指标数据
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCrqglpWbpR97BmdDibPbTv6CIP3yibMngP1BBv74wicyiaXDyicpa89TUiciaQ/640?wx_fmt=png)

*   返回结果
    

```
{
    "name": "gateway.requests",
    "description": null,
    "baseUnit": "seconds",
    "measurements": [
        {
            "statistic": "COUNT",
            "value": 1.0
        },
        {
            "statistic": "TOTAL_TIME",
            "value": 0.161376386
        },
        {
            "statistic": "MAX",
            "value": 0.161376386
        }
    ],
    "availableTags": [
        {
            "tag": "routeUri",            
            "values": [
                "http://localhost:8080"
            ]
        },
        {
            "tag": "routeId",
            "values": [
                "define_filter"
            ]
        },
        {
            "tag": "httpMethod",
            "values": [
                "GET"
            ]
        },
        {
            "tag": "outcome",
            "values": [
                "SUCCESSFUL"
            ]
        },
        {
            "tag": "status",
            "values": [
                "OK"
            ]
        },
        {
            "tag": "httpStatusCode",
            "values": [
                "200"
            ]
        }
    ]
}

```

【tag 解释】

*   **routeUri**——API 网关将路由到的 URI
    
*   **routeId**——路由 ID
    
*   **httpMethod**——请求所使用的 HTTP 方法
    
*   **outcome**——返回的状态码段，值的枚举类定义在 org.springframework.http.HttpStatus.Series 中
    
*   **status**——返回给客户端的 HTTP Status
    
*   **httpStatusCode**——返回给客户端的 HttpStatusCode，如 200
    

七、自定义过滤器  

-----------

Spring Cloud Gateway 提供了过滤器的扩展功能，开发者可以根据实际业务需求来自定义过滤器。同样，自定义过滤器也支持 GlobalFilter 和 GatewayFilter 两种。

### 7.1 > 自定义 GatewayFilter

*   自定义过滤器 GpDefineGatewayFilterFactory
    

```
/**
 * 自定义GatewayFilter
 * 【注意】
 * 类名必须要统一以GatewayFilterFactory结尾，因为默认情况下过滤器的name会采用该自定义类的前缀，这里的name=GpDefine
 */
@Service
public class GpDefineGatewayFilterFactory extends AbstractGatewayFilterFactory<GpDefineGatewayFilterFactory.GpConfig> {
    public GpDefineGatewayFilterFactory() {
        super(GpConfig.class);
    }
    @Override
    public GatewayFilter apply(GpConfig config) {
        return ((exchange, chain) -> {
            System.out.println("GpDefineGatewayFilterFactory [Pre] Filter Request, config.getName() = " + config.getName());
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {//在then方法中是请求执行结束之后的后置处理
                System.out.println("GpDefineGatewayFilterFactory [Post] Response Filter");
            }));
        });
    }
    /**
     * GpConfig只是一个配置类，该类中只有一个属性name。这个属性可以在yml文件中使用
     */
    public static class GpConfig {
        private String name;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
    }
}

```

【解释】

*   类名必须是**以 GatewayFilterFactory 结尾**，因为默认情况下过滤器的 name 会采用该自定义类的前缀。这里的 name=GpDefine。
    
*   在 apply 方法中，同时包含 Pre 过滤和 Post 过滤，在 then 方法中是请求执行结束之后的后置处理。
    
*   **GpConfig 是一个配置类**，该类中只有一个属性 name。这个属性可以在 yml 文件中使用。
    
*   该类要装载到 Spring IOC 容器中，所以使用 @Service 注解实现。
    

*   在 application.yml 文件中，配置自定义过滤器
    

```
#【请求路径匹配路由——配置自定义过滤器】
spring:
  cloud:
    gateway:
      routes:
        - id: define_filter
          uri: http://localhost:8080 #访问地址
          predicates:
            - Path=/gateway/** #路径匹配
          filters:
            - name: GpDefine #自定义过滤器的名字，即：GpDefineGatewayFilterFactory
              args:
                name: Gp_Mic #GpConfig.getName这个值
            - StripPrefix=1 #跳过前缀

```

*   访问 http://localhost:8088/gateway/say ，请求 ok
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCr8RgtkBMkuNEM8YSldgHJ6GzDGdqM9SEKKTibfFjzlQKe4rooyvdRwg/640?wx_fmt=png)

*   查看服务端日志
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCAIgfb4oYWjcgQnqwCkWib76fGY6JpN6Dg7qWUazolS5mGZjJDnicSKEQ/640?wx_fmt=png)

### 7.2> 自定义 GlobalFilter

GlobalFilter 的实现比较简单，它不需要额外的配置，只需要**实现 GlobalFilter 接口**，自动会过滤所有的 Route。

*   GpDefineFilter
    

```
@Service
public class GpDefineFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        System.out.println("GpDefineFilter [pre]-Enter GpDefineFilter");
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            System.out.println("GpDefineFilter [post]-return Result");
        }));
    }
    /**
     * 表示该过滤器的执行顺序，值越小，执行优先级越高
     */
    @Override
    public int getOrder() {
        return 0;
    }
}

```

【说明】

*   需要注意的是，我们实现的局部过滤器 GpDefineGatewayFilterFactory 没有指定 Order，它的默认值就为 0，如果想要设置多个过滤器的执行顺序，那么可以实现 Ordered 接口，并重写 getOrder 方法即可，并且 order 的值越小，执行优先级越高。
    

*   在 application.yml 文件中配置网关的转发规则
    

```
#【请求路径匹配路由】
# 请求POST http://localhost:8088/gateway/say会转发http://localhost:8080/say。
spring:
  cloud:
    gateway:
      routes:
        - id: path_routeHeaderRoutePredicateFactory
          uri: http://localhost:8080 #访问地址
          predicates:
            - Path=/gateway/** #路径匹配
          filters:
            - StripPrefix=1 #跳过前缀

```

*   访问 http://localhost:8088/gateway/say，请求 ok
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCqJqCM07C4nCOBicOAGRzhicMvZWWGu0WLKP4FpCnUzM8TibtRgeY8TtmA/640?wx_fmt=png)

*   查看服务端日志
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCYmlTLcianIGqVLZFERACq1BJvAIy9QvA1KiaB2Fzh9Svib3ORDtaBFQsA/640?wx_fmt=png)

八、Gateway 集成 Nacos 实现请求负载  

----------------------------

利用 Nacos 配置中心功能——可以实现网关**动态路由**功能。

利用 Nacos 服务注册功能——可以实现对后端服务的**负载均衡**。

### 8.1> 启动 Nacos

*   启动 nacos
    
    sh startup.sh -m standalone
    
*   访问 http://localhost:8848/nacos/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCIYJ2oK9aXj4YyK0ISMAj8icgxg1PzL7XIVmbnfZUyvD6N6Csic918PBw/640?wx_fmt=png)

*   整体架构图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCBzcWKNRvosCnZpxOd8UPssB62s4ZZsxZpqxjWcxotLr058G86mVZNQ/640?wx_fmt=png)

【说明】  

*   **gateway-nacos-provider**
    
    提供 REST 服务，并将服务注册到 Nacos 上。
    
*   **gateway-nacos-consumer**
    
    提供网关路由，基于 Nacos 服务注册中心。
    

### 8.2> gateway-nacos-provider

*   添加 jar 包依赖
    

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter</artifactId>
        <version>2.1.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
</dependencies>

```

*   创建 NacosController
    

```
@RestController
public class NacosController {
    @GetMapping("/sayHello")
    public String sayHello() {
        System.out.println("[gateway-nacos-provider]: sayHello");
        return "[gateway-nacos-provider]: sayHello";
    }
}

```

*   在 application.yml 中添加服务注册地址的配置
    

```
spring:
  application:
    name: gateway-nacos-provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
server:
  port: 8080

```

*   为了演示请求负载，将 gateway-nacos-provider 部署两份，端口分别 8080 和 8081。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCSibcdYjIwKE97ib8brFjiaCk1ly5ShFGr2DpUtu7FkYYZIlRX9mDwsvGA/640?wx_fmt=png)

*   进入 Nacos Dashboard 的服务列表，可以看到两个服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCjODhlicdaLicdmjMKEr4qCjwBbcKUwZMMPyf3zlDSv9QJ5zFFOHDph6Q/640?wx_fmt=png)

### 8.3> gateway-nacos-consumer  

*   添加 jar 包依赖
    

```
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter</artifactId>
        <version>2.1.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
        <version>2.1.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
</dependencies>

```

*   在 application.yml 中添加配置
    

```
spring:
  application:
    name: gateway-nacos-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能
          lower-case-service-id: true #是否使用service-id的小写，默认是大写
      routes:
        - id: gateway-nacos-provider
          uri: lb://gateway-nacos-provider #其中配置的lb://表示从注册中心获取服务，后面的gateway-nacos-provider表示目标服务在注册中心上的服务名
          predicates:
            - Path=/nacos/**
          filters:
            - StripPrefix=1
server:
  port: 8888

```

*   启动服务，多次请求 http://localhost:8888/nacos/sayHello
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaC0hnpVarkbfY211EoRRfgaBibJqGugW9Q8nUpEOJnm3eJlECjdzepgdw/640?wx_fmt=png)

*   通过服务端日志，发现请求被转发到两个 provider 服务上。通过 lb:// 实现了负载均衡。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCda0GBgiafGAnpkkXk7J7aibfZwWqiaSfywqAIveLHwJFZR6B81lcvsvjg/640?wx_fmt=png)

九、Gateway 集成 Sentinel 实现网关限流  

-------------------------------

### 9.1> Route 维度限流

*   网关限流规则 GatewayFlowRule
    

```
public class GatewayFlowRule {
    // 资源名称，可以是网关中的route名称或者用户自定义的API分组名称
    private String resource;
    // 资源模型，限流规则是针对API Gateway的route还是用户在Sentinel中定义的API分组。默认是route。
    // SentinelGatewayConstants.RESOURCE_MODE_ROUTE_ID = 0;
    // SentinelGatewayConstants.RESOURCE_MODE_CUSTOM_API_NAME = 1;
    private int resourceMode = 0;
    // 限流指标纬度，桶限流规则的grade字段
    private int grade = 1;
    // 限流阈值
    private double count;
    // 统计时间窗口，单位是秒，默认是1秒
    private long intervalSec = 1L;
    // 流量整形的控制效果，同限流规则的controlBehavior字段。目前支持快速失败和匀速排队两种模式，默认是快速失败。
    private int controlBehavior = 0;
    // 应对突发请求时额外允许的请求数目
    private int burst;
    // 匀速排队模式下的最长排队时间，单位时毫秒，仅在匀速排队模式下生效
    private int maxQueueingTimeoutMs = 500;
    // 参数限流配置。若不提供，则代表不针对参数进行限流，该网关规则将会被转换成普通流量规则；否则会转换成热点规则。
    private GatewayParamFlowItem paramItem;
    ... ...
}

```

*   添加一个配置类 GatewayConfiguration
    

```
/**
 * Route纬度限流
 */
@Configuration
public class GatewayConfiguration {
    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;
    public GatewayConfiguration(ObjectProvider<List<ViewResolver>> viewResolvers,
                                ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolvers.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }
    // 注入一个全局限流过滤器SentinelGatewayFilter
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }
    // 注入限流异常处理器
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }
    // 初始化限流规则
    @PostConstruct
    public void doInit() {
        initGatewayRules();
    }
    /**
     * Route维度限流
     */
    private void initGatewayRules() {
        Set<GatewayFlowRule> rules = new HashSet<>();
        GatewayFlowRule gatewayFlowRule = new GatewayFlowRule("gateway-nacos-provider").setCount(1).
                setIntervalSec(1);
        rules.add(gatewayFlowRule);
        GatewayRuleManager.loadRules(rules);
    }
}

```

*   application.yml 文件配置
    

```
server:
  port: 8888
#【Route维度限流】
spring:
  application:
    name: gateway-sentinel-demo
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      discovery:
        locator:
          enabled: false
          lower-case-service-id: false
      routes:
        - id: gateway-nacos-provider
          uri: lb://gateway-nacos-provider
          predicates:
            - Path=/sentinel/**
          filters:
            - StripPrefix=1

```

*   多次请求 http://localhost:8888/sentinel/sayHello ，当触发限流之后，会出现 {"code":429,"message":"Blocked by Sentinel: ParamFlowException"}，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCdwt4aJD9pyeujGibQX0ib5ClybInEiaWFPFPN4nfWlmyWEUenGINLPGug/640?wx_fmt=png)

### 9.2> 自定义 API 分组限流

*   自定义 API 分组限流实际上就是让多个 Route 共用一个限流规则。
    
*   application.yml 文件配置
    

```
#【自定义API分组限流】
spring:
  cloud:
    gateway:
      routes:
        - id: foo_route
          uri: http://localhost:8080
          predicates:
            - Path=/foo/** #被限流
          filters:
            - StripPrefix=1
        - id: baz_route
          uri: http://localhost:8080
          predicates:
            - Path=/baz/** #被限流
          filters:
            - StripPrefix=1
        - id: aaa_route
          uri: http://localhost:8080
          predicates:
            - Path=/aaa/** #不被限流
          filters:
            - StripPrefix=1

```

*   添加一个配置类 GatewayConfiguration1
    

```
/**
 * 自定义API分组限流
 */
@Configuration
public class GatewayConfiguration1 {
    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;
    public GatewayConfiguration1(ObjectProvider<List<ViewResolver>> viewResolvers,
                                 ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolvers.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }
    // 注入SentinelGatewayFilter
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }
    // 注入限流异常处理器
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }
    // 初始化限流规则
    @PostConstruct
    public void doInit() {
        initCustomizedApis();
        initGatewayRules();
    }
    /**
     * 自定义API分组限流，将/foo/**和/baz/**进行统一分组，并提供name=first_customized_api，然后在初始化网关限流规则时，针对该name设置
     * 限流规则。同时，我们可以通过setMatchStrategy来设置不同path下的限流参数策略
     */
    private void initCustomizedApis() {
        Set<ApiDefinition> definitions = new HashSet<>();
        ApiDefinition apiDefinition = new ApiDefinition("first_customized_api");
        apiDefinition.setPredicateItems(new HashSet<ApiPredicateItem>() {
            {
                add(new ApiPathPredicateItem().setPattern("/foo/**")
                        .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
                add(new ApiPathPredicateItem().setPattern("/baz/**")
                        .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
            }
        });
        definitions.add(apiDefinition);
        GatewayApiDefinitionManager.loadApiDefinitions(definitions);
    }
    /**
     * 针对分组name来设置限流规则
     */
    private void initGatewayRules() {
        GatewayFlowRule rule = new GatewayFlowRule("first_customized_api")
                .setResourceMode(SentinelGatewayConstants.RESOURCE_MODE_CUSTOM_API_NAME).setCount(1)
                .setIntervalSec(1);
        Set<GatewayFlowRule> rules = new HashSet<>();
        rules.add(rule);
        GatewayRuleManager.loadRules(rules);
    }
}

```

*   多次请求 http://localhost:8888/foo/sayHello 和 http://localhost:8888/baz/sayHello ，当触发限流之后，会出现 {"code":429,"message":"Blocked by Sentinel: ParamFlowException"}，但是，由于 / aaa/** 不在分组限流里，所以多次请求 http://localhost:8888/aaa/sayHello 则不会触发限流。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCGegopicnDLAAZjSXQpzDmbTdsHPZgb5TkYzmNaVeE7Sa5P2CG1UYzBg/640?wx_fmt=png)

### 9.3> 自定义异常

*   在前面演示的案例中，当触发限流规则时，会返回 {"code":429,"message":"Blocked by Sentinel: ParamFlowException"}，但是，在实际应用中，一般都是以 JSON 格式进行数据返回。那么针对于这种需求，就需要我们采用自定义异常的方式，来封装返回的异常信息。
    
*   application.yml 文件配置（我们可以借用 Route 维度限流使用的配置）
    

```
#【自定义异常】
spring:
  application:
    name: gateway-sentinel-demo
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      discovery:
        locator:
          enabled: false
          lower-case-service-id: false
      routes:
        - id: gateway-nacos-provider
          uri: lb://gateway-nacos-provider
          predicates:
            - Path=/sentinel/**
          filters:
            - StripPrefix=1

```

*   添加一个配置类 GatewayConfiguration2（关键点：提供自定义异常的 sentinelGatewayBlockExceptionHandler 方法）
    

```
/**
 * 自定义异常
 */
@Configuration
public class GatewayConfiguration2 {
    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;
    public GatewayConfiguration2(ObjectProvider<List<ViewResolver>> viewResolvers,
                                ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolvers.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }
    // 注入一个全局限流过滤器SentinelGatewayFilter
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }
    // 注入限流异常处理器
//    @Bean
//    @Order(Ordered.HIGHEST_PRECEDENCE)
//    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
//        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
//    }
    // 注入自定义的限流异常处理器
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public GpSentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
        return new GpSentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }
    // 初始化限流规则
    @PostConstruct
    public void doInit() {
        initGatewayRules();
    }
    /**
     * Route维度限流
     */
    private void initGatewayRules() {
        Set<GatewayFlowRule> rules = new HashSet<>();
        GatewayFlowRule gatewayFlowRule = new GatewayFlowRule("gateway-nacos-provider").setCount(1).
                setIntervalSec(1);
        rules.add(gatewayFlowRule);
        GatewayRuleManager.loadRules(rules);
    }
}

```

*   GpSentinelGatewayBlockExceptionHandler 自定义异常处理器（关键点：提供自定义异常的 writeResponse 方法）
    

```
public class GpSentinelGatewayBlockExceptionHandler implements WebExceptionHandler {
    private List<ViewResolver> viewResolvers;
    private List<HttpMessageWriter<?>> messageWriters;
    private final Supplier<ServerResponse.Context> contextSupplier = () -> {
        return new ServerResponse.Context() {
            public List<HttpMessageWriter<?>> messageWriters() {
                return GpSentinelGatewayBlockExceptionHandler.this.messageWriters;
            }
            public List<ViewResolver> viewResolvers() {
                return GpSentinelGatewayBlockExceptionHandler.this.viewResolvers;
            }
        };
    };
    public GpSentinelGatewayBlockExceptionHandler(List<ViewResolver> viewResolvers,
                                                  ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolvers;
        this.messageWriters = serverCodecConfigurer.getWriters();
    }
    /**
     * 该方法的作用是，将限流的异常信息写回客户端
     *
     * @param response
     * @param exchange
     * @return
     */
    private Mono<Void> writeResponse(ServerResponse response, ServerWebExchange exchange) {
        ServerHttpResponse serverHttpResponse = exchange.getResponse();
        serverHttpResponse.getHeaders().add("Content-type", "application/json;charset=UTF-8");
        byte[] datas = "{\"code\":999,\"msg\":\"访问人数过多\"}".getBytes();
        DataBuffer buffer = serverHttpResponse.bufferFactory().wrap(datas);
        return serverHttpResponse.writeWith(Mono.just(buffer));
    }
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        if (exchange.getResponse().isCommitted()) {
            return Mono.error(ex);
        } else {
            return !BlockException
                    .isBlockException(ex) ? Mono.error(ex)
                    : this.handleBlockedRequest(exchange, ex).flatMap((response) -> {
                        return this.writeResponse(response, exchange);
                    });
        }
    }
    private Mono<ServerResponse> handleBlockedRequest(ServerWebExchange exchange, Throwable throwable) {
        return GatewayCallbackManager.getBlockHandler().handleRequest(exchange, throwable);
    }
}

```

【说明】

*   所有代码都可以直接从 SentinelGatewayBlockExceptionHandler 中复制过来，我们只需要修改 writeResponse 方法即可，该方法的作用时将限流的异常信息写回客户端。
    

*   多次请求 http://localhost:8888/sentinel/sayHello ，我们可以看到自定义异常信息。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCMdUhU92sgkHo5B4pYVGI4g4b2HjoKYvO3fHPibHvKnKHEEGqVDj03eQ/640?wx_fmt=png)

### 9.4> 网关限流原理

*   流程图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibvTpibsgT7jGZrv0fbxj5iaCOzibQPro3HicXLdb0OZ90CHKF3H3SibEIDCdnYbPBtOVRrdFjgiaQt0osw/640?wx_fmt=png)

【解释说明】

*   **GatewayFlowManager**
    
    通过 GatewayFlowManager 加载**网关限流规则**（GatewayFlowRule）时，无论是否针对请求属性进行限流，Sentinel 底层都会将网关流控规则 GatewayFlowRule 转化为**热点参数规则**（ParamFlowRule）存储在 GatewayFlowManager 中，与正常的热点参数规则相互隔离。在转化时，Sentinel 会根据请求属性配置，为网关流控规则设置参数索引（idx），并添加到生成的热点参数规则中。
    
*   **SentinelGatewayFilter**
    
    在外部请求进入 API 网关时，会先经过 SentinelGatewayFilter，在该过滤器中**依次进行 Route ID/API 分组匹配**、**请求属性解析**和**参数组装**。
    
*   Sentinel 根据配置的**网关限流规则**来解析请求属性，并依照参数索引顺序组装**参数数组**，最终传入 SphU.entry(name,args) 中。
    
*   在 Sentinel API Gateway Adapter Common 模块中，在 SlotChain 中添加了一个 GatewayFlowSlot，专门用来处理**网关限流规则的检查**。
    
*   如果当前限流规则并没有指定限流参数，则 Sentinel 会在参数的最后一个位置置入一个**预设的常量 $D**，最终实现普通限流。
    
*   实际上，在网关限流中，我们所配置的**网关限流规则**最终都会转化为**参数限流规则**，通过 **ParamFlowCheck.passCheck** 进行参数限流规则检查。