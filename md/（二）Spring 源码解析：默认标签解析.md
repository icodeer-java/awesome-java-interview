> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491496&idx=1&sn=6500da670c30134e9a0e63e1abddf6cb&chksm=e9115d55de66d44323bb98b214c896c0608db965ad20ec249962c16ecdffeb64a973d929555a&scene=178&cur_album_id=2208187391738249217#rd)

一、概述
====

还记得我们在上一讲末尾提到的关于`默认标签解析`和`自定义标签解析`吧。本讲就来针对**默认标签解析**进行讲解。为了便于衔接上一讲的内容，我们将源码部分粘贴出来：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbf0nN2lwuZs8RaocdpUH5v9xFRhfO4hfoBHoNrra5u2Nkyia5KIaTgkQ/640?wx_fmt=png)

从上图中的源码中，我们可以看出默认标签的解析是在`parseDefaultElement(ele, delegate)`方法中实现的。我们来看一下这个方法如何实现的：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbylwFxbEyK2Sq7XUCn52D0s05EA5SrkqIdlROZ2oBaFyy4Mfw8eibmicA/640?wx_fmt=png)

在`parseDefaultElement(ele, delegate)`方法中我们可以看到，它分别针对 **4 种**不同的标签（即：`import`标签、`alias`标签、`bean`标签和`beans`标签）做了解析操作。那么下面我们就通过下面的 4 部分内容来对这些标签的解析进行深度剖析。

二、bean 标签的解析
============

在上面的 4 种标签中，对 bean 标签的解析最为复杂和重要，所以我们先从这个标签开始深入分析，如果能够理解它的解析过程，那么其他标签就不难理解了。我们废话不多说，言归正传。先来看看`processBeanDefinition(ele, delegate)`方法内部的具体实现逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhboUgyYhQNONeiaxYftZia9njKmAMcZBwM2fpDEKjibStTRONccRRumdYkQ/640?wx_fmt=png)

bean 标签的解析和注册的时序图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbSL2iaQNloico3Kk0AoUhAZJPTQBvC16Oibiab3SF5b7q86KPMHZQHic6Zow/640?wx_fmt=png)

2.1> parseBeanDefinitionElement(ele)
------------------------------------

首先我们来看一下元素解析部分内容，在`parseBeanDefinitionElement(ele)`方法中，但是这个方法其实只起到了 “周转” 的作用，它的内部其实又调用了`parseBeanDefinitionElement(ele, null)`方法，所以，我们看下一个方法的内部实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbPuhwKFMektuIbyLfUel76B6Uoo9x4iayaMPbPibcHk8rzRiaQxSXKXVOw/640?wx_fmt=png)

在 `parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean)` 方法中，其实一共执行了如下 **4 个**步骤（【注】但是下图中只列出了其中的第 1 步骤和第 2 步骤，后面文章内容，再给大家展示剩下的后两个步骤）。

> 【**步骤 1**】提取 **Element** 元素中的 “`id`” 和 “`name`” 属性，并将 name 解析为 aliases，然后为 beanName 赋值。  
> 【**步骤 2**】解析`其他属性`并封装到 **GenericBeanDefinition** 类型的实例中。  
> 【**步骤 3**】如果发现 **bean 没有指定 beanName**，那么使用默认规则生成`beanName`。  
> 【**步骤 4**】将获取到的信息封装到`GenericBeanDefinition`类型的实例中。

### 2.1.1> 步骤 1：解析 beanName 和 aliases

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbAnCIC7rq4EibRZ6yibB55zyO1IILBGq1icy2MRFHdVGFZpE4x8gYUF3cQ/640?wx_fmt=png)

其中，`checkNameUniqueness(beanName, aliases, ele)`是用于校验 **beanName** 和 **aliases** 是否是唯一的，即：不允许出现重复的名字（name）。如果发现有重复的，则直接抛出异常。具体逻辑实现，请见下图源码注释：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbib2kWEHv4j0fiahC6skEck95lBOnKRXXCuxM4zUnEiciaVYmfSadp8ZhSg/640?wx_fmt=png)

### 2.1.2> 步骤 2：解析其他属性

对于其他属性的解析，是在`parseBeanDefinitionElement(ele, beanName, containingBean)`方法中实现的，具体源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhb784XxMeAgwwibG2KaNyZPB4nJcCWSTr0WdggpmznKEJpyNzicSKAxZicA/640?wx_fmt=png)

#### a> 创建 GenericBeanDefinition 实例

创建 BeanDefinition 的这部分内容，是通过调用`createBeanDefinition(className, parent)`方法实现的。但是，在介绍整个方法内部逻辑之前，我们先来了解一下 BeanDefinition，它到底是做什么用的？

`BeanDefinition`是配置文件中 **<bean> 元素标签在 Spring 容器中的表现形式** ，也就是说，它是用来**承载 bean 信息**的。在配置文件中可以定义**父级 <bean>** 和 **子集 <bean>** ，它们分别由`RootBeanDefinition`和`ChildBeanDefinition`表示。而如果没有父级的话，则用`RootBeanDefinition`表示。而`GenericBeanDefinition`是从 2.5 版本之后加入进来的，用于为 bean 文件配置属性属性定义提供一站式服务。这三个类之间的关系如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbbE2E1NMIBTxAU9z7v2O8WuhFcJ2oYGo5ia2FYHCzN9W4E8VgnDyjvtA/640?wx_fmt=png)

了解了 BeanDefinition 的 3 个实现类之后，我们再来看一下`createBeanDefinition(className, parent)`方法的具体实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbInajBpqUETVZos8xrX1JqyQfqZJibT5uVqroPbEiadY98APBZMdcmusg/640?wx_fmt=png)

上图中的`createBeanDefinition(...)`方法内部基本没做什么，关键的内容在红框的`BeanDefinitionReaderUtils.createBeanDefinition(...)`方法调用上，下图是`createBeanDefinition(...)`方法的源码部分：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbIcSicNsK4VQDwvE3TTJZeMq9ib0JiapB95ibsA3RiaHFK4lqibOalBSYJ7RA/640?wx_fmt=png)

通过上图源码，我们可以看出来`createBeanDefinition(...)` 方法执行了很简单的 **4 步** 操作：

> 【**步骤 1**】创建`GenericBeanDefinition`实例对象 **bd**。  
> 【**步骤 2**】为 **bd** 设置`parentName`属性。  
> 【**步骤 3**】为 **bd** 设置`beanClass`属性。（如果 classLoader 不为空，则利用它去创建 className 对应的 Class 实例对象）  
> 【**步骤 4**】为 **bd** 设置`beanClassName`属性。（如果 classLoader 为空，则只需赋值 className 即可）

#### b> 解析 bean 的各种属性

有了可以承载 bean 信息的`GenericBeanDefinition`实例对象之后，我们就来继续往下分析，看看负责 **解析 bean 中各个属性** 的逻辑代码——`parseBeanDefinitionAttributes(ele, beanName, containingBean, bd)`：需要补充的一点是，此时入参 containingBean 等于 null。

在分析该方法源码之前，我们先来看如下两个遍历中的内容：一个是入参的`ele`，另一个是全局变量`defaults`。在下面的源码解析中，我们会经常回过来参照这两个参数所存储的值。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbl19Ad7KzCYKnPTzpC0uEk2GyrDYFYv84cVRend6FK43QcfDlJTRgxA/640?wx_fmt=png)

针对`parseBeanDefinitionAttributes(ele, beanName, containingBean, bd)`方法的源码，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbdaUB3yGrXgFKOaTQAtezrUfUp2S7kbicaImWVhuDmfZdBuhVejc1xEg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhb0q2G0LenPlwI2NbcDLX53IQ1P5gmWMUZQ5lGkaPtXVJS8356k4cnjg/640?wx_fmt=png)

#### c> 解析元数据

下面我们在来看一下元数据解析方法——`parseMetaElements(ele, bd);`再介绍源码之前，我们先来看一下 Spring 中的 meta 标签使用方式如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbNS5BMN7JzZ8yzNRicicre5TrldaG3P15iaHbTiaKxHIZ3JOEhzGxG4iaJ7w/640?wx_fmt=png)

> 从上面的例子我们可以看出来，使用了`meta`标签后，配置的`desc`并不会体现在`Gun`的属性当中，而只是一个额外的声明。当需要使用里面的信息的时候，可以通过`BeanDefinition`的`getAttribute(key)`方法进行获取。

下面我们再来看一下`parseMetaElements(ele, bd)`的源码部分：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbxKbCuJm4N3gEnWbGt9eg2hgpzcjGGGj9bJaqMK0UN7yJbMN18P2oVw/640?wx_fmt=png)

#### d> 解析 lookup-method 属性

我们平时对于`lookup-method`的使用其实是不多的，所以，我们在介绍关于`lookup-method`属性解析之前，先了解一下它是怎么使用的。如下代码所示，我们有一个抽象类 **Writer**，它有一个写作的方法`write()`，但是具体使用哪种类型的笔去写作，则需要抽象方法`getPen()`来决定。那么关于笔的类型，我们提供了铅笔（**Pencil**）和毛笔（**Brush**）这两种。具体如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbpBaQ7bNjFEiaMD7lBL4rnAgdEt1Wsy4wzwqgkDvg2Rx31rny8V6zXDg/640?wx_fmt=png)

然后，我们通过在 xml 的配置文件中，使用`lookup-method`标签，将 pencil 的 bean 赋值给`getPen()`方法，那么运行结果显示 “Pencil print !”

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbqs4yAicHW0k5CISMqO62mrbVpDP672bsiaRblqCHKOH9c5VcFhOh3Iiag/640?wx_fmt=png)

然后，我们可以通过修改 xml 配置中的`lookup-method`标签，**将原来的 “pencil” 替换为“brush”**，再运行一下，那么运行结果显示 “Brush print !”

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbqicVGQg4dXic8dYsENR5j6mVV9zPALCicBU85ViamcOAf6LUM0PYeaYK7Q/640?wx_fmt=png)

> 【**总结**】根据上面的演示，我们可以知道`lookup-method`它的作用是**获取器注入**。即：获取器注入是一种特殊的方法注入，它是把一个方法声明为返回某种类型的 bean，但实际要返回的 bean 是在配置文件里面配置的，此方法可用在设计有些`可插拔`的功能上，解除程序依赖。

好了，讲完了`lookup-method`的使用方法和作用之后，我们再来看一下`parseLookupOverrideSubElements(ele, bd.getMethodOverrides())`方法的源码实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbmT3s2BYzZTJvgTicPNDN4vU9ziaWkyC4Q8w3Im6EJVIfPCLDHAy7TXVg/640?wx_fmt=png)

> 【**总结**】`parseLookupOverrideSubElements(ele, bd.getMethodOverrides())`方法的内部逻辑跟我们解析元数据的方法`parseMetaElements(ele, bd)`非常类似。此次就不再赘述了。

#### e> 解析 replaced-method 属性

在介绍对`replaced-method`属性解析之前，我们还是来看一下它的使用场景吧。假设有个程序员`Coder`觉得工资又少，工作又多，非常烦躁。他打算去跟他们老板吼叫（`shout`）一番，表达自己内心的不满——“**老板！我要离职！我不想写代码了！烦死了！**”。具体实现如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbdwpmCib2sMX9hz2BHNDJBCsK8WJ0Q49bJkjicIKVtPVQHF1sib81UDInw/640?wx_fmt=png)

但是在他马上要到达老板办公室门口的时候，接到了他女朋友的电话，电话那头说：“我爸妈同意咱俩结婚了，但是有个前提，就是要买个楼房。” 程序员一想，自己买房的钱还没凑够呢！这不能离职啊！但是他此时已经推开了领导办公室的门。为了不喊出离职的那句话，我们可以采用`MethodReplacer`的方式改变`shout`方法的实现逻辑，从而让程序员 Coder 说出：“**老板！我最爱写代码了！我会一直忠于公司为您工作的！**” 这句话，具体实现如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbKP71kYWO1Bp7IQgwvqxvaXVv8ibsdIpeYRNCpWoLlOsHO0CzyafT1Fg/640?wx_fmt=png)

> 【**总结**】`replaced-mothod`可以实现方法替换，即：可以在运行时用新的方法替换现有的方法。

好了，理解了`replaced-mothod`标签的使用方式之后，我们来看一下`parseReplacedMethodSubElements(ele, bd.getMethodOverrides())`方法是如何对`replaced-method`属性进行解析的。详细解题源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhblXzyicm0sVgiaPzZkom2ULjCG9PHk7PKNBLBcA3yXPC1A1lQ9cct8Hvg/640?wx_fmt=png)

#### f> 解析构造函数的参数

对于构造函数的配置方式，请见如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbXdxuEdIaFl1Zzd5nNvnZV2lMnVRGcdqzqqAAickKHxfH7Db0oyHVy6g/640?wx_fmt=png)

> 【**解释**】默认情况下是按照参数的顺序注入的，当指定 index 索引后，就可以改变注入参数的顺序。

下面是`parseConstructorArgElements(ele, bd)`方法的源码实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbpmV9mqDre07sxticbVgdXQNSRuulGGMAcqpXEmublb2CfF4iaaQuKLew/640?wx_fmt=png)

我们可以在上面看到，方法内部又调用了`parseConstructorArgElement((Element) node, bd)`方法，这里面才是真正解析逻辑的地方：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbELJKTJGPZ5Ycq8h2HxibnfhibOsCdb8iapQnAibS9g8wMFBfTUv4KWnXCw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbts0LF3847diaxRSd7LdHJGUb8yqaNI9UHibsbGcsHRRuPT2CPNxIibGibQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbqibwrZIPDOxBUBJ29GicxdJMm7ItcHomlnBCVF4nRBSiaKGfGaQfouqicw/640?wx_fmt=png)

上面的代码还是比较好理解的，但是关于`parsePropertyValue(ele, bd, null)`方法，我们还需要再看一下。源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbMG4rgia8ugjKbSjlxictO83wXiciaxV1Uo1FjNGG2zC0XGIvjh6e7CibjAA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbPz4BPJBocNxwVMCBl7KAdcS7dHzwJib17gMpnDibkZX9bibQEocgd6dBw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhb4KRMBJI3lUK5UE0Hh7gdDC5oVZrBbiaSEqsj9lD2OaHo6ZOXaOfuVUw/640?wx_fmt=png)

那么关于上图蓝色框所标注的`parsePropertySubElement(subElement, bd)`方法，我们来看一下它的具体实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbL9mvTTUJicj2Pw4dHORGD7cicYX9mGK1vLxibWPmBBpFER4JEDmQrsFAw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbnHqJibgmumK2TV20TOSVdKa7n2osO9przYz6RBLTHmWia35Soy4gvuAg/640?wx_fmt=png)

通过上图中源码的注释，可以看出逐一的对 constructor-arg 的子元素进行解析，针对每个子元素的解析此处不再进行讲解。这些子元素都可以通过如下配置方式进行配置：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbrsW5Vaz0Hx5xGQsAU97dkqomAL7t0BErt9nh7bTmSoP5ZJkpothlLg/640?wx_fmt=png)

#### g> 解析 property 子元素

在方法`parsePropertyElements(ele, bd)`中，对`property`标签进行了解析。关于`property`标签的使用，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbsWFWLf4IgOpQJica94xtuHaohFBpKWeWNyg0ThH8rXISiaQWtqJFOq5Q/640?wx_fmt=png)

那么解析`property`子元素的源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbxmyR2J1kbaXLGwtqLbNoP2OGKNfibAgkjibibOYy5gC2IjjND2waZ364Q/640?wx_fmt=png)

> 【**解释**】可以看到上面函数与**构造函数注入**方式不同的是，返回值使用 **PropertyValue** 进行封装，并记录在了`BeanDefinition`的`propertyValues`属性里。

#### h> 解析 qualifier 子元素

当同一类型的 bean 注入到 IOC 之后，Spring 容器中匹配的候选 Bean 数目必须有且仅有一个，那么此时，我们可以通过 Qualifier 指出注入 Bean 的名称，这样其一就消除掉了，配置方式如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhb1jNZOrj3ksm6hEjsr6WuKTK66EfSd84ZAH6MWr1XKUIwoBcaJTs9uw/640?wx_fmt=png)

具体源码实现，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbjDVHfNMcXqGicYfia2aL9XPMQRKNQib8Kf3cePJXuX9R47XGHosG7Zibqg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbvQ3CRxXuztS5hjoH4qg6gp1YpL6ibzWu4hZ7Jg6uVZbc3btYLlUOAsQ/640?wx_fmt=png)

2.2> decorateBeanDefinitionIfRequired(ele, bdHolder)
----------------------------------------------------

上面是针对`parseBeanDefinitionElement(ele)`方法进行的解析，下面我们要解析的是下图中红框标注的方法：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbKfg8qhh1fY1Te246zcj7TMFoPsgfA8A3hKib3NwWK9xChKV6HMraMhg/640?wx_fmt=png)

当 Spring 中的`<bean>`标签的**子元素**使用了`自定义标签`配置，则会被`decorateBeanDefinitionIfRequired(ele, bdHolder)`方法解析，如下所示：

```
<bean id="test" class="com.muse.Test">
   <mybean:user user>
</bean>

```

那么，下面我们来看一下`decorateBeanDefinitionIfRequired(ele, bdHolder)`方法的源码实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbbnDoYDdAicVJRvY1Zt4hKZhFFL9NEDAiaXVkOwLNZiaiamTzW4Ix6FsByQ/640?wx_fmt=png)

> 【**解释**】上面调用`decorateBeanDefinitionIfRequired(ele, originalDef, null)`方法的时候，第三个参数传递的是 null，因为第三个参数是父类 bean，当堆某个嵌套配置进行分析时，这里需要传递父类的 BeanDefinition。分析源码得知这里传递的参数其实是为了使用父类的 scope 属性，以备子类若是没有设置 scope 时，默认使用父类的属性，这里分析的是顶层配置，所以传递 null。

在上面代码中，我们看到无论是对所有`属性`还是所有`子节点`，都会执行`decorateIfRequired(node, finalDefinition, containingBd)`方法，那么我们再来看一下这个方法的内部实现：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbdhg8GsTNB2LK9YywPSxT48qdZX5bqqe7DWh9nAv7sDicUHydnzRfiaiag/640?wx_fmt=png)

其中，`isDefaultNamespace(namespaceUri)`是通过判断 **namespaceUri** 不为空，并且等于 "`http://www.springframework.org/schema/beans`"，如果都满足，则是默认的命名空间。否则是自定义的命名空间。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbfd5O9pHetJYIjtXsSzz2IrKHlA92yo59eSookOvCQoFqnc0iape9Cqw/640?wx_fmt=png)

通过调用`readerContext.getNamespaceHandlerResolver()`，我们可以获得如下红框中的 **命名空间对应的处理器**（`NamespaceHandler`）。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbd5HGwj7vNKhxT6ib4dBJaLzmDqa7ick5EZLnKPfSLMEFodt2RpbZ486Q/640?wx_fmt=png)

其中关于自定义标签的解析过程，我们会在第 3 讲部分介绍，此处就直接略过了。

2.3> registerBeanDefinition(...)
--------------------------------

上面我们执行完了对配置的`解析`和`装饰`操作，那么下面就该到**注册阶段**了。涉及源码部分如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhb1eTKcT6vRSecPhggYSGkg0gPBu2lUI6SibAuc4gVuGbHZfCrm0DaBSQ/640?wx_fmt=png)

在下面的代码中，我们可以看到总共有两个步骤的操作，分别是：**注册 BeanDefinition** 和 **注册别名 Alias**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbFrc4xm5gicae4gWicKoC9aXVicIvEibaq3rZhWkrNtb0N4kFicicQsRjyo9Q/640?wx_fmt=png)

那么下面我们先来看一下**注册 BeanDefinition** 的方法`registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition())`的处理逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhboib9CiatC1Oricj4rBWGIZLgLQmLbibnupvFt8IsgjI9pdTwJE4xcgelpw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbZu8AazRLN7UNuh3826CPSaQuCFCuo6eBr2OsAaz63mDtgwGbr25lYA/640?wx_fmt=png)

> 【**解释**】从上面的代码中，我们可以看到针对 bean 的注册处理方式上，主要进行了以下几个步骤：  
> 【**步骤 1**】对 AbstractBeanDefinition 的校验。在解析 XML 文件的时候我们提过校验，但是此校验非彼校验，之前的校验是针对 XML 格式的校验，而此时的校验是针对于 AbstractBeanDefinition 的 methodOverrides 属性的。  
> 【**步骤 2**】对 beanName 已经注册的情况的处理。如果设置了不允许 bean 的覆盖，则需要抛出异常，否则直接覆盖。  
> 【**步骤 3**】加入 map 缓存。  
> 【**步骤 4**】清除解析之前留下的对应 beanName 的缓存。

那么下面我们先来看一下**注册别名 Alias** 的方法`registry.registerAlias(beanName, alias)`的处理逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhb3Bib0Jwv1wAdvurJYdH7cElEfLr2odf1sOQsGZ1vZiaUoqMqC43ZVaLg/640?wx_fmt=png)

> 【**解释**】由以上代码中可以得知，注册 alias 的步骤如下：  
> 【**步骤 1**】alias 与 beanName 相同情况处理。若 alias 与 beanName 名称相同，则不需要处理并删除掉原有 alias。  
> 【**步骤 2**】alias 覆盖处理。若 aliasName 之前已经被配置了，则进行 3 个判断处理。  
> 【**步骤 3**】alias 循环检查。  
> 【**步骤 4**】注册 alias 和 beanName 到 aliasMap 中。

2.4> fireComponentRegistered(...)
---------------------------------

该方法的目的是为了**通知监听器解析**及**注册完成**，这里的实现只为扩展，`目前Spring并没有对其进行任何实现`。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbBfTKibS2bAk30NKmofOsEMqib59nw4FpcO3ibP74PKnJMFgz50DKEkMRg/640?wx_fmt=png)

三、alias 标签的解析
=============

在对 bean 进行定义时，除了使用`id`属性来指定名称之外，为了提供多个 bean 的名称，我们可以使用`alias`标签来指定。例如，通过在`<bean>`标签中设置 **name 属性**来为 bean 设置别名（alias)。如下所示：

```
<bean id="gun"  />

```

另外，Spring 还有另外一种声明别名的方式：

```
<bean id="gun" class="com.muse.Gun" />
<alias  />

```

关于 alias 标签的解析的代码，是在`processAliasRegistration(ele)`方法内实现的，具体源码请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbotEq9HsPpwZKt2He0laaR9mCTWLINib3oYewbt863lf5Dv4D72rUq6Q/640?wx_fmt=png)

`processAliasRegistration(ele)`方法的内部源码如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhb1MbBicheYqeibK80xVOuDE7dLT6kyDLp2MT3kqqOd0QhIKtt6W7l7wicw/640?wx_fmt=png)

> 【**解释**】`processAliasRegistration(ele)`这个方法的代码逻辑比较简单，与在上文的 2.3 中讲的内容一样，都是将别名 alias 与 beanName 组成一对注册到 registry 中。此处不再赘述。

四、import 标签的解析
==============

对于项目中的大量 Spring 配置文件而言，如果我们采取**分模块维护**，那么更易于我们的管理。我们可以通过采用`<import>`标签，来引入不同模块的配置文件，具体如下所示：

```
<beans>
    <import resource="order.xml" />
    <import resource="stock.xml" />
</beans>

```

关于 import 标签的解析逻辑，我们来看如下源码：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbOGwfLFGhtWQDDhczVv4lHSEXicg60z6ibK7M0eazS6P0icqbwibxmUlVvA/640?wx_fmt=png)

其中，`importBeanDefinitionResource(ele)`方法的详细源码内容和注释，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbuEBddzUzpmmaMy7S2yKjz6LC11ksibfozID8FQ8YadmYgiaryZ2mgzibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbMfgf6Swb951DwPVLDTElYqNUxXFlvcL0PPoaic9I9oE68nJgqqTibL4A/640?wx_fmt=png)

> 【**解释**】  
> **1>** 获取`resource`属性所表示的路径。  
> **2>** 解析路径中的系统属性，格式如 “`${user.dir}`”。  
> **3>** 判定`location`是绝对路径还是相对路径。  
> **4>** 如果是绝对路径，则递归调用`bean`的解析过程，进行另一次的解析。  
> **5>** 如果是相对路径，则计算出绝对路径并进行解析。  
> **6>** 通知监听器，解析完成（Spring 没有实现内部逻辑）。

五、beans 标签的解析
=============

对于嵌入式的`beans`标签，非常类似于 import 标签所提供的功能。具体源码位置为下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicIpial2XP7oAu30ZSKAVHhbEbrLKfBX0g7DiaQzR1gyrhb3DbJbbI9jlCyEkBD6Z4lRgaibgzzmEb6w/640?wx_fmt=png)

对于嵌入式`beans`标签来讲，并没有太多可讲，**与单独的配置文件并没有太大的差别，无非是递归调用 beans 的解析过程**。并且，在`第1讲`的`2.3.2章节`，就对`doRegisterBeanDefinitions(ele)`方法进行了解析，此处就不在赘述了。

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的 **点赞** & **分享** 。

更多技术干货，欢迎大家关注公众号 “**爪哇缪斯**” ~ \(^o^)/ ~ 「干货分享，每天更新」

往期推荐
----

[（一）Spring 源码解析：容器的基本实现](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491156&idx=1&sn=b2ba03821b9be0fd8b7d6c86cb793256&chksm=e9115ca9de66d5bf094badcc0bf517e0f539693951403b0d68174c4066ab03e9c22c0c852eed&scene=21#wechat_redirect)  

[SpringBoot2.x——Part1](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486233&idx=1&sn=fda7d410bedf6b56457757225a4d6bfa&chksm=e91149e4de66c0f2acd4a95cba907f43dbbd0700c8f70ccd9fc0cc7f57048683732cf79fc77b&scene=21#wechat_redirect)  

[SpringBoot2.x——SpringBoot Web 源码解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486972&idx=1&sn=74556f31641a1bd3c29a2663bc533208&chksm=e9114f01de66c6177f47b26297b0836b5df0e92feea7a7420f611b3d3a2f8926eb952d20b117&scene=21#wechat_redirect)

[SpringCloud——Eureka（超详细图文）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247485219&idx=1&sn=15771c920bb990b2651b836bcc9c94a4&chksm=e91145dede66ccc8103ee7c9c5578debc408a5acc3437c077e2f5ea58664bd3b0a211b1c07a5&scene=21#wechat_redirect)  

[SpringCloud Alibaba——Sentinel](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247487759&idx=1&sn=b1a26b7f7654fade3bf15377de1cb77d&chksm=e91153f2de66dae4de1f1ed5e0f2d9490f53fb510a9ce534e6e6da316f80be4ae05fa4b1688e&scene=21#wechat_redirect)  

[SpringCloud Alibaba——Nacos 服务发现](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486401&idx=1&sn=ab5a818663fec1588f42125aee812ba3&chksm=e911493cde66c02a00a63db1af4dd0caf7bf17f88083817e5bf2b1fec037c54a7a0a8ce7d50b&scene=21#wechat_redirect)  

[SpringCloud——Gateway（万字图文）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486062&idx=1&sn=ff7c575533ee8dc3e4aa7a725ce58e7d&chksm=e9114893de66c1851b156907839ef83cbf05fba3e95ee16fc948215a8f5e5290c5e74b5b491c&scene=21#wechat_redirect)