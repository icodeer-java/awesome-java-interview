
一、概述
====

1.1> 整体概览
---------

在前面的内容中，我们针对`BeanFactory`进行了深度的分析。那么，下面我们将针对 **BeanFactory 的功能扩展类**`ApplicationContext`进行深度的分析。ApplicationConext 与 BeanFactory 的功能相似，都是用于向 IOC 中加载 Bean 的。由于 **ApplicationConext 的功能是大于 BeanFactory 的**，所以在日常使用中，建议直接使用 ApplicationConext 即可。下面是两个类使用示例：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOHz54GflRcpnVxtf3AqK9sGLaAdhD38rPeeWjInbr5WX78K3ak9f4ew/640?wx_fmt=png)

`ApplicationConext`是接口，所以要分析其加载 bean 的流程，就可以从`ClassPathXmlApplicationContext`入手，其构造函数如下所示：

```
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
    this(new String[] {configLocation}, true, null);
}

public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) {
    super(parent);
    setConfigLocations(configLocations); // 事情1：设置配置加载路径
    if (refresh) refresh(); // 事情2：所有功能的初始化操作
}

```

> 在上面的构造方法中，主要做了两个事情：  
> 【**事情 1**】设置`配置的加载路径`；  
> 【**事情 2**】执行 ApplicationConext 中，所有`功能的初始化`操作；  
> 下面文章的内容，我们就是会针对这两部分做详细的解析：

1.2> setConfigLocations(...) 设置配置加载路径
-------------------------------------

该方法逻辑不多，主要就是为应用上下文 ApplicationContext 设置配置路径（`config locations`），源码如下所示：

```
public void setConfigLocations(String... locations) { // 支持传入多个配置文件
    if (locations != null) {
        this.configLocations = new String[locations.length];
        for (int i = 0; i < locations.length; i++) 
            this.configLocations[i] = resolvePath(locations[i]).trim(); // 路径解析
    } else 
        this.configLocations = null;
}

// 如果路径中包含特殊符号（如：${var}）,那么在resolvePath方法中会搜寻匹配的系统变量并且进行替换操作
protected String resolvePath(String path) {
    // 调用AbstractPropertyResolver#resolveRequiredPlaceholders(path)方法
    return getEnvironment().resolveRequiredPlaceholders(path); 
}

```

1.3> refresh() 初始化
------------------

在`refresh()`方法中几乎包含了 ApplicationContext 中提供的全部功能，下面我们会针对这个方法进行详细的分析：

```
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        prepareRefresh(); /** 步骤1：为refresh操作做提前的准备工作 */
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory(); /** 步骤2：获得beanFactory实例对象 */
        prepareBeanFactory(beanFactory); /** 步骤3：准备用于此上下文的beanFactory */
        try {
            postProcessBeanFactory(beanFactory); // 允许在上下文子类中对BeanFactory进行后置处理（空方法，可由子类实现）
            invokeBeanFactoryPostProcessors(beanFactory); /** 步骤4：激活各种BeanFactory的后置处理器 */
            registerBeanPostProcessors(beanFactory); /** 步骤5：注册各种Bean的后置处理器，在getBean时才会被调用 */
            initMessageSource(); /** 步骤6：为上下文初始化消息源（即：国际化处理）*/
            initApplicationEventMulticaster(); /** 步骤7：为上下文初始化应用事件广播器 */
            onRefresh(); // 初始化特定上下文子类中的其他特殊bean（空方法，可由子类实现）
            registerListeners(); /** 步骤8：在所有注册的bean中查找listener bean，并注册到消息广播器中 */
            finishBeanFactoryInitialization(beanFactory); /** 步骤9：初始化剩下的单例（非惰性non-lazy-init）*/
            finishRefresh(); /** 步骤10：完成refresh，通知lifecycleProcessor刷新过程，同时发出ContextRefreshEvent通知 */
        }
        catch (BeansException ex) {
            destroyBeans(); // Destroy already created singletons to avoid dangling resources
            cancelRefresh(ex); // Reset 'active' flag
            throw ex;
        }
        finally {resetCommonCaches();}
    }
}

```

二、prepareRefresh() 准备工作
=======================

下面我们来分析一下`refresh()`方法中的`prepareRefresh()`代码段的处理逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOZmsiaFT2brrre12sTdOMs1MzKIEBn5WOGIEv6ccccgjwI4UL9rHMQgQ/640?wx_fmt=png)

针对`prepareRefresh()`方法来说，主要就包含如下注释的两个方法，看似没什么意义，但是如果我们**自定义了 ClassPathXmlApplicationContext 类**，那么可以通过重写`initPropertySources()`方法，来增加逻辑代码：

```
protected void prepareRefresh() {
    ...
    this.active.set(true); 
    initPropertySources(); // 空方法，可以由子类重写
    getEnvironment().validateRequiredProperties(); /** 验证需要的属性文件是否都已经放入环境中 */
    ...
}

```

```
public void validateRequiredProperties() throws MissingRequiredPropertiesException {
    this.propertyResolver.validateRequiredProperties();
}

```

```
public void validateRequiredProperties() {
    MissingRequiredPropertiesException ex = new MissingRequiredPropertiesException();
    for (String key : this.requiredProperties) 
        // 如果环境中没有配置所需属性，则将缺失的属性放到ex中，并抛出异常
        if (this.getProperty(key) == null) 
            ex.addMissingRequiredProperty(key); 
    if (!ex.getMissingRequiredProperties().isEmpty()) {throw ex;}
}

```

三、obtainFreshBeanFactory() 获得最新 BeanFactory
===========================================

下面我们来分析一下`refresh()`方法中的`obtainFreshBeanFactory()`代码段的处理逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOiaSoeSSBt1QlicCHibsJq7Cw4pv1YJF6PJGjSCpy6NRrpakxYic8w9h6eg/640?wx_fmt=png)

通过`obtainFreshBeanFactory()`这个方法，**ApplicationContext 就已经拥有了 BeanFactory 的全部功能**。而这个方法中也包含了前面我们介绍 BeanFactory 时候对于 xml 配置的加载过程。

```
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory(); /** 初始化BeanFactory */
    return getBeanFactory(); 
}

```

在`refreshBeanFactory()`方法中，在基本容器的基础上，增加了**是否允许覆盖** 和 **是否允许扩展**的设置，并提供了注解 **@Qualifier** 和 **@Autowired** 的支持。

```
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {destroyBeans();closeBeanFactory();}

    try {
        DefaultListableBeanFactory beanFactory = createBeanFactory(); // new一个beanFactory的实例对象
        beanFactory.setSerializationId(getId());
        customizeBeanFactory(beanFactory); // 定制化操作BeanFactory配置
        loadBeanDefinitions(beanFactory); /** 初始化XmlBeanDefinitionReader，并进行xml配置文件读取及解析 */
        this.beanFactory = beanFactory;
    }
    catch (IOException ex) {throw new ApplicationContextException(...);}
}

/** 该方法可以采用子类覆盖的方式改变里面的逻辑内容 */
protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
    if (this.allowBeanDefinitionOverriding != null) // 设置是否允许覆盖同名称且不同定义的beanDefinition实例
        beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
    if (this.allowCircularReferences != null) // 设置是否允许bean直接存在循环依赖
        beanFactory.setAllowCircularReferences(this.allowCircularReferences);
}

```

在`loadBeanDefinitions(beanFactory)`方法中，执行了初始化 **XmlBeanDefinitionReader** 操作，并且**进行 xml 配置文件读取及解析**

```
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    // 创建XmlBeanDefinitionReader，用于后续对配置的读取操作
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    initBeanDefinitionReader(beanDefinitionReader); // 对beanDefinitionReader进行设置，可以被子类覆盖
    loadBeanDefinitions(beanDefinitionReader); /** 加载xml配置信息 */
}

/** 对XmlBeanDefinitionReader进行设置（此方法可以由子类进行覆写） */
protected void initBeanDefinitionReader(XmlBeanDefinitionReader reader) {
    reader.setValidating(this.validating); // validating = true
}

```

当获得了 **XmlBeanDefinitionReader** 之后，我们就可以通过`loadBeanDefinitions(beanDefinitionReader)`方法对 xml 配置文件进行读取操作了。其中的`reader.loadBeanDefinitions(...)`方法我们在前面的 BeanFactory 配置文件读取操作中已经解析过了，此处就不再赘述。

```
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
    // 尝试获得xml配置文件的Resource实例集合，并进行配置加载
    Resource[] configResources = getConfigResources();  
    if (configResources != null) reader.loadBeanDefinitions(configResources);

    // 尝试获得xml配置文件路径集合，并进行配置加载
    String[] configLocations = getConfigLocations(); 
    if (configLocations != null) reader.loadBeanDefinitions(configLocations);
}

```

四、prepareBeanFactory(...) 准备用于此上下文的 beanFactory
===============================================

4.1> 整体概览
---------

下面我们来分析一下`refresh()`方法中的`prepareBeanFactory()`代码段的处理逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOXKeSXVic9eDExVYVV2SPayBlTXyUsq0Ib9wqMiagrW9r3tGOacjDHObA/640?wx_fmt=png)

当执行本方法之前，Spring 已经完成了对配置的解析操作，而从本方法开始，`ApplicationContext`在功能上的**扩展**也由此展开了。

```
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 将当前上下文的classLoader作为beanFactory的classLoader
    beanFactory.setBeanClassLoader(getClassLoader()); 

    // 是否支持SpEL表达式解析，即：可以使用#{bean.xxx}的形式来调用相关属性值
    if (!shouldIgnoreSpel) 
        beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));

    // 添加1个属性编辑器，它是对bean的属性等设置管理的工具
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // 添加1个bean的后置处理器
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this)); 

    // 设置7个需要忽略自动装配的接口
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationStartupAware.class);

    // 注册4个依赖
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // 添加1个bean的后置处理器
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    // 增加对AspectJ的支持
    if (!NativeDetector.inNativeImage() && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // 注册4个默认的系统环境bean
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) // environment
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) // systemProperties
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) // systemEnvironment
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    if (!beanFactory.containsLocalBean(APPLICATION_STARTUP_BEAN_NAME)) // applicationStartup
        beanFactory.registerSingleton(APPLICATION_STARTUP_BEAN_NAME, getApplicationStartup());
}

```

4.2> 添加对 SpEL 表达式的支持
--------------------

SpEL 全称是 “**Spring Expression Language**”，它能在`运行时构件复杂表达式`、`存取对象图属性`、`对象方法调用`等；它是单独模块，只依赖了`core`模块，所以可以单独使用。SpEL 使用 **#{...} **作为界定符，所有在**大括号中的字符**都会被认定为 SpEL，使用方式如下所示：

```
<bean id="user" value="com.muse.entity.User"/>
<bean ...>
    <property #{user}">
</bean>
    
相当于：
    
<bean id="user" value="com.muse.entity.User"/>
<bean ...>
    <property >
</bean>

```

在 **prepareBeanFactory(beanFactory)** 方法中，通过`beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(...))`注册 SpEL 语言解析器，就可以对`SpEL`进行解析了。

那么，在注册了解析器后，**Spring 又是在什么时候调用这个解析器进行解析操作的呢？**调用方式如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOnH3wmyPZBiaZgvxPDia5ibDIcsd3ut3hpK25eOt0Nl3Qbns1UA1BIVBCA/640?wx_fmt=png)

> 【**解释**】其实就是在 Spring 进行 bean 初始化的时候，有一个步骤是——**属性填充**（即：`populateBean()`），而在这一步中 Spring 会调用`AbstractAutowireCapableBeanFactory#applyPropertyValues(...)`方法来完成功能。而就在这个方法中，会创建`BeanDefinitionValueResolver`实例对象 **valueResolver** 来进行属性值的解析操作。同时，也是在这个步骤中，通过`AbstractBeanFactory#evaluateBeanDefinitionString(...)`方法去完成 **SpEL 的解析操作**。

相关源码如下所示：

```
protected Object evaluateBeanDefinitionString(String value, BeanDefinition beanDefinition) {
    if (this.beanExpressionResolver == null) return value;
    Scope scope = null;
    if (beanDefinition != null) {
        String scopeName = beanDefinition.getScope();
        if (scopeName != null) scope = getRegisteredScope(scopeName);
    }
    // 通过获得的beanExpressionResolver实例调用evaluate(...)方法执行解析操作
    return this.beanExpressionResolver.evaluate(value, new BeanExpressionContext(this, scope));
}

```

4.3> 添加属性编辑器
------------

下面我们继续来介绍一下`beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));`这行代码，那么通过调用的方法名称，我们可以猜到它的作用是给 BeanFactory 实例**添加属性编辑器**。那么，**什么是属性编辑器呢？** 在 Spring 加载 bean 的时候，可以把基本类型属性注入进来，但是对于类似 Date 这种复杂属性就无法被识别了。

### 4.3.1> 属性编辑器的应用

我们假设有一个`Schedule`类，其中包含了一个`Date`类型的属性，如果采用默认加载方式的话，是无法将 xml 中配置的 "2023-01-01" 值转换为 Date 类型的。那么，下面我们就来演示一下如何通过自定义属性编辑器来让 Spring 在加载 bean 的时候，**将 String 类型的值转换为 Date 类型的值**。如下所示：

```
@Data
public class Schedule {
    private String name;
    private Date date; // 添加Date类型的属性
}

```

```
<bean id="schedule" class="com.muse.springbootdemo.entity.propertyeditor.Schedule">
  <property />
  <property /> <!-- date是Date类型，需要将String转换为Date -->
</bean>

```

当我们运行的时候发现报错了，即：**无法将配置文件中的 String 类型转换为 Date 类型**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOgQUBLegBx5NsH7yicXLWSQbU66tIR8d1jjiaHtNGzjVKMia4pWZZicm1KQ/640?wx_fmt=png)

那么，我们添加用来`将String类型转换为Date类型`的属性编译器 **DatePropertyEditor**，并将其注入到 **CustomEditorConfigurer** 中即可。

```
public class DatePropertyEditor implements PropertyEditorRegistrar {
    @Override
    public void registerCustomEditors(PropertyEditorRegistry registry) {
        registry.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
    }
}

```

```
<!-- 注册属性编辑器 -->
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
    <property > <!-- 将自定义属性编辑器保存到propertyEditorRegistrars属性中 -->
        <list>
            <bean class="com.muse.springbootdemo.entity.propertyeditor.DatePropertyEditor"/>
        </list>
    </property>
</bean>

```

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOJ6qSrWxicTWWfSmgpaQ9PhsxjSFALyNrudv9sWic8rYwohGiaaD6ibHgJg/640?wx_fmt=png)

在调用`registerCustomEditor(...)`方法的时候，我们创建了 **CustomDateEditor** 实例对象，在此处我们只是来看一下它的类继承关系，下文还会涉及对它的介绍。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOWaMG3EtSdsNTwicCAekVvCbuMicOuUu7VpUDib8oObic8EHicFKH8FsDa5g/640?wx_fmt=png)

### 4.3.2> 源码解析

通过上面对于注册属性编辑器的配置，我们可以看到自定义的属性编辑器`DatePropertyEditor`被保存到了`CustomEditorConfigurer`的 **propertyEditorRegistrars 属性**中，那么由于`CustomEditorConfigurer`类实现了`BeanFactoryPostProcessor`接口，所以当 bean 初始化的时候，会调用它的`postProcessBeanFactory(...)`方法，那么在这个方法中，会将 **propertyEditorRegistrars 属性**中的所有属性编辑器添加到`beanFactory`中。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBONnpcDDyCDoyR6CoaAxKJkRPfT2pPXK347ogdMzr4chYtnFddmibKv1w/640?wx_fmt=png)

那么，我们在回头来看 **beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()))** 的这行代码，其实也是同样的调用方式：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOkO0CAPpxb6K06nDYA9w9vxbf6dnYnGWW1aQ8Po091haIjhuLThZXiaA/640?wx_fmt=png)

那么，我们分析到这一步之后，发现自定义属性编辑器都会保存到 **ConfigurableListableBeanFactory** 实例对象`beanFactory`的变量 **propertyEditorRegistrars** 中。那么问题来了——保存我们知道了，**那什么时候属性编辑器会被调用呢？** 其实就是在初始化 bean 的时候，在`initBeanWrapper(...)`方法中，会被调用。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOA7USrYCzFhnIwP9eUxCeicA1VsNzdtU0jmuHe7t5vSlg9YznxUw3v0Q/640?wx_fmt=png)

那么综上所述，我们来通过一张图，了解一下属性编辑器的注册和使用操作相关的类关系图：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOUm1agF2MXt0ibyD4GIzDmND3CTUk8D3NQdTMtj2cV6CamjfMGahz1Fg/640?wx_fmt=png)

在上面的例子中，我们自定义了一个属性编辑器`DatePropertyEditor`，其实 Spring 也内置了自己的属性编辑器 **ResourceEditorRegistrar**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBO3rdI9epvcHbSLkVaFNAhfJZJ77MicJxNNaxjrSNo11b6VUcpAePmS0g/640?wx_fmt=png)

在 **ResourceEditorRegistrar** 的`registerCustomEditors(...)`方法中，提供了 Spring 默认配置的所有属性编辑器集合。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBONO2eiaBD36hPUIAmrZEOVNuPoA3SZFic2B3vTyvC5qWNhdsToQWcAbdQ/640?wx_fmt=png)

在`doRegisterEditor(...)`中，我们发现该方法的内部实现与自定义属性编辑器实现方式基本一样，即：通过调用`registry.registerCustomEditor(requiredType, editor)`方法来注册属性编辑器。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBO50hnU6RA8ocfSiciaiaiaTgRVvldsXLXfr9WRgy0xMK3bcU3TyKibOqC9Rg/640?wx_fmt=png)

此处我们再提及一点，就是在 **PropertyEditorRegistrySupport** 类的`createDefaultEditors()`方法中，包含了 Spring 给我们提供的一系列默认编辑器。具体如下所示：

```
private void createDefaultEditors() {
    this.defaultEditors = new HashMap<>(64);
    
    // Simple editors, without parameterization capabilities.
    this.defaultEditors.put(Charset.class, new CharsetEditor());
    this.defaultEditors.put(Class.class, new ClassEditor());
    this.defaultEditors.put(Class[].class, new ClassArrayEditor());
    this.defaultEditors.put(Currency.class, new CurrencyEditor());
    this.defaultEditors.put(File.class, new FileEditor());
    this.defaultEditors.put(InputStream.class, new InputStreamEditor());
    if (!shouldIgnoreXml) 
        this.defaultEditors.put(InputSource.class, new InputSourceEditor());
    this.defaultEditors.put(Locale.class, new LocaleEditor());
    this.defaultEditors.put(Path.class, new PathEditor());
    this.defaultEditors.put(Pattern.class, new PatternEditor());
    this.defaultEditors.put(Properties.class, new PropertiesEditor());
    this.defaultEditors.put(Reader.class, new ReaderEditor());
    this.defaultEditors.put(Resource[].class, new ResourceArrayPropertyEditor());
    this.defaultEditors.put(TimeZone.class, new TimeZoneEditor());
    this.defaultEditors.put(URI.class, new URIEditor());
    this.defaultEditors.put(URL.class, new URLEditor());
    this.defaultEditors.put(UUID.class, new UUIDEditor());
    this.defaultEditors.put(ZoneId.class, new ZoneIdEditor());
    
    // Default instances of collection editors.
    this.defaultEditors.put(Collection.class, new CustomCollectionEditor(Collection.class));
    this.defaultEditors.put(Set.class, new CustomCollectionEditor(Set.class));
    this.defaultEditors.put(SortedSet.class, new CustomCollectionEditor(SortedSet.class));
    this.defaultEditors.put(List.class, new CustomCollectionEditor(List.class));
    this.defaultEditors.put(SortedMap.class, new CustomMapEditor(SortedMap.class));
    
    // Default editors for primitive arrays.
    this.defaultEditors.put(byte[].class, new ByteArrayPropertyEditor());
    this.defaultEditors.put(char[].class, new CharArrayPropertyEditor());
    
    // The JDK does not contain a default editor for char!
    this.defaultEditors.put(char.class, new CharacterEditor(false));
    this.defaultEditors.put(Character.class, new CharacterEditor(true));
    
    // Spring's CustomBooleanEditor accepts more flag values than the JDK's default editor.
    this.defaultEditors.put(boolean.class, new CustomBooleanEditor(false));
    this.defaultEditors.put(Boolean.class, new CustomBooleanEditor(true));
    
    // The JDK does not contain default editors for number wrapper types!
    // Override JDK primitive number editors with our own CustomNumberEditor.
    this.defaultEditors.put(byte.class, new CustomNumberEditor(Byte.class, false));
    this.defaultEditors.put(Byte.class, new CustomNumberEditor(Byte.class, true));
    this.defaultEditors.put(short.class, new CustomNumberEditor(Short.class, false));
    this.defaultEditors.put(Short.class, new CustomNumberEditor(Short.class, true));
    this.defaultEditors.put(int.class, new CustomNumberEditor(Integer.class, false));
    this.defaultEditors.put(Integer.class, new CustomNumberEditor(Integer.class, true));
    this.defaultEditors.put(long.class, new CustomNumberEditor(Long.class, false));
    this.defaultEditors.put(Long.class, new CustomNumberEditor(Long.class, true));
    this.defaultEditors.put(float.class, new CustomNumberEditor(Float.class, false));
    this.defaultEditors.put(Float.class, new CustomNumberEditor(Float.class, true));
    this.defaultEditors.put(double.class, new CustomNumberEditor(Double.class, false));
    this.defaultEditors.put(Double.class, new CustomNumberEditor(Double.class, true));
    this.defaultEditors.put(BigDecimal.class, new CustomNumberEditor(BigDecimal.class, true));
    this.defaultEditors.put(BigInteger.class, new CustomNumberEditor(BigInteger.class, true));
    
    // Only register config value editors if explicitly requested.
    if (this.configValueEditorsActive) {
        StringArrayPropertyEditor sae = new StringArrayPropertyEditor();
        this.defaultEditors.put(String[].class, sae);
        this.defaultEditors.put(short[].class, sae);
        this.defaultEditors.put(int[].class, sae);
        this.defaultEditors.put(long[].class, sae);
    }
}

```

4.4> 添加 ApplicationContextAwareProcessor 处理器
--------------------------------------------

下面我们继续来分析`prepareBeanFactory(...)`方法中的源码，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOr1ianFLq7wlpw7g7F46z0lAdQaKc73ey1gP55fBKDu4icfHJNK6ra1bg/640?wx_fmt=png)

> 【**解释**】在上面的源码红框中，只是将 ApplicationContextAwareProcessor 实例对象添加到了 beanFactory 的后置处理器集合中（即：`List<BeanPostProcessor> beanPostProcessors`）。

那既然 **ApplicationContextAwareProcessor 是后置处理器**，那它必然就已经实现了`BeanPostProcessor`接口。那么问题来了，**后置处理器会在什么时候被 Spring 调用呢？** 这个问题在前面章节的 **bean 实例化** 过程中其实已经介绍过了，我们再来回顾一下。请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOLTuKbjn9VGOzCX0kFh6j0qVqibHzLtIabMVicOJK4SwhLYbf3KJ688cw/640?wx_fmt=png)

> 【**解释**】在 bean 实例化的时候，也就是在 Spring 调用 **init-method** 方法的前后，会分别调用所有实现了 **BeanPostProcessor 接口**的实现类的`postProcessBeforeInitialization(...)`方法和`postProcessAfterInitialization(...)`方法。那么，下面我们就来看一下 ApplicationContextAwareProcessor 是如何自定义这两个方法的具体逻辑的。

**ApplicationContextAwareProcessor** 的`postProcessAfterInitialization(...)`方法没有做特殊逻辑处理，采用的就是它父接口 BeanPostProcessor 中默认的方法实现。源码如下所示：

```
default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    return bean;
}

```

**ApplicationContextAwareProcessor** 的`postProcessBeforeInitialization(...)`方法做了自定义的实现，源码如下所示：

```
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    // 只针对如下7种xxxAware进行特殊处理
    if (!(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
            bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
            bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware ||
            bean instanceof ApplicationStartupAware)) return bean;
    ... ...
    invokeAwareInterfaces(bean); // 为7种xxxAware设置对应的资源
    ... ...
    return bean;
}
private void invokeAwareInterfaces(Object bean) {
    if (bean instanceof EnvironmentAware) 
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    if (bean instanceof EmbeddedValueResolverAware) 
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    if (bean instanceof ResourceLoaderAware) 
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    if (bean instanceof ApplicationEventPublisherAware) 
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    if (bean instanceof MessageSourceAware) 
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    if (bean instanceof ApplicationStartupAware) 
        ((ApplicationStartupAware) bean).setApplicationStartup(this.applicationContext.getApplicationStartup());
    if (bean instanceof ApplicationContextAware) 
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
}

```

> 【**解释**】通过上面的源码，我们可以发现，在后置处理方法中，只是针对 7 种`Aware`实现设置了它们所需的资源。

4.5> 设置忽略依赖
-----------

我们继续来分析`prepareBeanFactory(...)`方法中如下红框的内容。在 4.3 中我们当通过调用 **ApplicationContextAwareProcessor** 类的`invokeAwareInterfaces()`方法之后，7 种 xxxAware 类型 bean 就都已经处理完毕了，那么下面的依赖注入操作就不需要再处理这些 xxxAware 类了。所以在如下红框种就对这 7 种 Aware 类执行了忽略依赖操作。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOdv0eVyjOCVTBp9tKerZds92F2f5WgZ9uoAic42uV8757W2lrekhUPkQ/640?wx_fmt=png)

4.6> 注册依赖
---------

下面我们再来继续分析`prepareBeanFactory(...)`方法中如下红框的内容。这部分内容是，当注册了依赖解析后，例如当注册了对 BeanFactory.class 的解析依赖后，当 bean 的属性注入的时候，一旦检测到属性为 BeanFactory 类型便会将 beanFactory 的实例注入进去。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOiaKNMW4icecYZKXb8D03VHbKicw0HDBwg1VfFiaSfvB6GQqeNhZ8L4mzgg/640?wx_fmt=png)

五、invokeBeanFactoryPostProcessors(...) 激活各种 BeanFactory 的后置处理器
==============================================================

下面我们来分析`refresh()`方法中的`invokeBeanFactoryPostProcessors(beanFactory)`这段代码的含义之前，通过方法名称，我们可以猜到这段方法是用来处理 BeanFactoryPostProcessor 的，那么 **它具体是做什么的呢？** 请见如下源码所示：

```
@FunctionalInterface
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}

```

> 【**解释**】`BeanFactoryPostProcessor`接口跟`BeanPostProcessor`类似，都可以对 bean 的定义（即：配置的元数据）进行处理。也就是说，**Spring 允许 BeanFactoryPostProcessor 在容器实际实例化任何其他的 bean 之前读取配置元数据，并进行修改。**如果配置多个`BeanFactoryPostProcessor`，可以通过实现`Ordered`接口来控制执行次序。

如果你想改变实际的 bean 实例，那么最好使用 BeanPostProcessor。因为 **BeanFactoryPostProcessor 的作用域范围是容器级别的**。它只和你所使用的容器有关。它不会对定义在另一个容器中的 bean 进行后置处理。

那么，我们先不着急解析`invokeBeanFactoryPostProcessors(beanFactory)`这段源码的具体逻辑，我们先插播一条 “娱乐新闻”，即：如何去使用`BeanFactoryPostProcessor`？

5.1> PropertySourcesPlaceholderConfigurer 的应用
---------------------------------------------

在某个路径下创建一个配置文件，我们以`prop/common.properties`为例，在该文件内配置相应的属性值。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOvdc6hich8devlTudD17XHFJEorq92yvwZFsLsBDsd5eiaUaMD1TTDxqQ/640?wx_fmt=png)

然后创建 message 的 Bean 配置信息，变量引用：`${message.msg}`。表明在其他的配置文件中指定了`message.msg`的值。那到底是哪个配置文件呢？我们创建 **PropertySourcesPlaceholderConfigurer** 的 Bean 配置信息。通过`locations`属性，来指定配置文件所在路径。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOuhwDcOylf5d6HQLJkcpRvFoo2HDJicibJ4MIQB2wstQbPHiaYW0SFjscw/640?wx_fmt=png)

从 IOC 中获取 message 的 bean 实例对象，然后输出 msg 的值。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOcNZbHuqAMZg2l5TBexXo0O51FutazYPyV6pjfzlPf6wTZ8hB48OmTQ/640?wx_fmt=png)

在上面的演示中，我们可以看到虽然在配置`PropertySourcesPlaceholderConfigurer`的 Bean 时指定了`common.properties`的文件路径，但是**这个文件是什么时候被加载解析的呢？** 答案是：**PropertySourcesPlaceholderConfigurer 其实就是一种 BeanFactoryPostProcessor**，那么当 Spring 加载任何实现了这个接口的 bean 的配置时，都会在 bean 工厂载入所有 bean 的配置之后执行`postProcessBeanFactory(beanFactory)`方法。类的继承关系图请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOoRLZTB97u956cgibk0rzCu39Po3BhiaFXMkAzz7JaZvBib8aBn8SAkB7A/640?wx_fmt=png)

本节开篇就说了，BeanFactoryPostProcessor 接口只有一个方法，即：`postProcessBeanFactory(beanFactory)`，那么下面我们来看一下 PropertySourcesPlaceholderConfigurer 类是怎么实现的这个方法：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOaYOJql85CO3aJ3QFJjtVDtk4lUZQKdASSzicsWaDFTfekPcEEMTiawvw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBO6RViacKG9w16rMQGRvJPOKiaM3WshW9RiaTdicz5RfStuBAKJ8jTgiadwkg/640?wx_fmt=png)

那么当配置文件信息被加载到了内存中后，就可以通过`message.msg`所配置的 “Hello World!” 值来替换`${message.msg}`了。此处替换流程暂且不深究。下面我们来看看如何自己实现一个 BeanFactoryPostProcessor。

5.2> 自定义 BeanFactoryPostProcessor 的应用
-------------------------------------

实现接口`BeanFactoryPostProcessor`，创建自定义处理类。在`postProcessBeanFactory(beanFactory)`方法中实现垃圾话过滤逻辑。需要注意的是，这个过滤垃圾话的**作用域是针对容器内所有的 bean 的**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBO6khKbMdl5CER6LTtqtksKjTn4mHBjAcj5bvQTKF0EicqOTdRh2F1p4g/640?wx_fmt=png)

配置 Bean，设置需要过滤的垃圾话列表。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBO468MtDNtsGJaczEr86cAEycBJGyDBccAB14xblSJIjCN7nlgBic7xQA/640?wx_fmt=png)

获取`allMessage`的 bean 实例，验证`msg`中的垃圾话是否被过滤掉了。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOuRibt45tt0jn3UAAdGg891poZymF8mNiaJMYYxUnFJr3X44AnreZCBTw/640?wx_fmt=png)

5.3> 激活 BeanFactoryPostProcessor 源码解析
-------------------------------------

### 5.3.1> 源码注释解析

通过上面的示例，我们基本了解到了`BeanFactoryPostProcessor`是如何使用的。那么，下面我们还是回到`refresh()`方法中，看一下`invokeBeanFactoryPostProcessors(beanFactory)`这段代码的具体实现逻辑：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOd9Dmsvd1pGR11b3NgmEHTD8g8SicnuRmnibu0iaCncGoadK6diax05lswQ/640?wx_fmt=png)

`invokeBeanFactoryPostProcessors(beanFactory)`方法的源码如下所示：

```
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 激活BeanFactoryPostProcessor操作
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
    
    if (!NativeDetector.inNativeImage() && beanFactory.getTempClassLoader() == null 
        && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }
}

```

激活的实际逻辑代码就在`invokeBeanFactoryPostProcessors(...)`方法中，里面逻辑很多，我们先看这个方法的源码和注释，然后我们在下面内容中，抽取重要的内容再详细的讨论。

```
public static void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, 
                                                 List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
    Set<String> processedBeans = new HashSet<>(); // 用于存储已经处理过的处理器名称
    
    /** 步骤1：对BeanDefinitionRegistry类型进行处理，包括【硬编码】+【配置方式】 */
    if (beanFactory instanceof BeanDefinitionRegistry) {
        BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory; 
        List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>(); // 通过【硬编码】注册的BeanFactoryPostProcessor
        List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>(); // 通过【硬编码】注册的BeanDefinitionRegistryPostProcessor

        /** 步骤1.1：遍历所有通过【硬编码】设置的BeanFactoryPostProcessor，按类型执行“分堆”操作，调用其postProcessBeanDefinitionRegistry(registry)方法 */
        for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
            if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) { 
                BeanDefinitionRegistryPostProcessor registryProcessor = (BeanDefinitionRegistryPostProcessor) postProcessor;
                registryProcessor.postProcessBeanDefinitionRegistry(registry); // BeanDefinitionRegistryPostProcessor接口中的方法，对BeanDefinitionRegistry执行后置处理
                registryProcessors.add(registryProcessor); // 通过【硬编码】注册的BeanDefinitionRegistryPostProcessor，会被放到registryProcessors中
            } else {
                regularPostProcessors.add(postProcessor); // 通过【硬编码】注册的其它BeanFactoryPostProcessor，会被放到regularPostProcessors中
            }
        }

        /** 步骤1.2：处理所有通过【配置方式】注册且实现了”PriorityOrdered接口“的BeanDefinitionRegistryPostProcessor实例集合，调用其postProcessBeanDefinitionRegistry(registry)方法 */
        List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();
        String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) { // 如果实现了PriorityOrdered接口，则进行排序
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
            }
        }
        sortPostProcessors(currentRegistryProcessors, beanFactory); // 排序
        registryProcessors.addAll(currentRegistryProcessors); // 合并【硬编码】和【配置方式（且实现了PriorityOrdered接口）】的BeanDefinitionRegistryPostProcessor集合
        // 调用所有currentRegistryProcessors实例的postProcessBeanDefinitionRegistry(registry)方法，实现对registry的后置处理
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
        currentRegistryProcessors.clear();

        /** 步骤1.3：处理所有通过【配置方式】注册且实现了”Ordered接口“的BeanDefinitionRegistryPostProcessor实例集合，调用其postProcessBeanDefinitionRegistry(registry)方法 */
        postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) { // 如果实现了Ordered接口，则进行排序
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
            }
        }
        sortPostProcessors(currentRegistryProcessors, beanFactory); // 排序
        registryProcessors.addAll(currentRegistryProcessors); // 合并【硬编码】和【配置方式（且实现了Ordered接口）】的BeanDefinitionRegistryPostProcessor集合
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
        currentRegistryProcessors.clear();

        /** 步骤1.4：处理所有通过【配置方式】注册且实现了BeanDefinitionRegistryPostProcessor实例集合（排除实现Ordered接口和PriorityOrdered接口），调用其postProcessBeanDefinitionRegistry(registry)方法 */
        boolean reiterate = true;
        while (reiterate) {
            reiterate = false;
            postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            for (String ppName : postProcessorNames) {
                if (!processedBeans.contains(ppName)) {
                    currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    processedBeans.add(ppName);
                    reiterate = true;
                }
            }
            sortPostProcessors(currentRegistryProcessors, beanFactory);
            registryProcessors.addAll(currentRegistryProcessors);
            invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
            currentRegistryProcessors.clear();
        }

        /** 步骤1.5：处理BeanDefinitionRegistryPostProcessor实例集合和BeanFactoryPostProcessor实例集合，调用其postProcessBeanFactory(beanFactory)方法 */
        invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
        invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
    }
        
    /** 步骤2：对ConfigurableListableBeanFactory而非BeanDefinitionRegistry类型进行处理【硬编码】+【配置方式】 ，调用其postProcessBeanFactory(beanFactory)方法*/
    else {
        invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
    }

    String[] postProcessorNames =
            beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);
    List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    List<String> orderedPostProcessorNames = new ArrayList<>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
        if (processedBeans.contains(ppName)) // skip - already processed in first phase above
        else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) // 如果实现了PriorityOrdered接口，则进行排序
            priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) // 如果实现了Ordered接口，则进行排序
            orderedPostProcessorNames.add(ppName);
        else 
            nonOrderedPostProcessorNames.add(ppName); // 否则，不排序了
    }

    /** 步骤3：对ConfigurableListableBeanFactory类型并且实现了”PriorityOrdered接口“的进行处理【配置方式】 ，调用其postProcessBeanFactory(beanFactory)方法*/
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);
    List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String postProcessorName : orderedPostProcessorNames) {
        orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }

    /** 步骤4：对ConfigurableListableBeanFactory类型并且实现了”Ordered接口“的进行处理【配置方式】 ，调用其postProcessBeanFactory(beanFactory)方法*/
    sortPostProcessors(orderedPostProcessors, beanFactory);
    invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);
    List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String postProcessorName : nonOrderedPostProcessorNames) {
        nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }

    /** 步骤5：对ConfigurableListableBeanFactory类型并且没有实现（PriorityOrdered接口或Ordered接口）进行处理【配置方式】 ，调用其postProcessBeanFactory(beanFactory)方法*/
    invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);
    beanFactory.clearMetadataCache();
}

```

通过上面的源码和源码注释，我们可以看到，其实就是针对两种情况进行分别的后置处理：

> 【**情况 1**】一个类实现了 **ConfigurableListableBeanFactory** 接口 + **BeanDefinitionRegistry** 接口  
> 【**情况 2**】一个类仅实现了 **ConfigurableListableBeanFactory** 接口

而对于后置处理器来说，有如下对应的处理关系：

> 【**针对 BeanDefinitionRegistry 接口**】我们采用 **BeanDefinitionRegistryPostProcessor** 后置处理器进行增强操作。  
> 【**针对 ConfigurableListableBeanFactory 接口**】我们采用 **BeanFactoryPostProcessor** 后置处理器进行增强操作。

而由于 BeanDefinitionRegistryPostProcessor 继承了 BeanFactoryPostProcessor，所以要区别处理：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOW3ojX714BWr1ickffHc5icwJQ3hKiaLgnLKfuPn0m7MMZ9o57yg3GwSuQ/640?wx_fmt=png)

### 5.3.2> 整体流程概述

**找出实现了 BeanDefinitionRegistry 接口 + ConfigurableListableBeanFactory 接口的 bean**

> 【**步骤 1**】调用 “`硬编码`” 设置的 **BeanDefinitionRegistryPostProcessor** 的`postProcessBeanDefinitionRegistry(registry)`方法进行增强；  
> 【**步骤 2**】调用 “`配置方式`” 设置的 **BeanDefinitionRegistryPostProcessor** 并实现了 **PriorityOrdered** 接口的`postProcessBeanDefinitionRegistry(registry)`方法进行增强；  
> 【**步骤 3**】调用 “`配置方式`” 设置的 **BeanDefinitionRegistryPostProcessor** 并实现了 **Ordered** 接口的`postProcessBeanDefinitionRegistry(registry)`方法进行增强；  
> 【**步骤 4**】调用 “`配置方式`” 设置的 **BeanDefinitionRegistryPostProcessor** 没有实现排序接口的`postProcessBeanDefinitionRegistry(registry)`方法进行增强；  
> 【**步骤 5**】调用 “`硬编码`”**BeanDefinitionRegistryPostProcessor** 的`postProcessBeanFactory(beanFactory)`方法进行增强；  
> 【**步骤 6**】调用 “`硬编码`”**BeanFactoryPostProcessor** 的`postProcessBeanFactory(beanFactory)`方法进行增强；

**找出仅仅实现了 ConfigurableListableBeanFactory 接口的 bean**

> 【**步骤 1**】调用 “`硬编码`” 设置 **BeanFactoryPostProcessor** 的`postProcessBeanFactory(beanFactory)`方法进行增强；  
> 【**步骤 2**】调用 “`配置方式`” 设置的 **BeanFactoryPostProcessor** 并实现了 **PriorityOrdered** 接口的`postProcessBeanFactory(beanFactory)`方法进行增强；  
> 【**步骤 3**】调用 “`配置方式`” 设置的 **BeanFactoryPostProcessor** 并实现了 **Ordered** 接口的`postProcessBeanFactory(beanFactory)`方法进行增强；  
> 【**步骤 4**】调用 “`配置方式`” 设置的 **BeanFactoryPostProcessor** 没有实现排序接口的`postProcessBeanFactory(beanFactory)`方法进行增强；

那么，**什么样的类即实现 BeanDefinitionRegistry 接口又实现 ConfigurableListableBeanFactory 接口呢？** 我们以 **DefaultListableBeanFactory** 为例，它就是既实现了`ConfigurableListableBeanFactory`接口，也实现了`BeanDefinitionRegistry`接口。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBO7mEG8H0uZTujUH93dmW80a5jD6R2zXiccHaGmXicLwaKmtKfbtAl8LQg/640?wx_fmt=png)

那么 **上面所说的【硬编码】，又是从哪里写入的呢？** 我们可以看到在 **AbstractApplicationContext** 类中有`addBeanFactoryPostProcessor(postProcessor)`方法，通过它，我们就可以采用硬编码的方式添加 PostProcessor 了。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOTSiaT69lLAxeIr2RJLKNOAEZaCDXOzBvaIictL46c93VVXAuTibvQYkkw/640?wx_fmt=png)

为了更便于大家对整个流程的理解，如下我提供了针对整体处理流程的图解。

### 5.3.3> 整体流程图解

针对实现了 **BeanDefinitionRegistry 接口 + ConfigurableListableBeanFactory 接口** 的 bean，调用 **BeanDefinitionRegistryPostProcessor** 类的`postProcessBeanDefinitionRegistry(registry)`方法的后置处理

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOQZYZNq34mQTO8Epo3u4YCwsAvaxtrtgUENckmL2AOCrcV2oCUZGVvQ/640?wx_fmt=png)

针对实现了 **BeanDefinitionRegistry 接口 + ConfigurableListableBeanFactory 接口** 的 bean，调用 **BeanFactoryPostProcessor** 类的`postProcessBeanFactory(beanFactory)`方法的后置处理

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOl68cCSFnl47lJwEHK7ZJTdekT2YHl1lRPd1ACD02fTbpGmKxY9ET0Q/640?wx_fmt=png)

针对仅仅实现了 **ConfigurableListableBeanFactory 接口**的 bean，调用 **BeanFactoryPostProcessor** 类的`postProcessBeanFactory(beanFactory)`方法的后置处理

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOybgx7fibm5ov9qsic6hpNoNNYw0pcOP60eEvmD7rn54JEcsQRbtbw89w/640?wx_fmt=png)

六、registerBeanPostProcessors(...) 注册各种 Bean 的后置处理器
==================================================

6.1> 使用示例
---------

创建一个`BeanPostProcessor`的实现类——**MuseBeanPostProcessor.java**

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOHPAKHRb0iccYKy4FBvs04uibKUCGw9tBPhLjG03O6F2kPPdFQxT7YO5g/640?wx_fmt=png)

在`oldbean.xml`配置文件中注册 **MuseBeanPostProcessor** 后置处理器。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOTWuSj1UPmK00CXTS7vtHVsPfviaaibdS7MfcmEeg4MEdX5FnnT39Khicg/640?wx_fmt=png)

启动项目，我们发现当使用 ApplicationContext 加载 bean 的时候，就可以调用我们创建的后置处理器；但是如果**使用 XmlBeanFactory 加载 bean，却无法调用我们创建的后置处理器**。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOg9Sc44icCBG8iaPMG30AMfPsMbMVS7uCmBp9TRd2ugypXFeR8Eyb8Oyg/640?wx_fmt=png)

其实这也印证了 **ApplicationContext 是 BeanFactory 的加强版本**，即：Plus 版本。在 Spring 中，其实绝大部分的功能都是通过后置处理器的方式进行扩展的，而 BeanFactory 其实只是初始化 IOC 容器了，并没有实现后置处理器的自动注册功能。

而 ApplicationContext 实现了自动添加后置处理器的能力，所以才会通过调用`applicationContext.getBean("allMessage")`的方式输出了 “-------allMessage---------”，而通过`beanFactory.getBean("allMessage")`的方式却没有输出。

6.2> 源码解析
---------

我们了解了怎样去使用`BeanPostProcessor`之后，下面我们来看一下涉及到注册`BeanPostProcessor`的逻辑代码，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOwy3zhVrFCq9BjVmUxZ1QP3yZLOpTibUmRUSDVgibf9BebqhruKSh9ic9A/640?wx_fmt=png)

如下是最核心的 BeanPostProcessor 方法，这里面的逻辑与 BeanFactoryPostProcessor 类似，请见如下源码和注释：

```
public static void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
    
    // BeanPostProcessorChecker是一个用于检查操作的后置处理器，即：当Spring配置中的后处理器还没有被注册就已经开始了bean的初始化是，那么Checker就会打印提示信息日志
    int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
    beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

    List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
    List<String> orderedPostProcessorNames = new ArrayList<>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) { // 存储实现了PriorityOrdered接口的后置处理器
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class); 
            priorityOrderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) 
                internalPostProcessors.add(pp); // 存储实现了PriorityOrdered接口并且是MergedBeanDefinitionPostProcessor类型后置处理器
        }
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) 
            orderedPostProcessorNames.add(ppName); // 存储实现了Ordered接口的后置处理器
        else 
            nonOrderedPostProcessorNames.add(ppName); // 存储未实现排序接口的后置处理器
    }
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);
    
    List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String ppName : orderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        orderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp); // 存储实现了Ordered接口并且是MergedBeanDefinitionPostProcessor类型后置处理器
        }
    }
    sortPostProcessors(orderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, orderedPostProcessors);

    List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String ppName : nonOrderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        nonOrderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp); // 存储没有实现排序接口并且是MergedBeanDefinitionPostProcessor类型后置处理器
        }
    }
    registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

    // 注册MergedBeanDefinitionPostProcessor类型后置处理器
    sortPostProcessors(internalPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, internalPostProcessors);
    
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
}

```

> 【**步骤 1**】针对实现了`PriorityOrdered`接口的 **BeanPostProcessor** 后置处理器执行排序和注册操作。  
> 【**步骤 2**】针对实现了`Ordered`接口的 **BeanPostProcessor** 后置处理器执行排序和注册操作。  
> 【**步骤 3**】针对没有实现排序接口的 **BeanPostProcessor** 后置处理器执行注册操作。  
> 【**步骤 4**】针对 **MergedBeanDefinitionPostProcessor** 类型的后置处理器执行注册操作。

在调用`registerBeanPostProcessors(...)`方法时，不用担心重复注册问题，因为都是**先执行 remove 再执行 add** 的，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBO4f7H2mrenFVCUryBtwFIBdkmWD7BP7s1AoBQjibW46frCfM0F7gLARg/640?wx_fmt=png)

七、initMessageSource() 为上下文初始化消息源
================================

Spring 国际化是根据客户端的系统语言类型返回对应的界面，这个便是所谓的 i18n 国际化（`internationalization`）。

**国际化信息**也称之为**本地化信息**，需要两个条件来最终决定：

> 【**条件 1**】语言类型（eg：中文）  
> 【**条件 2**】国家 / 地区的类型（ge：中国大陆、中国台湾、中国香港、新加坡、马来西亚……）

在 JDK 中，提供了 **java.util.Locale** 类用来支持国际化信息，它提供了如下使用方式：

> 【**方式 1**】指定语言和国家 / 地区——`new Locale("zh", "CN")` 或 `Locale.CHINA`  
> 【**方式 2**】指定语言——`new Locale("zh")` 或 `Locale.CHINESE`  
> 【**方式 3**】根据操作系统默认语言设置——`Locale.getDefault()`

**MessageSource** 是负责国际化信息的接口，它有如下几个重要实现类：

> **ResourceBundleMessageSource**：允许通过资源名加载国际化资源。  
> **ReloadableResourceBundleMessageSource**：与`ResourceBundleMessageSource`功能相似，额外提供定时刷新功能，不用重启系统，即可更新资源信息。  
> **StaticMessageSource**：主要用于程序测试，允许通过编程的方式提供国际化信息。  
> **DelegatingMessageSource**：为方便操作`MessageSource`的代理类。

7.1> 使用示例
---------

创建`i18n_zh.properties`文件，里面添加 “hello = 你好！世界！”，然后通过 **native2ascii** 将其转化为 utf8 的 ASCII 编码文件。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOwUfjh53HU80Kp7L7BtvpB5YSN5KY81XYFGpYEEcP2BJzpgJ9cVr9tA/640?wx_fmt=png)

创建`i18n_en.properties`文件，里面添加 “hello=Hello World！”，然后将`i18n_zh.properties`和`i18n_en.properties`都放入到 muse 文件夹下。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOwqB6P8f4musoRsnE3DHMw677v48VJiaOOJ9Yt7HiaUDiap78Md4fYoib6w/640?wx_fmt=png)

在配置文件`oldbean.xml`中配置国际化支持，其中 bean id 必须是 “**messageSource**”，否则就会报`NoSuchMessageException`异常

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOdHTZ4eGhbKwllbzVic5ibJwumXhXHj5eJjAsXNXA9ZokibyBLPDYSDmJg/640?wx_fmt=png)

执行测试代码：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOuVGXD8XjTs5YE1Sv2pbDLPqWIBNpQpwUwEhmmibic0zqFyibhIRRSrOFw/640?wx_fmt=png)

7.2> 源码解析
---------

了解了怎么使用国际化信息之后，我们现在来看一下**初始化国际化资源**的代码逻辑，如下图红框所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOlAmgCFFDJiatrXSHvic1d7huwibdvmnujia0D2u4BVTgKiceVe3R7ykaibpg/640?wx_fmt=png)

那么，在`initMessageSource()`方法中，通过判断是自定义国际化资源还是默认国际化资源，创建`MessageSource`实现类，然后赋值给 **AbstractApplicationContext** 的全局变量 **messageSource**。

```
protected void initMessageSource() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    // 如果需要用户自定义国际化资源，则需要创建一个bean的id必须是“messageSource”（硬编码）的bean
    if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) { 
        this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class); // 赋值给全局变量messageSource
        if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
            HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
            if (hms.getParentMessageSource() == null) {
                hms.setParentMessageSource(getInternalParentMessageSource());
            }
        }
    }
    // 否则，使用DelegatingMessageSource，一般后续通过调用getMessage方法返回国际化资源
    else {
        DelegatingMessageSource dms = new DelegatingMessageSource();
        dms.setParentMessageSource(getInternalParentMessageSource());
        this.messageSource = dms; // 赋值给全局变量messageSource
        beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
    }
}

protected MessageSource getInternalParentMessageSource() {
    return (getParent() instanceof AbstractApplicationContext ?
        ((AbstractApplicationContext) getParent()).messageSource : getParent());
}

public ApplicationContext getParent() {
    return this.parent;
}

```

在使用时，我们通过调用`applicationContext.getMessage("hello", null, Locale.US)`来获取国际化信息，那么我们看一下`getMessage(...)`方法的源码是怎么处理的：

```
public String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException {
    return getMessageSource().getMessage(code, args, locale);
}

private MessageSource getMessageSource() throws IllegalStateException {
    if (this.messageSource == null) 
        throw new IllegalStateException("MessageSource not initialized - call 'refresh' before accessing messages via the context: " + this);
    return this.messageSource; // 就是从全局messageSource中获取的国际化资源
}

```

八、initApplicationEventMulticaster() 为上下文初始化应用事件广播器
==================================================

8.1> 使用示例
---------

首先，继承`ApplicationEvent`，实现自定义 ApplicationEvent——**MuseApplicationEvent**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOW4Z7abwG5Mrhialvdia8ZtH5cRc6C5pkMbI8PXUL841buPr1rL7BC7KQ/640?wx_fmt=png)

其次，实现`ApplicationListener`接口，实现自定义 ApplicationListener——**MuseApplicationListener**：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOmPELZZ22ITu0uzstOQL1k4Teyu20IWJCcvxLfYzAAgiaJWXaV7QM7iag/640?wx_fmt=png)

将自定义应用监听器 **MuseApplicationListener** 注册倒 IOC 中：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOzRbBjK7hyGYddX865jH4JTcfK3XLQKuhI7XrLHaG7Wia9L9ibdmHBkCA/640?wx_fmt=png)

测试发布一个 **MuseApplicationEvent 类型**的事件，查看控制台输出内容：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBO5Dn6QqRVEfflBrU3o6jVdWfichia97S4Ue6VNJcQTRT1ajOajjTnicqUw/640?wx_fmt=png)

8.2> 源码解析
---------

好了，我们通过刚刚的示例，了解到了如何使用应用事件发起广播，那么下面我们就来看一下源码中如何进行初始化操作的，在下图`refresh()`方法的红框中：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOBDE2Egd7ibXxWNpiaE1VdyApevy65uhwQZJ4M56nApsUU2gzBHKuDZOA/640?wx_fmt=png)

我们再来看`initApplicationEventMulticaster()`方法的具体处理逻辑：

```
protected void initApplicationEventMulticaster() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    // 如果用户【自定义】了EventMulticaster，则向Spring中注册用户自定义的事件广播器
    if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
        this.applicationEventMulticaster = beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
    }
    // 如果用户【没有自定义】EventMulticaster，则向Spring中注册默认的SimpleApplicationEventMulticaster广播器
    else {
        this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
        beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
    }
}

```

> 上面的代码逻辑也是比较简单清晰的，就执行了两个判断逻辑：  
> 【**判断 1**】如果用户`自定义`了 EventMulticaster，则向 Spring 中**注册用户自定义的事件广播器**。  
> 【**判断 2**】如果用户`没有自定义`EventMulticaster，则向 Spring 中**注册默认的 SimpleApplicationEventMulticaster 广播器**。

那么，我们在看一下 **SimpleApplicationEventMulticaster** 中是怎么处理广播事件的：

```
public void multicastEvent(ApplicationEvent event) {
    multicastEvent(event, resolveDefaultEventType(event));
}
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
    Executor executor = getTaskExecutor();
    for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
        if (executor != null) 
            executor.execute(() -> invokeListener(listener, event)); // 采用并行方式执行广播操作
        else 
            invokeListener(listener, event); // 采用串行方式执行广播操作
    }
}

protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
    ErrorHandler errorHandler = getErrorHandler();
    if (errorHandler != null) 
        try {
            doInvokeListener(listener, event); // 触发广播监听行为
        }catch (Throwable err) {errorHandler.handleError(err);}
    else 
        doInvokeListener(listener, event); // 触发广播监听行为
}

private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
    try {
        listener.onApplicationEvent(event); // 发送监听事件
    }catch (ClassCastException ex) {...}
}

```

> 从上面的代码中，我们可以看出核心的逻辑就是两个步骤：  
> 【**步骤 1**】通过`getApplicationListeners(event, type)`方法获得所有的应用监听器列表。  
> 【**步骤 2**】遍历调用每一个 **ApplicationListener** 的`onApplicationEvent(event)`方法来发送监听事件。

需要注意的一点就是，当产生 Spring 监听事件的时候，**对于每一个监听器来说，其实他们都可以获得所有产生的监听事件**，那么具体处理哪一种类型的监听事件，可以在监听器中进行逻辑处理（如：上面 8.1 使用示例中的 MuseApplicationListener 实现方式）。当然，我们也可以通过使用**泛型**来约束处理的监听事件类型。如下图所示，那么`MuseApplicationListener`就只能处理`MuseApplicationEvent`类型的事件了。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOYxbzWFicDEvhGukMx0EIbY5a8LbnCvy6bxPibm1XLoB41ibkclgQeTH0w/640?wx_fmt=png)

九、registerListeners() 注册监听器
===========================

注册监听器的理解代码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOs9ZZqdborSxpYXolXw5zKG5v1iaibkCWFGhv2UE7FwPlVyc5hn2uMfPw/640?wx_fmt=png)

下面我们再来详细看一下`registerListeners()`方法的具体逻辑：

```
protected void registerListeners() {
    /** 添加以【硬编码】方式注册的监听器 */
    for (ApplicationListener<?> listener : getApplicationListeners()) 
        getApplicationEventMulticaster().addApplicationListener(listener);

    /** 添加以【配置文件】方式注册的监听器 */
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    for (String listenerBeanName : listenerBeanNames) 
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    
    /** 发送在多播设置之前发布的ApplicationEvent事件 */
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (!CollectionUtils.isEmpty(earlyEventsToProcess)) 
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) 
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
}

```

> 注册监听器的方法也比较简单清晰，它一共做到了如下 3 个步骤：  
> 【**步骤 1**】添加以`硬编码`方式注册的监听器  
> 【**步骤 2**】添加以`配置文件`方式注册的监听器  
> 【**步骤 3**】发送`在多播设置之前发布的ApplicationEvent`事件

十、finishBeanFactoryInitialization(...) 初始化非延迟加载单例
=================================================

10.1> 概述
--------

此处涉及到的就是我们完成了 BeanFactory 初始化工作之后的收尾工作了，包含：

> 【**步骤 1**】为上下文添加 ConversionService  
> 【**步骤 2**】为上下文添加 EmbeddedValueResolver（嵌入值解析器）  
> 【**步骤 3**】初始化 LoadTimeWeaverware 类型的 Bean  
> 【**步骤 4**】停止使用临时 ClassLoader 进行类型匹配  
> 【**步骤 5**】冻结配置  
> 【**步骤 6**】实例化所有剩余的（非惰性初始化）单例

10.2> 源码解析
----------

我们来看一下负责这部分逻辑的代码：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOjJLoJFapz4iaspGAChKUAiaaBVXhWr8hzTm9VCXmx6Du5oibvpSfKtl8Q/640?wx_fmt=png)

具体的逻辑处理就是在`finishBeanFactoryInitialization(beanFactory)`的方法中，如下所示：

```
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    /** 步骤1：为上下文初始化ConversionService */
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) && // 是否存在名称为“conversionService”并且类型是ConversionService的Bean
            beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) 
        beanFactory.setConversionService(beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));

    /** 步骤2：如果没有设置EmbeddedValueResolver（嵌入值解析器），则默认添加一个StringValueResolver */
    if (!beanFactory.hasEmbeddedValueResolver()) 
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    
    /** 步骤3：尽早初始化LoadTimeWeaverware类型的Bean，以便尽早注册其转换器 */
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) 
        getBean(weaverAwareName);
    
    beanFactory.setTempClassLoader(null); // 停止使用临时ClassLoader进行类型匹配
    beanFactory.freezeConfiguration(); // 冻结配置
    beanFactory.preInstantiateSingletons(); // 实例化所有剩余的（非惰性初始化）单例
}

```

冻结所有配置

```
public void freezeConfiguration() {
    this.configurationFrozen = true; // 开启配置冻结开关
    this.frozenBeanDefinitionNames = StringUtils.toStringArray(this.beanDefinitionNames);
}

```

初始化非延迟加载单例

```
public void preInstantiateSingletons() throws BeansException {
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
    /** 步骤1：初始化所有非惰性单例bean */
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            // 针对FactoryBean创建bean
            if (isFactoryBean(beanName)) {
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                if (bean instanceof FactoryBean) {
                    FactoryBean<?> factory = (FactoryBean<?>) bean;
                    boolean isEagerInit;
                    if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) 
                        isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit, getAccessControlContext());
                    else 
                        isEagerInit = (factory instanceof SmartFactoryBean && ((SmartFactoryBean<?>) factory).isEagerInit());
                    if (isEagerInit) // 对于配置了【非惰性加载】的bean执行创建操作
                        getBean(beanName);
                }
            } 
            // 针对非FactoryBean创建bean
            else getBean(beanName); 
        }
    }
    /** 步骤2：对于配置了SmartInitializingSingleton的Bean，触发初始化后回调（post-initialization callback）*/
    for (String beanName : beanNames) {
        Object singletonInstance = getSingleton(beanName);
        // 针对SmartInitializingSingleton类型触发初始化后回调，即：afterSingletonsInstantiated
        if (singletonInstance instanceof SmartInitializingSingleton) {
            SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
            if (System.getSecurityManager() != null) 
                AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                    smartSingleton.afterSingletonsInstantiated(); // 触发初始化后回调
                    return null;
                }, getAccessControlContext());
            else smartSingleton.afterSingletonsInstantiated(); // 触发初始化后回调
        }
    }
}

```

> 在上面代码中，总共执行了如下两个步骤：  
> 【**步骤 1**】步骤 1：初始化所有**非惰性**（isEagerInit）单例 bean。即：针对`FactoryBean`类型创建 bean 和 针对`其他类型`创建 bean；  
> 【**步骤 2**】对于配置了 **SmartInitializingSingleton** 的 Bean，触发初始化后回调，即：调用`afterSingletonsInstantiated()`方法；

十一、finishRefresh() 完成 refresh 操作
================================

最后这部分，我们来看一下`finishRefresh()`方法的逻辑代码：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicVBujflYWMBMhR3y2OuOBOzkbia9mQUiahevKHF6u7Ufmb6iccz0u5m1AF6oncQucccdNydaZ3TSyjA/640?wx_fmt=png)

在`finishRefresh()`方法中，主要就是针对 **Lifecycle 接口**进行处理，该接口主要包含如下两个重要方法：

> **start()** ：Spring 启动的时候，调用该方法开始生命周期；  
> **stop()** ：Spring 关闭的时候，调用该方法结束生命周期；

那么下面我们来看一下在`finishRefresh()`方法中，它是如何实现这个功能的：

```
protected void finishRefresh() {
    clearResourceCaches(); // 仅执行清除缓存操作：resourceCaches.clear()
    initLifecycleProcessor(); // 初始化生命周期处理器
    getLifecycleProcessor().onRefresh(); // 启动所有实现了Lifecycle接口的bean
    publishEvent(new ContextRefreshedEvent(this)); // 发布上下文已刷新的事件
    if (!NativeDetector.inNativeImage()) // 判断是不是在GraalVM虚拟机上运行
        LiveBeansView.registerApplicationContext(this);
}

```

初始化生命周期处理器——**LifecycleProcessor**

```
protected void initLifecycleProcessor() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    /** 步骤1：如果存在名字是“lifecycleProcessor”的生命周期处理器，则将其赋值给全局变量lifecycleProcessor */
    if (beanFactory.containsLocalBean(LIFECYCLE_PROCESSOR_BEAN_NAME)) {
        this.lifecycleProcessor = beanFactory.getBean(LIFECYCLE_PROCESSOR_BEAN_NAME, LifecycleProcessor.class);
        
    /** 步骤2：如果不存在，则创建默认的生命周期处理器——DefaultLifecycleProcessor，并赋值给全局变量lifecycleProcessor */
    else {
        DefaultLifecycleProcessor defaultProcessor = new DefaultLifecycleProcessor();
        defaultProcessor.setBeanFactory(beanFactory);
        this.lifecycleProcessor = defaultProcessor;
        beanFactory.registerSingleton(LIFECYCLE_PROCESSOR_BEAN_NAME, this.lifecycleProcessor);
    }
}

```

通过调用`onRefresh()`方法来启动所有实现了 **Lifecycle** 接口的 bean

```
public void onRefresh() {
    startBeans(true); // 启动所有实现了Lifecycle接口的bean
    this.running = true;
}

private void startBeans(boolean autoStartupOnly) {
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans(); // 获得所有Lifecycle集合
    Map<Integer, LifecycleGroup> phases = new TreeMap<>();

    /** 步骤1：如果autoStartupOnly是false，或者是SmartLifecycle类型的bean，并且其内部方法isAutoStartup()返回了true，则会保存到phase中 */
    lifecycleBeans.forEach((beanName, bean) -> {
        if (!autoStartupOnly || (bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())) {
            int phase = getPhase(bean);
            phases.computeIfAbsent(
                    phase,
                    p -> new LifecycleGroup(phase, this.timeoutPerShutdownPhase, lifecycleBeans, autoStartupOnly)
            ).add(beanName, bean);
        }
    });

    /** 步骤2：如果phase中存在待启动的LifecycleGroup，则调用每一个start()方法 */
    if (!phases.isEmpty()) 
        phases.values().forEach(LifecycleGroup::start); // 调用LifecycleGroup的start()方法
}

```

最后我们发现，会调用到 **DefaultLifecycleProcessor** 的内部类 **LifecycleGroup** 的`start()`方法；然后在`doStart()`方法中最终调用了每一个实现了 **Lifecycle** 接口的子类的`start()`方法。

```
public void start() {
    if (this.members.isEmpty()) return;
    Collections.sort(this.members);
    for (LifecycleGroupMember member : this.members) 
        doStart(this.lifecycleBeans, member.name, this.autoStartupOnly); // 在Spring中，真正执行某个逻辑都是以do开头的
}

private void doStart(Map<String, ? extends Lifecycle> lifecycleBeans, String beanName, boolean autoStartupOnly) {
    Lifecycle bean = lifecycleBeans.remove(beanName);
    if (bean != null && bean != this) {
        String[] dependenciesForBean = getBeanFactory().getDependenciesForBean(beanName);
        for (String dependency : dependenciesForBean) 
            doStart(lifecycleBeans, dependency, autoStartupOnly); // 如果有嵌套，则通过递归处理
        
        if (!bean.isRunning() && (!autoStartupOnly || !(bean instanceof SmartLifecycle) || ((SmartLifecycle) bean).isAutoStartup())) {
            try {
                bean.start(); // 调用Lifecycle实现类的start()方法触发生命周期开始操作
            } catch (Throwable ex) {...}
        }
    }
}

```

今天的文章内容就这些了：

> 写作不易，笔者几个小时甚至数天完成的一篇文章，只愿换来您几秒钟的 **点赞** & **分享** 。

更多技术干货，欢迎大家关注公众号 “**爪哇缪斯**” ~ \(^o^)/ ~ 「干货分享，每天更新」

往期推荐
----

[（一）Spring 源码解析：容器的基本实现](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491156&idx=1&sn=b2ba03821b9be0fd8b7d6c86cb793256&chksm=e9115ca9de66d5bf094badcc0bf517e0f539693951403b0d68174c4066ab03e9c22c0c852eed&scene=21#wechat_redirect)  

[（二）Spring 源码解析：默认标签解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491496&idx=1&sn=6500da670c30134e9a0e63e1abddf6cb&chksm=e9115d55de66d44323bb98b214c896c0608db965ad20ec249962c16ecdffeb64a973d929555a&scene=21#wechat_redirect)  

[（三）Spring 源码解析：自定义标签解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491585&idx=1&sn=8111bc892cafcf64af59b1a05e7e6acc&chksm=e912a2fcde652bea38e954cf2fc7a8d802ea1e332cf4c1eb538dd51d34631c9f1bf2bcfb72c6&scene=21#wechat_redirect)  

[（四）Spring 源码解析：bean 的加载流程](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491677&idx=1&sn=1c473369292838e30ca1c462c9b44734&chksm=e912a2a0de652bb6cb1650bfbe7cb6e6b33dc4fc5f79add14d6adedee718b6a8e39983290a66&scene=21#wechat_redirect)  

[SpringBoot2.x——Part1](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486233&idx=1&sn=fda7d410bedf6b56457757225a4d6bfa&chksm=e91149e4de66c0f2acd4a95cba907f43dbbd0700c8f70ccd9fc0cc7f57048683732cf79fc77b&scene=21#wechat_redirect)  

[SpringBoot2.x——SpringBoot Web 源码解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486972&idx=1&sn=74556f31641a1bd3c29a2663bc533208&chksm=e9114f01de66c6177f47b26297b0836b5df0e92feea7a7420f611b3d3a2f8926eb952d20b117&scene=21#wechat_redirect)