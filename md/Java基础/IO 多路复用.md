> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247489040&idx=1&sn=605a50877450d04decedbb53187ba2c7&chksm=e91154edde66ddfbadc12a14f54f65fe59e0b949b4b931e21803a1b8e84dc08be2cd59b512af&scene=178&cur_album_id=2451364959101059073#rd)

IO 多路复用
=======

> IO 多路复用，是很多中间件会采用的 IO 方式，由于它具备非阻塞，高吞吐的优点，而广泛被采用。那么介绍 IO 多路复用之前，我们还是从 IO 和 NIO 说起吧。

阻塞的 IO
------

默认情况下，我们通过套接字读取数据时，IO 是阻塞的，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxwFr60gnnsavOALN21LBoC1cKYibNNWumniaDurpVCicR2lPlYREBAm2RQ/640?wx_fmt=png)

> 如上图可以发现，在服务端`accept`方法和`read`方法以及客户端的`connect`方法都会发生阻塞。

阻塞 IO 的优化方式
-----------

那么为了解决 read 函数阻塞后，其他客户端无法连接服务端的问题。我们第一个想到的就是，**把原有的串行逻辑修改为并行**。也就是说，既然 read 会阻塞整个流程那么我们可不可以**把 read 函数读取数据的这块逻辑，单独拉到一个子线程中进行处理**。那么即使这个子线程中的 read 阻塞了，也不会影响主线程中客户端与服务端直接的连接逻辑。如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxib7J3JAX7F3DlmA2SEAv5LrQ1IHYoGhaozMz8EXDDkX1orXHdrqkDDA/640?wx_fmt=png)

> 不过，虽然通过我们采用多线程的方式，解决了由于 read 函数阻塞而导致的服务端无法连接客户端的问题。但是，read 函数依然还是老样子——阻塞的。那么这个就不是我们可以解决的了，我们需要借助操作系统帮助我们解决 read 阻塞的问题。

非阻塞 IO
------

操作系统给我们提供了非阻塞 IO，也就是**在调用 read 之前，将文件描述符设置为非阻塞**即可，如下所示：

```
fcntl(connfd, F_SETFL, O_NONBLOCK);
int n = read(connfd, buffer) != SUCCESS);

```

那么非阻塞 IO 中，read 的处理方式就变成了如下方式了：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxib0KiaTUibk7zSogAYbCcnljcibPEz3vnzUAfzXoG3DTeFtibm90AEroWlQ/640?wx_fmt=png)

> 通过上面的处理流程，我们也可以看出来，这种非阻塞的 IO 读，并没有完全的解决阻塞，因为只要网卡的数据被 copy 到内核缓冲区，那么文件描述符变为 “读已就绪”，读操作依然还是需要阻塞的，直到数据从内核缓冲区 copy 到用户缓冲区，才会解除阻塞，返回客户端发送的数据。

非阻塞 IO 的优化方式
------------

通过上面的方式，我们发现，如果很多客户端都来连接服务端，那么由于 read 操作开启独立子线程的，那么就面临着线程资源会被耗光的情况发生。针对这种情况我们**能不能采用一条线程来解决呢**？

也就是说，每当一个客户端连接到服务端之后，我们都**将它对应的文****件描述符 connfd 放到一个数组中****，然后由一条线程不断的去数组中遍历每个元素并调用 read 函数，然后发现返回的不是 - 1，则说明是读已就绪的状态了，那么我们就进行数据读取操作**。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxic4TTUkJUIyaP5TM9ib06mD9ua4TTZMiblpaXayFZKniciaiaWSlDVVKoZDA/640?wx_fmt=png)

> 这种方式虽然可以减少创建子线程的个数了，但是，如果还要再去发现其中不足的话，那就是我们每次遍历调用 read 返回 - 1 的时候，其实都是浪费资源的系统调用。**因为我们也不知道数组中哪个文本描述符（connfd）是 “读已就绪” 状态，我们只能采取遍历的方式。**那么，要是想解决这个问题，我们依然只能寄希望于操作系统，可以给我们提供帮助。

IO 多路复用
-------

### select

select 是操作系统提供的系统调用函数，通过它，我们可以**把一个文件描述符的数组发给操作系统，让操作系统去遍历，确定哪个文件描述符可以读写**，然后告诉我们去处理。不过，当 select 函数返回后，用户依然需要遍历刚刚提交给操作系统的数组。只不过，**操作系统会将准备就绪的文件描述符做上标识**，用户层将不会再有无意义的系统调用开销。

可以看出如下三个细节：

*   • select 调用需要传入 fd 数组，需要拷贝一份数组到内核，高并发场景下这样的**拷贝消耗的资源是惊人的**（可优化为不复制）；
    
*   • select 在内核层仍然是通过遍历的方式检查文件描述符的就绪状态，是个**同步过程**，只不过无系统调用切换上下文的开销（内核层可优化为异步事件通知）；
    
*   • select **仅仅返回可读文件描述符的个数**，具体哪个可读还是要用户自己遍历（可优化为只返回给用户就绪的文件描述符，无需用户做无效的遍历）；
    

整个 select 的流程图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxOr9BeOIDAJ1GCicviaZugRazkmfXAK0VyOvjs6ZlmGib7A5CFWkiax0mnA/640?wx_fmt=png)

### poll

它和 select 的主要区别就是，**去掉了 select 只能监听 1024 个文件描述符的限制**。

### epoll

epoll 是最终的大 boss，它解决了 select 和 poll 的一些问题。

针对上面说的 select 的细节，epoll 主要就是针对这三点进行了改进，即：

*   • 内核中保存一份文件描述符集合，**无需用户每次都重新传入，只需告诉内核修改的部分即可**；
    
*   • 内核不再通过轮询的方式找到就绪的文件描述符，而是通过**异步 IO 事件唤醒**；
    
*   • 内核**仅会将有 IO 事件的文件描述符返回给用户**，用户也无需遍历整个文件描述符集合。
    

具体，操作系统提供了这三个函数。

*   • 第一步：创建一个 epoll 句柄
    

```
int epoll_create(int size);

```

*   • 第二步：向内核添加、修改或删除要监控的文件描述符。
    

```
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

```

*   • 第三步：类似发起了 select() 调用
    

```
 int epoll_wait(  int epfd, struct epoll_event *events, int max events, int timeout);

```

处理流程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCiclXeibvFLPia7mZE4gAgqsuxebz77bGOkyhcjDsHxMT1ISTEiaxRJp9Ip0e3EhC6ls0RTD1jm5LiaLhw/640?wx_fmt=png)