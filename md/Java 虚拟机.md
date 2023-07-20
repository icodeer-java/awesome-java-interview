> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247487562&idx=1&sn=30db880222112093f8dbb325088c16e4&chksm=e91152b7de66dba10d26f9fff3bb4a0e8f2a0af443cd70b79a2092cbaf9b4a86f98566070128&scene=178&cur_album_id=2361652762658144256#rd)

    本篇文章包含了 Java 虚拟机中非常重要且面试中经常会遇到的知识点，内容比较多，从 JMM 到垃圾回收算法再到垃圾回收器和调优常用参数，其中并不包含 ZGC，需要这方面知识点的同学们可以期待后续这方面的文章。闲话少叙，本篇文章的大纲如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1QE3rxMNuZgHXoJb69NuVOmsC25LhvUlyt7vibwEiap5oRLLYVkzqnq4Q/640?wx_fmt=png)

一、JVM 的架构  

============

1.1> Java 程序的跨平台特性
------------------

*   在 Java 虚拟机中执行的指令，我们称之为 Java 字节码指令。下面显示了同一个 Java 程序，被编译为一组 Java 字节码的集合之后，可以通过 Java 虚拟机运行于不同的操作系统上，它以 Java 虚拟机为中介，实现了跨平台的特性。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1KnpFzHibw7LvnZXQu82Fsicd80ELviaXOoZiaDDtZZlPkZHiatuQXx7Etng/640?wx_fmt=png)

1.2> JVM 的基本结构  

-----------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd12uDB5goBOLKdiaq4icT7zsPz0IgHUMHicEj6tFLUicwUNTmh09q4cjKqNQ/640?wx_fmt=png)

*   **类加载子系统**
    

*   负责从文件系统或者网络中**加载 Class 信息**。
    

*   **方法区**
    

*   加载的类信息存放于一块称为**方法区**的内存空间。
    
*   除了类的信息外，方法区中可能还会存放**运行时常量池**信息，包括：**字符串字面量**和**数字常量**（这部分常量信息是 Class 文件中常量池部分的内存映射）是所有线程共享的。
    

*   **Java 堆**
    

*   Java 堆是在虚拟机启动的时候建立的，它是 Java 程序最主要的内存工作区域。**几乎所有的 Java 对象实例和数组都存放于 Java 堆中**。堆空间是所有线程共享的。
    

*   **直接内存**
    

*   Java 的 NIO 库允许 Java 程序使用直接内存，在 **NIO** 被广泛使用后，直接内存的使用也变得非常普通。
    
*   直接内存是 Java 堆外的、**直接向系统申请的内存空间**。访问速度会优于 Java 堆。它大空间大小只会受操作系统给出的最大内存影响。与 Java 堆相比，虽然在**访问读写**上直接内存有较大的优势，但是在**内存空间申请**时，堆空间的速度远远高于直接内存。
    
*   结论：**直接内存适合内存空间申请次数较少、访问较频繁的场合**。
    

*   **Java 栈**
    

*   它是线程私有的，它在线程创建的时候被创建。
    
*   Java 栈中保存着**栈帧信息**、**局部变量**、**方法参数**、同时和 Java 方法的**调用**、**返回**密切相关。
    

*   **本地方法栈**
    

*   它与 Java 栈非常类似，最大的不同在于 Java 栈用于 Java 方法的调用，而本地方法栈则**用于 Native 方法调用**。
    

*   **PC（Program Counter）寄存器**
    

*   它是每个线程私有的空间。
    
*   如果正在执行的方法不是本地方法，PC 寄存器就会指向**当前正在被执行的指令**。
    
*   如果当前方法是本地方法，那么 PC 寄存器的值就是 **undefined**。
    

*   **垃圾回收系统**
    

*   GC 可以对**方法区**、**Java 堆**和**直接内存**进行回收。
    
*   Java 堆是 GC 的工作重点，和 C、C++ 不同，Java 中所有的**对象空间释放都是隐式的**。
    

*   **执行引擎**
    

*   是 Java 虚拟机的最核心组件之一，它负责**执行虚拟机的字节码**。
    

1.3> Class 类加载  

-----------------

*   我们从事 java 开发，接触最多的也就是 java 源文件，但是**类是如何被 jvm 加载执行**的呢？我们来看图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1SHGPt1oq77uVe3SQDeFEYrQ56kNZGO8H88jyxhBibMaG8WgZOVibo4nA/640?wx_fmt=png)

### 1.3.1> Class 文件内容  

*   cafe babe 魔数，唯一作用是确定这个文件是否是一个能被 JVM 接收的 Class 文件。
    
*   0000 次版本号 （Minor Version）
    
*   0034 主版本号 （Major Version）
    
*   003a 常量池容量计数器
    
*   0a00 0e00 2409 ...  常量池（包含：直接引用和符号引用 javap -verbose Student）
    
*   一个 Class 文件中，包含 16 个部分，再此就不一一说了。
    

### 1.3.2> ClassLoader 对类进行加载  

#### 1.3.2.1> 主动加载的 4 种情况

*   情况 1：**new** 一个对象实例的时候。   
    
*   情况 2：利用**反射**或者或者 **clone** 的方式。
    
*   情况 3：初始化子类时，**父类**会被优先初始化。
    
*   情况 4：调用一个类的**静态方法**时。
    

#### 1.3.2.2> 类的加载分为 5 步

*   **第一步：加载 ClassLoader**
    

*   通过类的**全路径名称**，获取类的**二进制数据流**。
    
*   解析类的二进制数据流，转化为**方法区（永久代 or 元空间）**内部的数据结构。
    
*   创建 **java.lang.Class 类的实例对象**，表示该类型。
    

*   **第二步：验证**
    

*   它的目的是保证第一步中加载的字节码是**合法且符合规范的**。
    

*   大体分为 4 步验证
    

*   **格式检查**
    

    检查魔数、版本、长度等等。

*   **语义检查**
    

    抽象方法是否有实现类、是否继承了 final 类等等编码语义上的错误检查。

*   **字节码验证**
    

    跳转指令是否指向正确的位置，操作数类型是否合理等。

*   **符号引用验证**
    

    符号引用的直接引用是否存在

*   **第三步：准备**
    

*   准备阶段是正式**为类变量分配内存**并**设置类变量的初始值**阶段，即：在方法区中分配这些变量所使用的内存空间。
    
*   注意这里所说的初始值概念，比如一个类变量定义为：public static int v = 8080; 实际上变量 v 在准备阶段过后的初始值为 0 而不是 8080，将 v 赋值为 8080 的 put static 指令是程序被编译后，**存放于类构造器 <client> 方法之中**。但是注意如果声明为：public static **final** int v = 8080; 在编译阶段会为 v 生成 ConstantValue 属性，在准备阶段虚拟机会根据 ConstantValue 属性将 v 赋值为 8080。
    

*   **第四步：解析**
    

*   解析阶段是指虚拟机将**运行时常量池**中的**符号引用**替换为**直接引用**的过程。符号引用就是 class 文件中的：CONSTANT_Class_info、CONSTANT_Field_info、CONSTANT_Method_info 等类型的常量。（参见 1.3.3）
    

*   **第五步：初始化**
    

*   到达这个阶段，类就可以顺利加载到系统中。此时，类才会**开始执行 Java 字节码**。
    
*   初始化阶段是执行**类构造器 <client> 方法**的过程。<client> 方法是由编译器自动收集类中的**类变量**的赋值操作和**静态语句块**中的语句合并而成的。虚拟机会保证子 <client> 方法执行之前，父类的 < client > 方法已经执行完毕，**如果一个类中没有对静态变量赋值也没有静态语句块，那么编译器可以不为这个类生成 <client>() 方法**。
    

### 1.3.3> 符号引用和直接引用  

*   在**解析阶段**会有一个步骤，将**运行时常量池**当中二进制数据当中的**符号引用转化为直接引用**的过程。
    

#### 1.3.3.1> 符号引用

*   以一组符号来描述所引用的目标。
    
*   符号引用可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可，符号引用和虚拟机的布局无关。
    
*   为什么要有符号引用？
    

*   在编译的时候每个 java 类都会被编译成一个 class 文件，但**在编译的时候虚拟机并不知道所引用类的地址，所以就用符号引用来代替**，而在**解析阶段**就是为了把这个符号引用转化成为真正的地址的阶段。
    

#### 1.3.3.2> 直接引用

*   直接引用和虚拟机的布局是相关的，**不同的虚拟机对于相同的符号引用所翻译出来的直接引用一般是不同的**。
    
*   如果有了直接引用，那么直接引用的目标一定被加载到了内存中。
    
*   直接引用可以是：
    

*   直接指向目标的指针——指向对象，类变量和类方法的指针
    
*   相对偏移量——指向实例的变量，方法的指针
    
*   一个间接定位到对象的句柄。
    

#### 1.3.3.3> 示范例子

*   **创建一个类 Student.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1RAK0KG9EYxMicoMVNnbwBSalhOBmhZWrRibMvJCVVgdYXqLFWHjM76xw/640?wx_fmt=png)

*   **执行** **javap -verbose Student.class**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ZPHCAjIHOYnzEDib5ytOCJ5NqE9LyTgNz3CrF0Z95richtPfiatum0zuA/640?wx_fmt=png)

二、JMM  

========

*   讲解内存管理，此处是面试关键点，面试官常问的就是 JVM 内存管理分为几部分？每部分都是做什么的？哪些是线程私有的？哪些是线程共享的？
    
*   **Class 只有在必须要使用的时候才会被加载****。**
    

2.1> 程序计数器（线程私有）  

-------------------

*   是当前线程所执行的字节码的行号指示器，指向虚拟机字节码指令的位置。
    
*   被分配了一块**较小的内存空间**。
    
*   针对于**非 Native 方法**：是当前线程执行的字节码的行号指示器。
    

针对于 **Native 方法**，则为 undefined。

*   每个线程都有自己独立的程序计数器，所以，该内存是**线程私有**的。
    
*   这块区域是唯一一个在虚拟机中没有规定任何 OutOfMemoryError 情况的区域
    

2.2> 虚拟机栈（线程私有）  

------------------

*   为执行 **Java 方法**服务的，是**描述方法执行**的内存模型。
    
*   Java 栈是**线程私有**的内存空间。
    
*   每次**函数调用**的数据都是通过栈传递的。
    
*   在 Java 栈中保存的主要内容为**栈帧**。它的数据结构就是**先进后出**。每当函数被调用，该函数就会被**入栈**，每当函数执行完毕，就会执行**出栈**操作。而当前**栈顶**，即为正在执行的函数。
    
*   每个方法在执行的同时都会创建一个**栈帧**用于存储**局部变量表**、**操作数栈**、**帧数据区**等信息。
    
*   栈帧操作示意图——StackFrameTest.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1qumZOk2f92ckhL3lvpiaib1AIf5YIJ2qFdH2klzOrGJ7L44E3RjdclmA/640?wx_fmt=png)

*   由于每次函数调用都会生成对应的栈帧，从而占用一定的栈空间。因此，如果栈空间不足，那么函数调用自然无法继续进行下去。当请求的栈深度大于最大可用栈深度时，系统就会抛出 StackOverflowError 栈溢出错误，所以**函数嵌套调用的层次**在很大程度上**由栈的大小决定**：栈越大，函数可以支持的嵌套调用次数就越多。
    
*   可以通过**参数 - Xss** 来指定线程的最大栈空间。示例如下所示：
    

*   **StackOverflowTest.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1lTGKGjD1uB2yjDIoKy1mpgRJGXhWkq4FOGRrDfHYBmqHxrlBI5XYLQ/640?wx_fmt=png)

*   **设置最大栈内存为** **-Xss160K****，运行结果如下所示：**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1bDODibLd6fjKtHzqkFTdBsIu5Eu8aanibDatraSwDlGSbDJUPfjMbrcw/640?wx_fmt=png)

*   **设置最大栈内存为** **-Xss256K****，运行结果如下所示：**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1kNOTpp0R1RiaQbQT4hb3UhyN8JYXFpz9Fs0zFu09MtzrBucWx8xrjsw/640?wx_fmt=png)

### 2.2.1> 局部变量表  

*   局部变量表是栈帧的重要组成部分之一，它用于保存**函数的参数**以及**局部变量**。 
    
*   局部变量表中的变量只**在当前函数调用中有效**，当函数调用结束后，随着函数栈帧的销毁，局部变量表也会随之销毁。
    
*   由于**局部变量表在栈帧中**，因此，如果函数的参数和局部变量较多，会使得局部变量表膨胀，从而每一次函数调用就会占用更多的栈空间，最终导致函数的嵌套调用次数减少，如下所示：
    

*   **StackOverflow2Test.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1CWTBEmHaZ0CgDAVMLjFBGClZDUIib8NByQViaHEa7oELeicwoeYWtuHtw/640?wx_fmt=png)

*   **设置最大栈内存为** **-Xss160K****，运行结果如下所示：**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1eKg8icuNznY3ibOB8qY1skgR9FVrBdPCWDUwiaqh7UH0ojV6BEAzsv03g/640?wx_fmt=png)【解释】StackOverflowTest.java 执行同样栈大小，count=850

*   **设置最大栈内存为** **-Xss256K****，运行结果如下所示：**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1Evt1XoibbfOWVeYsjkzvNjIsCNkAU6Zf69MJvmV9vQ138iaLfG5HepeQ/640?wx_fmt=png)【解释】StackOverflowTest.java 执行同样栈大小，count=2131  

*   **使用 jclasslib 查看局部变量表中的内容**
    

*   在 idea 中添加 jclasslib 视图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1BMdic3gTsaGAD26UVJ1qM4vhgPr9bbg0RsVian9OFkmxibKFpHOZ1PicEw/640?wx_fmt=png)

*   添加后，使用 Show Bytecode With Jclasslib 查看 StackOverflow2Test.java 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1eyosNmpqlqI1dPsE3Oz6ob3HQ3r82wiaeGv2NOmdicT4F3TTcuyHxICA/640?wx_fmt=png)

*   查看结果如下所示，表明红框里的参数表示了在 Class 文件中的局部变量表的内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ndZzkSygOrQ5sEzVXyHU6hJnTRHpiapMOLq8llS72eBmAXApp8O41Qw/640?wx_fmt=png)

### 2.2.2> 操作数栈  

*   操作数栈也是栈帧中重要的内容之一，它主要用于**保存计算过程中的中间结果**，同时作为计算过程中的**变量临时的存储空间**。
    
*   操作数栈也是一个先进后出的数据结构。许多 Java 字节码指令都需要通过操作数栈进行参数传递。比如下面所示的 iadd 指令：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1bWibrYKCiapjicqZOwM1Hkqw7xGWy2NgB33bqnKa2jpqBaL2WOkOhNRjw/640?wx_fmt=png)

【解释】30 和 15 出栈，计算出结果为 45 后，再入栈。

### 2.2.3> 帧数据区  

*   栈帧还需要一些数据来支持**常量池解析**、**正常方法返回**和**异常处理**等。
    
*   **常量池解析**
    

*   大部分 Java 字节码指令需要进行常量池访问。在帧数据区中保存着**访问常量池的 “指针”**，方便程序访问常量池。
    

*   **正常方法返回**和**异常处理**
    

*   当函数返回或者出现异常时，虚拟机必须恢复调用者函数的栈帧，并让调用者函数继续执行下去。
    
*   对于异常处理，虚拟机必须有一个**异常处理表**。方便在发生异常的时候找到处理异常的代码。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1wOG6Gowj95w0V16mChhibibk6KzvrDO3e7TUsicpK8Sd9kdagPpPYWqiag/640?wx_fmt=png)

【解释】

*   表示在字节码偏移量 0~19 字节之间可能抛出任意异常，如果遇到异常，则跳到字节码偏移量为 19 处执行。
    

### 2.2.4> 本地方法栈（线程私有）  

*   本地方法栈则为 **Native 方法**服务。
    

### 2.2.5> 堆（线程共享）  

*   运行时数据区，**几乎所有的对象都保存在 java 堆中**。
    
*   Java 堆是**完全自动化管理**的，通过垃圾回收机制，垃圾对象会被自动清理，而不需要显示地释放。
    
*   堆是垃圾收集器**进行 GC 的最重要的内存区域**。
    
*   Java 堆可以分为：**新生代**（Eden 区、S0 区、S1 区）和 **老年代****。**
    
*   在绝大多数情况下，对象**首先分配在 eden 区**，在**一次新生代 GC 回收后**，如果对象还存活，则会进入 S0 或 S1，之后，每经历过一次新生代回收，对象如果存活，它的年龄就会加一。当对象的年龄达到一定条件后，就会被认为是老年代对象，从而进入老年代。
    
*   对象在内存中的分配
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1BL0JgzUcib0ib6w11LchpHT9ZhM5IZOH4uvqiaALZ1ict79rq9DA4TbVUw/640?wx_fmt=png)

### 2.2.6> 方法区 / 永久代 / 元空间（线程共享）  

#### 2.2.6.1> 方法区

*   逻辑上的东西，是 **JVM 的规范**，所有虚拟机必须遵守的**。**
    
*   是 JVM 所有**线程共享**的、用于**存储类信息**，例如：类的**字段**、**方法数据**、**方法代码****、****常量池**等。
    
*   方法区的大小决定了系统**可以保存多少个类**。
    

#### 2.2.6.2> 永久代（JDK8 之前）  

*   -XX:PermSize
    
    设置初始永久代大小。例如：-XX:PermSize=5m
    
*   -XX:MaxPermSize
    
    设置最大永久代大小，默认情况下为 64MB。例如：-XX:MaxPermSize=5m
    
*   指内存的永久保存区域，主要存放 Class 和 Meta（元数据）的信息，Class 在被加载的时候被放入永久区域，它和存放实例的区域不同，**GC 不会在主程序运行期对永久区域进行清理**。所以这也导致了永久代的区域会随着加载的 Class 的增多而胀满，最终抛出 OOM 异常。 
    
*   如果系统使用了一些**动态代理**，那么有可能会在运行时生成大量的类，从而造成内存溢出。所以，**设置合适的永久代大小**，对于系统的稳定性是至关重要的。
    

#### 2.2.6.3> 元空间（JDK8 及之后）  

*   -XX:MaxMetaspaceSize
    
    设置元空间默认初始大小，默认为 20.75MB。例如：-XX:MetaspaceSize=40m
    
    设置最大元数据空间。例如：-XX:MaxMetaspaceSize=40m
    
*   在 Java8 中，永久代已经被移除，被一个称为 “元数据区”（元空间）的区域所取代。
    
*   元空间的本质和永久代类似，元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用**堆外的直接内存**。
    
*   因此，与永久代不同，如果不指定大小，默认情况下，虚拟机会**耗尽所有的可用系统内存**。
    

#### 2.2.6.4> 为什么使用元空间替换永久代？  

*   表面上看是为了**避免 OOM 异常**。因为通常使用 PermSize 和 MaxPermSize 设置永久代的大小就决定了永久代的上限，但是不是总能知道应该设置为多大合适, 如果使用默认值很容易遇到 OOM 错误。当使用元空间时，可以加载多少类的元数据就不再由 MaxPermSize 控制, 而由系统的实际可用空间来控制。
    
*   更深层的原因还是要**合并 HotSpot 和 JRockit 的代码**，JRockit 从来没有所谓的永久代，也不需要开发运维人员设置永久代的大小，但是运行良好。同时也不用担心运行性能问题了, 在覆盖到的测试中, 程序启动和运行速度降低不超过 1%，但是这点性能损失换来了更大的安全保障。由于永久代内存经常不够用或者发生内存泄露，爆出异常 java.lang.OutOfMemoryError: PermGen 。字符串存在永久代中，容易出现性能问题和内存溢出。类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。永久代会位 GC 带来不必要的复杂度，而且回收效率偏低。Oracle 可能会将 HotSpot 和 JRockit 合二为一。
    

三、垃圾回收算法  

===========

3.1> 可触及性
---------

*   什么叫可触及性，就是 GC 时，是根据它来确定对象是否可被回收的。也就是说，**从根节点**开始是否可以访问到某个对象，也说明这个对象是否被使用。分为 3 种状态：
    

*   **可触及**：从根节点开始，可以到达某个对象。
    
*   **可复活**：对象引用被释放，但是可能在 **finalize()** 函数中被初始化复活。
    
*   **不可触及**：由于 **finalize() 只会执行一次**，所以，错过这一次复活机会的对象，则为不可触及状态。
    

*   看下面例子：**DieAliveObject.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1HXvpO8Qs91t5a4SdW8UrKiabmQLcutNjV2Roicfz0IGH85UibtFhAMkVg/640?wx_fmt=png)

【解释】  

*   Java9 中 finalize 方法为什么被废弃？
    

*   因为 finalize() 函数有可能发生引用外泄，在**无意中复活对象**。
    
*   由于 finalize() 函数是被系统调用的，**调用时间是不明确的**，因此不是一个好的资源释放方案，推荐在 try-catch-finally 语句中进行资源的释放。
    
*   java.lang.ref.Cleaner 和 java.lang.ref.PhantomReference 提供更灵活和有效的方式，在对象无法再访问时释放资源。
    

3.2> 引用级别  

------------

*   一共四个级别：
    
    **强引用**、**软引用**、**弱引用**、**虚引用**
    

### 3.2.1> 强引用

*   就是一般程序中的引用。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ds9n1tflskOUCJQZ3fOQqjqkSUIlzOderRygbqUGhZfqoWLjQgI4kg/640?wx_fmt=png)

*   上面例子中的两个引用都是强引用，强引用具备以下特点：
    

*   强引用可以**直接访问**目标对象。
    
*   强引用所指向的对象在任何时候都**不会被系统回收**，虚拟机宁愿抛出 OOM 异常，也不会回收强引用所指向的对象。
    
*   强引用**可能导致内存泄漏**。
    

### 3.2.2> 软引用（SoftReference）  

*   软引用是比强引用弱一点的引用类型。
    
*   一个对象只持有软引用，那么当**堆空间不足**时，才会被回收。因此，软引用对象不会引起内存溢出。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1KDCzr508WM0b2ghzjHmMRNM70c52QwzfIf5kXESga4ydYYDPnn1R7A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1fiafOdJn8VtaDnbbwkiciayM2SyK8bdyZKxIQpVNwvibxchrdiaGVrVXPWw/640?wx_fmt=png)  

### 3.2.3> 弱引用（WeakReference）  

*   当 **GC 的时候**，只要发现存在弱引用，**无论系统堆空间是否不足**，均会将其回收。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1eHL0DVicicB87X626VT9ibb1iccTTfDuFvBWUI5Uxtvk2ojz23NT4zBpcw/640?wx_fmt=png)

### 3.2.4> 虚引用（PhantomReference）  

*   如果对象持有虚引用，其实**与没有引用是一样**的。虚引用必须和**引用队列**在一起使用，它的作用是**用于跟踪 GC 回收过程**，所以可以将一些资源释放操作放置在虚引用中执行和记录。
    
*   示例如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1xXPjN7t9VBa7JtEibnBecNYDbHyrRKnwEUdyg8bzibiajj7LQNnFViaAOg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd10m1oibQpASHca7zJlshT9m89zfpRdnibcwqv5kPWuXbZeStPmLYqtW6A/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1kflLzYXz9rCug5wpewb1XyPvgpGnv1XTGyQzW8jFIrS3N9PbXeUk2g/640?wx_fmt=png)  

### 3.2.5> 相关面试题  

*   讲完了触及性和引用级别，我们来看一道面试题 LocalValueGC.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1VOu2fG4khtjzXfSibznUaJuiaCHVnFkygslkSiaTUZoKt26rniabPWXaqA/640?wx_fmt=png)

3.3> 槽位复用  

------------

*   槽位复用例子——LocalValueGC.java，通过 **jclasslib** 查看如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1Y3j3SsN5wib4DlRUExicJmuibXG1s0ibz1LibVTHIvicWl6A0Z0ARe8CW7hA/640?wx_fmt=png)

*   也可以执行 **javap -verbose LocalValueGC.class** 查看
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ZmvBtUr2RNfQxUnomBTTbYrnJgM7BXEnpuS0tO47AzoVcMvMvboI2w/640?wx_fmt=png)

*   图解如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1fnCUqRibUXPCRBPMIWZybCtNThAzckN0YdW7Tv3xicPv6YtLZOicDmnRw/640?wx_fmt=png)

3.4> 对象的分配  

-------------

*   Java 对象分配流程图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ZcqDhR3yJ345CkdjvnDUUwW8EdCJYzBictN85W3F3zVUjSLyiaa6ia8yw/640?wx_fmt=png)

【解释】

*   并不是所有对象都分配在堆上，除了**堆**（绝⼤多数对象分配到堆上）以外，还有两个地⽅可以存放对象——**栈**和 **TLAB**。
    

*   如果**开启栈上分配**，JVM 会先进行**栈上分配**；
    
*   如果**没有开启栈上分配**或**不符合条件**，则会进行 **TLAB 分配**；
    
*   如果 **TLAB 分配不成功且不满足进入老年代的条件**，则会在 **eden 区分配****；**
    
*   如果对象**满足了直接进入老年代的条件**，那就直接在**老年代分配**。
    

### 3.4.1> 栈上分配  

*   栈上分配是 JVM 提供的一项**优化技术**。
    
*   基本思想如下所示：
    

*   对于那些**线程私有的对象**（即：不可能被其他线程访问的对象），可以**将它们打散分配在栈上**，而不是分配在堆上。
    
*   分配在栈上的好处是可以在函数调用**结束后自行销毁**，而不需要垃圾回收器的介入，从而提高系统的性能。
    
*   对于**大量的零散小对象**，栈上分配提供了一种很好的对象分配优化策略，栈上分配速度快，并且可以有效避免 GC 带来的负面影响，但是由于和堆空间相比，**栈空间较小**，因此对于**大对象无法也不适合在栈上分配**。
    

*   **栈上分配的技术基础，两者必须都开启：**
    

*   **逃逸分析**：逃逸分析的目的是判断对象的作用域**是否有可能逃逸出函数体**。
    
*   **标量替换**：允许**将对象打散分配在栈上**。比如：若一个对象拥有两个字段，会将这两个字段视作局部变量进行分配。
    

*   只能在 **server 模式**下才能启用逃逸分析；
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1RcsagnEOd4uVpic0qdfsYcCsKMXOkIqdm4LxXiagWzKicMVKmOrTaZFCA/640?wx_fmt=png)

*   查看当前 jvm 所有配置的系统参数值状态
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1P1zH1mVJbuwWlGcIOBQGSjGasA0icjGLYeLpItRHgiaAWHfOo5MHMibwQ/640?wx_fmt=png)

*   参数 **-XX:+DoEscapeAnalysis** 启用逃逸分析；
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd17z2uePqA7OQs9XnNNVStoM3uiblO0OXOfIj6TL0WpXVvInecvcyRkdQ/640?wx_fmt=png)【解释】**Java SE 6u23 版本之后，HotSpot 中默认就开启了逃逸分析**，可以通过选项 - XX:+PrintEscapeAnalysis 查看逃逸分析的筛选结果。

*   参数 **-XX:+EliminateAllocations** 开启标量替换（默认打开）。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1YCdibcoTibpHU7dtmanibYGCGibiahxNyT8jpWfchr2cV0iasMtVvlia5eq0g/640?wx_fmt=png)

*   参数 **-XX:-UseTLAB** 关闭 TLAB（默认打开）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1qSp5VQB7KsR87Af4MXpSKz2zJdwictzHekEokPSDLRrt86ku4gvjfRg/640?wx_fmt=png)

*   举例：
    

*   **AssignOnStack.java**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1MMWv4uN2ogH0wIJ8OZ1NWytvVCKYk9I0mDqJhiaibgK15ENnMSlTOm9Q/640?wx_fmt=png)

*   **当开启栈上分配的时候，输出如下：**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd17VnPUeFpR10opChR0dGzbRFdK5JWAfrKBknMjdqnsp6KnlNfiarlmrQ/640?wx_fmt=png)

*   **当关闭栈上分配（即：关闭逃逸分析或标量替换中的任何一个）再次执行的时候，输出如下：**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1At1Hh2WBLSXbKJ0AOVg0lsowXLUvkSB7wGDeocgjZNFJXbC8M99NGA/640?wx_fmt=png)

### 3.4.2> TLAB  

*   TLAB 的全称是 **Thread Local Allocation Buffer**，即**线程本地分配缓存区**，这是一个线程专用的内存分配区域。
    
*   由于对象一般会分配在堆上，而堆是全局共享的。因此在同一时间，可能会有多个线程在堆上申请空间。因此，每次对象分配都必须要进行同步（虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性），而在竞争激烈的场合分配的效率又会进一步下降。
    
*   JVM 使用 TLAB 来**避免多线程冲突**，在给对象分配内存时，每个线程使用自己的 TLAB，这样可以避免线程同步，提高了对象分配的效率。
    
*   TLAB 本身**占用 Eden 区空间**，在开启 TLAB 的情况下，虚拟机会**为每个 Java 线程分配一块 TLAB 空间**。
    
*   参数 **-XX:+UseTLAB** 开启 TLAB，默认是开启的。
    
*   TLAB 空间的**内存非常小**，缺省情况下仅占有整个 **Eden 空间的 1%**，当然可以通过选项 - XX:TLABWasteTargetPercent 设置 TLAB 空间所占用 Eden 空间的百分比大小。
    
*   由于 TLAB 空间一般不会很大，因此**大对象无法在 TLAB 上进行分配**，总是会直接分配在堆上。TLAB 空间由于比较小，因此很容易装满。
    
*   示例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1LInibSdSOfuQxAqXsH7hZlByKJb0dbkHwAugqOYQibW4SWO45QnoDCcw/640?wx_fmt=png)【解释】

*   -XX:+UseTLAB
    

开启 TLAB，默认为开启。

*   -XX:+PrintTLAB
    

打开 TLAB 跟踪参数

*   -Xcomp（这里只是希望在相对一致的环境中测试）
    

JVM 在第一次使用时会把所有的字节码编译成本地代码，从而带来最大程度的优化。启用对所有函数的 JIT

*   -XX:-BackgroundCompilation （这里只是希望在相对一致的环境中测试）
    

禁止后台编译

*   -XX:-DoEscapeAnalysis
    

关闭逃逸分析

*   什么是 JIT？
    
    在部分商用虚拟机中（如 HotSpot），Java 程序最初是通过解释器（Interpreter）进行解释执行的，当虚拟机发现某个方法或代码块的运行特别频繁时，就会把这些代码认定为 “_热点代码_”。为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成与本地平台相关的机器码，并进行各种层次的优化，完成这个任务的编译器称为_即时编译器_（Just In Time Compiler，下文统称 JIT 编译器）。
    

### 3.4.3> 堆上分配

*   绝大多数对象，还是会分配在堆上的。
    

3.5> 逃逸分析  

------------

*   对于线程私有的对象，可以**分配在栈上**，⽽不是分配在堆上。好处是**⽅法执⾏完，对象⾃⾏销毁，不需要 gc 介⼊**。可以提⾼性能。
    
*   ⽽栈上分配的⼀个技术基础（如果**关闭逃逸分析**或**关闭标量替换**，那么⽆法将对象分配在栈上）就是逃逸分析。
    
*   逃逸分析的⽬的是判断**对象的作⽤域是否有可能逃逸出函数体**。如下所示：
    

```
Student student; // 属于逃逸了
public void say1() {
  student = new Student();
}
public void say2() {
  Student student = new Student(); // 没有逃逸
}

```

【注意】

*   对于 say2() ⽅法中的 new Student()，jvm 就有可能将 Student 分配在栈上，⽽不是堆上。
    
*   对于⼤量的零散⼩对象，栈上分配提供了⼀种很好的对象分配优化策略。
    
*   对于⼤对象，⽆法也不适合在栈上分配。
    

3.6> 标量替换  

------------

*   **标量**
    
    即**不可被进一步分解的量**，——JAVA 的基本数据类型就是标量（如：int，long 等基本数据类型以及 reference 类型等）
    
*   **聚合量**
    
    标量的对立就是**可以被进一步分解的量**。——JAVA 中对象就是可以被进一步分解的聚合量。
    
*   **替换过程**
    
    条件 1> 通过逃逸分析确定该对象不会被外部访问。
    
    条件 2> 对象可以被进一步分解，即聚合量。
    
    JVM 不会创建该对象，而会将该对象成员变量**分****解若干个被这个方法使用的成员变量所代替**。
    
    这些代替的成员变量在**栈帧**或**寄存器**上分配空间。
    

3.7> 垃圾回收算法  

--------------

*   主要的垃圾回收算法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1T1sg3MkzVLazkWW5dBedouJniavV20OiaxounteHEM01eNQRC2MnYuRQ/640?wx_fmt=png)

### 3.7.1> 引用计数法  

*   引用计数法是最经典也是**最古老**的垃圾收集方法，但是由于其固有的**循环引用和性能问题**，所以 JVM 并未选择此算法作为垃圾收集器算法。
    

*   对于一个对象 A，只要有任何一个对象引用了 A，则 A 的引用计数器就加 1，当引用失效时，引用计数器就减 1. 只要对象 A 的引用计数器的值为 0，则对象 A 就不可能再被使用。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd15ia2nUozrsvQlDRDkeFFydryeQuvFiaW7EBv2Ocy4kwp3vZItj9YnaBQ/640?wx_fmt=png)

*   但引用计数器有两个严重问题：
    

*   1> 无法处理循环引用的情况。
    

    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1xK6XyCHWvjg5x5sPOgv8icGkhTnm9tt17y1RNxCiciaRTMMYHWzV5cTFQ/640?wx_fmt=png)

*   2> 引用计数器要求在每次因引用产生和消除的时候，需要伴随一个加减法操作，对系统性能会有一定的影响。因此：JVM 并未选择此算法作为垃圾回收算法。
    

### 3.7.2> 标记清除法  

*   标记清除算法是现代垃圾回收算法的思想基础。分为两个阶段：标记阶段和清除阶段。标记清除算法产生最大的问题就是清除之后的空间碎片。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ibetvHML6yfXP0PvbBiaRsIMmDE9heIhKt8WLFHluRXx71ic0suN4urZQ/640?wx_fmt=png)

*   优点：
    
    实现简单，与保守式 GC 算法相兼容（由于保守式 GC 算法中，对象是不能被移动的。所以，适用于标记 - 清除算法。）
    
*   缺点：
    
    内存空间碎片化。
    
    由于分块不是连续的，因此每次分配都必须遍历空闲链表，找到足够大的分块。
    
    如果分配的是大的对象，最糟的情况就是得把空闲链表遍历到最后。
    
    标记和清除过程的效率都不高。
    
*   总结原理：
    
    它是最基础的 GC 算法，后续的 GC 算法都是针对它的缺点进行改良而产生的。JVM 回收器中的 CMS 就是使用的该算法
    
*   保守式 GC
    
    简单来说，保守式 GC(Conservative GC) 指的是 “**不能识别指针和非指针的 GC**”。
    
*   【优点】
    

*   1> 语言处理程序不依赖于 GC
    

保守式 GC 的优点在于容易编写语言处理程序。处理程序基本上不用在意 GC 就可以编写代码。语言处理程序的实现者即使没有意识到 GC 的存在，程序也会自己回收垃圾。因此语言处理程序的实现要比准确式 GC 简单。

*   【缺点】
    

*   1> 识别指针和非指针需要付出成本
    
*   2> 错误识别指针会压迫堆。当存在貌似指针的非指针时，保守式 GC 会把被引用的对象错误识别为活动对象。如果这个对象存在大量的子对象，那么它们一律都会被看成活动对象。因为程序把已经死了的非活动对象看成了活动对象，所以垃圾对象会严重压迫堆。
    
*   3> 能够使用的 GC 算法有限
    

### 3.7.3> 复制算法  

*   将原有内存空间分为两块。每次只使用其中一块内存，例如：A 内存，GC 时将存活的对象复制到 B 内存中。然后清除掉 A 内存所有对象。开始使用 B 内存。复制算法没有内存碎片，并且如果垃圾对象很多，那么这种算法效率很高。但是它的缺点是系统内存只能使用 1/2。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1JxGwxvaibMXl4TkxBI0BPtAmsNx9GKtOTJov472XM9Rx9OHHwqiadkGg/640?wx_fmt=png)

*   为了**解决效率问题**，复制算法应运而生。
    
*   优点：
    

执行效率很高。

可以保证回收后的内存空间没有碎片。

*   缺点：
    
    内存空间只能使用 1/2
    
*   总结原理：
    
    因为 90% 以上的新生代对象生命周期都很短暂，并且 GC 在新生代回收的特点就是频率高，耗时低，所以：针对以上特点，JVM 垃圾收集器都采用复制算法来回收新生代。
    
*   复制算法在 JVM 堆中的应用
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1KR50TyVM8W0uWWqEDIVrAThux3OgmkgcgiaJYcfTRicS8KQwSHiaUC2nw/640?wx_fmt=png)

*   由于 Eden 区于 S0 和 S1 比例默认是 8:1:1，新生代的空间 = Eden 区 + S0/S1=90%，那么浪费的空间也只有 10% 而已。
    
*   设置 **Eden 区与 Survivior 区比例**的 jvm 参数
    
    **-XX:SurvivorRatio**
    
*   设置 **BigObject** 的 jvm 参数
    
    **-XX:PretenureSizeThreshold**
    
*   设置 **OldObject** 的 jvm 参数
    
    **-XX:MaxTenuringThreshold**  每次 Minor GC，年龄加一岁。tenure：任期。
    
*   查看 JVM 某个参数的值：
    

```
muse@muse:/Users/muse/Desktop> jinfo -flag SurvivorRatio 11303                                   
-XX:SurvivorRatio=8
muse@muse:/Users/muse/Desktop> jinfo -flag PretenureSizeThreshold 11303                         
-XX:PretenureSizeThreshold=0 // 默认值是0，意思是不管多大都是先在eden中分配内存
muse@muse:/Users/muse/Desktop> jinfo -flag MaxTenuringThreshold 11303                      
-XX:MaxTenuringThreshold=15

```

### 3.7.4> 标记压缩算法  

*   标记压缩算法是一种老年代的回收算法。它首先标记存活的对象，然后将所有存活的对象压缩到内存的一端，然后在清理所有存活对象之外的空间。该算法不会产生内存碎片，并且也不用将内存一分为二。因此，其性价比比较高。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1YH7rGO0oZab3SAwHAedI6LkXWkhtOicPzlSJm8IR86fcibcEatIicEnlg/640?wx_fmt=png)

*   由于老年代的对象存活率很高，不容易被消亡，而复制算法不仅存在空间浪费，而且当老年代对象很多的时候，复制对象的效率会非常的低，所以，基于**老年代**的特性**，**产生了标记压缩算法。
    
*   它在**标记清除算法**的基础上做了优化，和标记清除算法一样，也是首先需要从根节点开始，对所有可达对象做一次标记。但之后，它并不是简单地清理未标记的对象，而是**将所有的存活对象压缩到内存的一端**，然后，清理边界外所有的空间。这种方法即避免了碎片的产生，又不需要两块相同的内存空间，因此，性价比很高。
    

### 3.7.5> 分代算法  

*   将堆空间划分为新生代和老年代，根据它们直接的不同特点，执行不同的回收算法，提升回收效率。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1k7ScbdicgPHyH4Ct4rXuXKvbWJQsBh8mJdvjtvU1WCCeiaodcPD9Ir2Q/640?wx_fmt=png)

*   当前 jvm 的垃圾回收，都是采用分代收集算法
    
*   针对**新生代**由于 GC 都有大量对象死去被回收，少数存量对象，只需要复制少量对象，就可以完全清除 S0/S1 的垃圾对象空间。所以采用 **“****复制算法****”** 更为合适；
    
*   而**老年代**对象存活率高，每次 GC 只清除少部分对象，所以采用 **“****标记 - 清除****”** 和 **“****标记 - 压缩****”** 算法来回收。
    

### 3.7.6> 分区算法

*   将堆空间划分成连续的不同小区间，每个区间独立使用、回收。由于当堆空间大时，一次 GC 的时间会非常耗时，那么可以控制每次回收多少个小区间，而不是整个堆空间，从而减少一次 GC 所产生的停顿。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1iaMtdvDo8ItLymlqickOfZia2BB8gRI9hkG3qkaKNqicbt1b9l90JTjshw/640?wx_fmt=png)

四、垃圾收集器  

==========

4.1> 串行回收器
----------

*   串行回收器也叫 **Serial 收集器**，是**最古老**收集器。
    
*   它在 JDK1.3 之前是虚拟机**新生代收集器**的唯一选择。在 client 模式下，默认是新生代收集器。
    
*   它是**单线程执行**回收操作的。它的特点就是，在单核或内核少的计算机来说，有更好的性能表现。它的优点就是**简单高效**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1xF9rCjyPrYEITPtMhswYkpcicdgiaPKu1GS92Ab5vZhHZWOa2icPzyXzQ/640?wx_fmt=png)

*   串行回收器的特点：
    
    只使用单线程进行 GC
    
    独占式的 GC
    
*   配置 JVM 参数启动指定的垃圾收集器
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1PKpmDxn8CGnXNPM91DvNElLsVEp6vdR8iczKk3StibpNtDiaxQWpxF7Tg/640?wx_fmt=png)

*   使用 **-XX:+UseSerialGC** 可以指定**新生代和老年代都是 Serial 收集器****。**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1KSbaNQXbN1ibvjBls59HTUH2PWyeeL4eN5mwzy3sy16Cqicze1QQhqyw/640?wx_fmt=png)

*   如何查看当前虚拟机使用什么垃圾收集器呢？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd12BYqRnahrzP8baWSUVKDx6po7rXiaIM6JqEtNyBYrbfliar3KF6FL56Q/640?wx_fmt=png)

【解释】  

*   **-XX:+PrintCommandLineFlags** 打印虚拟机显式和隐式参数
    

*   垃圾回收器算法的选择
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd13NbWUnYAhkHEKEGI78dT7ggaksstnsCRI5ic30kEuCORA7FmKabncLw/640?wx_fmt=png)

【解释】  

*   **新生代**：采用**复制算法**
    
*   **老年代**：采用**标记压缩算法**
    

*   查看垃圾回收情况，年轻代
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1hdJVCicZEToGEfeZ3J62wt1AS4N0L8ibicHSFCpDdl9xk9GzRLLLtdU0w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1EiaUiazNuUO4463SVeavJ17THkhtJrNN91T0tXWvjWCgWgegRT9R7icEw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1fE1eo3Pic28oFGiaEJFibzIZNSM1Z5V4r8ib3fVvH3PtWsibq5V9FYM36Hw/640?wx_fmt=png)  

4.2> 并行回收器  

-------------

*   将串行回收器多线程化。与串行回收器有相同的回收策略、算法、参数。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1DZP3LTPRDia8yPQadgS4PlLzqPgz3icFabXOM1gpImr1APVgWuBpE1yw/640?wx_fmt=png)

*   配置 JVM 参数启动指定的垃圾收集器
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1h6H6fW0caNrZ4EfBHSDCjyPop609S4PkjEEMvMLTH6dvEXYBanF0Wg/640?wx_fmt=png)

### 4.2.1> ParNew 回收器  

*   **-XX:+UseParNewGC**
    
    新生代：使用 ParNew 回收器
    
    老年代：使用 Serial 串行回收器
    
*   **-XX:+UseConcMarkSweepGC**
    
    新生代：使用 ParNew 回收器
    
    老年代：使用 CMS 回收器
    
*   是一个**新生代**的回收器，也是一个**独占式**的回收器，它与串行回收器唯一不同的，就是它采取**并发方式执行** GC。
    
*   大家一定要注意一点，就是在 cpu 核数少的机器，它的性能很可能比串行回收器差。
    
*   **-XX:ParallelGCThreads** 指定并行 GC 的线程个数，最好与 CPU 个数一致，否则会影响垃圾回收性能。
    
    默认情况下，当 CPU 数量少于等于 8 个的时候，并行线程数为 **8 个**。如果 CPU 数量大于 8，并行线程数量为 **3+(5*cpu_nums/8)**
    
*   操作示例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1oyRgokCe7dyXS53aktq0XwPYjeTkHRkyzmicgtpwDWut8icOyZ6icQFicg/640?wx_fmt=png)

### 4.2.2> ParallelGC 回收器（关注吞吐量）  

*   **-XX:+UseParallelGC**
    
    新生代：使用 ParallelGC 回收器
    
    老年代：使用 Serial 串行回收器
    
*   **-XX:+UseParallelOldGC**
    
    新生代：使用 ParallelGC 回收器
    
    老年代：使用 ParallelOldGC 收器
    
*   查询是否使用了该回收器
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1WZicXPepGKa3X9JZpBwPBeKakUwMLGwH9JZTj7cdOnLLmejHapT6iagg/640?wx_fmt=png)

*   它也是**新生代**的回收器，也采用的**复制算法**执行 GC 回收任务。它与 ParNew 有一个不同点就是，它提**供了一些设置系统吞吐量的参数**用来控制 GC 行为。
    
*   **-XX:MaxGCPauseMillis 最大的垃圾收集暂停时间**，它是一个大于 0 的整数，ParallelGC 会根据设置的值来调整堆的大小和其他 jvm 参数，使其把 GC 停顿时间控制在 MaxGCPauseMillis 之内，但是，大家要注意，如果将值设置很小，虽然停顿时间小了，却造成初始化的堆也变小了，垃圾回收会变得很频繁。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1IIrmYiaH2SakpL33fgfuU1omPv0lQSkBGryib0lGk3eXXpLKl1Xg0qIw/640?wx_fmt=png)

*   **-XX:GCTimeRatio 设置吞吐量大小**，可设置的值为 **0~100 之间的整数**。什么意思呢？就是说它影响的是垃圾回收时间，通过 1/(1+n) 来计算，假设 n=99，那么 1/(1+99)=1%，也就是说，系统会花费**小于 1% 的时间**用于垃圾回收。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd153ItCyvfuJoFXRTjaw9p72hicfn9IKj3ib3icYwu2jjuxUVMfuLFb55qA/640?wx_fmt=png)

*   **-XX:+UseAdaptiveSizePolicy** 如果你不倾向手动设置上面的参数，可以采用把**参数调整交由虚拟机自动设置**。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1AZhemdT630xleCCib01IibdRomtnh6bYFsicxSJ6qrL73cdmM873HR7aQ/640?wx_fmt=png)

### 4.2.3> ParallelOldGC 回收器（关注吞吐量）  

*   **-XX:+UseParallelOldGC**
    
    新生代：使用 ParallelGC 回收器
    
    老年代：使用 ParallelOldGC 收器
    
*   查询是否使用了该回收器
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1oHAmUCWJ5Qp2krdNcsBke1KC3W9AzULaicEjBCTPicRFiaJY79HR26Libg/640?wx_fmt=png)

*   它跟 ParallelGC 相似，也是**关注于吞吐量**的收集器。
    
*   从名字看，它比 ParallelGC 多了个 **Old**，其实就表示它是一个应用于**老年代**的回收器。
    
*   可以与 ParallelGC 搭配使用，即：**ParallelGC（新生代收集器）+ ParallelOldGC（老年代收集器）**。
    
*   它采用**标记压缩算法**进行 GC 操作。也可以使用 **-XX:ParallelGCThreads** 来指定并行 GC 的线程个数。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1hojNdrCY96bSpdFiamX1Fia0Aa4GGKdnTZb6z2NIeaJFNmLNCSoluQpg/640?wx_fmt=png)

### 4.2.4> CMS 回收器（关心系统的停顿时间）  

*   CMS 垃圾回收步骤
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ohT7uWd1PeA4s3DbaMniaLZ7vpMWmuP6YuOAERvqdhq3bxLqmvC49lg/640?wx_fmt=png)

*   **-XX:+UseConcMarkSweepGC**
    
    新生代使用 ParNew 回收器，老年代使用 CMS 回收器。
    
*   它的特点是，会**关心系统的停顿时间**。
    
*   CMS 全称为 Concurrent Mark Sweep，即：**并发标记清除**。它采用的是**标记清除算法**。也是多线程并发执行器。分为如下六个步骤：
    

*   1> **初始标记****（STW）**
    

    标记**根对象**

*   2> **并发标记**
    

    标记**所有对象**

*   3> **预清理**
    

    清理前的**准备**以及**控制停顿时间**（可以采用 **-XX:-CMSPrecleaningEnabled** 关闭，不进行预清理）

为什么要有预清理？

因为第 4 步**重新标记**是独占 CPU 的，如果 YoungGC 发生后，立即触发一次重新标记，那么一次停顿时间可能很长，为了避免这种情况，预处理时，会刻意等待一次新生代 GC 的发生，然后根据历史数据**预测**下一次 YoungGC 的时间，在当前时间和预测时间取**中间时刻**执行重新标记操作，目的就是**尽量避免 YoungGC 与重新标记重叠执行**。从而减少一次停顿时间。

*   4> **重新标记****（STW）**
    

 **修正并发标记**数据

*   5> **并发清理**
    

    清理垃圾（**真正的执行垃圾回收**）

*   6> **并发重置**
    

    重置状态等待下次 CMS 的触发

*   我们可以使用 **-XX:+UseConcMarkSweepGC** 来启用 CMS
    
*   那么由于它是多线程回收器，我们可以通过 **-XX:ConcGCThreads** 和 **-XX:PartallelCMSThreads** 设置并发线程数量
    
*   也可以通过 **-XX:CMSInitiatingOccupancyFraction** 来设置当老年代空间使用量达到某百分比时，会执行 CMS。默认 68，也就是老年代使用空间达到 68% 的时候，会执行一次 CMS 回收。
    
*   由于 CMS 采用的是标记清除算法，所以不可避免的就是内存碎片，那么我们可以通过如下两个参数进行解决：
    

*   **-XX:+UseCMSCompactAtFullCollection** 指定 GC 后，进行一次碎片整理
    
*   **-XX:CMSFullGCsBeforeCompaction** 指定执行多少次 GC 后，进行一次碎片整理
    

*   并行回收器 jvm 参数
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1SHV3pByHo7d3iah01hRYrrtU5AhfGBPCiaeQEJKKz9PNsviamcR7KrC6g/640?wx_fmt=png)

4.3> G1 回收器  

--------------

### 4.3.1> 概述

*   G1 回收器是 **JDK1.7** 正式使用的回收器，它的目标是来**取代 CMS 回收器**。
    
*   它属于**分代回收器**，也使用了**分区算法**。
    
*   G1 全称 Garbage First Garbage Collector。优先回收垃圾比例最高的区域。G1 收集器将堆划分为多个区域，每次收集部分区域来减少 GC 产生的停顿时间。
    
*   **-XX:+UseG1GC**
    
    标记打开 G1 收集器开关
    
*   它的优点有如下几个方面：
    

*   1> 它是**多个线程同时执行** GC 操作的，可以最大限度利用多 cpu 计算能力。
    
*   2> 虽然它在**初始标记**、**重新标记**和**独占清理**这三个阶段需要 STW。但是，对于整体 GC 过程来说，是可以与应用程序并行执行的。
    
*   3> 它即可以负责年轻代的 GC，也可以负责老年代的 GC。实现了一个回收器**对多代的完全统治**。
    
*   4> G1 不是采用 CMS 这种标记清除算法，它每次回收都会有效地复制对象，从而减少内存碎片。
    
*   5> G1 **支持分区算法**来执行垃圾回收，采取**局部回收**操作，它对内存进行了划分区域，每次 GC 收集只针对其中几个区域，极大减少由 GC 导致的停顿时间。
    

### 4.3.2> G1 的收集过程  

*   G1 的收集过程分为 4 个阶段：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ibXZMWibibcEtXkdr2B8IKcv0REOEP9VdRsaribBA1w6WPV2avmDBjicricQ/640?wx_fmt=png)

#### 阶段 1：新生代 GC

*   新生代 GC 的主要工作就是**回收 eden 区**和 **survivor 区**。
    
*   一旦 **eden 区被占满，新生代 GC 就会启动**。回收后，所有的 **eden 区都应该被清空**，而 survivor 区会被收集一部分数据，但是应该至少仍然存在一个 survivor 区。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1a7FXOIT6RoMlOeU8VeOeHLNicRGRsib3EJ6xvYhHNg7l9HYdic5VtgiboA/640?wx_fmt=png)

#### 阶段 2：并发标记周期 （过程与 CMS 很类似）  

*   整体流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1vuhAsuwqte1GIHH8VE8LI6ghzRmOBBibCrRByeYyzOmxhxHcNQ9qYtg/640?wx_fmt=png)

*   **初始标记（STW）**
    
    标记从**根节点**直接可到达的对象。
    
    这阶段会**伴随一次 Young GC**，会产生 STW（全局停顿），应用程序会停止执行。
    
*   **根区域扫描**
    
    由于 **Young GC** 的发生，所以初始标记后，**eden 被清空，存活对象放入 Survivor 区**。然后本阶段，则**扫描 survivor 区**，标记可直达老年代的对象。本阶段应用程序可以并行执行。但是，**根区域扫描不能和 YoungGC 同时执行**（因为根区域扫描依赖 survivor 区的对象，而新生代 GC 会修改这个区域），因此如果恰巧在此时需要进行 YoungGC，GC 就需要等待根区域扫描结束后才能进行，如果发生这种情况，这次 YoungGC 的时间就会延长。
    
*   **并发标记**
    
    用来**再次扫描整个堆的存活对象**，并做好标记。与 CMS 类似，该阶段可以被一次 Young GC 打断。
    
*   **重新标记（STW）**
    
    本阶段也会发生 STW，应用程序会停止执行。由于并发标记阶段中，应用程序也是并发执行的，所以本阶段，对标记结果进行**最后的修正**处理。
    
*   **独占清理（STW）**
    
    本阶段也会发生 STW，应用程序会停止执行。它用来计算各个区域的**存活对象**和 **GC 回收比例**，然后进行**排序，**从而识别出可以用来**混合收集**的区域。该阶段给出了需要被混合回收的区域并进行了标记，那么在混合收集阶段，是需要这些信息的。
    
*   **并发清理**
    
    本阶段会去识别并清理那些**完全空闲的区域**。
    

#### 阶段 3：混合收集

*   在第二步的并发标记周期过程中，虽然有部分对象被回收，但是总体回收比例还是比较低的。
    
*   由于 G1 已经明确知道哪些区域含有比较多的垃圾比例，所以就可以针对比例较高的区域进行回收操作。
    
*   G1 会**优先回收垃圾比例较高**的区域，因为这样性价比会比较高。
    
*   这个阶段叫作混合回收，是因为这段时期，**新生代和老年代 GC 都会同时进行**。
    
*   那么因为新生代 GC 后，eden 区必然被清空，此外被标记为垃圾比例最高的区域也被清理。
    
*   被清理区域中**存活对象就会被移动到其他的区域**，这样的好处就是可以减少空间碎片。
    

#### 阶段 4：Full GC（不是一定会执行，看情况来定）

*   由于 GC 回收过程，是与应用程序并发执行的，所以，如果在吞吐量很大的场景下，回收过程中内存不足，那么就会触发一次 Full GC。
    

4.4> GC 设置相关的 JVM 参数汇总  

-------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ItjrgWhLXJRBMqeQibwvNUibaaeGhuKfvcHcJMMgVDH3ezdUJxRW48Sg/640?wx_fmt=png)

五、常用的 JVM 参数  

===============

5.1> 垃圾回收日志
-----------

*   打印 GC 日志详细信息
    
    -XX:+PrintGCDetails
    
*   打印更全面的堆信息（会在每次 GC 前后分别打印堆的信息）
    
    -XX:+PrintHeapAtGC
    
*   每次 GC 发生时，额外输出 GC 发生的时间（该时间为虚拟机启动后的时间偏移量）
    
    -XX:+PrintGCTimeStamps
    
*   打印应用程序的执行时间
    
    -XX:+PrintGCApplicationConcurrentTime
    
*   打印应用程序由于 GC 而产生的停顿时间
    
    -XX:+PrintGCApplicationStoppedTime
    
*   跟踪系统内的软引用、弱引用、虚引用和 Finallize 队列
    
    -XX:+PrintReferenceGC
    
*   虚拟机允许将 GC 日志以文件的形式输出（在当前目录下的 log 文件夹中生成 gc.log 日志文件）
    
    -Xloggc:log/gc.log
    
*   打印虚拟机接受到的命令行显式参数
    
    -XX:+PrintVMOptions
    
*   打印虚拟机接受到的命令行显式和隐式参数
    
    -XX:+PrintCommandLineFlags
    
*   打印所有的系统参数值
    
    -XX:+PrintFlagsFinal
    

5.2> 类加载、类卸载的跟踪  

------------------

*   跟踪类的加载和卸载
    
    -verbose:class
    
*   跟踪类的加载
    
    -XX:+TraceClassLoading
    
*   跟踪类的卸载
    
    -XX:+TraceClassUnLoad
    
*   查看当前系统中占用空间最多的对象（在 Java 控制台按下 Ctrl+Break 组合键）
    
    -XX:+PrintClassHistogram
    

5.3> 配置 JMM 的参数  

------------------

*   堆和永久代的分配参数示意图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1viaCyQLrAYnsJibztEIgfV6XCZjqCNic96ibc0jpJcxbegFjo3jgq0aX2A/640?wx_fmt=png)

### 5.3.1> 堆配置  

*   设置初始堆空间
    
    -Xms30m
    
*   设置最大堆空间
    
    -Xmx30m
    
*   设置新生代的大小
    
    -Xmn10m
    
*   设置新生代 eden 和 from/to 空间的比例
    
    -XX:SurvivorRatio=8
    
*   设置老年代和新生代的比例（-XX:NewRatio = 老年代 / 新生代）
    
    -XX:NewRatio=2
    

### 5.3.2> 方法区配置  

*   设置初始永久代大小
    
    -XX:PermSize=5m
    
*   设置最大永久代大小（默认情况下为 64MB）
    
    -XX:MaxPermSize=5m
    
*   设置最大元数据空间
    
    -XX:MaxMetaspaceSize=20m
    

### 5.3.3> 栈配置  

*   指定栈的大小
    
    -Xss20m
    

### 5.3.4> 直接内存配置  

*   最大可用直接内存（默认为最大堆空间，即：-Xmx）
    
    -XX:MaxDirectMemorySize=200m
    

5.4> 堆溢出处理  

-------------

*   内存溢出时，导出整个堆的信息
    
    -XX:+HeapDumpOnOutOfMemoryError
    
    -XX:HeapDumpPath=/home/muse/logs/a.dump
    

六、性能监控工具  

===========

6.1> Linux
----------

### 6.1.1> top 指令

*   能够实时显示系统中各个进程的资源占用情况。分为两部分：**系统统计信息** & **进程信息**。
    
*   系统统计信息，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1B3A6ZhMuSWVHVFuPaibUvEHVyCJV0rzk4GhzzZa6qUeWO7pHAH7Iia8g/640?wx_fmt=png)

【解释】  

*   Line1：任务队列信息，从左到右依次表示：系统当前时间、系统运行时间、当前登录用户数。Load average 表示系统的平均负载，即任务队列的平均长度——**1 分钟**、**5 分钟**、**15 分钟**到现在的平均值。
    
*   Line2：进程统计信息，分别是：正在运行进程数、睡眠进程数、停止的进程数、僵尸进程数。
    
*   Line3：CPU 统计信息。us 表示用户空间 CPU 占用率、sy 表示内核空间 CPU 占用率、ni 表示用户进程空间改变过优先级的进程 CPU 占用率。id 表示空闲 CPU 占用率、wa 表示待输入输出的 CPU 时间百分比、hi 表示硬件中断请求、si 表示软件中断请求。
    
*   Line4：内存统计信息。从左到右依次表示：物理内存总量、已使用的物理内存、空闲物理内存、内核缓冲使用量。
    
*   Line5：从左到右表示：交换区总量、已使用交换区大小、空闲交换区大小、缓冲交换区大小。
    

*   进程信息，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1uPjLGPcEu2ApxibhcyibYib57xQay2WMtmQ02Tbgem3HZd6XpPvuaJPzQ/640?wx_fmt=png)

【解释】  

*   PID：进程 id
    
*   USER：进程所有者
    
*   PR：优先级
    
*   NI：nice 值，负值 -> 高优先级，正值 -> 低优先级
    
*   VIRT：进程使用虚拟内存总量 VIRT=SWAP+RES
    
*   RES：进程使用并未被换出的内存。CODE+DATA
    
*   SHR：共享内存大小
    
*   S：进程状态。
    
*   D = 不可中断的睡眠状态 R = 运行                    
    
*   S = 睡眠 T = 跟踪 / 停止 Z = 僵尸进程
    
*   %CPU：上次更新到现在的 CPU 时间占用百分比
    
*   %MEM：进程使用的物理内存百分比
    
*   TIME+：进程使用的 CPU 时间总计，单位 1/100 秒
    
*   COMMAND：命令行
    

### 6.1.2> vmstat 指令

*   性能监测工具，显示单位均为 kb。它可以统计 CPU、内存使用情况、swap 使用情况等信息，也可以指定采样周期和采用次数。例如：每秒采样一次，共计 3 次。**vmstat 1 3**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1nt5esiaye0aJQEZXwXKQVycmR2MQ8iaVGguQlTWL4y2Kzib7jSia6Rt9BA/640?wx_fmt=png)

【解释】  

*   procs 列
    

r 表示等待运行的进程数。

b 表示处于非中断睡眠状态的进程数。

*   memory 列
    

swpd 表示虚拟内存使用情况。

free 表示空闲内存量。

buff 表示被用来作为缓存的内存。

*   swap 列
    

si 表示从磁盘交换到内存的交换页数量。

so 表示从内存交换到磁盘的交换页数量。

*   io 列
    

bi 表示发送到块设备的块数，单位：块 / 秒。

bo 表示从块设备接收到的块数。

*   system 列
    

in 表示每秒的中断数，包括时钟中断。

cs 表示每秒的上下文切换次数。

*   cpu 列
    

us 表示用户 cpu 使用时间。

sy 表示内核 cpu 系统使用时间。

id 表示空闲时间。           

wa 表示等待 io 时间。

### 6.1.3> iostat 指令  

*   可以提供详尽的 I/O 信息。如果只看磁盘信息，可以使用 - d 参数。即：**Iostat –d 1 3** （每 1 秒采集一次持续 3 次）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1Weuo2RNolTlYysQWMCPjdhZNnib1sG1enB08VEHfE6vQjwF7fIjKiciaw/640?wx_fmt=png)

【解释】  

*   tps 列表示该设备每秒的传输次数。
    
*   Blk_read/s 列表示每秒读取块数。
    
*   Blk_wrtn/s 列表示每秒写入块数。
    
*   Blk_read 列表示读取块数总量。
    
*   Blk_wrtn 列表示写入块数总量。
    

6.2> JDK 自带工具  

----------------

### 6.2.1> jps 列出 Java 的进程

*   执行语法：**jps  [-options]** 
    

*   **jps** 列出 java 进程 id 和类名      
    

91275 FireIOTest

*   **jps -q** 仅列出 java 进程 id      
    

91275

*   **jps -m** 输出 java 进程的入参      
    

91730 FireIOTest a b

*   **jps -l** 输出主函数的完整路径      
    

91730 day1.FireIOTest

*   **jps -v** 显示传递给 JVM 的参数      
    

```
91730 FireIOTest -Xmx512m -XX:+PrintGC -javaagent:/Applications/IntelliJ     IDEA.app/Contents/lib/idea_rt.jar=51673:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8

```

### 6.2.2> jstat 查看堆中的运行信息  

*   执行语法：**jstat <-option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]**
    

*   **jstat -class -t 73608 1000 5**     查看进程 73608 的 ClassLoader 相关信息，每 1000 毫秒打印 1 次，一共打印 5 次，并输出程序启动到此刻的 Timestamp 数。
    
*   **jstat -compiler -t 73608** 查看指定进程的**编译信息**。
    
*   **jstat -gc 73608** 查看指定进程的**堆信息**。
    
*   **jstat -gccapacity 73608** 查看指定进程中**每个代的容量与使用情况**
    
*   **jstat -gccause 73608** 显示**最近一次 gc** 信息
    
*   **jstat -gcmetacapacity 73608** 查看指定进程的**元空间**使用信息
    
*   **jstat -gcnew 73608** 查看指定进程的**新生代**使用信息
    
*   **jstat -gcnewcapacity 73608** 查看指定进程的**新生代各区**大小信息
    
*   **jstat -gcold 73608** 查看指定进程的**老年代**使用信息
    
*   **jstat -gcoldcapacity 73608** 查看指定进程的**老年代各区**大小信息
    
*   **jstat -gcutil 73608** 查看指定进程的 **GC 回收**信息
    
*   **jstat -printcompilation 73608** 查看指定进程的 **JIT 编译方法统计信息**
    

### 6.2.3> jinfo 查看和设置运行中 java 进程的虚拟机参数  

*   执行语法： **jinfo [option] <pid>**
    

*   **jinfo -flag MaxTenuringThreshold 73608**  查看进程 73608 的虚拟机参数 MaxTenuringThreshold 的值
    
*   **jinfo -flag +PrintGCDetails 73608**     动态添加进程 73608 的虚拟机参数 + PrintGCDetails，开启 GC 日志打印
    
*   **jinfo -flag -PrintGCDetails 73608**     动态去除进程 73608 的虚拟机参数 - PrintGCDetails，关闭 GC 日志打印
    

### 6.2.4> jmap 用于生成指定 java 进程的 dump 文件  

*   可以查看堆内对象实例的统计信息，查看 ClassLoader 信息和 finalizer 队列信息。
    
*   执行语法： **jmap [option] <pid>**
    

*   **jmap -histo 73608 > /Users/muse/a.txt**   输出进程 73608 的**实例个数**与合计到文件 a.txt 中
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1ziaca8NXvGXmrjjuSPj5KU4IDpeKhgjuTO4hiakqYjpxldM48EP07fag/640?wx_fmt=png)

*   **jmap -dump:format=b,file=/Users/muse/b.hprof 73608**  输出进程 73608 的堆快照，可使用 jhat、visual VM 等进行分析
    

### 6.2.5> jhat 用于分析 jmap 生成的堆快照  

*   命令用于分析 jmap 生成的堆快照。
    
*   执行语法： **jhat [-stack <bool>] [-refs <bool>] [-port <port>] [-baseline <file>] [-debug <int>] [-version] [-h|-help] <file>**
    

**jhat b.hprof**      

*   分析 jmap 生成的堆快照 b.hprof，http://127.0.0.1:7000 通过这个地址查看。OQL（Object Query Language）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1d0RNskAxtt2EBd61AfsME3RjeFeYaUhUbqabqeA12rstoXMnJ6aW8w/640?wx_fmt=png)

### 6.2.6> jstack 用于导出指定 java 进程的堆栈信息  

*   执行语法： **jstack [-l] <pid>**
    
*   jstack -l 73608 > /Users/muse/d.txt    输出进程 73608 的实例个数与合计到文件 d.txt 中
    
*   cat /Users/muse/d.txt
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1a4JqMhlxzVibYL8qRh6t7gRtpgXtbSHZdWT3tpFml6B3s7KovNtzfdA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9vcLXgic6zUTV9lWEgibIOd1by3QYS4xydvM9Vibib9v92WiblTQDSHOjdXicxhfTfAXvcaCbPsJ5Qr9dg/640?wx_fmt=png)