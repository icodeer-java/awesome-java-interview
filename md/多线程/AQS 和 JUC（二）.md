
    本篇文章主要介绍 AQS 和 JUC 这两部分。因为 JUC 底层大量的在使用 AQS，所以，这两部分将作为整体来进行介绍。下面是本篇文章的提纲，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbibqfP3YmjBichVvvPLGqgdMXEkvq4nBAXrV21Lzf2hp7rm7wiaLkqXiabw/640?wx_fmt=png)

一、ReentrantLock 重入锁
===================

1.1> 概述
-------

*   重入锁可以完全替代 synchronized 关键字。在 JDK5.0 的早期版本中，重入锁的性能远远好于 synchronized，但从 JDK6.0 开始，JDK 在 synchronized 上做了大量的优化，使得两者的性能差距并不大。重入锁对逻辑控制的灵活性要远远好于 synchronized。
    
*   重入锁常用方法
    
    **void** **lock()**：获得锁，如果锁已经被占用，则等待。
    
    **void** **lockInterruptibly()**：获得锁，但优先响应中断。
    
    **boolean** **tryLock()**：尝试获得锁，如果成功，返回 true；如果失败则返回 false；获得不到锁，则不进行等待，立即返回。
    
    **boolean** **tryLock(long time, TimeUnit unit)**：在给定时间内尝试获得锁。
    
    **boolean** **isHeldByCurrentThread()**：判断担心线程是否持有锁。
    
    **void** **unlock()**：释放锁。
    
*   下面是使用重入锁的简单示例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbT6bAq90u1A9rCaNepB83I2oPYwUFJpLP2cibH50ZF1Fs3kJXzVobfNw/640?wx_fmt=png)

*   之所以称之为**重入锁**，就是一个线程允许反复进入。当然，这里的反复仅仅局限于一个线程；如果同一个线程多次获锁，那么在释放锁的时候，也必须释放相同次数。如果释放锁的次数多，那么会得到一个 **java.lang.IllegalMonitorStateException** 异常，反之，如果释放锁的次数少，那么相当于线程还持有这个锁。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbxibv7FRHTLicop2ibH3IzibUpLRTfsEQia6w5KNPzwtQnbHibwZm38GP0rbA/640?wx_fmt=png)

1.2> 中断响应 lockInterruptibly()  

--------------------------------

*   如果使用 synchronized，要么获得锁，要么保持等待。而如果使用了重入锁，则提供了另一种可能，那就是线程可以被中断。也就是在等待锁的过程中，程序可以根据需要取消对锁的请求。即：**如果一个线程正在等待锁，那么它依然可以收到一个通知，被告知无须再等待，可以停止工作了**。可以很好的应对死锁问题。示例如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbrxa0TQUiaVNaGK7FO2P6ibHj5kIplT2CnX4pBQ3DNicqxibFDZrflsy7Rw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbNs97HImkFicXnI0HIKBwI4UTiaugfa15GRueibwItziac6ER9EYVI4lFuw/640?wx_fmt=png)

1.3> 锁申请等待限时 tryLock(long time, TimeUnit unit)
----------------------------------------------

*   除了等待外部通知之外，要避免死锁还有另外一种方式，就是**限时等待**。以下面为例，线程尝试获得锁，如果没有获得锁，则等待 5 秒钟。如果 5 秒钟之后依然没有获得锁，则返回 false，表示获得锁失败。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbHkL2GLCyiaDJKiaAoHNwnvf3icSqMhb7Inx3FG79oroWDRxIQQgVG6tcw/640?wx_fmt=png)

*   tryLock() 方法也可以不带参数直接运行。在这种情况下，当前线程会尝试获得锁，如果锁并未被其他线程占用，则申请锁会成功，并立即返回 true。如果锁被其他线程占用，则**当前线程不会进行等待，而是立即返回 false**。这种模式不会引起线程等待，因此也不会产生死锁。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRb1X0KIsutH3Lc02GLhM0NSicTXCUzFHhIjF1pIibzOyNg5NjNg97NX19A/640?wx_fmt=png)

1.4> 公平锁和非公平锁  

----------------

*   在大多数情况下，锁的申请都是非公平锁。系统只是会从这个锁的等待线程中随机选择一个。类似大家买票不去排队，乱哄哄的围在售票窗口前，售票员忙得焦头烂额，也顾不及谁先谁后，随便找一个人出票就完事儿了。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbseW1gRCXdufYWF6ky3uTt3M4lwKNvbJETxSRjpIZThR2SIp5UfwhNA/640?wx_fmt=png)

*   当入参为 true 时，则采用公平锁方式。要求系统维护一个有序队列，因此公平锁的实现成本比较高，性能相对也非常低下。因此，默认情况下，锁是非公平的。如果没有特别的需求，也不需要使用公平锁。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRb3MffyCLpyrJYLiaRuSOwZE01BKfic54hibMf26B87gCL7S91JlpliaE8kw/640?wx_fmt=png)

1.5> AQS 源码解析  

----------------

*   [详情请见：_【源码解析】AbstractQueuedSynchronizer.pdf_（点击可跳转）](http://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247487138&idx=1&sn=1522585c311c7de703f1fe599fdc4077&chksm=e9114c5fde66c54986a0a87b1d89d9accafad30f8a5c99878320868b14557ea976d1031bc583&scene=21#wechat_redirect)
    

二、Condition 重入锁的搭配类  

======================

*   相同点
    
    Condition 和 Object 的 wait()、notify() 方法的作用是大致相同的，对应关系如下所示：
    

*   Condition.**await()**--->Object.**wait()**
    
*   Condition.**signal()**--->Object.**notify()**
    
*   Condition.**signalAll()**--->Object.**notifyAll()**
    

*   不同点
    
    Object.wait() 和 Object.notify() 方法是和 **Synchronized** 关键字合作使用的；
    
    Condition.await() 和 Condition.signal() 是与 **ReentrantLock** 相关联的。
    
*   Condition 常用方法
    

*   **void await()：**
    
    会使**当前线程等待**，同时**释放当前锁**，当其他线程中使用 signal() 或者 signalAll() 方法时，线程会重新获得锁并继续执行。或者当线程被中断是，也能跳出等待。这和 Object.wait() 方法很相似。
    
*   **void awaitUninterruptibly()：**
    
    与 await() 方法基本相同，区别是它**不会在等待过程中响应中断**（Uninterruptibly）。
    
*   **long awaitNanos(long nanosTimeout)：**
    
    如果 **nanosTimeout 时间内**，没有被执行 signal，则解除等待状态。
    
*   **boolean await(long time, TimeUnit unit)：**
    
    如果 **time 时间内**，没有被执行 signal，则解除等待状态。
    
*   **boolean awaitUntil(Date deadline)：**
    
    如果 **deadline 时间内**，没有被执行 signal，则解除等待状态。
    
*   **void signal()：**
    
    用于**唤醒一个**正在等待中的线程。
    
*   **void signalAll()：**
    
    用于**唤醒所有**正在等待中的线程。
    

*   Condition 使用示例:
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbWV64dwUAufs5jxLNRhuygHGjME91mPveckseR14HXCTYuvuDAS7icDg/640?wx_fmt=png)

三、Semaphore 信号量  

==================

*   广义上说，Semapore 信号量是对锁的一种扩展；因为无论是内部锁 synchronized，还是重入锁 ReentrantLock，一次都只允许一个线程访问某一资源，而信号量却可以**指定多个线程同时访问某一个资源**。
    
*   信息量主要提供了一下构造函数，必须要指定信号量的**准入数**，即：同时能申请多少个**许可**
    
    public **Semaphore(int permits)**; // permits：准入数
    
    public **Semaphore(int permits, boolean fair)**; // permits：准入数，fair：是否公平获得锁
    
*   信息量主要方法如下所示：
    

*   **public void acquire();**
    
    尝试获得一个准入的许可。若无法获得，则线程会**等待**，直到有线程释放一个许可或者当前线程被中断。
    
*   **public void acquireUninterruptibly();** 
    
    具有 acquire 一样的功能，但是**不响应中断**（Uninterruptibly）。
    
*   **public void tryAcquire();**             
    
    尝试获得一个许可，如果成功就**立即返回** true，失败则立即返回 false。
    
*   **public void tryAcquire(long timeout, TimeUnit unit);**
    
    在**指定时间内**，尝试获得一个许可，如果成功就返回 true，失败则返回 false。
    
*   **public void release();  **                
    
    资源访问结束后，**释放一个许可**。
    

*   申请了 4 个准入，循环 10 个线程，那么将会以 4 个线程一组为单位进行执行输出
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbCkhdGRCU4SzbDlLwsial2F9zrNvZ6tYCtbVfEnJAKQDS2AH4lKac45Q/640?wx_fmt=png)

四、ReadWriteLock 读写锁  

======================

*   ReadWriteLock 是 JDK5 中提供的读写分离锁。它允许多个线程同时读。但是考虑到数据的完整性，写写操作和读写操作间依然是需要相互等待和持有锁的。读写锁的访问约束情况如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRb2SiaKuS6qAuoATsuMo5AHLEicBfN2tt3WenbpIxa0ggarYgOjCdozhaQ/640?wx_fmt=png)

*   下面例子中，我们使用**普通锁**，执行读写操作：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbgAu7Lk3oeDk6BEQHSOJsXP6ckyOFKBjwF6GC9ic8jDdd3f3S7FYT0hw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbKwmwsRicRARSDrLh52hMjUJ5iaEVMMdzT3UHQsDwkyS2sJnAAWvldmFQ/640?wx_fmt=png)

*   下面例子中，我们使用**读写锁**（只需要将上面例子中 openRWLock=true 即可）执行读写操作，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbXsuW2QHntDTPk4uNXxVxxzEy4y61BORPhVlF7XH3M70aicBlw3fo7lg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbZfZYJOe9mCEMPnZomM7vBjibHgxRIDyjibhp262ChnJoq2bxFo9zZXAA/640?wx_fmt=png)

*   所以，如果在系统中，读操作的次数远远大于写操作，那么读写锁就可以发挥最大的效果，提升系统的性能。
    

五、CountDownLatch 倒计时器  

========================

*   CountDownLatch 是一个多线程控制工具。用来控制线程的等待。设置需要 countDown 的数量 num，然后每一个线程执行完毕后，调用 **countDown()** 方法，而主线程调用 **await()** 方法执行等待，直到 num 个子线程执行了 countDown() 方法 ，则主线程开始继续执行。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbYMvLETylEcxTN63MOeSNVCjpMUdib7gR5PYB7ettw4c7yONstt1gcvg/640?wx_fmt=png)

六、CyclicBarrier 循环栅栏  

=======================

*   CyclicBarrier 与 CountDownLatch 非常类似，它支持计数器的**反复使用**，CyclicBarrier 可以理解为循环栅栏。CyclicBarrier 可以接收一个参数作为 **Runnable barrierAction**，每当计数器一次计数完成后——**CyclicBarrier.await()** 时，系统会执行的动作。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbiao0YtkT2pycZktTXobPKnAgkBUx1rOicaIibCmu1QUk9iaQs8P2O97mIQ/640?wx_fmt=png)

*   CyclicBarrier.await() 方法可能会**抛出两种异常**：一个是 **InterruptedException**，也就是在等待过程中，线程被中断，应该说这是一个非常通用的异常，大部分迫使线程等待的方法都可能会抛出这个异常，使得线程在等待时依然可以响应外部紧急事件。另外一个异常则是 CyclicBarrier 特有的 **BrokenBarrierException**，一旦遇到这个异常，则表示当前的 CyclicBarrier 已经破损了，可能系统已经没有办法等待所有线程到齐了。如果继续等待，可能就是徒劳无功的，因此就此结束吧。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbZ0vFTPPEPicQc5xg21oPEbv3AwSS2Yz6K9icpRk9MZZAttaXicKNMuG1Q/640?wx_fmt=png)

七、LockSupport 线程阻塞工具类  

========================

*   LockSupport 是一个非常方便实用的**线程阻塞工具**，它可以在线程内**任意位置**让线程阻塞。和 Thread.suspend() 相比，它弥补了由于 resume() 在前发生，导致线程无法继续执行的情况。和 Object.wait() 方法相比，它**不需要先获得某个对象的锁**，也不会抛出 InterruptedException 异常。
    
*   park() 可以阻塞当前线程，其中每一个线程都有一个许可，该许可**默认为 [不可用]**。如果该许可是 [可用] 状态，那么 park()方法会立即返回，消费这个许可，将该许可**变更为 [不可用]** 状态，流程代码可以继续执行。**如果该许可是 [不可用] 状态，那么 park()方法将会阻塞**；unpark 方法，将指定线程的一个许可**变为 [可用]** 状态。如下表所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbPf1Eply3YqUZOCGQ9pQCN9C83xrPNJ2oEQ7n1Mia04cibtNanz01wY1A/640?wx_fmt=png)

*   示例一：先执行 unpark() 方法再执行 park() 方法，也不会造成永久卡死线程。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRb2xwbMjuic5l4E7iaC3Mv5pHDdU28ekhmkuXPgtogEFIA9Pxp9M9oJzmg/640?wx_fmt=png)

*   示例二：LockSupport.park() 还能**支持中断**。但是它**不会抛 InterruptedException 异常**。它只会默默的返回，但是我们可以从 Thread.interrupted() 等方法获得中断标记。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC9CMLdmHTxb85CdZdeJFSRbERU64Wbv9tGUCJqyKNh49HE4plGULkNfvSAgY61sS1NkF5XhftQoMw/640?wx_fmt=png)