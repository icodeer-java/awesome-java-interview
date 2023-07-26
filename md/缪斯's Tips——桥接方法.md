> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489061&idx=1&sn=d356fa2bbaeae1880e44595f189e5ba4&chksm=e91154d8de66ddcece9870ea95bf862f511db6f8ff4a6862b4aa6bc86af57e54943655cbb8bc&scene=178&cur_album_id=2451364959101059073#rd)

桥接方法
====

> 一提到桥接方法，最常见的应该是 23 种设计模式其中的 1 种，但是我们此处提到的桥接方法，是由于在 JDK5 中泛型的诞生而随之产生的。那么既然要提到桥接方法，就不得不先聊一下它所产生的前因——类型擦除。

类型擦除
----

泛型是提供给 javac 编译器使用的，它用于限定集合的输入类型，让编译器在**源代码级别上**，挡住向集合中插入非法数据。但编译器编译完带有泛型的 java 程序后，**生成的 class 文件中将不再带有泛型信息**，以此使程序运行效率不受到影响，这个过程称之为 “类型擦除”。

下面我们以 Animal 作为接口，Cat 作为实现，描述一下类型擦除。如下是**泛型未被擦除**的样子：

```
public interface Animal<T> {
    void eat(T t);
}

public class Cat implements Animal<String> {
    @Override
    public void eat(String s) {
        System.out.println("cat eat " + s);
    }
}

```

由于生成的 class 文件中将不带有泛型信息，那么我们**将泛型擦除掉，以 Object 来代替**：

```
public interface Animal {
    void eat(Object t);
}

public class Cat implements Animal {
    @Override
    public void eat(String s) {
        System.out.println("cat eat " + s);
    }
}

```

整体处理方式如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxmiapuMYToW7MQhdoRUbV6AQxwXR1O9hiaYIyp18xBtRYTvCmNQZiaia1Pg/640?wx_fmt=png)

> 泛型被擦除掉了，我们大功告成了！！哎？不对啊！细心的同学们会发现，这个接口定义的是`void eat(Object t)`，但是 Cat 类中 Override 的却是`void eat(String s)`，这个是错误的呀，方法签名都不对，不能 Override 的啊。所以，我们发现，虽然泛型被擦除掉了，并且以 Object 来代替了，但是**程序代码却是错误的**了。那怎么办呢？OK，别着急，桥接方法它来了。

桥接方法
----

由于类型被擦除了，为了维持多态性，所以编译器就自动生成了桥接方法。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxL2Yjv7qHwP2jnUzQ0DemPictXHyT0oURc2kx7XwUF6dbibiaLXKrZA7PQ/640?wx_fmt=png)

> 通过上图中红色框框住的方法我们可以看到，生成了一个`void eat(Object s)`方法，这样就可以维持 Animal 和 Cat 之间的 Override 关系。并且在这个方法内，我们通过将入参 s 强转型为 String 的方式，增加了对入参的限制，并且最终调用的方法依然是`void eat(String s)`这个方法。这么一看，泛型被擦除了，并且依然可以保证对入参类型的限制，完美！

不过，这时候对于一些严谨的同学们就会有质疑了，你说的这个真实存在吗？能证明给我们看吗？可以的。还记得反射那节课吧，给大家介绍过如何查看类中所有的方法，那么，我们就来查看一下 Cat 类中所有的方法有哪些吧。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxkW4LnANclzibxggImgpDzjRbR7GZso1ib4hswroHhiaaNj9NnZFia3prZg/640?wx_fmt=png)

> 我们发现，通过`Cat.class.getDeclaredMethods()`方式获得 Cat 中的方法时，出现了一个根本不是我们编写的类——即：`eat(Object o)`，那么该方法就是桥接方法了。

桥接方法的应用
-------

上面介绍了类型擦除和桥接方法，那么会有同学们疑惑了，这东西有啥用呢？其实它的用处还真的不少呢，尤其是在应用框架上面，当需要使用反射方式访问方法时，就需要先过滤掉桥接方法，因为这个方法毕竟不是我们自己编写的嘛。下面我们来找一下桥接方法的身影吧。

### MyBatis

桥接方法在 Mybatis 中的身影如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxJXkhSunGgPEHxsL868iaYmoRYFJibibYlVQB99XAt9icRY8ckq7jTdY6icA/640?wx_fmt=png)

### Spring

桥接方法在 Spring 中的身影如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxHDFfOY8dm0tCuNFYh8TG6J8IibvaqkMV3LB9GXEKFzXyhxZNlMeaiaDg/640?wx_fmt=png)