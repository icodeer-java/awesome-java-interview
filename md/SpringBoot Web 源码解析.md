> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486972&idx=1&sn=74556f31641a1bd3c29a2663bc533208&chksm=e9114f01de66c6177f47b26297b0836b5df0e92feea7a7420f611b3d3a2f8926eb952d20b117&scene=178&cur_album_id=2208187391738249217#rd)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpicGt7BenMFuoWJ0JRqMmy5M4icVOO3AZhTuccibHWW4qYnbfJYNhg8oRA/640?wx_fmt=png)

一、静态资源  

=========

1.1> 静态资源访问
-----------

*   官方文档 7.7.1 The "Spring Web MVC Framework"，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpWvno6VtC0ODbTkNolwYNEF2IIkcxCxMMmsSSSUMMrEL5AiajL5Jv2cw/640?wx_fmt=png)

*   静态资源访问路径为：/static、/public、/resources、/META-INF/resources
    
*   通过【当前项目根路径 / + 静态资源名】即：http://localhost:8080/kangxi.jpg 的方式，访问静态资源。
    

*   静态资源访问原理：请求进来，先去找 Controller 看能不能处理。如果不能处理，则尝试去寻找静态资源。
    

即：如果我们有一个 Controller 的接口，请求地址也是 http://localhost:8080/kangxi.png，那么则会访问该 Controller，而不会访问静态资源。

*   实战一下，尝试把静态资源放入以下四个路径下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpCpvNHYjwr2yX4XVTialKsZ6MOmzHM1RiboSdTo4JuVqIwpT1mjIFxoOA/640?wx_fmt=png)

*   访问如下请求，都可以查看响应的静态图片：
    

```
http://localhost:8888/qianlong.jpg
http://localhost:8888/yongzheng.jpg
http://localhost:8888/kangxi.png
http://localhost:8888/mingchao.jpeg

```

*   由于默认静态资源的请求路径是 /**，如果想改变静态资源的请求路径，也可以通过如下两种方式：
    

```
spring.mvc.static-path-pattern=/static/**

```

或

```
spring:
    mvc:
        static-path-pattern: "/static/**"

```

*   访问如下请求，都可以查看响应的静态图片：
    

```
http://localhost:8888/static/qianlong.jpg
http://localhost:8888/static/yongzheng.jpg
http://localhost:8888/static/kangxi.png
http://localhost:8888/static/mingchao.jpeg

```

*   如果我们想要自定义一个目录用于存放静态资源，我们可以使用如下方式指定
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibprbOemWP3IpJVtqcsgc1OcTGCvwOqCtjqL6cqXkh7LYO4fiarPvSwFibQ/640?wx_fmt=png)

1.2> 欢迎页  

-----------

*   官方文档 7.7.1 The "Spring Web MVC Framework"，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpWKyekC6gShia2bputgfst9zU1wXL2cPwrHnYI6lvYVejLhPHeP1BlRQ/640?wx_fmt=png)

*   有两种方式支持欢迎页 http://localhost:8080
    

*   方法一：静态资源路径下放入 index.html（不能配置静态资源前缀，否则失效）
    
*   方法二：Controller 里面有支持 /index 路径请求的接口方法
    

1.3> 自定义 Favicon
----------------

*   静态资源路径下放入名为 favicon.ico 即可。（不能配置静态资源前缀，否则失效）
    

1.4> 静态资源配置原理解析
---------------

*   相关源码在 WebMvcAutoConfiguration.addResourceHandlers(...) 方法中，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpvP3VQ4B2514r4UeGDkwXJYpNT6CZhiboVXwYQpDCduiaFfhySAXzhtibg/640?wx_fmt=png)

*   **spring.web.resources.add-mappings** 的默认值
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpGAhQjNHt7OcX7e5hOGDhBLXuIbPRVc7e65fwDibKxBmsCcTOxeaJxSw/640?wx_fmt=png)

*   **spring.mvc.static-path-pattern** 的默认值
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpXYj9WgIklGUGmhT4MG13o4ib1kCibm039MsI1aakuYhW1NbuyphmU7YA/640?wx_fmt=png)

*   **spring.web.resources.static-locations** 的默认值
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpEvsKoDgnRnoNaxoX7D3R48qic32gqCNRNtuKkn25ibd9teAAr2KIqaibg/640?wx_fmt=png)

*   欢迎页相关代码，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpOYsQu79fmAQVojwTAIgnfP9ZVbwokhMOdcr8p6icM386rCYS4Pe2yxw/640?wx_fmt=png)

二、Rest 请求映射  

==============

2.1> 概述
-------

*   请求路径，采用 @RequestMapping 或 @XxxMapping
    
*   Rest 风格支持（使用 HTTP 请求方式动词来表示对资源的操作）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpeWMLqNiaryiagNaG6tKKvnic18F4phAK89Otib4YZL20ub8btvafKAWCFg/640?wx_fmt=png)

*   核心 Filter：HiddenHttpMethodFilter
    
    用法：表单 method=POST， 隐藏域 _method=PUT/DELETE
    
*   Rest 原理（表单提交要使用 Rest 的时候，因为表单提交只支持 GET 和 POST 两种；如果用 Postman，则无所谓了）
    
*   表单提交会带上**_method=PUT** 或 **_method=DELETE**
    
*   请求会被 **HiddenHttpMethodFilter** 拦截，具体处理如下所示：
    
    请求是否正常，并且是 POST
    

*   获取到_method 的值
    
*   兼容以下请求：PUT、DELETE 和 PATCH
    
*   原生 request(post)，包装模式 requestWrapper 重新转换了 getMethod 方法，返回的是传入的值。
    
*   过滤器链放行的时候用 wrapper。以后的方法调用 getMethod 是调用 requestWrapper 的。
    

2.2> 具体操作  

------------

*   **student.html**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpMeIKicibUC3avOquxN0xV8ngX4IvCCaTFb44mGgibIZc2qYpt6od84uPQ/640?wx_fmt=png)

*   **RestDemoController.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp0iaBdWxTTE5KibTEAcG76KHGLYnDTKv5kZoL7zIXXTzxpUEV5RgdkgUw/640?wx_fmt=png)

*   添加配置文件，开启 HiddenHttpMethodFilter 过滤器
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpRAJveg7pbPfGVYaQsHOS7NByL3Yhx2F0aWg0XdDF4EVznIXX26olIg/640?wx_fmt=png)

*   请求测试 http://localhost:8888/static/student.html
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp3K6VcwMdRkGt22YjkhRrcNf5MX5bnpfxhBlmCFn61uWM9hOTd7aibzQ/640?wx_fmt=png)

2.3> hiddenMethod 源码解析  

-------------------------

*   配置开启 hiddenMethod 过滤器，CTRL + 鼠标点击对应配置项
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpAWAduMPU4QQsibLPibEuu8Xmq0WnEItWSYW2hibk9Y7el4prjzn8SD6LA/640?wx_fmt=png)

*   跳转到 spring-configuration-metadata.json 文件中
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibptQu0iaMnbPmFcNhWN45gaXFNws1s8FlcpKUiaK1ys5uQSr6sulLslF0A/640?wx_fmt=png)

*   全局搜索一下 “spring.mvc.hiddenmethod.filter”
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpgHbSG2sB8IjmeVugXNJ4qP8ib1mhnuibRwLNZicCBTa3t2DumBTgWGUXQ/640?wx_fmt=png)

*   WebMvcAutoConfiguration.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpSCiagbjd2PCLP3RWcBRicEGHPHNrw0EsbZWjzj7E0A59Ul5E7FXLibMcA/640?wx_fmt=png)

*   OrderedHiddenHttpMethodFilter.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpr38GukGKIMr7T8RPNp93tT1gEjZzegJ6sR15zvyK8c9Ih8tMFpm4icg/640?wx_fmt=png)

*   HiddenHttpMethodFilter.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpd9RrjGUdPn0qnFbJT8EfhkHfhyUAFtFicW2ibOM05ceo0ibodOcZvIJWw/640?wx_fmt=png)

三、SpringMVC 请求映射原理  

=====================

*   当我们执行 http://localhost:8888/student?method=GET 的时候，会通过 DispatchServlet 进行请求的转发处理，但是它没有实现 doGet 方法，而是由它的父类 FrameworkServlet 中实现了 doGet 方法。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp1RVK99fCKgetfRDu3hsPjFncdeVTfwiaDJUOoiaBHa3GqNVgls7cOhgg/640?wx_fmt=png)

*   FrameworkServlet 中的 processRequest() 方法用于处理 Request 请求。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpsPFq1R16LkSic9wJw2qPicor5Nt4V3iczdiaHbz62TgXySvaQMgibsr35mA/640?wx_fmt=png)

*   DispatcherServlet.doService(...)，其中 _isIncludeRequest(request) 方法究竟是什么作用呢？_要想明白这个问题，我们可以借助一条 JSP 的指令来理解：<jsp:incluede page="xxx.jsp"/> ，这条指令是指在一个页面中嵌套了另一个页面，那么我们知道 JSP 在运行期间是会被编译成相应的 Servlet 类来运行的，所以在 Servlet 中也会有类似的功能和调用语法，这就 RequestDispatch.include() 方法。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpwxSBMt7Q2XQZJ6UzZHTB1dPibIBKd9tlm3t0lbNRAd1xwJ4TbkPzbxg/640?wx_fmt=png)

*   DispatcherServlet.doDispatch(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpj9ueEXu8MhtG8X5Mz72U0WSgVLhaicmvla19bK5urn6LowxLoKiaABXg/640?wx_fmt=png)

*   DispatchServlet.getHandler() 方法尝试遍历所有 HandlerMappings，从中获得 HandlerExecutionChain 实例，如果不为空，则表明找到了，返回该实例即可。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpbKQ0qdOtS9jb5f4p8nUyez0ETv3qItwqCBhxQCpYic0Pw2fJvbbWETA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibperqW0DsYSLFDtFKCq3Sn1GGkruxLRRatD7YkiakibpNyoKibOFzbUvNNw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpaR0jUh4bEu0cf07zvL2nL0mRpdXefzvSnLkqJWLr1aAv9Iw9w1e2pQ/640?wx_fmt=png)

*   AbstractHandlerMapping.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpoINw8ThGpo1eLj5OGELicgIIF51N12BFKVlElSpMf4KARmpjfk6uUOg/640?wx_fmt=png)

*   RequestMappingInfoHandlerMapping.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpqUe4N8iaJbB2QHzJvlYhLKaUqKBy3EhAYlX42q5ibt3uNbmFnhe3PIicg/640?wx_fmt=png)

*   AbstractHandlerMethodMapping.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpG0etbh4ic8Ln7icAaPqFfu8YJBdbWULtIeZKmhCWmNAQ0LBPsTv7epxg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp5BUu0D4ibuukYn4ku4sibLvYVkia0LJI6p7mbehKsjGKZycqcxibaeuNvw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpdSFlXsf2sHf8sORGnnyodBCJrN81OwBrPYokdYze67CpScEz3lqUaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpf691OBibg4fWCm2N3aNz4Bgy0COphRL09oHJ3bj0C1avEtPXriaNYlnQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpldzF9xczP5BAedbG4P4xiaOk4Q4mfsv2bEvMo2c4vme1uReOUL1PCkw/640?wx_fmt=png)  

*   整体处理流程，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpCntQY8IPB4ibstQF1MicsyLw5S1PNsX36XogdyBJbozHbUVia9BtoDMcw/640?wx_fmt=png)

*   补充内容：HandlerMapping 的 bean 初始化
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpE5vopibIALbql8RAAoOfVQvUlibyCib3WWnI3zYRogibichlYjB1nppFSwA/640?wx_fmt=png)

*   总结
    

*   SpringMVC 功能源码分析都从 org.springframework.web.servlet.DispatcherServlet 的 doDispatch(...) 方法开始。
    
*   所有的请求映射都在 HandlerMapping 中。
    
*   SpringBoot 自动配置欢迎页的 WelcomePageHandlerMapping，访问 / 能访问到 index.html
    
*   SpringBoot 自动配置了默认的 RequestMappingHandlerMapping
    
*   请求进来，挨个尝试所有的 HandlerMapping 看是否有请求信息
    

*   如果有，就找到这个请求对应的 Handler
    
*   如果没有，就找下一个 HandlerMapping
    

*   如果我们需要一些自定义的映射处理，我们也可以自己给容器中放入 HandlerMapping
    

3.1> 注解的使用  

-------------

*   @PathVariable\@RequestHeader\@ModelAttribute\@RequestParam
    
    @MatrixVariable\@CookieValue\@RequestBody
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpiatnDYU91gzT3cQib7EJSB2A6eDRyDCC6UfibXjRvGoaArqArviawsicaaw/640?wx_fmt=png)

3.2> 注解源码解析  

--------------

*   针对上面的接口执行请求，即：http://localhost:8080/get/aaaaaaa/detail/bbbbbbb?name=muse&age=10
    

*   原理概述
    

*   在 HandlerMapping 中找到能够处理请求的 Handler，也就是 Controller 的某个方法。
    
*   为当前的 Handler 找到一个适配器，即：HandlerAdapter
    

*   通过调用 getHandlerAdapter(...) 方法，获得 HandlerAdaper 对象
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp3ticobGiam1gkfPNx64nHBKOxXEwL4qtQVicpKmnZKcYXW4ibsWnIhR7Yg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibppIIy5kibXmNn01CaxF2adkKtVrXvFoaPSZmJCHNbcoY2S8OHlybyibTA/640?wx_fmt=png)如下图所示，默认有四种 **HandlerAdapter**：  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpeEaPnbhVwzdcqCv2zdFjvlog8D2EgZcGZN6jazxP0M0JtxvzmBia5bw/640?wx_fmt=png)【注】

*   RequestMappingHandler：支持方法上标注 @RequestMapping
    
*   HandlerFunctionAdapter：支持函数式编程
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpaPvg9ibvx8LCeeNfLwMq9lZPy5HnGXLO8f70sjeCqEhKZZMOhCajzKg/640?wx_fmt=png)

如下图所示：返回第一种 RequestMappingHandlerAdapter

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpTkrhMIBQ1VWKmyQGq4ClINWic3GiaPSoIrqoKw9nZvLiaZ9ap6WjTyX3A/640?wx_fmt=png)

*   执行完 getHandlerAdapter(...) 方法后，继续往下执行，lastModified 值为 - 1
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpxERWFhnPqrs6JKv249sIKVcs6iaWgofjibFD03cwDRWMA67qBnI1aYSg/640?wx_fmt=png)

*   checkNotModified(...) 方法用来判断，是否有改动
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibplLEIyBH1BJM7RRYHaM4qpBmicD7iaIpy4A0EE80eEr5GBUHyMNgsJaSA/640?wx_fmt=png)

*   继续往下执行。我们看到关键的方法 ha.handle(...)，真正的调用 handler 的方法。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp6YGcydCOd5EneSTnZUerL4QbPzvWAoA0HicWhraIdFicV5j9EKIsoX1A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpFMJHe19dt7Nl5dApfWTpTPrnYrib3x8vlQzLG1CjDIuHMQWX92HsprQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpeZXtHLDGwDHhSlbVZO8ibicF62ynNVumhIkvNPAYqNHP7uEicIBzqbrhw/640?wx_fmt=png)

如下图所示：获得 27 个 argumentResolvers 参数解析器——HandlerMethodArgumentResolver，用于对请求入参进行解析  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpBGfKQHGMt6frwvGQOjGN1ibAqr0DJVokp1o2Zzwyiax7ugzI2zKRdFhQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpIEL30CzyCkrYP1P1bnvdZvsZdS8dqORycjrUhprW8cwuw0CS51vT2A/640?wx_fmt=png)  

获得 15 个 returnValueHandlers 结果解析器，用于对结果进行处理

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp85cOu4vuoicEMk3qTIj3xEd2WzHbLPHBsG6YRK9U3xfIOib9Vdw0ibvIA/640?wx_fmt=png)

*   跳过中间不重要的方法，我们直接到第 894 行的 invokeAndHandle(...) 方法，这块是真正执行请求处理的地方
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp4ichBgZNaSZ2jcZYib6DIibgGvGXSib5eNoiaRw6S3Mib7aYic5pmW9nFVHHA/640?wx_fmt=png)

调用 invokeForRequest(...) 方法，下一步，会直接跳到响应 Controller 上对应的处理方法，执行完毕后，将返回值赋值 returnValue

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpWQicSEYkUJfy3AttVSnhE6gY7QVwwGP3aXziaDyI47aJZ1Gx80y9BshQ/640?wx_fmt=png)

下图中，doInvoke(args) 方法，利用反射，调用 Controller 中的响应方法。此方法就不过多说明了。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpmKIaUd4RTUDKoVVBZDqUmVbaCC0VHTYVqjTxrrxHkxU8b4STCNQMicQ/640?wx_fmt=png)

我们回过头来，看 doInvoke(args) 方法上面红框中的 getMethodArgumentValues(...) 方法是如何确定参数值的。下图中 parameters 表示方法所有的入参声明（即：参数所在的位置，使用的注解是什么，入参类型是什么，等等）。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibps1mHs9BicpDhNz9JgfVeweSMWDYDnbvLtzqibZDrDxe3csF820XJk9ow/640?wx_fmt=png)

如下图所示，由于 providedArgs 是空的数组，所以都会返回 null

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpk9lfUkgy695dZ8qef337aRQS7RYnYOSLqaq766zBxBWQoZiaPGwljTQ/640?wx_fmt=png)

红框代码用来判断当前入参是否可以被解析。会在外层循环中，将所有的 Controller 的入参都进行一次遍历的校验。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpCznJzibmIKX6AziaqIUibDnOHnRKic26J7J7OibeIwK5oyYotHIoJ2C0ukg/640?wx_fmt=png)

以入参以 @PathVariable("orderId") String orderId 为例，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpYtgBeyz7dn9a2qTdTDVPkDd6icTDlxvFGJfXTjCS5QxEPib9ZKoffu8A/640?wx_fmt=png)【注】上图含义说明：

*   针对 @PathVariable("orderId") String orderId 采用 PathVariableMethodArgumentResolver 解析器进行解析
    
*   针对 @PathVariable Map<String, String> pathVariableMap 采用 PathVariableMapMethodArgumentResolver 解析器进行解析
    

*   下图中的 resolveArgument(...) 便是参数解析的重点处理逻辑
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibprg3PeHAJ5CYbicIBjF0Sk1JAMMxjgXicib0t6eo9hjg0TXEZsAbGibDDjQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpLFStGM75B1sPyKJGsN7gqdwdT7v2dkic9CCd8oJtjLa3lHdDp4fLCAw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpvdGmARTeeqq4w81F35horSrn5QEAbfLVZht7Hqt665JXI0EZGJMBIg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp6okUKtDpXrEzgescLZicuia7wYviab9licDicX3kTf4bo37ND1ibBC3y0guA/640?wx_fmt=png)  

3.3> Servlet API  

-------------------

*   请求类型如下所示：
    
    WebRequest
    
    ServletRequest
    
    MultipartRequest
    
    HttpSession
    
    javax.servlet.http.PushBuilder
    
    Principal
    
    InputStream
    
    Reader
    
    HttpMethod
    
    Locale
    
    TimeZone
    
    ZoneId
    
*   由 ServletRequestMethodArgumentResolver 对上面参数进行解析，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpl7EibiaxuQvOWLhCibV2DBtq0vMIx3tpgQY8m1vBAaiba0JgBCIGUnGLAw/640?wx_fmt=png)

3.4> 复杂参数  

------------

*   Map、Model
    
    以他们作为参数，就相当于 Map 和 Model 里面的数据会被放在 request 的请求域中，即：request.setAttribute(...)
    
*   RedirectAttributes
    
    重定向携带数据
    
*   ServletResponse
    
    Servlet API 中的 response 响应
    
*   Errors/BindingResult、Model、RedirectAttributes、ServletResponse、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder
    
*   举例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpA9uHzoPbRsibkVmCXQdcGwkQ5Snxqv3HUdbjLibCh1GR86ibia4jZQh5KQ/640?wx_fmt=png)

3.5> 复杂参数源码解析  

----------------

*   Map 入参由 MapMethodProcessor 进行解析，Model 入参由 ModelMethodProcessor 进行解析。
    
*   目标方法执行完毕后，会将所有的数据都放在 ModelAndViewContainer 中，包含要跳转的页面地址 view 和 model 数据。
    

*   由于 2.5.3.2 已经对请求流程做了分析，所以，针对 Map 和 Model 请求参数的源码分析，我们从下图红框处开始：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpeqgQJ71CTXxbCoOmqcTzYiceY5oRbfSIjgSgcVa9iaiapsMpXYicibKoDAw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpIfibfc7Exe3NQOftQCBu1pa2MRCIDz9V3SGfrm3OtaPAT9ZmRnibB8rg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpUvqUOHQXoI3XzfYrg0loJm0MnskR06tzzFuMBYmu4JOf8Ap11fbnPQ/640?wx_fmt=png)

*   参数解析完毕后，调用 invoke 方法，通过反射调用 Controller 中响应的请求方法。调用完毕后，运行下图位置：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp0WDOELuCPc4U5MiaSfCU5drKsPZ7EkKrViaB3xMEphpv6NLHh3YNdQMw/640?wx_fmt=png)

*   下面，我们来看一下，defaultModel 中的这两个值，是如何放到请求属性中的？执行完 invokeAndHandler 方法后，跳到上一层，在 getModelAndView 处，获得了模型和视图对象。一直继续往上层走。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp7wESJNnHYSWXbmhlxibicK57nwLZ3xSCFyJO0EwPQ83PG9Qib5ibn8pXeQ/640?wx_fmt=png)

*   一直往上层代码走，直到 DispatchServlet 的 doDispatch(...) 方法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpM5Sicj0hXISENkC4ZSWfIJkqCsSbnEPSryQFXBPkMVAGS95nnCnOZyQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpodv4kh5FWwb8UUI5TL5QFrZCncqQUZXPnXwSs0mEgmv5bBDibdoSWWQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpvXylGrelA8acsEfTAkibknDYxUxzIfiac8r7AXbwzbj4qjIyIGjtnrwA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpfvpXk4N8iaGMpRTjoMRkJK4QsMAHEkf7yPPiaY1lVBJFxdc2XfJr4jcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpZoTcKZx2HOWR1JbbQR71fCcGjicj8amwGica6UELgLcE0hu511JkqE9w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpKibWHbdmFbSGxhjDrOlGZPeib0IQ8icJOIficF408wR5ZElAYkFETOODZg/640?wx_fmt=png)  

3.6> 自定义对象参数  

---------------

*   可以自动类型转换与格式化，可以级联封装。
    
*   举例
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpvmtxmTsptYlYibcoHasLfeR2GorR2PL63uib7IzNYibaNFwV8ib8XHGswQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpMMDbs6eotCX7coAdzhFY8k49t3DlT6NtibMD0uo382TU14gSV3fzxaQ/640?wx_fmt=png)

3.7> 自定义对象参数源码解析  

-------------------

*   我们来验证一下，请求参数如何赋值给对象的
    
*   由于自定义类型参数是由 ServletModelAttributeMethodProcessor 进行解析的，但是，它没有提供 supportsParameter(...) 方法和 resolveArgument(...) 方法，所以我们直接看这个类的父类 ModelAttributeMethodProcessor 的 supportsParameter(...) 方法：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpxciajzwJNY6MxtLA3tb7ZILpq71Ymar6YEP0d6SscO4TraXPceA2ZyQ/640?wx_fmt=png)

*   我们再来看一下 ModelAttributeMethodProcessor 的 resolveArgument(...) 方法：
    
*   WebDateBinder，即：web 数据绑定器；作用是将请求参数的值绑定到指定的 JavaBean 里面。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpjOlYc2icLV0dG0pjYicutO8kkzDq8b0kibvdvuRKScCmgOrYJg0zOvyWA/640?wx_fmt=png)

*   WebDateBinder 利用它里面的 Converters，将请求数据转换成指定的数据类型，然后再次封装到 Teacher 中
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp9TH9w8AAQpmHLukQkoicugIHwu4IaIMousEgXo8UMAP9l8o1lU6P7Sg/640?wx_fmt=png)

*   binder 里包含了空属性的 Teacher 对象和 124 个类型转换器，webRequest 中保存着 web 请求参数信息。然后通过 bindRequestParameters 方法，将 request 的请求数据通过类型转换器赋值给 Teacher 对象。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpjQkx1EFvuITjTdasg12p9YlhaX1wiaQRVZxfU3yjsTPLRQmGZnj2clA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpc2WlUj3a7VZZcHLNxzzKWDIYDg1SKff5AdX8P8BBJGKDE7RGIz5gxQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpxxbicuRYuRTVRdykXJ588eqqicWjHy9U6gzBe9SibFCIickxDrIjCpKjPw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpcicR8yV2wvN3BbEOrnuMjHichM2Q4H2oFtLC1t7Wdg4N0AMh6ZpTv4ibw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpnmf0YzsBykNB94yxq0qK20FAuoicTxPIkOWMhxIcpFwXxibDOeMKNFXw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpwUAHWbwwGVWnxialNDgbLpAuzY0icY57K5Xt6EBZztkicC72ic9WPheNRA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpt9GTZKXm2ibDI5GtAJJ3Lr0mtkv1INnvuMG3ezmcAYTBruiaG1ZV00tw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpLuG7rZflIm2TbFU2anDcntQmq2SYeaPKe6SAdIpzFQzcNnXvJKobKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpH3aBq9S2dISbibPTudVGrQscxYk5OiaibScqcSLwJP9Q8zJsQQOVtlkXg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpHYebuDuJQTKJ79CEW7CiajplaYuibKicwazBnve33nZemYoSZ7DbMoCYg/640?wx_fmt=png)

*   我们往上层走，回到 canConvert 这层方法中，
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp7pRFwIBMwQHz69tf1ykiasPpaqMEvDPgvNwV9GCa9OEk9fqxRR2YogQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpjMhsBtXib7569KpKUljqarBqibwhdpeXVDtNYnRczQppYMlrFibicq2wkA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibp8LhU6Rh9oHEibRiclykPSa7shE1D14JgRdEvbtAGibtNQibzKzCJb7Aiaiaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpCYlfKcfwWyEV26TAJaTIVJiay4CzGJrHczy3ic7vnPDuWOHnOR6fMN8A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpsW1zibAT4OVcF85Ar5CUXrJI0ouWvYx2JMTQdpib22A6RcymSBnpkTUQ/640?wx_fmt=png)

*   类型转换完毕后，往上回到 processLocalProperty(...) 这个方法，对 ph 进行赋值操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpzpJQFTy2GExD5JowgEYt1ib71EnoD74ToAK1ItDDqXMLHBW5HmWPs6w/640?wx_fmt=png)

3.8> 实现自定义 Converter  

-----------------------

*   有 teacher 接口，用来保存教师信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpO0CaeBxmXo7zicbnZ9KaRpfsKp2mUEqrWDPGiaLia6ojlmQrichElYaY9w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpKGgt6Oc98Kmt30c52pGHs9riasiaRNUL886iaYFZwHaQ8Gc07eLib0mcHQ/640?wx_fmt=png)

*   希望请求 teacher 接口保存教师信息时，通过 teacher 参数，将 name，age 和 sex 的值用逗号分割进行赋值。
    
*   创建 WebMvcConfigurer，并重写 addFormatters 方法。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpE9s0eFwWs97WcOQhPoPicRXSAS7lLMAJnK4RxOtj7a6G2VqlOCz6bWQ/640?wx_fmt=png)

*   测试结果
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicd4kbIMa28icuYRQrUIfKibpw1icaRzUou5xjRDQ6ZHmxuSicBg8DAoX7ziceAyKta78Gl4te1Xafic6LQ/640?wx_fmt=png)