> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491585&idx=1&sn=8111bc892cafcf64af59b1a05e7e6acc&chksm=e912a2fcde652bea38e954cf2fc7a8d802ea1e332cf4c1eb538dd51d34631c9f1bf2bcfb72c6&scene=178&cur_album_id=2208187391738249217#rd)

一、使用示例
======

**步骤 1**：创建`User`实体

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3JqJtrY4FeibemnTmtu3vNOFaQNzeCTLcPjA4r3TNgCrL0EVOsUnJBTw/640?wx_fmt=png)

**步骤 2**：定义一个`XSD`文件描述组件内容

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3iaMpD2bGN6RYv2I7MvDCauGAKZt74ufGjfVEIUe1aujMwERgjicb41zg/640?wx_fmt=png)

**步骤 3**：创建`BeanDefinitionParser`接口的实现类，用来**解析 XSD 文件中的定义和组件定义**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3E22ZgHtkV4eoYlJBMIKWueDvXzaFrgxhwNh1lrFvvPaB7QvTg0cHfA/640?wx_fmt=png)

**步骤 4**：创建`NamespaceHandlerSupport`实现类，目的是**将组件注册到 Spring 容器中**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3HtFZG7xwpOkPSZsQEaVYjBYeGibvSsYP9ceibhoFvBo8B35ib3XibKwIFQ/640?wx_fmt=png)

**步骤 5**：编写`spring.handlers`和`spring.schemas`文件，默认位置是`/META-INF`目录下

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3X9Ioib0Nru6Obf5CgjYTFzA793EXyYYtZRKhFQQ5eNLnmJFCjaARYpA/640?wx_fmt=png)

**步骤 6**：在配置文件`oldbean.xml`中引入对应的命名空间以及 XSD 之后，就可以配置`<myname:user ... />`了

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3OPEzxxzs2E5bKvazRNw5z1yUbMS2MDpwUM31Chpa2s5bh8tQLCm6iaA/640?wx_fmt=png)

**步骤 7**：进行测试

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3k9vxlNTd8rcAhkiasNrQUeyAic1Kjo3bVdgPbluJezyUMwm6Ze201jPQ/640?wx_fmt=png)

二、源码解析
======

在第 2 讲中，我们已经介绍了关于**默认标签**的解析过程。那么我们还是需要将视角在回到`parseBeanDefinitions(...)`方法上来，从下图源码截图中我们可以看出来，我们首先是来判断`root`和`root的子节点`是否是**默认表空间**，即：通过`delegate.isDefaultNamespace(...)`来进行判断：

> 【**如果是默认表空间**】执行`默认标签`解析——delegate.parseDefaultElement(ele, delegate);  
> 【**如果不是默认表空间**】则执行`自定义标签`解析——delegate.parseCustomElement(ele);

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3XOibru2eVHF5sKY8hfvwzSndIIoFr0zmhXjp91rRXTCS7jqRGhAa7gQ/640?wx_fmt=png)

下面我们来看一下`parseCustomElement(...)`方法的具体实现，具体来说有如下 3 个步骤：

> 【**首先**】获得`namespaceUri`, 此处是通过 **org.w3c.dom.Node** 中的`getNamespaceURI()`方法进行获取的；  
> 【**其次**】获得解析该自定义标签的`NamespaceHandler`实现类。  
> 【**最后**】调用该实现类的`parse(...)`方法进行解析操作。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3b3m435q1dFebAOmMt75cvVgOGW3dSh3TESAGxP2tSeuq75OydLY8lg/640?wx_fmt=png)

2.1> getNamespaceURI(ele) 方法解析
------------------------------

此方法是用于**获得 namespaceUri**，此处是通过 **org.w3c.dom.Node** 中的`getNamespaceURI()`方法进行获取的，以使用示例中为例，将会返回的 **namespaceUri=“http://www.muse.com/schema/user”**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3lXj9ZsMhsfjzUGXyq3vzhOM2ibNb8vYZ5gib2XBkG1A51oqtbGJKG9iaA/640?wx_fmt=png)

2.2> resolve(namespaceUri) 方法解析
-------------------------------

此方法是用来**获得解析该自定义标签的 NamespaceHandler 实现类**，为下图中红框处代码：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3fHPQEIiaRtoOPemRAiatN6MqAZnV5dHPBR2X6KYruGVkn5VbCiacUNtFw/640?wx_fmt=png)

在此处的`this.readerContext.getNamespaceHandlerResolver()`方法中，实际会返回 **DefaultNamespaceHandlerResolver** 实例对象。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3LB8hCia5T9AY8tia55QewkG1XODbceWo1QycSATpal0JRtAh3vEuIJqA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3w0LFSLF0fVrbXia9YEvkTQ62gs9fv5SR0jSlKricnFYpy8qfibG3tQcdw/640?wx_fmt=png)

上面我们了解了 **DefaultNamespaceHandlerResolver** 实例对象的创建过程之后，那么下面我们就来分析一下它的`resolve(namespaceUri)`这个方法的内部实现，下面是该方法的源码部分：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3VxMzUK8XM6bPyPpdHodHmN8icu6N2ktPWIoyWkQUzdA0rVApmxSwgKg/640?wx_fmt=png)

在`getHandlerMappings()`方法中，我们获得了系统加载的所有`NamespaceHandler`实例对象的映射，映射关系为：**key=uri**，**value=NamespaceHandler 实现类**。但是，如果我们发现加载的`handlerMappings等于null`，那么我们就需要去加载`META-INF/spring.handlers`文件中的配置信息，将其生成`NamespaceHandler`实例对象的映射。所以，综上所示，`getHandlerMappings()`方法的主要功能就是读取`spring.handlers`的配置文件并将配置文件缓存在 map 中。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3oDGiby3P4al3G4WlNBZ4ibvbVYMlv9xR68okDql5NOZicAicmvhFAhXjtw/640?wx_fmt=png)

那么以我们的演示例子来说，`handlerMappings`中是包含了 **11** 个`xxxNamespaceHander`实例对象的映射关系的，在下图中，红框部分就是我们自定义的`UserNamespaceHandler`。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F34K0zwPSl6licASaNicNG1dZbuT8DJnoy2a7cqBRicBE23pHOJgqs4FzWg/640?wx_fmt=png)

那么在调用 **namespaceHandler.init()** 方法的时候，其实调用的是`UserNamespaceHandler`实例的`init()`方法，该方法是我们自己实现的。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3qX5qhsly5mHYrW7rz3stj2Jtszr5HxA4R5Iwg84fkmLWT3iciaIXo9gg/640?wx_fmt=png)

当我们调用 **handlerMappings.put(namespaceUri, namespaceHandler)** 方法时，那么就将原本 String 类型的 value 值 “`com.muse.springbootdemo.UserNamespaceHandler`”，替换为`UserNamespaceHandler`实例对象了。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3iaFAGue9erw8icl1va39ZbagCT4ib0Z90tniah3jwgt16h5MAgdQmWLYxA/640?wx_fmt=png)

2.3> parse(...) 方法解析
--------------------

下面我们再来看一下的`parse(...)`方法，该方法是用来进行**自定义标签的解析操作**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3zWIxkwiayCibBfk4rSQh6EWwUsjictNjDOljcM1dUYECTvFe0f6LkMCqQ/640?wx_fmt=png)

在`parse(...)`方法中我们可以看到，首先是通过`findParserForElement(element, parserContext)`方法来找到 localName 对应的解析器。以我们上面的示例为例，我们在 **oldbean.xml** 中配置的是`<myname:user muse@163.com"/>`，那么获得了`localName`就等于 “**user**”。由于我们在`UserNamespaceHandler`类中已经配置了 **user** 与 **UserBeanDefinitionParser 实例对象**的对应关系，所以`parser`就拿它作为方法的返回值。具体详情，请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3N8tLyibUa9DBZtfKSQVuDd7uQp1iaymxg9SSKkKDWWYib77oiaTSE7dSnA/640?wx_fmt=png)

在上面的代码逻辑中，我们已经获得到了`parser`示例对象（`UserBeanDefinitionParser`实例对象），那么我们通过调用 parser 对象的 **parser(element, parserContext)** 方法对自定义标签执行解析操作。下面是该方法涉及的源码部分：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3T5YBCzdMI5OUo56HAicjz5FoGxEFnwCGjnDWYXNVgp0oO6eWB7j2Yfg/640?wx_fmt=png)

我们在上面可以看到，对自定义标签进行解析是在`parseInternal(element, parserContext)`方法中执行的，

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3WfOTO5whYEfaVQAOhZeiaoDE34J4DOdE3Qy1bcZic60XKdxObLknzJ0w/640?wx_fmt=png)

在`doParse(element, parserContext, builder)`方法中，执行了真正的自定义标签解析逻辑，那么既然是自定义标签，是无法通过 Spring 进行解析的，而是需要我们自己提供自定义解析类`XxxBeanDefinitionParser`来实现`doParse(...)`方法的，具体如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F30r4bbIAueVQamcGWveo7V6lSbmGxdVTuFTg5Aiclc438kTYaw09ZvYw/640?wx_fmt=png)

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的 **点赞** & **分享** 。

更多技术干货，欢迎大家关注公众号 “**爪哇缪斯**” ~ \(^o^)/ ~ 「干货分享，每天更新」

往期推荐
----

[（一）Spring 源码解析：容器的基本实现](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491156&idx=1&sn=b2ba03821b9be0fd8b7d6c86cb793256&chksm=e9115ca9de66d5bf094badcc0bf517e0f539693951403b0d68174c4066ab03e9c22c0c852eed&scene=21#wechat_redirect)

[（二）Spring 源码解析：默认标签解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491496&idx=1&sn=6500da670c30134e9a0e63e1abddf6cb&chksm=e9115d55de66d44323bb98b214c896c0608db965ad20ec249962c16ecdffeb64a973d929555a&scene=21#wechat_redirect)  

[SpringBoot2.x——Part1](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486233&idx=1&sn=fda7d410bedf6b56457757225a4d6bfa&chksm=e91149e4de66c0f2acd4a95cba907f43dbbd0700c8f70ccd9fc0cc7f57048683732cf79fc77b&scene=21#wechat_redirect)  

[SpringBoot2.x——SpringBoot Web 源码解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486972&idx=1&sn=74556f31641a1bd3c29a2663bc533208&chksm=e9114f01de66c6177f47b26297b0836b5df0e92feea7a7420f611b3d3a2f8926eb952d20b117&scene=21#wechat_redirect)  

[字节一面挂了，面试官问 DDD，我却不知道](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489168&idx=1&sn=9bc88df8701f09a6aba6869d5c873bce&chksm=e911546dde66dd7b873c5b6a7407fa28b9acc6dcef0fdb2d54ae20abe34548ec14ba5977fabd&scene=21#wechat_redirect)