
〇、AOP 概念
========

**Aspect**：切面

> 给业务方法增加到功能，切面泛指交叉业务逻辑。上例中的事务处理、日志处理就可以理解为切面。常用的切面是通知（Advice）。实际就是对主业务逻辑的一种增强。

**Pointcut**：切入点

> 切入点指声明的一个或多个连接点的集合，通过切入点指定一组方法。被标记为 final 的方法是不能作为连接点与切入点的。因为最终的是不能被修改的，不能被增强的。

**Advice**：通知、增强

> 通知表示切面的执行时间，Advice 也叫增强。换个角度来说，通知定义了增强代码切入到目标代码的时间点，是目标方法执行之前执行，还是之后执行等。通知类型不同，切入时间不同。

**JoinPoint**：连接点

> 连接切面的业务方法，连接点指可以被切面织入的具体方法。通常业务接口中的方法均为连接点。

**Target**：目标对象

> 目标对象指将要被增强的对象。即包含主业务逻辑的类的对象。

AOP 中重要的三个要素：`Aspect`、`Pointcut`、`Advice`

> 意思是说：在 Advice 的时间、在 Pointcut 的位置，执行 Aspect。

为了方便大家理解 AOP 中的相关概念，请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpLaQ00fZInLRpKX1iaMgJpAsqibQOgSZt3y8HQnaeCVJFeJjU2uRsLEdg/640?wx_fmt=png)

一、动态 AOP 的使用示例
==============

当我们对某些类有**横切性**的逻辑时，为了不破坏目标类，我们则可以使用 AOP 的方式将增强逻辑注入到目标类上。为了更清晰的了解 AOP 的用法，下面我们通过一个使用案例，实现一下面向切面编程。

首先，我们创建一个普通的业务类 **MuseAop**

```
public class MuseAop {
    public void goToWork() {
        System.out.println("Muse去上班");
    }
}

```

其次，创建 **Advisor**，然后对 MuseAop 的`goToWork()`方法进行增强操作

```
@Aspect
public class MuseAspectJ {
    @Pointcut("execution(* *.goToWork(..))")
    public void goToWork() {}

    @Before("goToWork()")
    public void beforeGoToWork() {
        System.out.println("@Before：起床、洗漱、穿衣");
    }

    @After("goToWork()")
    public void afterGoToWork() {
        System.out.println("@After：开始工作了");
    }

    @SneakyThrows
    @Around("goToWork()")
    public Object aroundGoToWork(ProceedingJoinPoint point) {
        System.out.println("@Around-1：听一首摇滚歌曲提提神");
        Object result = point.proceed();
        System.out.println("@Around-2：听一首钢琴乐舒缓情绪");
        return result;
    }
}

```

然后，在配置文件中添加 bean，并且通过配置 **<aop:aspectj-autoproxy />** 来开启 aop 动态代理

```
<aop:aspectj-autoproxy />
<bean/>
<bean/>

```

最后，编写测试类，查看测试结果

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygp7huZU2pBg1JQZkk4FDMPT4QD2owsaUnpxof5uwZnKKZdv0g8hniaYgg/640?wx_fmt=png)

二、动态 AOP 自定义标签
==============

根据上面我们使用 AOP 的示例，我们可以看到是通过配置`<aop:aspectj-autoproxy>`来开启动态代理的，因此我们可以将它为 AOP 源码分析的切入点。请见下图所示，我们在全项目中搜索了`aspectj-autoproxy`，然后发现注入了新的`BeanDefinitionParser`实现类—— **AspectJAutoProxyBeanDefinitionParser**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygphckBX9W6fyTYO21bRCg6icvAqVNhzEHoycXghicESvELQo3J6MCD9oUw/640?wx_fmt=png)

那么下面，我们来看一下 **AspectJAutoProxyBeanDefinitionParser** 类的具体实现，由于 AspectJAutoProxyBeanDefinitionParser 实现了`BeanDefinitionParser`接口，而 BeanDefinitionParser 只有一个方法，即：`parse(element, parserContext)`，所以，我们来看一下 **AspectJAutoProxyBeanDefinitionParser** 类是如何实现`parse`方法的。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpwiakTYFCEWo1XRK98GicUPmibCZ6GttuYwibiaXhTHopU1wqQkfdjk4McCA/640?wx_fmt=png)

下面，我们先看一下 **AopNamespaceUtils** 类的`registerAspectJAnnotationAutoProxyCreatorIfNecessary(parserContext, element)`方法的具体实现逻辑。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpzia30RNcIsF3KTDZjaIGtMNKb0FuLprRIXlXOetITYUzPiaA18aKEmFQ/640?wx_fmt=png)

2.1> registerAspectJAnnotationAutoProxyCreatorIfNecessary 方法解析
--------------------------------------------------------------

该方法是用于**注册或者升级 AnnotationAwareAspectJAutoProxyCreator 类型的 APC**，对于 AOP 的实现，基本都是在 **AnnotationAwareAspectJAutoProxyCreator** 类中完成的，它可以根据`@Point`注解定义的切点来自动代理相匹配的 bean。但是为了配置简便，Spring 使用了自定义配置来帮助我们自动注册`AnnotationAwareAspectJAutoProxyCreator`，注册流程如下所示：

```
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry, Object source) {
    return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
}

```

**AUTO_PROXY_CREATOR_BEAN_NAME**（“`org.springframework.aop.config.internalAutoProxyCreator`”）的作用是内部管理的自动代理创建器的`bean名称`。如果名称为 “`AUTO_PROXY_CREATOR_BEAN_NAME`” 的 apc 实例在容器中已经存在，则试图替换为该 bean 为`AnnotationAwareAspectJAutoProxyCreator`类型；否则，在容器中创建`AnnotationAwareAspectJAutoProxyCreator`类型的 APC 实例。

```
private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
    /** 步骤1：如果容器中已经存在apc实例，则试图替换为AnnotationAwareAspectJAutoProxyCreator类型 */
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
            int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
            int requiredPriority = findPriorityForClass(cls);
            if (currentPriority < requiredPriority) apcDefinition.setBeanClassName(cls.getName());
        }
        return null;
    }
    /** 步骤2：如果不存在，则向容器中创建AnnotationAwareAspectJAutoProxyCreator类型的apc实例对象 */
    RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
    beanDefinition.setSource(source);
    beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
    beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
    return beanDefinition;
}

```

### 2.1.1> 关于 APC 优先级别的补充说明

APC 的抽象类 **AbstractAdvisorAutoProxyCreator** 默认有 **4 个**实现类，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpZPgoNvZxv3tH6qAKJT7dH52v2nMtSpfbChcb7YKXF281gBRjJOnBDA/640?wx_fmt=png)

我们可以通过`AopConfigUtils.findPriorityForClass(...)`方法来获得当前 APC 实现类的优先级别，`数字越大，优先级别越高`。**这个优先级，其实就是 ArrayList 中存储的 APC 实现类的 index 序号**。源码请见下图所示：

```
/** 默认初始化3个APC（AutoProxyCreator）实现类 */
private static final List<Class<?>> APC_PRIORITY_LIST = new ArrayList<>(3);
static {
    APC_PRIORITY_LIST.add(InfrastructureAdvisorAutoProxyCreator.class); // 第0级别
    APC_PRIORITY_LIST.add(AspectJAwareAdvisorAutoProxyCreator.class); // 第1级别
    APC_PRIORITY_LIST.add(AnnotationAwareAspectJAutoProxyCreator.class); // 第2级别
}

/** 通过Class获得优先级别 */
private static int findPriorityForClass(Class<?> clazz) {
    return APC_PRIORITY_LIST.indexOf(clazz);
}

/** 通过className获得优先级别 */
private static int findPriorityForClass(@Nullable String className) {
    for (int i = 0; i < APC_PRIORITY_LIST.size(); i++) {
        Class<?> clazz = APC_PRIORITY_LIST.get(i);
        if (clazz.getName().equals(className)) return i;    
    }
}

```

> 【**第 0 级别**】InfrastructureAdvisorAutoProxyCreator  
> 【**第 1 级别**】AspectJAwareAdvisorAutoProxyCreator  
> 【**第 2 级别**】AnnotationAwareAspectJAutoProxyCreator

2.2> useClassProxyingIfNecessary 方法解析
-------------------------------------

步骤 1：获得属性 **PROXY_TARGET_CLASS_ATTRIBUTE**（“`proxy-target-class`”）

> 如果`proxy-target-class`等于 true，才会执行 AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry) 方法；

步骤 2：属性 **EXPOSE_PROXY_ATTRIBUTE**（“`expose-proxy`”）

> 如果`expose-proxy`等于 true，才会执行 AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry) 方法；

`useClassProxyingIfNecessary()`方法源码如下所示：

```
private static void useClassProxyingIfNecessary(BeanDefinitionRegistry registry, Element sourceElement {
    if (sourceElement != null) {
        // 设置参数proxy-target-class
        boolean proxyTargetClass = Boolean.parseBoolean(sourceElement.getAttribute(PROXY_TARGET_CLASS_ATTRIBUTE));
        if (proxyTargetClass) AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);

        // 设置参数expose-proxy
        boolean exposeProxy = Boolean.parseBoolean(sourceElement.getAttribute(EXPOSE_PROXY_ATTRIBUTE));
        if (exposeProxy) AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
    }
}

```

这两个方法逻辑基本一致的，就是首先判断是否存在名字为 **AUTO_PROXY_CREATOR_BEAN_NAME** 的 BeanDefinition 实例对象，如果存在的话，将该对象的属性`proxyTargetClass`或者属性`exposeProxy`赋值为 true 即可。

```
/** 将definition的proxyTargetClass属性设置为true */
public static void forceAutoProxyCreatorToUseClassProxying(BeanDefinitionRegistry registry) {
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        BeanDefinition definition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        definition.getPropertyValues().add("proxyTargetClass", Boolean.TRUE);
    }
}
/** 将definition的exposeProxy属性设置为true */
public static void forceAutoProxyCreatorToExposeProxy(BeanDefinitionRegistry registry) {
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        BeanDefinition definition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        definition.getPropertyValues().add("exposeProxy", Boolean.TRUE);
    }
}

```

`proxyTargetClass`属性的作用是什么？

> **<aop:config proxy-target-class="true">** ：配置为强制使用`CGLIB`代理  
> **<aop:aspectj-autoproxy proxy-target-class="true"/>** ：配置为`CGLIB`代理 +`@AspectJ`自动代理支持

`exposeProxy`属性的作用是什么？

> **<aop:aspectj-autoproxy export-proxy="true"/>** ：支持通过`AopContext.currentProxy()`来暴露当前代理类。

`exposeProxy`场景举例：

```
public interface AService {
    public void a();
    public void b();
}

@Service
public class AserviceImpl implements AService {
    @Transactional(propagation = Propagation.REQUIRED)
    public void a() {
        this.b(); // 由于this指向target对象，所以不会执行b事务切面
        ((AService) AopContext.currentProxy()).b(); // 暴露了当前的代理类，所以可以执行b事务切面
    }
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void b() {}
}

```

三、创建 AOP 代理
===========

通过上面的介绍，我们可以看到 AOP 在极力的创建 **AnnotationAwareAspectJAutoProxyCreator** 对象作为 APC 的 bean 实例：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpV2Nkvv207TvvfMZv5TShIV9wI5Suib42rSP2qFCLYsU6U6l9jDF2xrg/640?wx_fmt=png)

既然是这样的，那我们就来看一下`AnnotationAwareAspectJAutoProxyCreator`类的继承结构，请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpT8gFHU2fMEl80355OMIHhcD1MCtwLcdJI7cGQysFGYxicibdDko9eWsA/640?wx_fmt=png)

从上图中我们可以看到它实现了 **BeanPostProcessor** 接口，那么这个接口里有我们熟悉的`postProcessAfterInitialization(...)`方法，该方法是由 **AbstractAutoProxyCreator** 类实现的，源码请见如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpB0XABMnj0JyHXudyX8sdJibzwUeiasvVzavLbibKFKyTxavQWmtUsq5ew/640?wx_fmt=png)

下面真正执行对 bean 增强的方法就是`wrapIfNecessary(...)`了，在这里有如下几种情况是不需要增加的：

> 【**情况 1**】**targetSourcedBeans** 中保存的是已经处理过的 bean，如果在其中，则不需要增强；  
> 【**情况 2**】**advisedBeans** 中 value 值为 false 表示不需要增强；  
> 【**情况 3**】**基础设施类**（实现`Advices`、`Pointcut`、`Advisors`、`AopInfrastructureBeans`这四个接口的类），则不需要增强；  
> 【**情况 4**】如果是 **Original 实例**（以`beanClass.getName()`开头，并且以 "`.ORIGINAL`" 结尾），则不需要增强；

在`wrapIfNecessary(...)`方法中，主要有两个重要的方法：**getAdvicesAndAdvisorsForBean(...)** 和 **createProxy(...)** ，后续我们会针对这两个方法进行解析。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpWTqP2VAQkYpAeacGqp32u2zUPMtchibrU6ZhBF4mLkjmiaQpoxGXUf2w/640?wx_fmt=png)

根据上面的描述，我们可以知道`getAdvicesAndAdvisorsForBean(...)`方法就是用来获得增强器的方法了，这里通过调用`findEligibleAdvisors(beanClass, beanName)`方法来获得增强器列表，并进行结果返回；如果没有获得增强器，则返回`DO_NOT_PROXY`（其值为 null），其源码如下所示：

```
protected Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, TargetSource targetSource) {
    /** 寻找增强列表 */
    List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
    if (advisors.isEmpty()) return DO_NOT_PROXY;
    return advisors.toArray();
}

```

`Eligible`的英文翻译是 “**符合条件的**”，那么`findEligibleAdvisors(...)`方法的主要作用就是——**找到符合条件的增强器**，具体操作有如下两步：

> 【**步骤 1**】通过调用`findCandidateAdvisors(...)`方法，获取所有的增强；  
> 【**步骤 2**】通过调用`findAdvisorsThatCanApply(...)`方法，寻找所有增强中适用于 bean 的增强并进行应用；

相关源码，请见如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpUxOCTeg6wXDkwCVfPuvPMibbyVOs6MibaLI7C6Zt4ib0GozZHiaY5y1RPA/640?wx_fmt=png)

3.1> findCandidateAdvisors() 获取所有增强器
------------------------------------

在`findCandidateAdvisors()`方法中，我们可以获得所有增强器，此处一共执行了两个步骤来获得所有增强器：

> 【**步骤 1**】获得 **xml** 中配置的 AOP 声明;  
> 【**步骤 2**】获得带有 **aspectj 注释**的 AOP 声明;

如下就是`findCandidateAdvisors()`方法的相关源码：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpjsEH0Q51d1QxnhhUFvwicMT4dpbaSlyE3uNFCI2WEfCiblK84Vuer3tw/640?wx_fmt=png)

### 3.1.1> findCandidateAdvisors() 寻找在 IOC 中注册过的 Advisor 接口实现类

```
protected List<Advisor> findCandidateAdvisors() {
    return this.advisorRetrievalHelper.findAdvisorBeans();
}

```

在`findAdvisorBeans()`方法中我们可以看到，如下逻辑：

> 【**步骤 1**】首先，尝试从**缓存**（`cachedAdvisorBeanNames`）中获得 Advisor 类型的 bean 名称列表。  
> 【**步骤 2**】如果没有获得到，则试图去 **IOC 容器**中获得所有 Advisor 类型的 bean 名称列表。  
> 【**步骤 3**】如果都没有获得 Advisor 类型的 bean 名称列表，则直接返回空集合。  
> 【**步骤 4**】如果不为空，则通过`beanFactory.getBean(name, Advisor.class)`来获得 Advisor 实例集合，并进行返回。

如下就是`findAdvisorBeans()`方法的相关源码：

```
public List<Advisor> findAdvisorBeans() {
    //【步骤1】获得所有被缓存的Advisor的bean名称列表
    String[] advisorNames = this.cachedAdvisorBeanNames;

    //【步骤2】如果缓存中没有，那么我们就从BeanFactoryUtils中获得Advisor的bean名称列表，然后作为已缓存的bean名称列表
    if (advisorNames == null) {
        //【官方注释】这里不要初始化FactoryBeans:我们需要保留所有未初始化的常规bean，以便让自动代理创建器应用于它们!
        // 返回容器中所有Advisor类型的bean名称列表
        advisorNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(this.beanFactory, Advisor.class, true, false);
        this.cachedAdvisorBeanNames = advisorNames;
    }

    //【步骤3】如果缓存也没有并且从从BeanFactoryUtils中也没获得到，则直接返回空集合
    if (advisorNames.length == 0) return new ArrayList<>();

    //【步骤4】从IOC容器中获得Advisor对象实例集合，并返回
    List<Advisor> advisors = new ArrayList<>();
    for (String name : advisorNames) {
        if (isEligibleBean(name)) {
            if (this.beanFactory.isCurrentlyInCreation(name)) 
                if (logger.isTraceEnabled()) logger.trace("Skipping currently created advisor '" + name + "'");
            else {
                try {
                    advisors.add(this.beanFactory.getBean(name, Advisor.class));
                } catch (BeanCreationException ex) { ... ... }
            }
        }
    }
    return advisors;
}

```

### 3.1.2> buildAspectJAdvisors() 获得带有 aspectj 注释的 AOP 声明

在`buildAspectJAdvisors()`方法中，主要逻辑有四个步骤：

> 【**步骤 1**】获得 Aspect 的`beanName`列表；  
> 【**步骤 2**】通过`beanName`来获得 **MetadataAwareAspectInstanceFactory** 实例，具体如下所示：如果`per-clauses`(aspect 实例化模式) 类型等于 SINGLETON，则创建 **BeanFactoryAspectInstanceFactory** 类型的`factory`；否则，则创建 **PrototypeAspectInstanceFactory** 类型的`factory`；  
> 【**步骤 3**】通过调用`advisorFactory.getAdvisors(factory)`来获得 Advisor 列表；  
> 【**步骤 4**】维护 **advisorsCache** 缓存或 **aspectFactoryCache** 缓存；如果 beanName 是单例的，则将`beanName`和`Advisor列表`维护到 **advisorsCache** 缓存中；否则，将`beanName`和`factory`维护到 **aspectFactoryCache** 缓存中；

在`buildAspectJAdvisors()`方法中，源码及注释如下所示：

```
public List<Advisor> buildAspectJAdvisors() {
    // 获得已经解析过的Aspect的beanName列表
    List<String> aspectNames = this.aspectBeanNames;

    // 步骤1：如果aspectNames为空，则试图从IOC中解析出Aspect的beanName列表
    if (aspectNames == null) {
        synchronized (this) { // 加锁
            aspectNames = this.aspectBeanNames; // double check
            if (aspectNames == null) {
                List<Advisor> advisors = new ArrayList<>();
                aspectNames = new ArrayList<>();
                // 获得IOC容器中所有的beanName
                String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(this.beanFactory, Object.class, true, false);
                for (String beanName : beanNames) {
                    if (!isEligibleBean(beanName)) continue; // isEligibleBean方法默认返回true
                    // 官方注释：我们必须小心不要急切地实例化bean，因为在这种情况下，它们将被Spring容器缓存，但不会被织入
                    Class<?> beanType = this.beanFactory.getType(beanName, false);
                    if (beanType == null) continue;

                    // 一个类如果包含@Aspect注解并且不是被ajc编译的类，则返回true
                    if (this.advisorFactory.isAspect(beanType)) {
                        aspectNames.add(beanName);
                        AspectMetadata amd = new AspectMetadata(beanType, beanName);
                        if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
                            // 单例则创建BeanFactoryAspectInstanceFactory类型的factory
                            MetadataAwareAspectInstanceFactory factory = new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
                            List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
                            if (this.beanFactory.isSingleton(beanName)) 
                                this.advisorsCache.put(beanName, classAdvisors); // 缓存Advisor
                            else 
                                this.aspectFactoryCache.put(beanName, factory); // 缓存MetadataAwareAspectInstanceFactory
                            advisors.addAll(classAdvisors);
                        } else {
                            if (this.beanFactory.isSingleton(beanName)) {
                                throw new IllegalArgumentException("Bean with name '" + beanName +
                                        "' is a singleton, but aspect instantiation model is not singleton");
                            }
                            // 非单例则创建PrototypeAspectInstanceFactory类型的factory
                            MetadataAwareAspectInstanceFactory factory = new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
                            this.aspectFactoryCache.put(beanName, factory); // 缓存MetadataAwareAspectInstanceFactory
                            advisors.addAll(this.advisorFactory.getAdvisors(factory));
                        }
                    }
                }
                this.aspectBeanNames = aspectNames;
                return advisors; // 将解析好的Advisor列表执行返回操作
            }
        }
    }
    if (aspectNames.isEmpty()) return Collections.emptyList();

    // 步骤2：如果没有“经历过”步骤1，则再次处解析Advisor列表
    List<Advisor> advisors = new ArrayList<>();
    for (String aspectName : aspectNames) {
        List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
        if (cachedAdvisors != null) // 情况1：如果在advisorsCache缓存中存在，则直接返回Advisor列表
            advisors.addAll(cachedAdvisors); 
        else { // 情况2：如果在aspectFactoryCache缓存中存在，则需要调用factory的getAdvisors方法来获得Advisor列表
            MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
            advisors.addAll(this.advisorFactory.getAdvisors(factory)); 
        }
    }
    return advisors;
}

```

在上面的源码中，我们可以发现，其中通过 **advisorFactory.getAdvisors(factory)** 来获得 Advisor 集合是非常核心的代码，因为只有它才能帮助我们获得 Advisor 列表，部分截取代码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygp6g3hLehaxrQlrwMiazbTMnpFLdyotI8OhfXjoY8RPIuWmfywO1k8BMQ/640?wx_fmt=png)

`advisorFactory.getAdvisors(factory)`方法的源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpv3bT8LK5xpSS1pLtw3LoiaOnznFT4sxQlpscia0CspEqahoERYGL8tWw/640?wx_fmt=png)

### 3.1.3> getAdvisor(...) 获得普通增强器

`getAdvisor(...)`方法的源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpTicZRvNZicDZKlk6GdnNJjZo2rNZdCfUU9Sx0kxewXpibnicT0McwiaTiaVw/640?wx_fmt=png)

#### a> 步骤 1：获得切点表达式的相关信息

下面我们来看一下**步骤 1** 中的**获得切点表达式的相关信息**的`getPointcut(...)`方法源码逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpgGyibMPDu98Gq1V5NiayHMqsmWWGITBEiafS9EtuErFibpicM4ogoo2Gm5g/640?wx_fmt=png)

方法上的 AspectJ 相关注解（`AspectJAnnotation`），查找注解的顺序按照：`@Pointcut——>@Around——>@Before——>@After——>@AfterReturning——>@AfterThrowing` ，如果找到了某个注解，则直接返回`AspectJAnnotation`实例对象，不需要继续寻找了。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpyPqicpbIat311R8MIJFRHquZficERkzjGfJMPmk9tTn8JAZmDgEGwoibw/640?wx_fmt=png)

#### b> 步骤 2：根据切点信息生成增强

我们可以看到，在步骤 2 中，是通过创建一个`InstantiationModelAwarePointcutAdvisorImpl`实例对象来生成切点的增强的，源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpCXXz9HO4dAOVnUzRDuicLhv6XPFvDRYQdPU4ib4sGStQr34kTI1MjicOw/640?wx_fmt=png)

在`getAdvice`方法中，我们根据上面解析出的方法上面使用的 AspectJ 注解来生成相应的 **AbstractAspectJAdvice**，代码如下所示：

```
public Advice getAdvice(Method candidateAdviceMethod, AspectJExpressionPointcut expressionPointcut,
                        MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder,
                        String aspectName) {
    Class<?> candidateAspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass(); // 获得切面类
    validate(candidateAspectClass);
    // 获得切面上的AspectJ相关注解
    AspectJAnnotation<?> aspectJAnnotation = AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod); 
    if (aspectJAnnotation == null) return null; // 如果没有AspectJ相关注解，那么直接返回null即可

    // 如果类上有@Aspect注解 并且 该类是被AspectJ编译的，则直接抛出异常
    if (!isAspect(candidateAspectClass))
        throw new AopConfigException("Advice must be declared inside an aspect type: Offending method '" +
                candidateAdviceMethod + "' in class [" + candidateAspectClass.getName() + "]");

    // 根据不同的AspectJ相关注解，生成对应不同的springAdvice的实例对象
    AbstractAspectJAdvice springAdvice;
    switch (aspectJAnnotation.getAnnotationType()) {
        case AtPointcut: // @Pointcut
            return null;
        case AtAround: // @Around
            springAdvice = new AspectJAroundAdvice(candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtBefore: // @Before
            springAdvice = new AspectJMethodBeforeAdvice(candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtAfter: // @After
            springAdvice = new AspectJAfterAdvice(candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtAfterReturning: // @AfterReturning
            springAdvice = new AspectJAfterReturningAdvice(candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterReturning afterReturningAnnotation = (AfterReturning) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterReturningAnnotation.returning()))
                springAdvice.setReturningName(afterReturningAnnotation.returning());
            break;
        case AtAfterThrowing: // @AfterThrowing
            springAdvice = new AspectJAfterThrowingAdvice(candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterThrowing afterThrowingAnnotation = (AfterThrowing) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterThrowingAnnotation.throwing()))
                springAdvice.setThrowingName(afterThrowingAnnotation.throwing());
            break;
        default:
            throw new UnsupportedOperationException("Unsupported advice type on method: " + candidateAdviceMethod);
    }
    // 为springAdvice实例对象赋值
    springAdvice.setAspectName(aspectName);
    springAdvice.setDeclarationOrder(declarationOrder);
    String[] argNames = this.parameterNameDiscoverer.getParameterNames(candidateAdviceMethod);
    if (argNames != null) springAdvice.setArgumentNamesFromStringArray(argNames);
    springAdvice.calculateArgumentBindings();
    return springAdvice;
}

```

### 3.1.4> new SyntheticInstantiationAdvisor(...) 创建同步实例化增强器

如果寻找的**增强器不为空**而且又**配置了增强延迟初始化**，那么就需要在首位加入同步实例化增强器。同步实例化增强器`SyntheticlnstantiationAdvisor`源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpribvHdXniccXPFQiaxeESVGN3tmgRSibFkKYibCnT12Tq4ZD8kMNtt457Xw/640?wx_fmt=png)

### 3.1.5> getDeclareParentsAdvisor(field) 获得 @DeclareParents 配置的增强器

DeclareParents 主要用于**引介增强**的注解形式的实现，如果属性上使用了`@DeclareParents`注解，那么我们就来创建 **DeclareParentsAdvisor** 类型的增强器，其中关于 @DeclareParents 注解的使用场景，请参照 3.1.6 部分即可，源码如下所示：

```
private Advisor getDeclareParentsAdvisor(Field introductionField) {
    // 获得属性上使用了@DeclareParents注解的注解类
    DeclareParents declareParents = introductionField.getAnnotation(DeclareParents.class);
    if (declareParents == null) return null;
    if (DeclareParents.class == declareParents.defaultImpl()) throw new IllegalStateException(...);
    // 创建DeclareParentsAdvisor类型的增强器
    return new DeclareParentsAdvisor(introductionField.getType(), declareParents.value(), declareParents.defaultImpl());
}

```

### 3.1.6> @DeclareParents 注解的使用

创建接口 **IPay** 和实现类 **OnlinePay**

```
public interface IPay {
    void pay();
}

```

```
@Service
public class OnlinePay implements IPay {
    @Override
    public void pay() {
        System.out.println("-------OnlinePay--------");
    }
}

```

创建接口 **IPayPlugin** 和实现类 **AlipayPlugin**

```
public interface IPayPlugin {
    void payPlugin();
}

```

```
@Service
public class AlipayPlugin implements IPayPlugin {
    @Override
    public void payPlugin() {
        System.out.println("-------Alipay--------");
    }
}

```

使用`@DeclareParents`注解配置切面。该注解的作用是：**可以在代理目标类上增加新的行为**（即：新增新的方法）。

```
@Aspect
@Component
public class PayAspectJ {
    @DeclareParents(value = "com.muse.springbootdemo.entity.aop.IPay+",defaultImpl = AlipayPlugin.class)
    public IPayPlugin alipayPlugin;
}

```

创建配置类 **MuseConfig**，开启 AspectJ 的自动代理，然后就会为使用`@Aspect`注解的 bean 创建一个代理类。

```
@Configuration
@ComponentScan
@EnableAspectJAutoProxy
public class MuseConfig {
}

```

进行测试，我们发现从 IOC 中获得的 bean **实现了 IPay 和 IPayPlugin 这两个接口**，并且会在后台输出 “`-------OnlinePay--------`” 和 “`-------Alipay-------`”

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpWfz9qkLXAD1r50noljrv42iaw1ybF5x8c3ZDkM7nN9bdrsExkiaMYrMg/640?wx_fmt=png)

3.2> findAdvisorsThatCanApply(...) 寻找匹配的增强器
-------------------------------------------

在 3.1 中，我们已经分析完获取所有增强器的方法`findCandidateAdvisors()`，那么本节我们将在获取的所有增强（`candidateAdvisors`）基础上，再去寻找匹配的增强器，即：`findAdvisorsThatCanApply(...)`方法，相关源码如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpXibibV8dOHuicPDbrHoPd6nn8aD2OcaxR8K6icUYYO9icWBKOaffqxzMqKw/640?wx_fmt=png)

在`findAdvisorsThatCanApply(...)`方法中，其主要功能是**获得所有增强器 candidateAdvisors 中，适用于当前 clazz 的增强器列表**。而由于针对`引介增强`和`普通增强`的处理是不同的， 所以采用分开处理的方式，请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpKXZ2BAnExa5bjLibkcyF5P8eDUsmpAXN5hxoL4UeAPl85nVo4AcwM9A/640?wx_fmt=png)

> 那么，**什么是引介增强呢？** 引介增强是一种特殊的增强，其它的增强是方法级别的增强，即只能在方法前或方法后添加增强。而引介增强则不是添加到方法上的增强， 而是添加到**类级别的增强**，即：可以为目标类动态实现某个接口，或者动态添加某些方法。具体实现请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpNv7sFYz6ib3U0ELKmak4Fsu8vm6ACIDcH6ibZLcia403g4cwtREPesiaqg/640?wx_fmt=png)

那么，在上面的`findAdvisorsThatCanApply(...)`方法源码中，我们可以发现，`canApply(...)`方法是其中很重要的判断方法，那么它内部主要做了什么操作呢？在其方法内部，依然根据**引介增强**和**普通增强**两种增强形式分别进行的判断，其中，如果是引介增强的话，则判断该增强是否可以应用在 targetClass 上，如果可以则返回 true，否则返回 false。那么，如果是普通增强，则需要再调用`canApply(...)`方法继续进行逻辑判断，相关源码请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygp3icichKVpfxW0SJCJQvgayNkImytpuibnvnc691EKTIDSt37upibDbQ7EQ/640?wx_fmt=png)

在`canApply(...)`方法中，主要是逻辑是获得 **targetClass 类（非代理类）** 及 **targetClass 类的相关所有接口** 中的所有方法去匹配，是否满足对 targetClass 类的增强，如果找到了，则返回 false；如果找不到，则返回 true；相关源码，请见下图所示：

```
public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
    // 如果Pointcut不能应用于targetClass类上，则直接返回false
    if (!pc.getClassFilter().matches(targetClass)) return false;

    MethodMatcher methodMatcher = pc.getMethodMatcher();
    if (methodMatcher == MethodMatcher.TRUE) return true;

    IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
    if (methodMatcher instanceof IntroductionAwareMethodMatcher) 
        introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;

    Set<Class<?>> classes = new LinkedHashSet<>();
    if (!Proxy.isProxyClass(targetClass)) classes.add(ClassUtils.getUserClass(targetClass)); // 只添加非代理类
    classes.addAll(ClassUtils.getAllInterfacesForClassAsSet(targetClass)); // 添加targetClass的所有接口类

    // 遍历所有相关类的所有方法，只要有与targetClass匹配的方法，则返回ture
    for (Class<?> clazz : classes) {
        Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
        for (Method method : methods) {
            if (introductionAwareMethodMatcher != null ?
                    introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions) :
                    methodMatcher.matches(method, targetClass)) {
                return true;
            }
        }
    }
    return false;
}

```

3.3> createProxy(...) 创建 AOP 代理
-------------------------------

在 3.1 和 3.2 章节中，我们介绍了如何获取 bean 所对应的 Advisor 增强器，那么，下面我们就该开始利用这些增强器去配合创建代理对象了，这部分工作由`createProxy()`方法负责，源码如下所示：

```
/** 为beanClass创建AOP代理 */
protected Object createProxy(Class<?> beanClass, String beanName, Object[] specificInterceptors, 
                             TargetSource targetSource) {
    if (this.beanFactory instanceof ConfigurableListableBeanFactory)
        AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, 
                                         beanName, beanClass);

    ProxyFactory proxyFactory = new ProxyFactory();
    proxyFactory.copyFrom(this); // 将当前对象中的信息复制到proxyFactory实例中

    // 调用proxyFactory.setProxyTargetClass(...)用于设置是否应该使用targetClass而不是它的接口代理 
    // 调用proxyFactory.addInterface(...)用于添加代理接口
    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) proxyFactory.setProxyTargetClass(true);
        else evaluateProxyInterfaces(beanClass, proxyFactory);
    }

    /** 3.3.1> 获得所有增强器 */
    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    proxyFactory.addAdvisors(advisors); // 添加增强器集合
    proxyFactory.setTargetSource(targetSource); // 设置被代理的类
    customizeProxyFactory(proxyFactory); // 定制代理（空方法）
    proxyFactory.setFrozen(this.freezeProxy); // 默认false，表示代理被配置后，就不允许修改它的配置了
    if (advisorsPreFiltered()) proxyFactory.setPreFiltered(true);
    ClassLoader classLoader = getProxyClassLoader();
    if (classLoader instanceof SmartClassLoader && classLoader != beanClass.getClassLoader())
        classLoader = ((SmartClassLoader) classLoader).getOriginalClassLoader();

    /** 3.3.2> 获得代理对象 */
    return proxyFactory.getProxy(classLoader);
}

```

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpTJS9kTlKDnMjfbuTcricibJsMZ8aPMIp6We6uV8eKG0PiaQXhhLDbbyyQ/640?wx_fmt=png)

> 从上面的源码我们可以整理出`createProxy(...)`方法的操作流程：  
> 【**步骤 1**】创建 **ProxyFactory** 实例对象，后续会对其各个参数进行初始化赋值，为最后调用 proxyFactory 的`getProxy(...)`方法做准备；  
> 【**步骤 2**】将**当前对象**（this）中的部分信息复制到 proxyFactory 实例中；  
> 【**步骤 3**】调用 proxyFactory.`setProxyTargetClass(...)`用于**设置是否应该使用 targetClass 而不是它的接口代理**；  
> 【**步骤 4**】调用 proxyFactory.`addInterface(...)`用于**添加代理接口**；  
> 【**步骤 5**】获得所有**增强器 Advisor** 并添加到 proxyFactory 中；  
> 【**步骤 6**】向 proxyFactory 中设置**被代理的类**；  
> 【**步骤 7**】可以对 proxyFactory **进行定制化操作**，默认是空方法；  
> 【**步骤 8】**通过调用 proxyFactory 的`setFrozen(...)`方法，来控制代理工厂被配置之后，**是否还允许修改通知**；  
> 【**步骤 9**】调用 proxyFactory 的`getProxy(...)`方法获得**代理对象**。

### 3.3.1> buildAdvisors(...）获得所有增强器

在`buildAdvisors(...)`方法中，我们可以看到大致做了两个步骤：

> 【**步骤 1**】获得所有拦截器（普通的 + 指定的）；  
> 【**步骤 2**】通过这些拦截器生成 Advisor 增强集合，

`buildAdvisors(...)`方法中的源码及注释如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpgVbASQ0oX4UcibmIcUFHiaWL1YpliaH6qibzdDDxwOsLQ9YP0MtFvybZ5A/640?wx_fmt=png)

收集拦截器的代码并不复杂，那么下面我们再来看`wrap(...)`方法是如何通过拦截器生成 Advisor 增强的，源码及注释请见下图所示：

```
public Advisor wrap(Object adviceObject) throws UnknownAdviceTypeException {
    // 如果待封装的adviceObject本来就是Advisor类型，则直接返回即可
    if (adviceObject instanceof Advisor) return (Advisor) adviceObject;    
    // 如果既不是Advisor类型也不是Advice类型，则直接抛出异常，无法执行包装操作
    if (!(adviceObject instanceof Advice)) throw new UnknownAdviceTypeException(adviceObject);

    Advice advice = (Advice) adviceObject;
    // 如果adviceObject是MethodInterceptor类型，则包装成DefaultPointcutAdvisor实例对象
    if (advice instanceof MethodInterceptor) return new DefaultPointcutAdvisor(advice);
    // 遍历适配器列表adapters，如果也支持advice，则包装成DefaultPointcutAdvisor实例对象
    for (AdvisorAdapter adapter : this.adapters) 
        if (adapter.supportsAdvice(advice)) return new DefaultPointcutAdvisor(advice);

    throw new UnknownAdviceTypeException(advice);
}

```

### 3.3.2>proxyFactory.getProxy(...) 获得代理对象

通过上面的操作，我们已经获得了**增强 Advisor 列表**了，并且也做好了对`proxyFactory`的赋值准备操作，下面就该到了获得代理对象的步骤了，具体逻辑代码在`getProxy(...)`方法中，源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpZibwYJ4wP3NDeaZHbZ0b2J7jArIgribvUe3V7CpiamqPSAfQIxibISpYyg/640?wx_fmt=png)

#### a> createAopProxy() 创建 AOP 代理

创建 aop 代理的时候，首先激活所有的 Listener，然后再去创建 AOP 代理，这部分代码很少，很好理解，请见如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpM3IicsXW96Xum059H4GtMicueeAibyxzqMRyEcmW7ibgCIESNic4bswHmCQ/640?wx_fmt=png)

在上面代码中，我们需要再继续分析红框内的`createAopProxy(this)`方法，

```
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (!NativeDetector.inNativeImage() &&
            (config.isOptimize() || // 默认false
             config.isProxyTargetClass() || // 默认false
             hasNoUserSuppliedProxyInterfaces(config))) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) throw new AopConfigException(...);

        // 如果是接口或者是代理类，则使用JDK动态代理
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) 
            return new JdkDynamicAopProxy(config);

        // 否则，使用CGlib代理
        return new ObjenesisCglibAopProxy(config);
    }
    else return new JdkDynamicAopProxy(config); // 使用JDK动态代理
}

```

> **isOptimize()** ：用来控制通过 CGLIB 创建的代理是否使用激进的优化策略。除非完全了解 AOP 代理如何处理优化，否则不推荐用户使用这个设置。目前这个属性仅用于 CGLIB 代理，对于 JDK 动态代理（默认代理）无效。  
> **isProxyTargetClass()** ：这个属性为 true 时，目标类本身被代理而不是目标类的接口。如果这个属性值被设为 true, CGLIB 代理将被创建，设置方式为：`<aop:aspectj-autoproxy-proxy-target-class＝"true"/>` 。  
> **hasNoUserSuppliedProxyInterfaces(config)** ：是否存在代理接口。

通过 createAopProxy(config) 方法，根据不同情况，会返回不同代理对象，在下面内容中，我们会分别分析不同代理对象的代理流程：

> 如果采用 **JDK 动态代理**，则返回`JdkDynamicAopProxy`代理对象；  
> 如果采用 **CGlib 代理**，则返回`ObjenesisCglibAopProxy`代理对象；

#### b> JdkDynamicAopProxy.getProxy(classLoader) 获得代理对象

我们先来看一下 **JdkDynamicAopProxy** 类对`getProxy(...)`方法的实现，我们发现，里面只调用了 Proxy.newProxyInstance(...) 方法，源码如下所示：

```
public Object getProxy(@Nullable ClassLoader classLoader) {
    return Proxy.newProxyInstance(classLoader, this.proxiedInterfaces, this);
}

```

自己编写过 JDK 动态代理的朋友应该对`getProxy(...)`方法中的内容并不陌生，它的编写结构如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygprNx8M4AJ0qdeOvqyZsAXqHYGJYKWmcMAMlBiavqSYCeq2m22R6l4ic7A/640?wx_fmt=png)

> 概括来说就是 3 点：  
> 【**第 1 点**】要实现`InvocationHandler`接口；  
> 【**第 2 点**】重写`invoke`方法；  
> 【**第 3 点**】通过调用`Proxy.newProxyInstance(...)`获得代理对象；

那么我们再来看 JdkDynamicAopProxy 类，它也实现了`InvocationHandler`接口，即：满足了第 1 点；

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygp4oaPOInFQ1EruhrXAwSoAnb6nv0rt6oU8nRkyfG3ueVnhLIcopx4jA/640?wx_fmt=png)

那么就剩下第 2 点，即：重写`invoke`方法了，我们把目光注视到 JdkDynamicAopProxy 类的 invoke 方法，看看它是如何实现的：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9ovYMrvnH3Za65icuGKKygpLF80ORR2Pkbzo1mQGwj7oqSsIJ5kJkH5n2KFVKaxqib3RSKAVviadrYw/640?wx_fmt=png)

将拦截器封装到 ReflectiveMethodInvocation 类中，并通过调用`proceed`方法逐一调用拦截器，下面是 proceed 源码内容：

```
public Object proceed() throws Throwable {
    // 执行完所有增强后执行切点方法
    if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) 
        return invokeJoinpoint();

    // 获取下一个要执行的拦截器
    Object interceptorOrInterceptionAdvice = this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
    //  动态匹配
    if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) { 
        InterceptorAndDynamicMethodMatcher dm = (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
        Class<?> targetClass = (this.targetClass != null ? this.targetClass : this.method.getDeclaringClass());
        if (dm.methodMatcher.matches(this.method, targetClass, this.arguments)) 
            return dm.interceptor.invoke(this); // 调用拦截器
        else 
            return proceed(); // 不匹配不调用拦截器
    }
    // 普通拦截器，直接调用拦截器即可
    else return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this); // 将this作为参数传递以保证当前实例中调用链的执行
}

```

> 在`proceed`方法中，或许代码逻辑并没有我们想象得那么复杂，ReflectiveMethodlnvocation 中的主要职责是维护了**链接调用的计数器**，记录着当前调用链接的位置，以便链可以有序地进行下去，那么在这个方法中并没有我们之前设想的维护各种增强的顺序，而是将此工作委托给了各个增强器，使**各个增强器在内部进行逻辑实现**。

#### c> ObjenesisCglibAopProxy.getProxy(classLoader) 获得代理对象

上面介绍完 JDK 动态代理之后，我们下面来介绍 Cglib 动态代理是如果获得代理对象的。对于 Cglib 大家一定不会陌生，下面我们就手写一下通过 Cglib 获得代理对象的示例，请见如下代码所示：

```
public class CGlibTest {
    public static void main(String[] args) {
        // 创建增强器
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(CGlibTest.class);
        enhancer.setCallback(new MyMethodInterceptor());
        // 通过增强器，获得CGlib代理对象
        CGlibTest test = (CGlibTest) enhancer.create();
        test.play();
    }

    public void play() {
        System.out.println("play game!");
    }
}

/**
 * 创建GClib拦截器
 */
class MyMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("-----------------preExecutor-----------------");
        Object result =  methodProxy.invokeSuper(o, objects);
        System.out.println("-----------------afterExecutor-----------------");
        return result;
    }
}


```

重温了 cglib 动态代理之后，我们来看 Spring AOP 是如何通过它来获得代理的，此处我们来看一下 **ObjenesisCglibAopProxy** 类的`getProxy(...)`方法是如何实现的：

```
public Object getProxy(@Nullable ClassLoader classLoader) {
    try {
        Class<?> rootClass = this.advised.getTargetClass();
        Class<?> proxySuperClass = rootClass;
        // 如果类名中包含"$$"
        if (rootClass.getName().contains(ClassUtils.CGLIB_CLASS_SEPARATOR)) {
            proxySuperClass = rootClass.getSuperclass();
            Class<?>[] additionalInterfaces = rootClass.getInterfaces();
            for (Class<?> additionalInterface : additionalInterfaces) 
                this.advised.addInterface(additionalInterface);
        }
        // 针对类进行验证操作
        validateClassIfNecessary(proxySuperClass, classLoader);

        // 创建Cglib的Enhancer并对其进行配置操作
        Enhancer enhancer = createEnhancer();
        if (classLoader != null) {
            enhancer.setClassLoader(classLoader);
            if (classLoader instanceof SmartClassLoader && ((SmartClassLoader) classLoader).isClassReloadable(proxySuperClass)) 
                enhancer.setUseCache(false);
        }
        enhancer.setSuperclass(proxySuperClass);
        enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
        enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
        enhancer.setStrategy(new ClassLoaderAwareGeneratorStrategy(classLoader));

        /** 设置拦截器 */
        Callback[] callbacks = getCallbacks(rootClass);
        Class<?>[] types = new Class<?>[callbacks.length];
        for (int x = 0; x < types.length; x++) types[x] = callbacks[x].getClass();
        enhancer.setCallbackFilter(new ProxyCallbackFilter(this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
        enhancer.setCallbackTypes(types);

        // 生成代理类并创建代理实例对象
        return createProxyClassAndInstance(enhancer, callbacks);
    } 
    catch (CodeGenerationException | IllegalArgumentException ex) {...}
}

/** 创建代理对象 */
protected Object createProxyClassAndInstance(Enhancer enhancer, Callback[] callbacks) {
    enhancer.setInterceptDuringConstruction(false);
    enhancer.setCallbacks(callbacks);
    return (this.constructorArgs != null && this.constructorArgTypes != null ?
            enhancer.create(this.constructorArgTypes, this.constructorArgs) : // 采用有参构造函数创建实例对象
            enhancer.create()); // 采用无参构造函数创建实例对象
}

```

> 通过上面的代码，我们可以大致了解 getProxy 方法的处理逻辑分为 3 个步骤：  
> 【**步骤 1**】创建 Enhancer 实例对象，并对其进行初始化；  
> 【**步骤 2**】获得 Callback 拦截器，并赋值到 Enhancer 实例对象中；  
> 【**步骤 3**】通过 Enhancer 实例对象的 create(...) 方法来创建代理对象；

由于我们手工创建过 Cglib 动态代理了，所以对于上述步骤都会比较熟悉，但是对于第二步获得拦截器，我们还是比较陌生的，那么我们就来着重分析一下这个方法，其源码如下所示：

```
private Callback[] getCallbacks(Class<?> rootClass) throws Exception {
    // Parameters used for optimization choices...
    boolean exposeProxy = this.advised.isExposeProxy();
    boolean isFrozen = this.advised.isFrozen();
    boolean isStatic = this.advised.getTargetSource().isStatic();
    // 步骤1：创建aopInterceptor拦截器
    Callback aopInterceptor = new DynamicAdvisedInterceptor(this.advised);

    // 步骤2：创建targetInterceptor拦截器
    Callback targetInterceptor;
    if (exposeProxy) targetInterceptor = (isStatic ?
            new StaticUnadvisedExposedInterceptor(this.advised.getTargetSource().getTarget()) :
            new DynamicUnadvisedExposedInterceptor(this.advised.getTargetSource()));
    else targetInterceptor = (isStatic ?
            new StaticUnadvisedInterceptor(this.advised.getTargetSource().getTarget()) :
            new DynamicUnadvisedInterceptor(this.advised.getTargetSource()));

    // 步骤3：创建targetDispatcher调度器
    Callback targetDispatcher = (isStatic ?
            new StaticDispatcher(this.advised.getTargetSource().getTarget()) : new SerializableNoOp());

    // 步骤4：将不同的拦截器或调度器都保存到Callback数组中
    Callback[] mainCallbacks = new Callback[] {
            aopInterceptor,  // 用于一般的增强器
            targetInterceptor,  // 如果被优化了，调用target而不考虑调用增强
            new SerializableNoOp(),  // 映射到this的方法不能重写
            targetDispatcher,
            this.advisedDispatcher,
            new EqualsInterceptor(this.advised),
            new HashCodeInterceptor(this.advised)
    };

    // 步骤5：如果目标是静态的并且Advice链是冻结的，那么我们可以通过使用该方法的固定链直接将AOP调用发送到目标来进行一些优化。
    Callback[] callbacks;
    if (isStatic && isFrozen) {
        Method[] methods = rootClass.getMethods();
        Callback[] fixedCallbacks = new Callback[methods.length];
        this.fixedInterceptorMap = CollectionUtils.newHashMap(methods.length);
        for (int x = 0; x < methods.length; x++) {
            Method method = methods[x];
            List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, rootClass);
            fixedCallbacks[x] = new FixedChainStaticTargetInterceptor(
                    chain, this.advised.getTargetSource().getTarget(), this.advised.getTargetClass());
            this.fixedInterceptorMap.put(method, x);
        }
        // callbacks = mainCallbacks + fixedCallbacks;
        callbacks = new Callback[mainCallbacks.length + fixedCallbacks.length];
        System.arraycopy(mainCallbacks, 0, callbacks, 0, mainCallbacks.length);
        System.arraycopy(fixedCallbacks, 0, callbacks, mainCallbacks.length, fixedCallbacks.length);
        this.fixedInterceptorOffset = mainCallbacks.length;
    }
    else 
        callbacks = mainCallbacks; // callbacks = mainCallbacks;
    return callbacks;
}

```

从上面我们自定义演示 Cglib 例子中可以看到，通过`enhancer.setCallback(new MyMethodInterceptor())`这段代码，可以将我们自定义的拦截器注入到增强中，那么，在上面源码中，我们在`步骤1`中将 advised 保存到 **DynamicAdvisedInterceptor** 中，并在下面的步骤里，将其保存到`Callback数组`中，那么，当执行 Cglib 代理调用的时候，就会调用 **DynamicAdvisedInterceptor** 类中的`intercept(...)`方法了，代码如下所示：

```
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
    Object oldProxy = null;
    boolean setProxyContext = false;
    Object target = null;
    TargetSource targetSource = this.advised.getTargetSource();
    try {
        if (this.advised.exposeProxy) {
            oldProxy = AopContext.setCurrentProxy(proxy);
            setProxyContext = true;
        }
        target = targetSource.getTarget();
        Class<?> targetClass = (target != null ? target.getClass() : null);
        // 获取拦截器链
        List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
        Object retVal;
        // 没有拦截链，直接调用原方法即可
        if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
            Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
            retVal = methodProxy.invoke(target, argsToUse);
        }
        else // 调用拦截链
            retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
        retVal = processReturnType(proxy, target, method, retVal);
        return retVal;
    }
    finally {
        if (target != null && !targetSource.isStatic()) targetSource.releaseTarget(target);
        if (setProxyContext) AopContext.setCurrentProxy(oldProxy);
    }
}

```

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的 **点赞** & **分享** 。

更多技术干货，欢迎大家关注公众号 “**爪哇缪斯**” ~ \(^o^)/ ~ 「干货分享，每天更新」

往期推荐
====

[（五）Spring 源码解析：ApplicationContext 解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491687&idx=1&sn=f68c5b65f7adad93c96ea0d70d3590d9&chksm=e912a29ade652b8c5575ee13f129db0b329619c53c5bebbeea407c543cfc714e7438cbe5abac&scene=21#wechat_redirect)  

[（四）Spring 源码解析：bean 的加载流程](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491677&idx=1&sn=1c473369292838e30ca1c462c9b44734&chksm=e912a2a0de652bb6cb1650bfbe7cb6e6b33dc4fc5f79add14d6adedee718b6a8e39983290a66&scene=21#wechat_redirect)  

[（三）Spring 源码解析：自定义标签解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491585&idx=1&sn=8111bc892cafcf64af59b1a05e7e6acc&chksm=e912a2fcde652bea38e954cf2fc7a8d802ea1e332cf4c1eb538dd51d34631c9f1bf2bcfb72c6&scene=21#wechat_redirect)

[（二）Spring 源码解析：默认标签解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491496&idx=1&sn=6500da670c30134e9a0e63e1abddf6cb&chksm=e9115d55de66d44323bb98b214c896c0608db965ad20ec249962c16ecdffeb64a973d929555a&scene=21#wechat_redirect)

[（一）Spring 源码解析：容器的基本实现](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491156&idx=1&sn=b2ba03821b9be0fd8b7d6c86cb793256&chksm=e9115ca9de66d5bf094badcc0bf517e0f539693951403b0d68174c4066ab03e9c22c0c852eed&scene=21#wechat_redirect)  
[](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491585&idx=1&sn=8111bc892cafcf64af59b1a05e7e6acc&chksm=e912a2fcde652bea38e954cf2fc7a8d802ea1e332cf4c1eb538dd51d34631c9f1bf2bcfb72c6&scene=21#wechat_redirect)