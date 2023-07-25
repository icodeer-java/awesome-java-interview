
一、Spring 的整体架构
==============

Spring 的整体架构图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgmeyjLCYV0vXnSadgcQpZZibLg0ibMmsZQTn5U4pUK8jlPrjR0J1FF8hA/640?wx_fmt=png)

二、容器的基本实现
=========

2.1> 核心类介绍
----------

### 2.1.1> DefaultListableBeanFactory

`DefaultListableBeanFactory`是整个 bean 加载的核心部分，是 Spring **注册**及**加载** bean 的默认实现。

`XmlBeanFactory`集成自 DefaultListableBeanFactory，不同的地方是在 XmlBeanFactory 中使用了**自定义的 XML 读取器**`XmlBeanDefinitionReader`，实现了个性化的 BeanDefinitionReader 读取。源码部分如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wg9brIdibgCZwyV7SBPVic1FTicK3IImmmgfZNFGzukGianEUbrHAwKWaNUg/640?wx_fmt=png)

`DefaultListableBeanFactory`类的继承关系如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgsIyB8icDHOLl12ronQpkTuYul54LyiaDI47sZcJnZ2iaW2ogLFTJX8POg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgYbvAIdYEJs6LgOQXhOWH5x7M7Z8uEo0VquYfVKmBOeQqQ8rXPGKKxA/640?wx_fmt=png)

### 2.1.2> XmlBeanDefinitionReader

XML 配置文件的读取是 Spring 中重要的功能，而`XmlBeanDefinitionReader`可以实现该功能。那么我们先来看一下这个类的继承关系：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgWCibicHR45LgfZtWMbqn4OiajlKtPgYH4rlCAbrn6xjtmicDga2l2PpsYg/640?wx_fmt=png)

> *   • **ResourceLoader（**接口**）**：定义资源加载器，主要应用于根据给定的资源文件地址返回对应的 Resource。
>     
> *   • **DocumentLoader（**接口**）**：定义从资源文件加载到转换为 Document 的功能。
>     
> *   • **BeanDefinitionDocumentReader（**接口**）**：定义读取 Document 并注册 BeanDefinition 功能。
>     
> *   • **BeanDefinitionParserDelegate（**接口**）**：定义解析 Element 的各种方法。
>     
> *   • **BeanDefinitionReader（**接口**）**：主要定义资源文件读取并转换为 BeanDefinition 的各个功能。
>     
> *   • **EnvironmentCapable（**接口**）**：定义获取 Environment 方法。
>     
> *   • **AbstractBeanDefinitionReader**：对 EnvironmentCapable、BeanDefinitionReader 类定义的功能进行实现。
>     

2.2> XmlBeanFactory 的创建
-----------------------

如果想使用 Spring，我们可以通过创建`BeanFactory`示例，然后调用`getBean(...)`来获得相应的实例对象，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgxCGTH1zrosJr9xeiaicltUFic52oYNslSdibf3MoQWq65jsB02WIZWB9vw/640?wx_fmt=png)

但是在获得 BeanFactory 的过程中，我们其实大体是经历了如下的几个步骤：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wg9Rjy79VKeNnpic4q0dqRf0q5mxetOfOD7yodBcibe22h7gyCsAVvqb9Q/640?wx_fmt=png)

我们可以看到，第一步是获得了`Resource`。那么 **Resource 是 Spring 用于封装底层资源**，比如：`File`、`URL`、`Classpath`等。对于不同来源的资源文件都有相应的 Resource 实现，比如：

> 针对**文件**资源：`FileSystemResource`  
> 针对 **Classpath** 资源：`ClassPathResource`  
> 针对 **URL** 资源：`UrlResource`  
> 针对 **InputStream** 资源：`InputStreamResource`  
> 针对 **Byte 数组**资源：`ByteArrayResource`  
> ……

我们在日常开发工作中，如果需要对资源文件进行加载，也可以直接使用 Spring 提供的`XxxResource`类。比如，我们要加载`oldbean.xml`文件，则可以使用如下方式将其**先转化为 Resource** ，然后**再调用 getInputStream()** ，就可以获得到输入流了。那么针对于输入流的后续操作，与我们以往的处理方式是一样的。当然，除了能从 Resource 中获得`InputStream`之外，还可以获得`File`、`URI`和`URL`等。具体请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wg1icR3156HeHaCwOPYc3Fguzib1A0BCicrTSIXLypZVTwXRYryBJ5akoKA/640?wx_fmt=png)

将 Spring 相关的配置文件封装为 Resource 类型实例之后，我们就可以继续**创建 XMLBeanFactory 实例对象**。如下是其构造函数源码：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgOYZhTkEWvf4vwtQzsZNTHZ98PEqePvRztyqkZrtr7u4icIX82SEU0IQ/640?wx_fmt=png)

2.3> loadBeanDefinitions(resource) 加载 bean
------------------------------------------

我们来看一个`loadBeanDefinitions(...)`方法的源码实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wg2A5f7LR0HV4nh4qkVvZibbUhH56VYwiblMIk6pvBBrT4kmib8mB64GmJQ/640?wx_fmt=png)

主要代码逻辑是用来根据 xml 配置文件的内容进行解析，然后使 Spring 加载 bean。函数执行的时序图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgh1cva39mNtfnmVEuSthARD5ibMXRSiaSnwA9OIDyaiab69tic4MCswy7Gw/640?wx_fmt=png)

### 2.3.1> EncodedResource 解析

我们发现 resource 又被`EncodedResource`类包装了一层。在构造 EncodedResource 实例的时候，我们可以指定`resource`、`encoding`和`charset`。具体源码如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgibJQmaRQAUl7b8zcN7QvGPkiaZficBU8EG8spTVflzOh5OicypMibWzolZw/640?wx_fmt=png)

那么当调用它的`getReader()`方法时，就会使用相应的**字符集 charset** 和**编码 encoding** 作为输入流的`charset`和`encoding`。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wg46ibCdj0eB9maoZQyoCNrZjy9G6jNoOo8OXKNEgMN9advnibeVRmB4iag/640?wx_fmt=png)

### 2.3.2> loadBeanDefinitions(EncodedResource encodedResource)

本方法就是针对 EncodedResource 实例对象进行 bean 的加载。具体执行如下 3 个步骤：

> **步骤 1**：将入参`encodedResource`保存到`currentResources`中，用于记录当前被加载的资源。如果发现已经存在了，则抛异常，终止资源加载。  
> **步骤 2**：从`encodedResource`中获得输入流`InputStream`，并创建`inputSource`实例对象。如果在`encodedResource`中配置了编码（**encoding**），则为`inputSource`配置该编码。  
> **步骤 3**：调用`doLoadBeanDefinitions(...)`方法从资源中加载 bean。

具体源码实现逻辑，请见下图：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgPNg2cYkJtWlU1Z0gWXsI6icZshUrPicVp89YhqUnTOIVK71jUkRF7Ipw/640?wx_fmt=png)

> 需要注意的一点是，InputSource 不是 Spring 提供的类，它的全路径名是`org.xml.sax.InputSource`，用于**通过 SAX 读取 XML 文件**的方式来创建 InputSource 对象。

下面我们来看一下`doLoadBeanDefinitions(...)`方法的具体实现。在这个方法中，主要做了两件事件：

> **步骤 1**：加载`XML`配置文件，然后将其封装为`Document`实例对象。  
> **步骤 2**：根据`Document`实例和`Resource`实例，执行 Bean 的注册。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wggWaFIicxoL37YYX6q9iaQaYRjb9NgVElhb6jxFLkLkAYF8VEqPvlxibEw/640?wx_fmt=png)

#### a> doLoadDocument(...)

在`doLoadDocument(...)`方法中，我们需要关注下图中红框的两部分代码：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgiauwVkiabL0WSZ1nE4MUmneV5hTAKLDodMAEIN7c5iaovCuSywrNKmVrA/640?wx_fmt=png)

首先，我们来看一下`getValidationModeForResource(Resource resource)`，具体源码逻辑如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgICBicmAVPia7CibNOFToxSE4UtyPqLYaibpjU8lvj8xQB1UulGY0YcA0nA/640?wx_fmt=png)

> 默认值为：**VALIDATION_AUTO**，如果发现现在的 Mode 不是 VALIDATION_AUTO 了，则说明有人自定义了，那么就返回自定义的 Mode。如果没有被自定义，那么则通过`detectValidationMode(resource)`方法根据 xml 配置文件的格式，来确定 Mode 是 **DTD** 还是 **XSD**。

最后，我们来看一下`detectValidationMode(resource)`方法的具体实现，它到底是如何判断 Mode 的：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgo53kBIiaXEYAFgRw4p7Pyo0B6UX3KgSkh2ztibDsUViawEfYRHO7VTDMg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgUfibzWPrvNXoDMDnFP2KppOVtwY3PtuU1P0mIDPtFGLlrUrBmCaiburA/640?wx_fmt=png)

> XML 文件的验证模式保证了 XML 文件的正确性，而比较常用的有两种，即：**DTD** 和 **XSD**。  

> **DTD**（`Document Type Definition`）：它是一种 XML 约束模式语言，要使用 DTD 验证模式的时候需要在 XML 文件的头部声明 ****，并且它引用的是后缀名为`.dtd`的文件。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgUataicy14Ib54xCOh4AaJdSSmgEDvzHSucWNI3Z0uqdPXiaic2xObfDJg/640?wx_fmt=png)

> **XSD**（`XML Schemas Definition`）：用于描述 XML 文档的结构。它引用的是后缀名为`.xsd`的文件。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgxzp8RfiaAOCIxQgH37XyfgaAkV4r5zWJf6ticBUibgZ58aqHCUIkqhFRA/640?wx_fmt=png)

看完`getValidationModeForResource(resource)`方法之后，我们再来看一下`documentLoader.loadDocument(...)`方法。为了便于理解，我们再次将相关代码粘贴出来：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgiauwVkiabL0WSZ1nE4MUmneV5hTAKLDodMAEIN7c5iaovCuSywrNKmVrA/640?wx_fmt=png)

`loadDocument(...)`方法是通过 SAX 解析 XML 文档，这段代码是套路性的代码，没什么好说的。就是**创建 DocumentBuilderFactory**——> **创建 DocumentBuilder**——> **调用 parse(inputSource) 方法解析 inputSource 实例然后返回 Document 对象**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgwh0ic7MugwUWFUYSpvk81tCz3wBOYrFuaaDTtYPMKnUgZqnUmHzS5iaQ/640?wx_fmt=png)

在上面黄框圈中的`EntityResolver`实例，它的作用是：DTD 默认寻找规则是通过网络（即：声明的 DTD 的 URI 地址）来下载相应的 DTD 声明，并进行认证。由于网络原因，下载速度本身就是耗时的。那么，**我们可以通过 EntityResolver 来实现寻找 DTD 声明的过程**，比如：我们将 DTD 文件放到项目中的某个路径下，在实现时直接将此文档读取并返回给 SAX 即可。

黄框中的`EntityResolver`实例，它是一个接口，并且提供了一个`resovleEntity(...)`方法，源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wg1DHTYxIcdH6Bic4IodRic2JkyE57cZFNh5GRhpZ9ZuTibqgB7lUtHUbag/640?wx_fmt=png)

那么`publicId`和`systemId`是什么呢？如果是下图中的 XSD 的配置文件，那么 **publicId**=`null`，**systemId**=`http://www.springframework.org/schema/beans/spring-beans.xsd`

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgp0iaAayu81w3knevT4PWlzxfjcpkhsgstRDeQjYfXUu5xehWe6PftGQ/640?wx_fmt=png)

如果是下图中的 DTD 的配置文件，那么 **publicId**=`-//SPRING//DTD BEAN 2.0//EN`，**systemId**=`https://www.springframework.org/dtd/spring-beans-2.0.dtd`

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgYVTSBc0PxeYjP7zqibqTianibVzicmYuibT1YWImmgfqqPiaxpkADjqVjkLw/640?wx_fmt=png)

好了，了解了 publicId 和 systemId 之后，我们要将关注点放在对`EntityResolver`实例的获取过程了，在`doLoadDocument(...)`方法中，是通过 `getEntityResolver()`获得的，那我们就来看一下`getEntityResolver()`的具体实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgf8ZL4XrCZoKfRvaQX44yiaMpbREB6EyXRJ5VK6YagiaomibqYaoKXBSrw/640?wx_fmt=png)

既然最终都是要通过调用`DelegatingEntityResolver`的构造方法，我们就来看看它的内部实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgZ4O4p1rcW6JayFLql3NOIF3cOdt0DQxdrw4WhJMicRoCuFqeFMbSVEg/640?wx_fmt=png)

> **BeansDtdResolver**：是直接截取 systemId 最后的 xx.dtd，然后去**当前路径下**寻找。  
> **PluggableSchemaResolver**：是默认到 **META-INF/spring.schemas** 文件中找到 systemId 所对应的 XSD 文件并加载。

#### b> registerBeanDefinitions(...)

在上面的内容中，我们已经将`xml配置文件`通过 **SAX** 解析成了`Document实例对象`了。那么下面，我们就要操作这个 Document 实例对象 doc **进行 bean 的注册操作**了。在介绍具体操作细节之前，我们先看一下相关源码部分：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgK1rxBW8icHuCfrYG0MOjMkK3J2nicPeicoKjMxjkyAm5mu68JaGmgxyKA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgSXoaegwmz33UfcTYn1zhq3TSaWwd3ggdYZ5bNrBFME8HFgJ0icvKK7Q/640?wx_fmt=png)

由于`BeanDefinitionDocumentReader`只是**接口**，所以通过`createBeanDefinitionDocumentReader()`方法其实创建的是它的**实现类**——**DefaultBeanDefinitionDocumentReader**。具体代码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgE0qKqs7WowCw9S8iaVfc6k9KUIibP99vNEdINBVmm3bx77icbBSzd3sFA/640?wx_fmt=png)

好了，看完了`createBeanDefinitionDocumentReader()`方法创建了`DefaultBeanDefinitionDocumentReader`实例之后，我们再继续看`documentReader.registerBeanDefinitions(...)`方法：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgL4glmYE1lSja9u9Tcyib9Oiao14ibYDp7pOqXf8aCpchuvgkGqZVdIcKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wg5pWmR3wIveZc2uS8a3B31sNspTjyVHOcNXcqSlzxRlp5nf9kEUwGWw/640?wx_fmt=png)

profile 最主要的目的就是**可以区分不同的环境，进而对不同环境进行配置**。例如在`dev`、`test`、`preonline`、`online`等环境有各自的配置文件，我们就可以通过设置 profile 来在特定的环境下读取对应的配置文件。使用方式如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgY9tMYorW9icjEFKibIR4OuHHYicNqO7Ctb0wRkUA6N4oYMqIk5CC1gliaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgsyqYw0jZU8VibBejPCcleWPVqU0QxDgiafxbvrgdzbsGWQBJ427hibGZQ/640?wx_fmt=png)

了解了 profile 之后，我们来看一下要**执行 xml 解析**的关键方法`parseBeanDefinitions(root, this.delegate)`，它会根据表空间（如果命名空间等于 "`http://www.springframework.org/schema/beans`"，则表示是默认表空间，否则是自定义表空间）来判断，是进行**默认标签**解析还是**自定义标签**解析。相关源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8QXUJX4urk1icVibUGWj69wgn215sGz4Eez2QOTNljMuNbjnVZOOMA07lbv6da30DWRTW7yybJcgdw/640?wx_fmt=png)

**默认标签**：`<bean>`

**自定义标签**： `<tx:annotation-driven>`

对于默认标签来说，Spring 自己就知道如何去解析；而对于自定义标签来说，就需要用户实现一些接口及配置了。那么关于这两种标签类型的解析，也就是我们后续关注的重点了。具体内容**请见下一篇文章：默认标签解析**。

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的 **点赞** & **分享** 。

更多技术干货，欢迎大家关注公众号 “**爪哇缪斯**” ~ \(^o^)/ ~ 「干货分享，每天更新」

往期推荐
----

[图解 LeetCode——2325. 解密消息（难度：简单）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491114&idx=1&sn=ae71135785681ae5ffc486a2381d5f95&chksm=e9115cd7de66d5c1421a157fe7965f98ab91b9b516a11255fbb3022061c38b0cc7bff28d32f3&scene=21#wechat_redirect)  

[图解 LeetCode——1780. 判断一个数字是否可以表示成三的幂的和（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491106&idx=1&sn=4d56134335e9283480b9f7e8202a7545&chksm=e9115cdfde66d5c9f2287b206a67b68d7a38a3127c21ec2bf0801dc7db69026dae8086bf1bd1&scene=21#wechat_redirect)  

[图解 LeetCode——1812. 判断国际象棋棋盘中一个格子的颜色（难度：简单）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491099&idx=1&sn=732a0236048834258d7e0865c728f444&chksm=e9115ce6de66d5f00e7eb43afaf8126623313c626f53f56baa162bb3973a8176345a2b377a9c&scene=21#wechat_redirect)  

[图解 LeetCode——1775. 通过最少操作次数使数组的和相等（难度：中等）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491090&idx=1&sn=84dce8c917c069e0a923b4afa7edcaf7&chksm=e9115cefde66d5f9402be753aa866c50a01566aeb3a24978b41935c781ce79c60e5e622bfc73&scene=21#wechat_redirect)  

[图解 LeetCode——1796. 字符串中第二大的数字（难度：简单）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491081&idx=1&sn=d03438ba72934cfe314bc2d8c0dc9bc7&chksm=e9115cf4de66d5e201e463690b2c945ab18cf5cb6c3d642a9cf1238d4dfd3ad778f663a42419&scene=21#wechat_redirect)