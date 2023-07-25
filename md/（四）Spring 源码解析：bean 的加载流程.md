> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247491677&idx=1&sn=1c473369292838e30ca1c462c9b44734&chksm=e912a2a0de652bb6cb1650bfbe7cb6e6b33dc4fc5f79add14d6adedee718b6a8e39983290a66&scene=178&cur_album_id=2208187391738249217#rd)

一、概述
====

在前几讲中，我们着重的分析了 Spring 对 xml 配置文件的**解析**和**注册**过程。那么，本节内容，将会试图分析一下 **bean 的加载过程**。具体代码，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3b9T0cpjQicGufSltrYMUs6xDDDkRAKzDD9bpmib15ceQlt2Sib0AwyGpA/640?wx_fmt=png)

1.1> doGetBean(...)
-------------------

针对 bean 的创建和加载，我们可以看出来逻辑都是在`doGetBean(...)`这个方法中的，所以，如下就是针对于这个方法的整体源码注释：

```
@SuppressWarnings("unchecked")
protected <T> T doGetBean(String name, Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) {
    String beanName = transformedBeanName(name); // 提取真正的beanName（去除‘&’或者将别名name转化为beanName)

    /** 步骤1：尝试根据beanName，从缓存中获得单例对象 */
    Object beanInstance, sharedInstance = getSingleton(beanName); 
    if (sharedInstance != null && args == null) 
        beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null); // 从bean中获得真正的实例对象
        
    /** 步骤2：缓存中不存在实例，则采取自主创建实例对象 */
    else {
        // 如果【原型模式】出现循环依赖，则无法处理，直接抛出异常
        if (isPrototypeCurrentlyInCreation(beanName)) throw new BeanCurrentlyInCreationException(beanName); 
        
        /** 步骤3：如果存在parentBeanFactory，并且配置中也没有beanName的配置信息，则尝试从parentBeanFactory中获取实例 */
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            String nameToLookup = originalBeanName(name);
            if (parentBeanFactory instanceof AbstractBeanFactory) 
                return ((AbstractBeanFactory) parentBeanFactory).doGetBean(nameToLookup, requiredType, args, typeCheckOnly);
            else if (args != null) 
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            else if (requiredType != null) 
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            else 
                return (T) parentBeanFactory.getBean(nameToLookup);  
        }

        if (!typeCheckOnly) markBeanAsCreated(beanName); // 如果不执行类型检查，则将beanName保存到alreadyCreated中
        StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate").tag("beanName", name);
        try {
            if (requiredType != null) beanCreation.tag("beanType", requiredType::toString);
            
            /** 步骤4：将GenericBeanDefinition转换为RootBeanDefinition，如果是子Bean，则与父类的相关属性进行合并 */
            RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);

            /** 步骤5：如果存在依赖，那么需要递归每一个依赖的bean并对其进行实例化创建 */
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                for (String dep : dependsOn) {
                    // 如果发生了循环依赖，则直接抛出异常
                    if (isDependent(beanName, dep)) throw new BeanCreationException(...);
                    registerDependentBean(dep, beanName); // 缓存依赖调用
                    try {
                        getBean(dep); // 创建每一个依赖（dep）的实例Bean
                    } catch (NoSuchBeanDefinitionException ex) {throw new BeanCreationException(...);}
                }
            }

            /** 步骤6：创建单例对象 */
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args); // 创建Bean实例对象
                    } catch (BeansException ex) {destroySingleton(beanName); throw ex;}
                });
                beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd); // 获得真正的bean
            }
            /** 步骤7：创建原型对象 */
            else if (mbd.isPrototype()) {
                Object prototypeInstance = null;
                try {
                    beforePrototypeCreation(beanName);
                    prototypeInstance = createBean(beanName, mbd, args); // 创建Bean实例对象
                } finally {
                    afterPrototypeCreation(beanName);
                }
                beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd); //获得真正的bean
            }
            /** 步骤8：创建指定scope类型的对象 */
            else {
                String scopeName = mbd.getScope();
                if (!StringUtils.hasLength(scopeName)) throw new IllegalStateException(...);
                
                Scope scope = this.scopes.get(scopeName);
                if (scope == null) throw new IllegalStateException(...);
                
                try {
                    Object scopedInstance = scope.get(beanName, () -> {
                        beforePrototypeCreation(beanName);
                        try {
                            return createBean(beanName, mbd, args); // 创建Bean实例对象
                        } finally {
                            afterPrototypeCreation(beanName);
                        }
                    });
                    beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);//获得真正的bean
                }
                catch (IllegalStateException ex) {throw new ScopeNotActiveException(...);}
            }
        }
        catch (BeansException ex) {
            beanCreation.tag("exception", ex.getClass().toString());
            beanCreation.tag("message", String.valueOf(ex.getMessage()));
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
        finally {
            beanCreation.end();
        }
    }

    /** 步骤9：检查需要的类型是否符合bean的实际类型，如果不同，则对其进行类型转换 */
    return adaptBeanInstance(name, beanInstance, requiredType); 
}

```

1.2> doGetBean(...)
-------------------

通过上面针对 doGetBean(...) 方法的源码注释，我们可以将其主要的流程总结一下：

**1：对 beanName 进行解析和转换——transformedBeanName(name)**

> 【**第 1 步**】去除 FactoryBean 的修饰符 “`&`”，因为如果 beanName 是以 “&” 开头的，则表明是 **FactoryBean**。所以需要去掉 “&” 前缀。  
> 【**第 2 步**】如果 beanName 传入的是 alias 值，则通过`aliasMap`获取真正的 beanName。

**2：尝试从缓存中获取单例实例——getSingleton(beanName)**

> 因为单例在 Spring 的同一个容器内只会被创建一次，后续再获取 bean，就直接从单例缓存 **singletonObjects** 中获取了。所以，首先会尝试从缓存中加载 bean，如果加载不到，再尝试从 **singletonFactories** 中加载。

> 因为在创建单例 bean 的时候会存在依赖注入的情况，而在创建以来的时候，为了避免循环依赖，所以 **Spring 不等 bean 创建完成就会将创建 bean 的**`ObjectFactory`**提早曝光加入到缓存中**，一旦另外的 bean 创建时候需要依赖这个 bean 的时候，则直接使用`ObjectFactory#getObject()`方法来获得单例实例。

> 具体逻辑如下所示：  
> 【**第 1 步**】尝试从 **singletonObjects** 中获得单例；  
> 【**第 2 步**】如果当前`beanName`所对应的实例正处于创建中，则尝试从 **earlySingletonObjects** 中获得单例；  
> 【**第 3 步**】尝试从 **singletonFactories** 中获得`ObjectFactory`对象，然后通过调用`getObject()`方法获得单例；

**3：bean 的实例化——getObjectForBeanInstance(...)**

> 其实我们从缓存中获得的是 bean 的原始状态，并不一定是我们最终想要的 bean。比如：我们需要对工厂 bean 进行处理，那么这里得到的其实是工厂 bean 的初始状态，而我们真正需要的是工厂 bean 中定义的`factory-method`方法中返回的 bean，那么 **getObjectForBeanInstance** 就可以完成这样的工作。

**4：原型模式的依赖检查——isPrototypeCurrentlyInCreation(beanName)**

> 只有**单例**才可以解决循环依赖，而原型模式如果发生了循环依赖，则直接抛异常。

**5：parentBeanFactory 相关逻辑处理——getParentBeanFactory()**

> 如果**存在 parentBeanFactory**，并且当前所加载的 **XML 配置信息中不包含 beanName**，那么我们就只能通过`parentBeanFactory#getBean()`方法来获得 beanName 对应的实例对象。

**6：将 GenericBeanDefinition 转换为 RootBeanDefinition——getMergedLocalBeanDefinition(beanName)**

> 因为从 XML 配置文件中读取到的 bean 信息是存储在 **GenericBeanDefinition** 中的，但是后续的所有 bean 处理都是针对 **RootBeanDefinition** 的，所以这里需要进行一下类型转换，在转换的同时，如果父类 bean 不为空的话，那么会合并父类的属性。

**7：针对所有依赖的 bean 执行初始化操作——mbd.getDependsOn()**

> 在 Spring 的加载顺序中，初始化一个 bean 的时候，首先**优先初始化这个 bean 所对应的所有依赖**。

**8：针对不同的 scope 进行 bean 的创建——createBean(beanName, mbd, args)**

> 此处会针对**单例**（Singleton）、**原型**（Prototype）和**其他 scope** 进行不同的初始化策略。但是最终都是会调用`createBean方法`来创建 bean。

**9：类型转换——adaptBeanInstance(name, beanInstance, requiredType)**

> 只有**当 requiredType 不为 null** 的时候，才会执行类型转换。而我们调用的`getBean(String name)`方法中所调用的`doGetBean(name, null, null, false)`方法的第 2 个参数，就是`requiredType`，而这里硬编码传入的就是 null，所以不会执行类型转换操作。  
> 但是**如果 requiredType 传入了一个类型，与 bean 的类型不同，则要执行类型转换操作**。在 Spring 中提供了各种各样的转换器，用户也可以自己扩展转换器来满足需求。

二、FactoryBean 的用法
=================

如果在某些情况下，**实例化 bean 的过程比较复杂**，如果在中进行配置的话，需要提供大量的配置信息，这种情况下的配置就失去了灵活性。所以，此时我们可以采取编码的方式来实例化这个 bean，即：通过实现`FactoryBean接口`，在`getObject()`方法中去实现 bean 的创建过程。那么**当 Spring 发现配置文件中的 class 属性配置的实现类是 FactoryBean 的子类时**，就会通过调用`FactoryBean#getObject()`方法返回 bean 的实例对象。如下是演示例子：

```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Car {
    private int maxSpeed;
    private String brand;
    private double price;
}

```

```
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    @Nullable
    T getObject() throws Exception;

    @Nullable
    Class< ？> getObjectType();

    default boolean isSingleton() {
        return true;
    }
}

```

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3XiaE9DI3y278ku3s5vfaW3bfaV7ia6EBbqW5ce0vRwHZ1wpmN2LicwuNw/640?wx_fmt=png)

如上面例子所示，当我们需要获取 Car 实例对象时，通过调用 **getBean("car")** 即可；那么，如果我们就是想要获得 CarFactoryBean 的实例对象，则可以通过调用 **getBean("&car")** 即可。

三、getSingleton(beanName)
========================

由于单例在 Spring 容器中只会被创建一次，即：创建出来的单例实例对象就会被缓存到 **singletonObjects** 中。所以，当要获得某个 beanName 的实例对象时，会首先尝试从`singletonObjects`中加载，如果加载不到，则再尝试从`singletonFactories`中加载。

因为在创建单例 bean 的时候可能会存在依赖注入的情况，所以为了**避免循环依赖**，Spring 创建 bean 的原则是不等 bean 创建完成就会将创建 bean 的`ObjectFactory`提早曝光加入到缓存 **singletonFactories** 中，一旦下一个 bean 创建时需要依赖上一个 bean，则直接使用`ObjectFactory`。具体代码逻辑，请见下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F32lGELcsnjjaksr5HotgEj13sgoIr4tXUPuFJa0YNTxajVFCh5AFuVA/640?wx_fmt=png)

> 【**singletonObjects**】用于保存 **beanName** 和 **bean 实例**之间的关系。  
> 【**singletonFactories**】用于保存 **beanName** 和**创建 bean 的工厂**之间的关系。  
> 【**earlySingletonObjects**】用于保存 **beanName** 和 **bean 实例**之间的关系。与 singletonObjects 的不同之处在于，当一个单例 bean 被放到这里面后，那么当 bean 还在创建过程中，就可以通过`getBean方法`获取到了，其目的是用来检测循环引用。  
> 【**registeredSingletons**】用来保存当前所有**已注册的 bean**。

四、getObjectForBeanInstance(...)
===============================

当我们得到 bean 的实例之后，要做的第一步就是调用`getObjectForBeanInstance(...)`方法来检测正确性，即：**检测当前 bean 是否是 FactoryBean 类型**，如果是，那么需要调用它的`getObject()`方法作为返回值。源码注释如下所示：

```
Object getObjectForBeanInstance(Object beanInstance, String name, String beanName, RootBeanDefinition mbd) {
    /** 步骤1：如果name是以“&”开头的，则表示就是要返回FactoryBean的实例对象，不需要处理，直接返回即可 */
    if (BeanFactoryUtils.isFactoryDereference(name)) {
        if (beanInstance instanceof NullBean) return beanInstance;
        if (!(beanInstance instanceof FactoryBean)) throw new BeanIsNotAFactoryException(...);
        if (mbd != null) mbd.isFactoryBean = true;
        return beanInstance;
    }

    /** 步骤2：如果beanInstance不是FactoryBean类型的实例对象，则不需要处理，直接返回即可 */
    if (!(beanInstance instanceof FactoryBean)) return beanInstance;

    /** 步骤3：beanInstance是FactoryBean类型的实例对象，则调用getObject()方法获的真实的bean对象 */
    Object object = null;
    if (mbd != null) mbd.isFactoryBean = true;
    else object = getCachedObjectForFactoryBean(beanName); // 首先尝试从缓存中获得bean
    if (object == null) {
        FactoryBean< ？> factory = (FactoryBean< ？>) beanInstance;
        // 将存储XML配置信息的GenericBeanDefinition转换为RootBeanDefinition，如果指定beanName是子bean，则合并父类属性
        if (mbd == null && containsBeanDefinition(beanName)) mbd = getMergedLocalBeanDefinition(beanName);
        boolean synthetic = (mbd != null && mbd.isSynthetic()); // true：用户自定义的 false：应用程序定义的
        object = getObjectFromFactoryBean(factory, beanName, !synthetic); // 调用getObject()方法获的真实的bean对象
    }
    return object;
}

```

上面代码比较简单，大多是一些辅助代码以及一些功能性的判断，而真正的核心代码是 **getObjectFromFactoryBean(factory, beanName, !synthetic)** ，下面我们来着重分析一下这个方法，源码注释如下所示：

```
protected Object getObjectFromFactoryBean(FactoryBean< ？> factory, String beanName, boolean shouldPostProcess) {
  /**
   * 步骤1：如果factory是单例的，并且beanName对应的bean已经被创建了;
   * 如果没有创建缓存，则向缓存factoryBeanObjectCache中添加beanName与Bean实例对象的对应关系;
   */
  if (factory.isSingleton() && containsSingleton(beanName)) {
      synchronized (getSingletonMutex()) {
          Object object = this.factoryBeanObjectCache.get(beanName); // 尝试从缓存中获得Bean实例对象
          if (object == null) {
              // 创建bean的实例对象。即：调用FactoryBean#getObject()获得
              object = doGetObjectFromFactoryBean(factory, beanName); 
              
              Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
              if (alreadyThere != null) object = alreadyThere;
              else {
                  if (shouldPostProcess) { // 是否执行后置处理（xxxPostProcess）
                      if (isSingletonCurrentlyInCreation(beanName)) return object; // 如果bean正在创建，则直接返回
                      beforeSingletonCreation(beanName); // 将beanName加入到缓存singletonsCurrentlyInCreation中
                      try {
                          // bean在初始化后会调用所有注册的BeanPostProcessor类的postProcessAfterInitialization方法
                          object = postProcessObjectFromFactoryBean(object, beanName); 
                      } catch (Throwable ex) {
                          throw new BeanCreationException(...);
                      } finally {
                          afterSingletonCreation(beanName); //将beanName从缓存singletonsCurrentlyInCreation中移除
                      }
                  }
                  // 如果bean已经被创建，则向缓存中维护beanName与FactoryBean实例对象的对应关系
                  if (containsSingleton(beanName)) this.factoryBeanObjectCache.put(beanName, object);
              }
          }
          return object;
      }
  }
  /** 步骤2：如果factory不是单例的，或者bean没有被创建，则只获得bean实例，不需要维护到factoryBeanObjectCache中 */
  else {
      // 创建bean的实例对象。即：调用FactoryBean#getObject()获得
      Object object = doGetObjectFromFactoryBean(factory, beanName); 
      if (shouldPostProcess) {
          try {
              // bean在初始化后会调用所有注册的BeanPostProcessor类的postProcessAfterInitialization方法
              object = postProcessObjectFromFactoryBean(object, beanName); 
          } catch (Throwable ex) {throw new BeanCreationException(...);}
      }
      return object;
  }
}

```

在上面代码中，我们还需要再谈一谈`postProcessObjectFromFactoryBean(object, beanName)`方法，它的作用其实就是尽量保证所有 bean 在初始化之后，都会调用所有注册了的 **BeanPostProcessor** 类的 **postProcessAfterInitialization(result, beanName)** 方法，在实际开发过程中可以针对此特性设计自己的业务逻辑。源码如下所述：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3VMBgIVOLp09icUZ4z3PBSVUlZRlRLpCagq0CP1QULoOrnoAGxOzUfLA/640?wx_fmt=png)

五、getSingleton(beanName, singletonFactory)
==========================================

在上面的文章中我们已经介绍过`getSingleton(beanName)`方法了，它的主要作用就是**从缓存中获取单例对象**。那么下面我们要介绍的方法`getSingleton(beanName, singletonFactory)`，是针对于**缓存中并不存在**单例 bean 的时候的处理流程。源码注释如下所示：

```
public Object getSingleton(String beanName, ObjectFactory< ？> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    synchronized (this.singletonObjects) {
        /** 首先，尝试从缓存中获取bean实例 */
        Object singletonObject = this.singletonObjects.get(beanName); 
        if (singletonObject == null) {
            if (this.singletonsCurrentlyInDestruction) throw new BeanCreationNotAllowedException(...);
        
            // 将beanName加入到缓存inCreationCheckExclusions和缓存singletonsCurrentlyInCreation中
            beforeSingletonCreation(beanName);
            
            boolean newSingleton = false, recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) this.suppressedExceptions = new LinkedHashSet<>();
            
            try {
                /** 其次，尝试调用ObjectFactory#getObject()方法，获取bean实例 */
                singletonObject = singletonFactory.getObject(); // ObjectFactory是接口，所以getObject()方法需要实现
                newSingleton = true;
            } catch (IllegalStateException ex) {
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) throw ex;
            } catch (BeanCreationException ex) {
                if (recordSuppressedExceptions) 
                    for (Exception suppressedException : this.suppressedExceptions) 
                        ex.addRelatedCause(suppressedException);
                throw ex;
            } finally {
                if (recordSuppressedExceptions) this.suppressedExceptions = null;
                afterSingletonCreation(beanName); // 将beanName从singletonsCurrentlyInCreation中移除掉
            }
            if (newSingleton) 
                // 添加缓存：singletonObjects 和 registeredSingletons
                // 移除缓存：singletonFactories 和 earlySingletonObjects
                addSingleton(beanName, singletonObject);
        }
        return singletonObject;
    }
}

```

> 通过源码可以看到如下流程：  
> 【**首先**】尝试从缓存 **singletonObjects** 中获取 bean 实例，如果获取到了，就执行 return 返回该实例对象。  
> 【**其次**】如果没有从缓存中获取到 bean 实例，则通过调用 **ObjectFactory#getObject() **方法，获取 bean 实例。 由于 ObjectFactory 是接口，所以** getObject() 方法需要单独实现**。  
> 【**最后**】将实例对象 return 返回即可。

六、createBean(...)
=================

在上面我们介绍`getSingleton(beanName, singletonFactory)`方法源码的时候，提到了其中的 singletonFactory，它是 **ObjectFactory** 类型的，它是一个接口，并且这个接口只提供了一个方法`getObject()`，需要单独实现这个方法，来完成 bean 实例对象的创建，那么具体创建代码在哪个地方呢，即下图中红框的 **createBean(beanName, mbd, args)** 方法。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3oodlA3b59Qib6XynZdISL65oXcS1ZX5FXbEibWzcbF7zkYvrrv2T15cw/640?wx_fmt=png)

其中 **createBean(beanName, mbd, args)** 方法的源码注释如下所示：

```
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    RootBeanDefinition mbdToUse = mbd;
    /** 步骤1：根据class属性或className来获得Class实例对象，如果mbd中没有设置beanClass，则创建新的mbdToUse，设置beanClass */
    Class< ？> resolvedClass = resolveBeanClass(mbd, beanName); 
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }

    /** 步骤2：验证和准备覆盖的方法（MethodOverrides） */
    try {
        mbdToUse.prepareMethodOverrides();
    } catch (BeanDefinitionValidationException ex) {throw new BeanDefinitionStoreException(...);}

    /** 步骤3：给BeanPostProcessors一个机会，来返回一个替代真正实例的代理对象，并直接return返回*/
    try {
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) return bean; // 如果存在代理对象，则返回代理对象
    } catch (Throwable ex) {throw new BeanCreationException(...);}

    /** 步骤4：真正开始创建bean实例对象 */
    try {
        Object beanInstance = doCreateBean(beanName, mbdToUse, args); 
        return beanInstance;
    } catch (Throwable ex) {throw new BeanCreationException(...);}
}

```

针对 “**步骤 1**” 的 **resolveBeanClass(mbd, beanName)** 方法的源码和注释如下所示：

```
protected Class< ？> resolveBeanClass(RootBeanDefinition mbd, String beanName, Class< ？>... typesToMatch) {
    try {
        /** 1：如果mbd中配置了BeanClass，则直接返回 */
        if (mbd.hasBeanClass()) return mbd.getBeanClass(); 

        /** 2：如果没有配置，则通过doResolveBeanClass(mbd, typesToMatch)方法解析出来 */
        if (System.getSecurityManager() != null)  
            return AccessController.doPrivileged((PrivilegedExceptionAction<Class<?>>)
                    () -> doResolveBeanClass(mbd, typesToMatch), getAccessControlContext());
        else 
            return doResolveBeanClass(mbd, typesToMatch); // 解析bean的类型Class
    } catch (PrivilegedActionException pae) {...}
}

```

```
private Class< ？> doResolveBeanClass(RootBeanDefinition mbd, Class< ？>... typesToMatch) {
    ClassLoader beanClassLoader = getBeanClassLoader(), dynamicLoader = beanClassLoader;
    if (!ObjectUtils.isEmpty(typesToMatch)) { // eg：typesToMatch为null，不执行这段代码
        ClassLoader tempClassLoader = getTempClassLoader();
        if (tempClassLoader != null) {
            dynamicLoader = tempClassLoader;
            freshResolve = true;
            if (tempClassLoader instanceof DecoratingClassLoader) {
                DecoratingClassLoader dcl = (DecoratingClassLoader) tempClassLoader;
                for (Class< ？> typeToMatch : typesToMatch) 
                    dcl.excludeClass(typeToMatch.getName());
            }
        }
    }
    
    boolean freshResolve = false;
    String className = mbd.getBeanClassName();
    /** 1：如果可以获得className，并且evaluated与className不同，则以evaluated为准 */
    if (className != null) {
        // 针对mdb（可能将其作为表达式进行解析），解析出className或者Class实例
        Object evaluated = evaluateBeanDefinitionString(className, mbd); 
        if (!className.equals(evaluated)) {
            if (evaluated instanceof Class) return (Class< ？>) evaluated; // 返回通过mbd解析出来的evaluated实例
            else if (evaluated instanceof String) {
                className = (String) evaluated; 
                freshResolve = true;
            } else throw new IllegalStateException(...);
        }
        if (freshResolve) {
            if (dynamicLoader != null) {
                try {
                    return dynamicLoader.loadClass(className); // 返回通过mbd解析出来的evaluated的Class实例
                } catch (ClassNotFoundException ex) {...}
            }
            return ClassUtils.forName(className, dynamicLoader);
        }
    }
    
    /** 2：以mbd.getBeanClassName()为准，来创建Class实例对象 */
    return mbd.resolveBeanClass(beanClassLoader);
}

```

> 主要执行如下几个步骤：  
> 【**步骤 1**】尝试从 mbd 中**获得 beanClass**——`mbd.getBeanClass()`。  
> 【**步骤 2**】如果无法获得`beanClass`，那么再尝试根据 mbd 的配置内容，**解析出 beanClass**。  
> 
> *   • 从 mbd 中**获得 beanClassName**——`mbd.getBeanClassName()`。
>     
> *   • 再针对 mdb（可能将其作为表达式进行解析），解析出 **evaluated**（有可能是 className 或者 Class 实例）
>     
> *   • 如果`beanClassName`与`evaluated`不同，则**以 evaluated 为准**。
>     
> *   • 否则，通过`beanClassName`获得它所对应的 Class 实例对象。
>     

针对 “**步骤 2**” 的 **mbdToUse.prepareMethodOverrides() **方法是用于**检查查找方法是否存在并确定其重载状态**，其源码和注释如下所示：

```
public void prepareMethodOverrides() throws BeanDefinitionValidationException {
    if (hasMethodOverrides()) // 如果配置中存在lookup-method和replace-method，那么hasMethodOverrides()返回true
        getMethodOverrides().getOverrides().forEach(this::prepareMethodOverride);
}

protected void prepareMethodOverride(MethodOverride mo) throws BeanDefinitionValidationException {
    // 根据在lookup-method和replace-method上配置的方法名，去bean类中查找相同方法名称的方法数量
    int count = ClassUtils.getMethodCountForName(getBeanClass(), mo.getMethodName());
    if (count == 0) throw new BeanDefinitionValidationException(...); // 如果存在0个，则抛出异常
    else if (count == 1) mo.setOverloaded(false); // 如果存在1个，则标记为未重载，以避免arg类型检查的开销
}

```

> 在 Spring 中，虽然没有`override-method`这样的配置，但是针对配置的 **lookup-method** 和 **replace-method** 会被的存放在 BeanDefinition 中的`methodOverrides`属性里。

> 然后，会通过 MethodOverride 中的方法名，来校验 bean 类中是否存在对应的方法。并且，**如果只匹配到了 1 个方法，那么将重写标记为未重载，以避免 arg 类型检查的开销**。因为对于方法的匹配来说，如果在一个类中存在多个重载方法，那么在函数调用及增强的时候，还需要根据参数类型进行匹配，这样才能最终确认当前调用的到底是哪个方法。但是，Spring 将一部分匹配工作在这里完成了，即：如果当前类中匹配的方法只有 1 个，那么就设置重载该方法为 false，这样在后续调用的时候就可以直接使用这个方法，而不需要进行方法的参数匹配操作了。

针对 “**步骤 3**” 的 **resolveBeforeInstantiation(beanName, mbdToUse)** 方法的源码和注释如下所示：

```
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    // 默认beforeInstantiationResolved为null，所以会进入if语句中
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        // 如果不是自定义的mbd，并且配置了一些InstantiationAwareBeanPostProcessor
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) { 
            Class< ？> targetType = determineTargetType(beanName, mbd); // 获得beanClass
            if (targetType != null) {
                // 调用InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation方法
                bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                if (bean != null) {
                    // 调用BeanPostProcessor的postProcessAfterInitialization方法
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }
        mbd.beforeInstantiationResolved = (bean != null); // 设置是否执行了beforeInstantiation的解析操作
    }
    return bean;
}

```

**applyBeanPostProcessorsBeforeInstantiation()** 方法如下所示：

```
protected Object applyBeanPostProcessorsBeforeInstantiation(Class< ？> beanClass, String beanName) {
    for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
        Object result = bp.postProcessBeforeInstantiation(beanClass, beanName);
        if (result != null) return result; // 只要返回的result值【不为空】，则中断循环调用，返回结果
    }
    return null;
}

```

> 【解释】**实例化前的后处理器应用——即：创建 bean 的代理对象**。会在 bean 的实例化操作之前进行调用，也就是将 AbstractBeanDefinition 转换为 BeanWrapper 前的处理。给子类一个修改 BeanDefinition 的机会，也就是说当程序经过这个方法后，**bean 可能已经不是我们认为的那个 bean 了**，而是或许成为了一个经过处理的代理 bean，或者可能是通过 cglib 生成的 bean，也可能是通过某些其他技术生成的 bean。

**applyBeanPostProcessorsAfterInstantiation()** 方法如下所示：

```
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) {
    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) return result; // 只要出现了返回的result值为【空】，则中断循环调用，返回结果
        result = current;
    }
    return result;
}

```

> 【解释】**实例化后的后处理器应用——即：对 bean 进行后置处理**。Spring 会在 bean 的初始化后尽可能保证将注册的后处理器的`postProcessAfterInitialization`方法应用到这个 bean 中，**因为如果返回的 bean 不为空，那么便不会再次经历普通 bean 的创建过程，所以只能在这里应用后处理器的 postProcessAfterInitialization 方法**。

针对 “**步骤 4**” 的 **doCreateBean(beanName, mbdToUse, args)** 方法的源码解析，我们会再下面其他章节中进行详细解析和说明，此处暂略。

七、循环依赖
======

对于循环依赖，就是 A 类中引用了 B 类，B 类中引用了 C 类，而 C 类中引用了 A 类，那么这样就会出现循环依赖的情况。针对循环依赖，有如下情况：

【**单例类型**】——构造器循环依赖，则无法被解决。

```
<bean id="testA" class="com.muse.TestA">
    <constructor-arg index="0" ref="testB"/>
</bean>
<bean id="testB" class="com.muse.TestB">
    <constructor-arg index="0" ref="testC"/>
</bean>
<bean id="testC" class="com.muse.TestC">
    <constructor-arg index="0" ref="testA"/>
</bean>

```

【**单例类型**】——setter 循环依赖，可以通过提前暴露刚完成构造器注入但未完成其他步骤的 bean 来解决。

```
<bean id="testA" class="com.muse.TestA">
    <property />
</bean>
<bean id="testB" class="com.muse.TestB">
    <property />
</bean>
<bean id="testC" class="com.muse.TestC">
    <property />
</bean>

```

【**原型类型**】——无法被解决。

```
<bean id="testA" class="com.muse.TestA" scope="prototype">
    <property />
</bean>
<bean id="testB" class="com.muse.TestB" scope="prototype">
    <property />
</bean>
<bean id="testC" class="com.muse.TestC" scope="prototype">
    <property />
</bean>

```

八、doCreateBean(...)
===================

8.1> 概述
-------

我们跟踪了这么多 Spring 代码，经历了这么多函数，或多或少也会发现这么一个规律，就是：**一个真正干活的函数，大多是以 do 开头命名的**。那么，我们马上要介绍的这个`doCreateBean(...)`方法，就是**负责常规 bean 创建**的。相关的源码和注释如下所示：

```
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    /** 步骤1：获得BeanWrapper实例对象instanceWrapper */
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) instanceWrapper = this.factoryBeanInstanceCache.remove(beanName); // 清除缓存
    if (instanceWrapper == null) instanceWrapper = createBeanInstance(beanName, mbd, args); // 创建BeanWrapper实例
    
    /** 步骤2：调用所有配置了MergedBeanDefinitionPostProcessor实现类的postProcessMergedBeanDefinition方法 */
    Object bean = instanceWrapper.getWrappedInstance();
    Class< ？> beanType = instanceWrapper.getWrappedClass();
    if (beanType != NullBean.class) mbd.resolvedTargetType = beanType;
    synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            try {
                // MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition(mbd, beanType, beanName)
                applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            } catch (Throwable ex) {...}
            mbd.postProcessed = true;
        }
    }

    /** 步骤3：针对“正在创建”的“允许循环依赖”的“单例“执行【提前曝光】 */
    boolean earlySingletonExposure = (mbd.isSingleton() && // 是否是单例的
                                      this.allowCircularReferences && // 是否允许循环依赖
                                      isSingletonCurrentlyInCreation(beanName)); // 单例bean是否正在创作中
    if (earlySingletonExposure) 
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    
    Object exposedObject = bean;
    try {
        /** 步骤4：对bean进行填充操作，将各个属性值进行注入 */
        populateBean(beanName, mbd, instanceWrapper);
        
        /** 步骤5：调用初始化方法，例如：init-method */
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    } catch (Throwable ex) {...}

    /** 步骤6：针对需要执行”提前曝光“的单例 */
    if (earlySingletonExposure) {
        Object earlySingletonReference = getSingleton(beanName, false);
        if (earlySingletonReference != null) {
            if (exposedObject == bean) exposedObject = earlySingletonReference; // bean没有被增强改变
            else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
                String[] dependentBeans = getDependentBeans(beanName); // 获得所有依赖
                Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);           
                for (String dependentBean : dependentBeans) 
                    if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) // 执行依赖检测
                        actualDependentBeans.add(dependentBean);
                // 因为bean创建后，它所依赖的bean一定创建了，那么不为空则表示所依赖的bean没有全部创建完，即：存在循环依赖
                if (!actualDependentBeans.isEmpty()) throw new BeanCurrentlyInCreationException(...);
            }
        }
    }

    try {
        /** 步骤7：如果配置了destroy-method，这里需要注册以便于在销毁时候进行调用 */
        registerDisposableBeanIfNecessary(beanName, bean, mbd);
    } catch (BeanDefinitionValidationException ex) {...}
    
    return exposedObject;
}

```

下面我们就针对流程中的重要逻辑进行更深入的源码解析。

8.2> createBeanInstance() 创建 bean 的实例
-------------------------------------

首先，我们先来分析一下用于**创建 bean 的实例**的`createBeanInstance(beanName, mbd, args)`方法。相关的源码和注释如下所示：

```
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    /** 步骤1：解析beanClass */
    Class<?> beanClass = resolveBeanClass(mbd, beanName);
    if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) 
        throw new BeanCreationException(...);

    /** 步骤2：如果配置了instanceSupplier，则通过调用Supplier#get()方法来创建bean的实例，并封装为BeanWrapper实例 */
    Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
    if (instanceSupplier != null) return obtainFromSupplier(instanceSupplier, beanName);

    /** 步骤3：如果配置了factoryMethodName或者配置文件中存在factory-method，则使用工厂方法创建bean的实例 */
    if (mbd.getFactoryMethodName() != null) return instantiateUsingFactoryMethod(beanName, mbd, args);

    boolean resolved = false, autowireNecessary = false;
    if (args == null) {
        synchronized (mbd.constructorArgumentLock) {
            // 一个类有多个构造函数，每个构造函数有不同的参数，所以调用前需要先根据参数锁定构造函数或者对应的工厂方法
            if (mbd.resolvedConstructorOrFactoryMethod != null) {
                resolved = true;
                autowireNecessary = mbd.constructorArgumentsResolved;
            }
        }
    }

    /** 步骤4：如果已经解析过（resolved=true），那么就使用解析好的构造函数方法，不需要再次锁定 */
    if (resolved) {
        if (autowireNecessary) 
            return autowireConstructor(beanName, mbd, null, null);  // 构造函数自动注入
        else 
            return instantiateBean(beanName, mbd); // 使用默认构造函数构造
    }

    /** 步骤5：如果没解析过，那么则需要根据参数解析构造函数 */
    Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
            mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
        return autowireConstructor(beanName, mbd, ctors, args); // 构造函数自动注入
    }

    /** 步骤6：尝试获取默认构造的首选构造函数 */
    ctors = mbd.getPreferredConstructors();
    if (ctors != null) return autowireConstructor(beanName, mbd, ctors, null); // 构造函数自动注入

    /** 步骤7：如果以上都不行，则使用默认构造函数构造bean实例 */
    return instantiateBean(beanName, mbd); 
}

```

在上面的代码中，主要负责创建 bean 的相关方法有两个，分别是`autowireConstructor(...)`和`instantiateBean(...)`，下面我们就针对这两个方法进行源码解析。

### 8.2.1> autowireConstructor(...) 有参数的实例化构造

首先是`autowireConstructor(...)`方法，它是负责**有参数的实例化构造**，这部分流程比较复杂，下面是该方法的源码和注释：

```
BeanWrapper autowireConstructor(String beanName, RootBeanDefinition mbd, Constructor<?>[] ctors, 
                                Object[] explicitArgs){
    return new ConstructorResolver(this).autowireConstructor(beanName, mbd, ctors, explicitArgs);
}

```

```
public BeanWrapper autowireConstructor(String beanName, RootBeanDefinition mbd, Constructor<?>[] chosenCtors, 
                                       Object[] explicitArgs) {
    BeanWrapperImpl bw = new BeanWrapperImpl();
    this.beanFactory.initBeanWrapper(bw);
    Constructor<?> constructorToUse = null;
    ArgumentsHolder argsHolderToUse = null;
    
    /** 步骤1：尝试获得构造函数（constructorToUse）和方法入参（argsToUse）*/
    Object[] argsToUse = null;
    if (explicitArgs != null) // case1：如果getBean方法调用的时候指定了方法参数，则直接使用
        argsToUse = explicitArgs; 
    else { // case2：如果没有指定explicitArgs，则尝试从mbd中获取构造函数入参argsToUse和构造函数constructorToUse
        Object[] argsToResolve = null;
        synchronized (mbd.constructorArgumentLock) {
            constructorToUse = (Constructor<?>) mbd.resolvedConstructorOrFactoryMethod;
            if (constructorToUse != null && mbd.constructorArgumentsResolved) {
                argsToUse = mbd.resolvedConstructorArguments;
                if (argsToUse == null) argsToResolve = mbd.preparedConstructorArguments;
            }
        }
        if (argsToResolve != null) 
            // 转换参数类型。假设构造函数为A(int, int)，可以通过如下方法将入参的("1", "1")转换为(1, 1)
            argsToUse = resolvePreparedArguments(beanName, mbd, bw, constructorToUse, argsToResolve);
    }

    /** 步骤2：如果constructorToUse和argsToUse没有全部解析出来，则尝试从配置文件中解析获取 */
    if (constructorToUse == null || argsToUse == null) {
        Constructor<?>[] candidates = chosenCtors; 
        if (candidates == null) { // 如果入参chosenCtors为空，则获取bean中所有的构造方法作为“候选”构造方法
            Class<?> beanClass = mbd.getBeanClass();
            try {
                candidates = (mbd.isNonPublicAccessAllowed() ? 
                              beanClass.getDeclaredConstructors() : 
                              beanClass.getConstructors());
            } catch (Throwable ex) {...}
        }
        
        // 如果类中只有1个无参的构造函数，则创建bean的实例对象并且return
        if (candidates.length == 1 && explicitArgs == null && !mbd.hasConstructorArgumentValues()) {
            Constructor<?> uniqueCandidate = candidates[0];
            if (uniqueCandidate.getParameterCount() == 0) {
                synchronized (mbd.constructorArgumentLock) {
                    mbd.resolvedConstructorOrFactoryMethod = uniqueCandidate;
                    mbd.constructorArgumentsResolved = true;
                    mbd.resolvedConstructorArguments = EMPTY_ARGS;
                }
                bw.setBeanInstance(instantiate(beanName, mbd, uniqueCandidate, EMPTY_ARGS));
                return bw;
            }
        }

        boolean autowiring = (chosenCtors != null || 
                              mbd.getResolvedAutowireMode() == AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR);
        ConstructorArgumentValues resolvedValues = null;
        // 解析构造函数参数个数minNrOfArgs
        int minNrOfArgs;
        if (explicitArgs != null) minNrOfArgs = explicitArgs.length;
        else {
            ConstructorArgumentValues cargs = mbd.getConstructorArgumentValues(); // 提取配置文件中配置的构造函数参数
            resolvedValues = new ConstructorArgumentValues(); // 用于承载解析后的构造函数参数的值
            minNrOfArgs = resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues); // 解析参数个数
        }
        
        // 对构造函数执行排序操作，其中：public构造函数优先且参数数量降序排列，然后是非public构造函数参数数量降序排列
        AutowireUtils.sortConstructors(candidates); 

        // 遍历所有构造函数，对每个构造函数进行参数匹配操作
        int minTypeDiffWeight = Integer.MAX_VALUE;
        Set<Constructor< ？>> ambiguousConstructors = null;
        Deque<UnsatisfiedDependencyException> causes = null;
        for (Constructor< ？> candidate : candidates) {
            int parameterCount = candidate.getParameterCount();
            if (constructorToUse != null && argsToUse != null && argsToUse.length > parameterCount) break;
            if (parameterCount < minNrOfArgs) continue; 

            /** 创建构造函数的”参数持有者（ArgumentsHolder）“实例对象argsHolder */
            ArgumentsHolder argsHolder;
            Class<?>[] paramTypes = candidate.getParameterTypes(); // 获得构造函数的参数类型集合
            if (resolvedValues != null) { // 只有当explicitArgs等于null时，resolvedValues才满足不为空
                try {
                    // 获得@ConstructorProperties({"x", "y"})注解里配置的参数名称
                    String[] paramNames = ConstructorPropertiesChecker.evaluate(candidate, parameterCount);
                    if (paramNames == null) {
                        // 从BeanFactory中获得配置的ParameterNameDiscoverer实现类
                        ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();
                        if (pnd != null) 
                            paramNames = pnd.getParameterNames(candidate); // 获得构造函数的参数名称
                    }
                    // 根据【paramTypes】和【paramNames】创建参数持有者ArgumentsHolder
                    argsHolder = createArgumentArray(beanName, mbd, resolvedValues, bw, paramTypes, paramNames,
                            getUserDeclaredConstructor(candidate), autowiring, candidates.length == 1);
                } catch (UnsatisfiedDependencyException ex) {...}
            } else {
                if (parameterCount != explicitArgs.length) continue;
                argsHolder = new ArgumentsHolder(explicitArgs);
            }

            /** 探测是否有不确定性的构造函数存在，例如：不同构造函数的参数为父子关系 */
            int typeDiffWeight = (mbd.isLenientConstructorResolution() ?
                argsHolder.getTypeDifferenceWeight(paramTypes) : argsHolder.getAssignabilityWeight(paramTypes));
            if (typeDiffWeight < minTypeDiffWeight) { // 如果它代表着当前最接近的匹配，则选择作为构造函数
                constructorToUse = candidate;
                argsHolderToUse = argsHolder;
                argsToUse = argsHolder.arguments;
                minTypeDiffWeight = typeDiffWeight;
                ambiguousConstructors = null;
            } else if (constructorToUse != null && typeDiffWeight == minTypeDiffWeight) {
                if (ambiguousConstructors == null) {
                    ambiguousConstructors = new LinkedHashSet<>();
                    ambiguousConstructors.add(constructorToUse);
                }
                ambiguousConstructors.add(candidate);
            }
        }
        
        if (constructorToUse == null) {
            if (causes != null) {
                UnsatisfiedDependencyException ex = causes.removeLast();
                for (Exception cause : causes) 
                    this.beanFactory.onSuppressedException(cause);
                throw ex;
            }
            throw new BeanCreationException(...);
        } else if (ambiguousConstructors != null && !mbd.isLenientConstructorResolution()) 
            throw new BeanCreationException(...);

        // 将解析的构造函数加入到缓存中
        if (explicitArgs == null && argsHolderToUse != null) 
            argsHolderToUse.storeCache(mbd, constructorToUse); 
    }
    Assert.state(argsToUse != null, "Unresolved constructor arguments");
    
    /** 步骤3：通过constructorToUse和argsToUse创建bean的实例对象，并存储到BeanWrapper中 */
    bw.setBeanInstance(instantiate(beanName, mbd, constructorToUse, argsToUse));
    return bw;
}

```

针对上面的源码内容，我们可以总结出如下几个步骤：

> 【**步骤 1**】确定构造函数的参数 **argsToUse**。首先：根据`explicitArgs`参数进行判断；其次：尝试从`mbd`中获取；最后：尝试从`配置文件`中获取  
> 【**步骤 2**】确定构造函数 **constructorToUse**。  
> 【**步骤 3**】根据确定的构造函数转换对应的参数类型。  
> 【**步骤 4**】构造函数不确定性的验证。  
> 【**步骤 5**】根据实例化策略类中的`instantiate(mbd, beanName, this)`方法以及`constructorToUse`和`argsToUse`来实例化 bean，并封装到 **BeanWrapper** 中。

### 8.2.2> instantiateBean(...) 无参数的实例化构造

上面我们介绍了带参数的构造方法解析，那么下面我们就针对**不带参数的构造函数**的实例化过程进行解析操作，其相关注释和源码如下所示：

```
protected BeanWrapper instantiateBean(String beanName, RootBeanDefinition mbd) {
    try {
        Object beanInstance;
        if (System.getSecurityManager() != null) 
            beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () -> getInstantiationStrategy().instantiate(mbd, beanName, this), getAccessControlContext());
        else 
            beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, this); //实例化策略
        BeanWrapper bw = new BeanWrapperImpl(beanInstance);
        initBeanWrapper(bw); // 将创建好的实例封装为BeanWrapper对象
        return bw;
    }
    catch (Throwable ex) {throw new BeanCreationException(...);}
} 

```

通过上面针对 instantiateBean 方法源码之后，我们会发现，主要只有两个操作：

> 【**操作 1**】通过实例化策略类的`instantiate(mbd, beanName, this)`方法创建 bean 实例对象。  
> 【**操作 2**】将创建的 bean 实例封装为`BeanWrapper`对象。

### 8.2.3> instantiate(...)

在上面我们提到的 “通过实例化策略类的 **instantiate(mbd, beanName, this)** 方法创建 bean 实例对象”，那么下面我们就来分析一下这个方法的内部逻辑：

```
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
    /** 步骤1：如果没有配置lookup-method或replace-method，则直接使用反射创建bean的实例对象即可 */
    if (!bd.hasMethodOverrides()) {
        Constructor< ？> constructorToUse;
        synchronized (bd.constructorArgumentLock) {
            constructorToUse = (Constructor< ？>) bd.resolvedConstructorOrFactoryMethod;
            // 获得构造函数实例
            if (constructorToUse == null) {
                final Class< ？> clazz = bd.getBeanClass();
                if (clazz.isInterface()) throw new BeanInstantiationException(...);
                try {
                    if (System.getSecurityManager() != null) 
                        constructorToUse = AccessController.doPrivileged((PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
                    else 
                        constructorToUse = clazz.getDeclaredConstructor();
                    bd.resolvedConstructorOrFactoryMethod = constructorToUse;
                }
                catch (Throwable ex) {throw new BeanInstantiationException(...);}
            }
        }
        return BeanUtils.instantiateClass(constructorToUse); // 通过反射创建bean实例
    }
    /** 步骤2：否则，需要使用cglib创建代理对象，将动态方法织入到bean的实例对象中 */
    else 
        return instantiateWithMethodInjection(bd, beanName, owner); // Must generate CGLIB subclass.
}

```

通过 **instantiateWithMethodInjection(bd, beanName, owner)** 方法，使用`cglib`创建代理对象

```
protected Object instantiateWithMethodInjection(RootBeanDefinition bd, String beanName, BeanFactory owner) {
    return instantiateWithMethodInjection(bd, beanName, owner, null);
}

protected Object instantiateWithMethodInjection(RootBeanDefinition bd, String beanName, BeanFactory owner, 
                                                Constructor< ？> ctor, Object... args) {
    return new CglibSubclassCreator(bd, owner).instantiate(ctor, args);
}

public Object instantiate(@Nullable Constructor< ？> ctor, Object... args) {
    /** 步骤1：创建Cglib代理类 */
    Class< ？> subclass = createEnhancedSubclass(this.beanDefinition); 

    /** 步骤2：创建Cglib代理类实例对象 */
    Object instance;
    if (ctor == null) instance = BeanUtils.instantiateClass(subclass);
    else {
        try {
            Constructor< ？> enhancedSubclassConstructor = subclass.getConstructor(ctor.getParameterTypes());
            instance = enhancedSubclassConstructor.newInstance(args);
        }
        catch (Exception ex) {...}
    }

    /** 步骤3：将代理对象封装成Factory实例对象，并注入lookup-method和replace-mehtod */
    Factory factory = (Factory) instance;
    factory.setCallbacks(new Callback[] {NoOp.INSTANCE,
            new LookupOverrideMethodInterceptor(this.beanDefinition, this.owner), // 注入lookup-method
            new ReplaceOverrideMethodInterceptor(this.beanDefinition, this.owner)}); // 注入replace-mehtod
    return instance;
}

private Class< ？> createEnhancedSubclass(RootBeanDefinition beanDefinition) {
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(beanDefinition.getBeanClass());
    enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
    if (this.owner instanceof ConfigurableBeanFactory) {
        ClassLoader cl = ((ConfigurableBeanFactory) this.owner).getBeanClassLoader();
        enhancer.setStrategy(new ClassLoaderAwareGeneratorStrategy(cl));
    }
    enhancer.setCallbackFilter(new MethodOverrideCallbackFilter(beanDefinition));
    enhancer.setCallbackTypes(CALLBACK_TYPES);
    return enhancer.createClass(); // 创建Cglib代理类
}

```

8.3> getEarlyBeanReference(...) 记录创建 bean 的 ObjectFactory
---------------------------------------------------------

好了，经过上面 8.2 章节的一大段解析之后，我们还是要把视角放到`doCreateBean(...)`，在这个方法里，有如下一段代码，是用来处理单例提前曝光逻辑的：

```
/** 步骤3：判断是否【提前曝光】单例 */
boolean earlySingletonExposure = (mbd.isSingleton() && // 单例
                                  this.allowCircularReferences && // 允许循环依赖
                                  isSingletonCurrentlyInCreation(beanName)); // 正在创建的单例
if (earlySingletonExposure)  
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));

```

对于 **addSingletonFactory(...)** 方法，主要是为了避免后期循环依赖，可以在 bean 初始化完成前将用于创建 bean 实例的`ObjectFactory`加入缓存中

```
protected void addSingletonFactory(String beanName, ObjectFactory< ？> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}

```

对于 **getEarlyBeanReference(beanName, mbd, bean)** 方法，它会调用所有`SmartInstantiationAwareBeanPostProcessor#getEarlyBeanReference(...)`方法。

```
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {
            exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);
        }
    }
    return exposedObject;
}

```

我们以 **AB 循环依赖**为例，类`A`中含有属性类`B`，而类`B`中又会含有属性类`A`，那么初始化 beanA 的过程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F3S4o4M6Iibog3A12oTr3XRu46ibcD4DicKoGocicg9icxOIbrdgtOEBE238w/640?wx_fmt=png)

当调用`getBean(A)`的时候，并不是直接去实例化 A，而是**先去检测缓存中是否有已经创建好的 bean**，或者**是否已经存在创建好的 ObjectFactory**，而此时对于 A 的 ObjectFactory 我们早已经创建，所以便不会再去向后执行，而是直接调用`ObjectFactory#getObject()`方法去创建 A。

8.4> populateBean(...) 属性注入
---------------------------

针对属性注入的操作，是由 **populateBean(...)** 方法进行负责的，其相关源码和注释如下图所示：

```
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    /** 步骤1：如果bean的实例bw为null，但是却定义了bean的属性值，则抛异常；否则直接return返回 */
    if (bw == null) {
        if (mbd.hasPropertyValues()) throw new BeanCreationException(...);
        else return;
    }

    /** 步骤2：针对配置了InstantiationAwareBeanPostProcessor实现类，那么会调用postProcessAfterInstantiation方法 */
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) 
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) 
            if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) 
                return;

    /** 步骤3：获得配置的bean属性，然后根据注入类型（byName/byType）执行注入操作 */
    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);
    int resolvedAutowireMode = mbd.getResolvedAutowireMode(); // 获得自动装配模型AutowireMode
    if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
        if (resolvedAutowireMode == AUTOWIRE_BY_NAME) 
            autowireByName(beanName, mbd, bw, newPvs); // 通过set方法方法，根据name自动注入
        if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) 
            autowireByType(beanName, mbd, bw, newPvs); // 通过set方法方法，根据type自动注入
        pvs = newPvs;
    }

    /** 步骤4：获取加工处理后的属性pvs */  
    boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors(); sor
    boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE); 
    PropertyDescriptor[] filteredPds = null;
    if (hasInstAwareBpps) { // 是否配置了后置处理器InstantiationAwareBeanPostProcesor
        if (pvs == null) pvs = mbd.getPropertyValues();
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            // 执行InstantiationAwareBeanPostProcessor#postProcessProperties(...)方法
            PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                if (filteredPds == null)
                    // 从给定的BeanWrapper中提取一组经过筛选的PropertyDescriptor，排除忽略的依赖关系类型或在忽略的依赖接口上定义的属性。
                    filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                // 执行InstantiationAwareBeanPostProcessor#postProcessPropertyValues(...)方法
                pvsToUse = bp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                if (pvsToUse == null) return;
            }
            pvs = pvsToUse;
        }
    }

    /** 步骤5：执行依赖检查 */  
    if (needsDepCheck) { // 是否需要执行依赖检测操作
        if (filteredPds == null) filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
        checkDependencies(beanName, mbd, filteredPds, pvs);
    }

    /** 步骤6：将属性应用到bean中 */
    if (pvs != null) 
        applyPropertyValues(beanName, mbd, bw, pvs);
}

```

### 8.4.1> autowireByName(...) 根据名称进行注入

在传入的参数 pvs 中找出已经加载的 bean，然后递归实例化相关 bean，最后将其加入到 pvs 中。源码如下所示：

```
protected void autowireByName(String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {
    String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw); // 寻找bw中需要依赖注入的属性
    for (String propertyName : propertyNames) {
        if (containsBean(propertyName)) {
            Object bean = getBean(propertyName); // 递归初始化相关bean
            pvs.add(propertyName, bean);
            registerDependentBean(propertyName, beanName); // 注册依赖
            if (logger.isTraceEnabled()) logger.trace(...);
        }
        else if (logger.isTraceEnabled()) logger.trace(...);
    }
}

```

### 8.4.2> autowireByType(...) 根据类型进行注入

由于需要根据类型进行注入，所以需要进行类型的解析和对比操作，相关的代码逻辑就变得复杂了，如下是相关源码：

```
protected void autowireByType(String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {
    TypeConverter converter = getCustomTypeConverter();
    if (converter == null) converter = bw;

    Set<String> autowiredBeanNames = new LinkedHashSet<>(4);
    String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw); // 寻找bw中需要依赖注入的属性
    for (String propertyName : propertyNames) {
        try {
            PropertyDescriptor pd = bw.getPropertyDescriptor(propertyName);
            if (Object.class != pd.getPropertyType()) {
                MethodParameter methodParam = BeanUtils.getWriteMethodParameter(pd); // 探测指定属性的set方法
                boolean eager = !(bw.getWrappedInstance() instanceof PriorityOrdered);
                DependencyDescriptor desc = new AutowireByTypeDependencyDescriptor(methodParam, eager);
                /**
                 * 解析指定beanName的属性所匹配的值，并把解析到的属性名称存储在autowiredBeanNames中。
                 * 当属性存在多个封装bean时（@Autowired private List<A> aList）将会找到所有匹配A类型的bean并将其注入进去
                 */    
                Object autowiredArgument = resolveDependency(desc, beanName, autowiredBeanNames, converter);
                if (autowiredArgument != null) 
                    pvs.add(propertyName, autowiredArgument);
                for (String autowiredBeanName : autowiredBeanNames) {
                    registerDependentBean(autowiredBeanName, beanName); // 注册依赖
                    if (logger.isTraceEnabled()) logger.trace(...);
                }
                autowiredBeanNames.clear();
            }
        }
        catch (BeansException ex) {throw new UnsatisfiedDependencyException(...);}
    }
}

```

> 【解释】Spring 中提供了对集合类型注入的支持，如果使用注解的方式，则如下所示：  
> `@Autowired`  
> `private List<Test> tests;`  
> Spring 将会把所有与 Test 匹配的类型找出来并注入到`tests`属性中，正式由于这一原因，所以在 autowireByType(...) 方法中，新建了局部变量`autowiredBeanNames`，用于存储所有依赖的 bean，如果只是对非集合类的属性注入的话，那么这个属性就没啥用处了。

下面我们再来分析一下 **resolveDependency(...)** 方法，其源码如下所示：

```
public Object resolveDependency(DependencyDescriptor descriptor, 
                                String requestingBeanName,
                                Set<String> autowiredBeanNames, 
                                TypeConverter typeConverter) throws BeansException {
    descriptor.initParameterNameDiscovery(getParameterNameDiscoverer());
    if (Optional.class == descriptor.getDependencyType()) // Optional类型的特殊处理
        return createOptionalDependency(descriptor, requestingBeanName);
    else if (ObjectFactory.class == descriptor.getDependencyType() || // ObjectFactory类型的特殊处理
             ObjectProvider.class == descriptor.getDependencyType()) // ObjectProvider类型的特殊处理
        return new DependencyObjectProvider(descriptor, requestingBeanName);
    else if (javaxInjectProviderClass == descriptor.getDependencyType()) // javaxInjectProviderClass类型的特殊处理
        return new Jsr330Factory().createDependencyProvider(descriptor, requestingBeanName);
    else {
        Object result = getAutowireCandidateResolver().getLazyResolutionProxyIfNecessary(descriptor, requestingBeanName);
        if (result == null) // 默认的getLazyResolutionProxyIfNecessary(...)方法返回null
            // 通用处理逻辑
            result = doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter);
        return result;
    }
}

```

下面我们再来分析一下 **doResolveDependency(...)** 方法，其源码如下所示：

```
public Object doResolveDependency(DependencyDescriptor descriptor, 
                                  String beanName, 
                                  Set<String> autowiredBeanNames, 
                                  TypeConverter typeConverter) throws BeansException {
    InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
    try {
        Object shortcut = descriptor.resolveShortcut(this);
        if (shortcut != null) return shortcut;
        Class< ？> type = descriptor.getDependencyType();
        /** 步骤1：针对Spring中@Value注解的获取和解析 */
        Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor); 
        if (value != null) {
            if (value instanceof String) {
                // 如果实现并注册了StringValueResolver接口的实现，则调用resolveStringValue方法对value进行处理
                String strVal = resolveEmbeddedValue((String) value); 
                BeanDefinition bd = (beanName != null && containsBean(beanName) ?
                        getMergedBeanDefinition(beanName) : null);
                // 如果配置了BeanExpressionResolver，则对value值进行表达式解析
                value = evaluateBeanDefinitionString(strVal, bd);
            }

            // 获得类型转换器，并对value值进行转换处理
            TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
            try {
                return converter.convertIfNecessary(value, type, descriptor.getTypeDescriptor());
            } catch (UnsupportedOperationException ex) {
                return (descriptor.getField() != null ?
                        converter.convertIfNecessary(value, type, descriptor.getField()) :
                        converter.convertIfNecessary(value, type, descriptor.getMethodParameter()));
            }
        }

        /** 步骤2：如果注入的是StreamDependencyDescriptor、Collection、Map、数组 */
        Object multipleBeans = resolveMultipleBeans(descriptor, beanName, autowiredBeanNames, typeConverter);
        if (multipleBeans != null) 
            return multipleBeans;

        
        Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
        if (matchingBeans.isEmpty()) {
            if (isRequired(descriptor)) 
                raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
            return null;
        }
        String autowiredBeanName;
        Object instanceCandidate;
        if (matchingBeans.size() > 1) {
            autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
            if (autowiredBeanName == null) {
                if (isRequired(descriptor) || !indicatesMultipleBeans(type)) 
                    return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
                else 
                    return null;
            }
            instanceCandidate = matchingBeans.get(autowiredBeanName);
        }
        else {
            Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
            autowiredBeanName = entry.getKey();
            instanceCandidate = entry.getValue();
        }
        if (autowiredBeanNames != null) 
            autowiredBeanNames.add(autowiredBeanName);
        if (instanceCandidate instanceof Class) 
            instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
        Object result = instanceCandidate;
        if (result instanceof NullBean) {
            if (isRequired(descriptor)) 
                raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
            result = null;
        }
        if (!ClassUtils.isAssignableValue(type, result)) throw new BeanNotOfRequiredTypeException(...);
        return result;
    } finally {
        ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
    }
}

```

> 【解释】寻找类型的匹配执行顺序是，首先尝试使用解析器进行解析，如果解析器没有成功解析，那么可能是使用默认的解析器没有做任何处理，或者是使用了自定义的解析器，但是对于集合等类型来说并不在解析范围之内，所以再次对不同类型进行不同情况的处理，虽然说对于不同类型处理的方式不一致，但是大致的思路还是相似的。

### 8.4.3> applyPropertyValues(...)

程序运行到这里，已经完成了对所有注入属性的获取，但是**获取的属性是以**`PropertyValues`**形式存在的，并没有应用到已经实例化的 bean 中**，这项工作是在`applyPropertyValues(...)`方法中实现的，具体源码如下所示：

```
protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
    if (pvs.isEmpty()) return;
    
    if (System.getSecurityManager() != null && bw instanceof BeanWrapperImpl) 
        ((BeanWrapperImpl) bw).setSecurityContext(getAccessControlContext());
    
    MutablePropertyValues mpvs = null;
    List<PropertyValue> original;
    if (pvs instanceof MutablePropertyValues) {
        mpvs = (MutablePropertyValues) pvs;
        if (mpvs.isConverted()) { // 如果mpvs中的值已经被转换为对应的类型，那么可以直接设置到bw中
            try {
                bw.setPropertyValues(mpvs);
                return;
            }
            catch (BeansException ex) {throw new BeanCreationException(...);}
        }
        original = mpvs.getPropertyValueList();
    } else 
        // 如果pvs不是MutablePropertyValues类型，那么直接使用原始的属性获取方法
        original = Arrays.asList(pvs.getPropertyValues());
    
    TypeConverter converter = getCustomTypeConverter();
    if (converter == null) converter = bw;

    // 获取对应的解析器
    BeanDefinitionValueResolver valueResolver = new BeanDefinitionValueResolver(this, beanName, mbd, converter);
    List<PropertyValue> deepCopy = new ArrayList<>(original.size());
    boolean resolveNecessary = false;
    // 遍历属性，将其转换为对应类的对应属性类型
    for (PropertyValue pv : original) {
        if (pv.isConverted()) deepCopy.add(pv);
        else {
            String propertyName = pv.getName();
            Object originalValue = pv.getValue();
            if (originalValue == AutowiredPropertyMarker.INSTANCE) {
                Method writeMethod = bw.getPropertyDescriptor(propertyName).getWriteMethod();
                if (writeMethod == null) throw new IllegalArgumentException(...);
                originalValue = new DependencyDescriptor(new MethodParameter(writeMethod, 0), true);
            }
            Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue); // 执行类型转换
            Object convertedValue = resolvedValue;
            boolean convertible = bw.isWritableProperty(propertyName) &&
                    !PropertyAccessorUtils.isNestedOrIndexedProperty(propertyName);
            if (convertible) 
                convertedValue = convertForProperty(resolvedValue, propertyName, bw, converter);

            if (resolvedValue == originalValue) {
                if (convertible) pv.setConvertedValue(convertedValue);
                deepCopy.add(pv);
            }
            else if (convertible && originalValue instanceof TypedStringValue &&
                    !((TypedStringValue) originalValue).isDynamic() &&
                    !(convertedValue instanceof Collection || ObjectUtils.isArray(convertedValue))) {
                pv.setConvertedValue(convertedValue);
                deepCopy.add(pv);
            }
            else {
                resolveNecessary = true;
                deepCopy.add(new PropertyValue(pv, convertedValue));
            }
        }
    }
    if (mpvs != null && !resolveNecessary) 
        mpvs.setConverted();
    
    try {
        bw.setPropertyValues(new MutablePropertyValues(deepCopy));
    } catch (BeansException ex) {throw new BeanCreationException(...);}
}

```

8.5> initializeBean(...) 初始化 bean
---------------------------------

这个方法主要是针对我们配置的`init-method`属性，当 Spring 中程序已经执行过 bean 的实例化，并且进行了属性的填充，而就在这时将会调用用户设定的初始化方法。具体源码如下所示：

```
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
            invokeAwareMethods(beanName, bean);
            return null;
        }, getAccessControlContext());
    }
    else invokeAwareMethods(beanName, bean); 

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) 
        // 调用配置的所有BeanPostProcessor#postProcessBeforeInitialization方法
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);

    try {
        invokeInitMethods(beanName, wrappedBean, mbd); // 激活用户自定义的init方法
    } catch (Throwable ex) {throw new BeanCreationException(...);}
    
    if (mbd == null || !mbd.isSynthetic()) 
        // 调用配置的所有BeanPostProcessor#postProcessAfterInitialization方法
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    
    return wrappedBean;
}

```

### 8.5.1> invokeAwareMethods(...) 激活 Aware 方法

Spring 中提供了一些 **Aware 接口**实现，比如：`BeanFactoryAware`、`ApplicationContextAware`、`ResourceLoaderAware`、`ServletContextAware`等，实现这些 Aware 接口的 bean 在被初始化之后，可以取得一些相对的资源。我们可以通过示例来了解一下 Aware 的用法。

```
public class Hello {
    public void say() {
        System.out.println("hello");
    }
}

```

```
public class Test implements BeanFactoryAware {
    private BeanFactory beanFactory;

    // 声明bean的时候，Spring会自动注入BeanFactory实例
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void testAware() {
        // 通过hello这个bean，从BeanFactory中获得实例
        Hello hello = (Hello) beanFactory.getBean("hello");
        hello.say();
    }
}

```

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9bga3PYJ7HPcNyic9THf0F32x1R3eZ5ljkyePb4KonkjM1Hn0qLE1uI9UbiceHcNpPZTVlFELNWwPw/640?wx_fmt=png)

按照上面的方法我们可以获取到 Spring 中的 BeanFactory，并且可以根据 BeanFactory 获取所有的 bean，以及进行相关设置。当然还有其他 Aware 的使用方法也都是大同小异的，此时，我们再来看一下 invokeAwareMethods(...) 的源码实现：

```
private void invokeAwareMethods(String beanName, Object bean) {
    if (bean instanceof Aware) {
        if (bean instanceof BeanNameAware) 
            ((BeanNameAware) bean).setBeanName(beanName); // 向BeanNameAware中注入beanName
        
        if (bean instanceof BeanClassLoaderAware) {
            ClassLoader bcl = getBeanClassLoader();
            if (bcl != null) 
                ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl); // 向BeanClassLoaderAware中注入classLoader
        }
        
        if (bean instanceof BeanFactoryAware) // 向BeanFactoryAware中注入beanFactory
            ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
    }
}

```

### 8.5.2> invokeInitMethods(...) 激活自定义的 init 方法

客户定制的初始化方法除了我们熟知的使用配置`init-method`外，还有使自定义的 bean 实现`InitializingBean接口`，并在`afterPropertiesSet()`方法中实现自己的初始化业务逻辑。其中，**InitializingBean 的 afterPropertiesSet() 方法先被执行，而 init-method 后执行**。下面是相关源码实现：

```
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    boolean isInitializingBean = (bean instanceof InitializingBean);
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        if (logger.isTraceEnabled()) logger.trace(...);
        
        if (System.getSecurityManager() != null) {
            try {
                AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                    ((InitializingBean) bean).afterPropertiesSet(); // 属性初始化后的处理
                    return null;
                }, getAccessControlContext());
            }
            catch (PrivilegedActionException pae) {throw pae.getException();}
        }
        // 属性初始化后的处理，调用InitializingBean#afterPropertiesSet()方法
        else ((InitializingBean) bean).afterPropertiesSet(); 
    }
    if (mbd != null && bean.getClass() != NullBean.class) {
        String initMethodName = mbd.getInitMethodName();
        if (StringUtils.hasLength(initMethodName) &&
                !(isInitializingBean && 
                "afterPropertiesSet".equals(initMethodName)) &&
                !mbd.isExternallyManagedInitMethod(initMethodName)) {
            // 调用自定义的init-method方法
            invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}

```

调用自定义的`init-method`方法，源码如下所示：

```
protected void invokeCustomInitMethod(String beanName, Object bean, RootBeanDefinition mbd) throws Throwable {
    String initMethodName = mbd.getInitMethodName(); // 获得init-method方法
    Assert.state(initMethodName != null, "No init method set");

    /** 步骤1：获得init-method对应的Method实例对象 */
    Method initMethod = (mbd.isNonPublicAccessAllowed() ?
            BeanUtils.findMethod(bean.getClass(), initMethodName) :
            ClassUtils.getMethodIfAvailable(bean.getClass(), initMethodName));
    if (initMethod == null) 
        if (mbd.isEnforceInitMethod()) throw new BeanDefinitionValidationException(...);
        else return;
    Method methodToInvoke = ClassUtils.getInterfaceMethodIfPossible(initMethod);

    /** 步骤2：通过反射，执行init-method的方法调用 */
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
            ReflectionUtils.makeAccessible(methodToInvoke);
            return null;
        });
        try {
            AccessController.doPrivileged((PrivilegedExceptionAction<Object>)
                    () -> methodToInvoke.invoke(bean), getAccessControlContext());
        } catch (PrivilegedActionException pae) {...}
    }
    else {
        try {
            ReflectionUtils.makeAccessible(methodToInvoke);
            methodToInvoke.invoke(bean); // 通过反射，执行init-method的方法调用
        }
        catch (InvocationTargetException ex) {throw ex.getTargetException();}
    }
}

```

8.6> registerDisposableBeanIfNecessaryn(...) 注册 DisposableBean
--------------------------------------------------------------

Spring 同时也提供了销毁方法的扩展入口，对于销毁方法的扩展，除了我们熟知的配置属性 **destroy-method** 方法外，用户还可以注册后处理器 **DestructionAwareBeanPostProcessor** 来统一处理 bean 的销毁方法，具体源码如下所示：

```
protected void registerDisposableBeanIfNecessary(String beanName, Object bean, RootBeanDefinition mbd) {
    AccessControlContext acc = (System.getSecurityManager() != null ? getAccessControlContext() : null);
    if (!mbd.isPrototype() && requiresDestruction(bean, mbd)) {
        /** 
         * 单例模式下注册需要销毁的bean，此方法中会处理实现DisposableBean的bean，
         * 并且对所有的bean使用DestructionAwareBeanPostProcessor处理
         */
        if (mbd.isSingleton())  
            registerDisposableBean(beanName, new DisposableBeanAdapter(bean, 
                                                             beanName, 
                                                             mbd, 
                                                             getBeanPostProcessorCache().destructionAware, 
                                                             acc));
        else { // 自定义scope的处理
            Scope scope = this.scopes.get(mbd.getScope());
            if (scope == null) throw new IllegalStateException(...);
            scope.registerDestructionCallback(beanName, new DisposableBeanAdapter(bean, 
                                                             beanName, 
                                                             mbd, 
                                                             getBeanPostProcessorCache().destructionAware, 
                                                             acc));
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

[SpringBoot2.x——Part1](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486233&idx=1&sn=fda7d410bedf6b56457757225a4d6bfa&chksm=e91149e4de66c0f2acd4a95cba907f43dbbd0700c8f70ccd9fc0cc7f57048683732cf79fc77b&scene=21#wechat_redirect)  

[SpringBoot2.x——SpringBoot Web 源码解析](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247486972&idx=1&sn=74556f31641a1bd3c29a2663bc533208&chksm=e9114f01de66c6177f47b26297b0836b5df0e92feea7a7420f611b3d3a2f8926eb952d20b117&scene=21#wechat_redirect)