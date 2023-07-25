
> 超详细图文之 Redis（上篇）
> 
> java_muse，公众号：爪哇缪斯[超详细图文之 Redis（上篇）](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247485288&idx=1&sn=6e5c9d236f83595264d85fa143043cfb&chksm=e9114595de66cc837fd976b1e17fc2f99e72a9e276f3b68a108d89be2f6cd0ecdd205561d319&token=1050689176&lang=zh_CN#rd)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz3RLiaasw30HibUfkIXqQ1t9dYZWHysM7eftm64JOWjicqOXXGiajNcdYtw/640?wx_fmt=png)

一、NoSQL 四大分类整体对比  

===================

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzHFCgzY1VHnMA87GTTV2EHDDxT7Haa3xOBINVtx8l6iaRHHmeMBw6uYg/640?wx_fmt=png)

二、Redis 安装  

=============

*   官网下载最新版本 redis
    
    （https://redis.io 或 http://www.redis.cn）  
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzNxftwVlKmGLHgibHJg1XoWyfavVAYMOEVe9W8zOosnOuKsemmycxG8w/640?wx_fmt=png)

*   解压 redis-6.2.6.tar，然后进入 redis 的解压目录，执行 make 进行安装操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzAKMNba0MNJTPyLxff2mBY6Y2ctJv7EDPsUmx3ObeNUr21taFaWegGg/640?wx_fmt=png)

*   修改 redis.conf，配置它后台运行
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzsdsDzrtm7Liagc8o2K2TiaWlkChZibNP1dYGqsX7FPTDq5a9HibFN5lWTg/640?wx_fmt=png)

*   运行 redis
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzMYfRdvXfO7J69UZrXQzz2PmPUK45eX7O3WYVoFHW2hqIrCZDqibVqVw/640?wx_fmt=png)

*   使用 redis 客户端连接 redis（如果是本机安装且采用默认 port，则可以不指定 - h 和 - p）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzBIcxBtfmuo4wICq3WGict661Yp85MqhvDqGm5EH7H6mz1SXOMy8vjvA/640?wx_fmt=png)

*   关闭 redis 服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzaic742RicGicf13cKALj00mqFXBicCsWZt0dSpfHNZZ6kulA7x8xqoskicg/640?wx_fmt=png)

三、Redis-benchmark 测试工具  

=========================

*   redis 自带的用于测试 redis 性能的工具，它具有如下的可配置参数
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz4y7aLKdQuHqGlmxEt5OSvvvWEto2sibcj9dkQWDQWToviaQvDPbnAa2Q/640?wx_fmt=png)

*   我们来进行压测，开启 200 个并发，总请数为 20 万
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzkv9tX6FY7y3BY1hauqjovF91cPH8GkwpcK0ukNK6IQaTqYR3scPy3w/640?wx_fmt=png)

*   我们看一下 GET 的压测结果
    

```
====== SET ======
  200000 requests completed in 1.74 seconds
  100 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 3600 1 300 100 60 10000
  host configuration "appendonly": no
  multi-thread: no
Latency by percentile distribution:
0.000% <= 0.151 milliseconds (cumulative count 1)
50.000% <= 0.423 milliseconds (cumulative count 115987)
75.000% <= 0.447 milliseconds (cumulative count 153373)
87.500% <= 0.495 milliseconds (cumulative count 175602)
93.750% <= 0.583 milliseconds (cumulative count 187813)
96.875% <= 0.671 milliseconds (cumulative count 193905)
98.438% <= 0.751 milliseconds (cumulative count 197076)
99.219% <= 0.807 milliseconds (cumulative count 198570)
99.609% <= 0.871 milliseconds (cumulative count 199245)
99.805% <= 0.975 milliseconds (cumulative count 199623)
99.902% <= 1.103 milliseconds (cumulative count 199807)
99.951% <= 1.431 milliseconds (cumulative count 199903)
99.976% <= 2.047 milliseconds (cumulative count 199952)
99.988% <= 2.391 milliseconds (cumulative count 199976)
99.994% <= 2.583 milliseconds (cumulative count 199988)
99.997% <= 2.663 milliseconds (cumulative count 199994)
99.998% <= 2.711 milliseconds (cumulative count 199997)
99.999% <= 2.735 milliseconds (cumulative count 199999)
100.000% <= 2.751 milliseconds (cumulative count 200000)
100.000% <= 2.751 milliseconds (cumulative count 200000)
Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 0)
0.003% <= 0.207 milliseconds (cumulative count 6)
0.015% <= 0.303 milliseconds (cumulative count 30)
20.573% <= 0.407 milliseconds (cumulative count 41146)
88.780% <= 0.503 milliseconds (cumulative count 177561)
94.910% <= 0.607 milliseconds (cumulative count 189819)
97.677% <= 0.703 milliseconds (cumulative count 195355)
99.285% <= 0.807 milliseconds (cumulative count 198570)
99.704% <= 0.903 milliseconds (cumulative count 199409)
99.843% <= 1.007 milliseconds (cumulative count 199687)
99.903% <= 1.103 milliseconds (cumulative count 199807)
99.939% <= 1.207 milliseconds (cumulative count 199878)
99.947% <= 1.303 milliseconds (cumulative count 199893)
99.950% <= 1.407 milliseconds (cumulative count 199900)
99.955% <= 1.503 milliseconds (cumulative count 199910)
99.959% <= 1.607 milliseconds (cumulative count 199917)
99.963% <= 1.703 milliseconds (cumulative count 199925)
99.966% <= 1.807 milliseconds (cumulative count 199933)
99.971% <= 1.903 milliseconds (cumulative count 199942)
99.975% <= 2.007 milliseconds (cumulative count 199949)
99.978% <= 2.103 milliseconds (cumulative count 199956)
100.000% <= 3.103 milliseconds (cumulative count 200000)
Summary:
  throughput summary: 115273.77 requests per second
  latency summary (msec):
  avg       min       p50       p95       p99       max
0.447     0.144     0.423     0.615     0.783     2.751

```

四、Redis 的数据库介绍  

=================

*   Redis 默认配置了 16 个数据库，默认我们使用的是序号 = 0 的这个数据库。
    
    /Users/muse/redis-6.2.6/redis.conf 配置信息如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzOEbkYBLCxiaicThJ2qTd2iaJnw4tpK2KBpWetkAQSkHxQAZ1ib6J3rl19w/640?wx_fmt=png)

*   切换数据库 **SELECT [数据库序号（从 0 开始）]**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzls6YYnicUKL33k3FS99fOzOSQlatIYTvjBMnvtdBXtgSZa7HqDsWEyw/640?wx_fmt=png)

*   查看该数据库的大小 **DBSIZE**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzbjY9oJThgaUpIgGSvxuelKc4sdW3b0gDiaU76N9j1Vhohibm600bAHBg/640?wx_fmt=png)

*   不同的数据库，存储不同的信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzF2k7KyXzg48zBC9jEo4YeBHDYaLH06mBuv5ntic8wpVDcVqk0GzcnCg/640?wx_fmt=png)

*   清空**当前**数据库中的数据 **FLUSHDB**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzwuibn4uFo9O6nqxERibSialDNA7nOdbb4Qvu2OgxsLMIHGKusD8KvDa3Q/640?wx_fmt=png)

*   清空**所有**数据库中的数据 **FLUSHALL**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzo80Oib2U5G4MibJ0HBUZf0pm1GNtq17uLXicfUFVeicDtVk4OnXMbIGqcA/640?wx_fmt=png)

五、Redis 三种特殊类型  

=================

5.1> geospatial 地理位置
--------------------

### 5.1.1> 概述

*   可以用于基于地理位置的业务场景。比如查询两地直接的举例，方圆几里存在的地理位置等等。
    
*   Redis 提供了 geospatial 相关的 8 个指令，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz0ZjqvUvw16uDTltQQ8a8ItFU3FqxLvlhJNIlE93WOhicPos7Q9C8RAQ/640?wx_fmt=png)

### 5.1.2> GEOADD（v3.2.0）  

*   官方文档：
    
    http://www.redis.cn/commands/geoadd.html
    
*   指令格式：**GEOADD key longitude latitude member [longitude latitude member ...]**
    
*   将指定的地理空间位置（纬度、经度、名称）添加到指定的 key 中。
    
*   这些数据将会存储到 **Zset**，这样的目的是为了方便使用 **GEORADIUS** 或者 **GEORADIUSBYMEMBER** 命令对数据进行半径查询等操作。
    
*   该命令以采用标准格式的**参数 x,y**，所以**经度必须在纬度之前**。
    
*   这些坐标的限制是可以被编入索引的，区域面积可以很接近极点但是不能索引。具体的限制，由 EPSG:900913 / EPSG:3785 / OSGEO:41001 规定如下：
    
*   有效的**经度**从 - 180 度到 180 度。
    
*   有效的**纬度**从 - 85.05112878 度到 85.05112878 度。当坐标位置超出上述指定范围时，该命令将会返回一个错误。
    
*   操作示范
    
    经纬度查询 https://jingweidu.bmcx.com）：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzBOjOEYVtkQpqoq0icgqaHBwIlfxFG00SqYzvLEa307iciaeUNicI9ZHLTg/640?wx_fmt=png)

### 5.1.3> GEODIST（v3.2.0）  

*   官方文档：
    
    http://www.redis.cn/commands/geodist.html
    
*   指令格式：**GEODIST key member1 member2 [unit]**
    
*   返回两个给定位置之间的距离。如果两个位置之间的其中一个不存在， 那么命令返回空值。
    
*   指定单位的参数 **unit** 必须是以下单位的其中一个：
    

<table width="281"><tbody><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(145, 213, 255);"><p><strong>unit</strong></p></td><td width="235" data-style="border-color: rgb(217, 217, 217); background-color: rgb(145, 213, 255);"><p><strong>解释</strong></p></td></tr><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(255, 251, 143);"><p>m</p></td><td width="235" data-style="border-color: rgb(217, 217, 217);"><p>表示单位为米（默认值）</p></td></tr><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(255, 251, 143);"><p>km</p></td><td width="235" data-style="border-color: rgb(217, 217, 217);"><p>表示单位为千米</p></td></tr><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(255, 251, 143);"><p>mi</p></td><td width="235" data-style="border-color: rgb(217, 217, 217);"><p>表示单位为英里</p></td></tr><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(255, 251, 143);"><p>ft</p></td><td width="235" data-style="border-color: rgb(217, 217, 217);"><p>表示单位为英尺</p></td></tr></tbody></table>

*   如果用户没有显式地指定单位参数， 那么 GEODIST 默认使用米作为单位。
    
*   GEODIST 命令在计算距离时会假设地球为完美的球形，在极限情况下，这一假设最大会造成 0.5% 的误差。
    
*   操作示范
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzlOIrPGnIA10icRsYPxbicYbNZKI6bbFWnAweKMIfpH1iaZFr3xxwich0xA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzB7ibP9nfCpZcicUjibHoGXf20ICS9oWYAibupMRBnBDlymsB3Oezt3YLwA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzhicx7TiaibaEeRbrsj6fiaicfHwlQeRLbDDt6zSVeWcVUfA2rTJickesKWog/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzstks3Xn1Zv0hfa3q5hB5xW8LmdvuiclmzXGrXSkwGpPHOI7iachregDw/640?wx_fmt=png)

### 5.1.4> GEOHASH（v3.2.0）

*   官方文档：
    
    http://www.redis.cn/commands/geohash.html
    
*   指令格式：**GEOHASH key member [member ...]**
    
*   返回一个或多个位置元素的 Geohash 表示。通常使用表示位置的元素使用不同的技术，使用 Geohash 位置 52 点整数编码。由于编码和解码过程中所使用的初始最小和最大坐标不同，编码的编码也不同于标准。
    
*   Geohash 字符串属性
    
    该命令将返回 11 个字符的 Geohash 字符串，所以没有精度 Geohash。返回的 geohashes 具有以下特性：
    

*   他们可以缩短从右边的字符。它将失去精度，但仍将指向同一地区。
    
*   它可以在 geohash.org 网站使用，网址 http://geohash.org/<geohash-string>。查询例子：http://geohash.org/sqdtr74hyu0
    

*   操作示范
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzRVpuG1HqczBQY1Crib6dia62wuAIUjic1ibvk4WibsTQeuFnntsGL2RvyMg/640?wx_fmt=png)

### 5.1.5> GEOPOS（v3.2.0）

*   官方文档：
    
    http://www.redis.cn/commands/geopos.html
    
*   指令格式：**GEOPOS key member [member ...]**
    
*   从 key 里返回所有给定位置元素的位置（经度和纬度）
    
*   因为 GEOPOS 命令接受可变数量的位置元素作为输入，所以即使用户只给定了一个位置元素，命令也会返回数组回复。
    
*   操作演示
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzfBemnGxfT0r9icvibauEkUfLJ6vpMgBoxBib0NJWnOA12KEJozlWGRJnA/640?wx_fmt=png)

5.1.6> GEORADIUS（v3.2.0）

*   官方文档：
    
    http://www.redis.cn/commands/georadius.html
    
*   指令格式：**GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]**
    
*   以给定的经纬度为中心，返回键包含的位置元素当中，与中心的距离不超过给定最大距离的所有位置元素。范围可以使用以下其中一个单位：
    

<table width="281"><tbody><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(145, 213, 255);"><p><strong>unit</strong></p></td><td width="235" data-style="border-color: rgb(217, 217, 217); background-color: rgb(145, 213, 255);"><p><strong>解释</strong></p></td></tr><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(255, 251, 143);"><p><strong>m</strong></p></td><td width="235" data-style="border-color: rgb(217, 217, 217);"><p>表示单位为米（默认值）</p></td></tr><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(255, 251, 143);"><p><strong>km</strong></p></td><td width="235" data-style="border-color: rgb(217, 217, 217);"><p>表示单位为千米</p></td></tr><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(255, 251, 143);"><p><strong>mi</strong></p></td><td width="235" data-style="border-color: rgb(217, 217, 217);"><p>表示单位为英里</p></td></tr><tr><td width="46" data-style="border-color: rgb(217, 217, 217); background-color: rgb(255, 251, 143);"><p><strong>ft</strong></p></td><td width="235" data-style="border-color: rgb(217, 217, 217);"><p>表示单位为英尺</p></td></tr></tbody></table>

*   在给定以下可选项时，命令会返回额外的信息：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzibrqSq5gCtHEtdEJH3uD8j7JFvviaXjYDEoUGqnibdUHhwCdm045ShTZw/640?wx_fmt=png)

*   命令默认返回未排序的位置元素。通过以下两个参数，用户可以指定被返回位置元素的排序方式：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzLN4GDSwGr5ssUpuAPzLdxSGKf7RklwibicScOibpKRhXudL21UwicZk1Yg/640?wx_fmt=png)

*   在默认情况下，GEORADIUS 命令会返回所有匹配的位置元素。虽然用户**可以使用 COUNT <count> 选项去获取前 N 个匹配元素**，但是因为命令在内部可能会需要对所有被匹配的元素进行处理， 所以在对一个非常大的区域进行搜索时，即使只使用 COUNT 选项去获取少量元素，命令的执行速度也**可能会非常慢**。但是从另一方面来说，使用 COUNT 选项去减少需要返回的元素数量，对于减少带宽来说仍然是非常有用的。
    
*   操作演示
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzbtnUSbgBFQIsJiaJQVWehRddpPcWytJV9V6hg2iabqicmDViaDj1OA7kNw/640?wx_fmt=png)

### 5.1.7> GEORADIUSBYMEMBER（v3.2.0）  

*   官方文档：
    
    http://www.redis.cn/commands/georadiusbymember.html
    
*   指令格式：**GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]**
    
*   这个命令和 GEORADIUS 命令一样，都可以找出位于指定范围内的元素，但是 GEORADIUSBYMEMBER 的中心点是由给定的位置元素决定的，而不是像 GEORADIUS 那样，使用输入的经度和纬度来决定中心点指定成员的位置被用作查询的中心。
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzNAZUWvdPDxYGPyqMhk6bzHLpPULzroar3SiaYBNfXejWD8w1BhMYQng/640?wx_fmt=png)

5.2> hyperloglog 预估集合的基数  

---------------------------

### 5.2.1> 概述

*   hyperloglog 常用的使用场景，一般是非精准性的统计计数。比如：统计访问网站的 UV 数，商品评论数或点击量等等。
    
*   HyperLogLog 是一种用于计算唯一事物的概率数据结构（从技术上讲，这称为预估集合的基数）
    

*   它占用的空间很小，只需要 12KB 的内存，可以存储 2^64 不同的元素数量。但是它的统计是有小于 1% 的误差的，所以并不适合精准统计的使用场景。
    
*   Redis 提供了 hyperloglog 相关的 3 个指令，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzIUFFkjdpH1G3t1iaDMtWiavYYqMDBQBq1CKzb4R846Cic9g13ThEcqmUQ/640?wx_fmt=png)

### 5.2.2> PFADD（v2.8.9）  

*   官方文档：
    
    http://www.redis.cn/commands/pfadd.html
    
*   指令格式：**PFADD key element [element ...]**
    
*   将 element 集合存储到以 key 为变量名的 HyperLogLog 结构中.
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz4pvxgLkDOJPicJwWjicichhzGOksicekfnQ3E8N27ZVjvttsqzySMkVZrg/640?wx_fmt=png)

### 5.2.3> PFCOUNT（v2.8.9）

*   官方文档：
    
    http://www.redis.cn/commands/pfcount.html
    
*   指令格式：**PFCOUNT key [key ...]**
    
*   获得指定 key 为变量名的 HyperLogLog 结构中中元素的个数
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzj3bbQxZg1QXddz7T8nrDNPceILbia64C5a8n9h0vPHicnY9Xa7YibzogQ/640?wx_fmt=png)

### 5.2.4> PFMERGE（v2.8.9）

*   官方文档：
    
    http://www.redis.cn/commands/pfmerge.html
    
*   指令格式：**PFMERGE destkey sourcekey [sourcekey ...]**
    
*   将多个 HyperLogLog 合并（merge）为一个新的 HyperLogLog ，合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。合并得出的 HyperLogLog 会被储存在目标变量（第一个参数）里面，如果该键并不存在，那么命令在执行之前， 会先为该键创建一个空的.
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzzarwdiarsDd8SSFxQibCaLq0SjYJktIN29BtBxxic9nbMznym9AkrAefA/640?wx_fmt=png)

5.3> bitmap 位图  

-----------------

### 5.3.1> 概述

*   我们可以利用 bitmap 指定其二进制位是 0 或 1，来实现类似 “是”or“否” 的相关操作。它的特点也是占用内存空间特别的小。比如，我们要记录每个用户当天是否活跃（即：是否登录过系统），那么如果我们要记录他一年的是否登录的记录，只需要 365 个 bit 即可存储。
    
*   Redis 提供了位图相关的 7 个指令，我们只针对其中常用的 3 个进行操作演示。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzAcibvL0W5fZx5aXDHZwBd1QeFaJXR2Yyde9S7A7C397rnrVExcWorRw/640?wx_fmt=png)

### 5.3.2> SETBIT（v2.6.0）  

*   官方文档：
    
    http://www.redis.cn/commands/setbit.html
    
*   指令格式：**SETBIT key offset value**
    
*   设置或者清空 key 的 value(字符串) 在 offset 处的 bit 值。那个位置的 bit 要么被设置，要么被清空，这个由 value（只能是 0 或者 1）来决定。当 key 不存在的时候，就创建一个新的字符串 value。要确保这个字符串大到在 offset 处有 bit 值。参数 offset 需要大于等于 0，并且小于 2^32（限制 bitmap 大小为 512MB）。当 key 对应的字符串增大的时候，新增的部分 bit 值都是设置为 0。
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzTEwhWHianX2CkzXopIUJrFHhicoic1uCZ57KicUufotlegKdlFw4reYVpg/640?wx_fmt=png)

### 5.3.3> GETBIT（v2.2.0）  

*   官方文档
    
    http://www.redis.cn/commands/getbit.html
    
*   指令格式：**GETBIT key offset**
    
*   获取 key 中某个 offset 位置上的值
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzocGibLdcYJPAaicnErMB84K3suWWicUxD8H1HRmXy0eHD08KhoRccKMYg/640?wx_fmt=png)

### 5.3.4> BITCOUNT（v2.6.0）  

*   官方文档：
    
    http://www.redis.cn/commands/pfmerge.html
    
*   指令格式：**BITCOUNT key [start end]**
    
*   获取 key 中从 start 到 end 范围内 1 的个数
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzweSN7IETK6I0Ogiapga8iaSTDp3rs6rP1yHnk9qohg0mPfGoEhNPkKCQ/640?wx_fmt=png)

六、Redis 的事务管理  

================

6.1> 概述
-------

*   事务的本质，其实就是一组命令的集合。一个事务中的所有命令都会按照命令的顺序去执行，而中间不会被其他命令加塞。
    
*   Redis 提供了事务相关的 5 个指令，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz9FPU1QojxrX70VNgDSiao2Fh48a89mFGlppTcKHDxVEWJpqTvaJK9dw/640?wx_fmt=png)

6.2> MULTI（v1.2.0）  

---------------------

*   官方文档：
    
    http://www.redis.cn/commands/multi.html
    
*   指令格式：**MULT**
    
*   标记一个事务块的开始。随后的指令将在执行 EXEC 时作为一个原子执行。简而言之，我们可以使用 MULTI 来开启一个事务。
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzJFNgwTjuwEjH1eKiaicItqmejOaFvgtE2W2dZY9Is40WbBZIqwzSN7dw/640?wx_fmt=png)

【解释】

我们发现，在事务中每次执行一条指令，就会返回 QUEUED，表明指令已经存入了这个事务的执行队列中了。但是需要注意的一点是，只是放入了事务队列，但并没有去执行。_那什么时候会执行呢？_那就来看一下下个指令 EXEC。

6.3> EXEC（v1.2.0）
-----------------

*   官方文档：
    
    http://www.redis.cn/commands/bitfield.html
    
*   指令格式：**EXEC**
    
*   执行事务中所有在排队等待的指令并将链接状态恢复到正常。当使用 WATCH 时，只有当被监视的键没有被修改，且允许检查设定机制时，EXEC 会被执行。简而言之，我们可以使用 EXEC 来提交一个事务。
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzUJicmVQRO12KEa4VOIcPLnA6bUH2D6qoTfo7L5BDphw6xic9Rdmtes5g/640?wx_fmt=png)

【解释】

调用完 EXEC 之后，正确执行的都会返回 OK，并且当我们再次查询 account 里面的金额的时候，也正确的返回了 1100。这就说明，一个事务内的指令是按照顺序执行的。

6.4> DISCARD（v2.2.0）
--------------------

*   官方文档：
    
    http://www.redis.cn/commands/discard.html
    
*   指令格式：**DISCARD**
    
*   刷新一个事务中所有在排队等待的指令，并且将连接状态恢复到正常。如果已使用 WATCH，DISCARD 将释放所有被 WATCH 的 key。
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzalicXooA4E7VdKXiagnpvhicmxRHTbsCRXW76O0ibgPafUUAG9qh8Y2dqw/640?wx_fmt=png)

6.5> WATCH（v2.2.0）
------------------

*   官方文档：
    
    http://www.redis.cn/commands/watch.html
    
*   指令格式：**WATCH**
    
*   标记所有指定的 key 被监视起来，在事务中有条件的执行。可以利用 WATCH 实现 Redis 的**乐观锁**
    
*   操作演示 1：
    

*   客户端 A：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzuxaSDvNBPdJxoS6v23l7znoHE2iaB5ic9loC1biblupoBr5HOvic9JDsOg/640?wx_fmt=png)

*   客户端 B：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzQciaX6zmAR0KrtkKMY2mI0tyrx4xjIc9YJDY2OSmEgmictV0HWnv6icxA/640?wx_fmt=png)

*   操作演示 2：
    

*   在客户端 A 中我们再次开始事务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzHZibym4icj9Sdqj0qiazo4micW61uwqQcp0lWDAjNxgEpWYyvY9pLp1IoQ/640?wx_fmt=png)

*   客户端 B：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzQciaX6zmAR0KrtkKMY2mI0tyrx4xjIc9YJDY2OSmEgmictV0HWnv6icxA/640?wx_fmt=png)

【结论】当执行了 EXEC 指令之后，watch 就被隐式的执行了 unwatch。如果需要再次监控，就需要再次调用 WATCH 指令。  

6.6> UNWATCH（v2.2.0）
--------------------

*   官方文档：
    
    http://www.redis.cn/commands/unwatch.html
    
*   指令格式：**UNWATCH**
    
*   刷新一个事务中已被监视的所有 key。**如果执行** **EXEC** **或者** **DISCARD****，则不需要手动执行** **UNWATCH**
    
*   操作演示：（略）
    

6.7> 事务中异常的处理
-------------

### 6.7.1> 命令语法错误

*   针对语法错误，会导致整个事务执行被中断
    
*   操作演示
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzoFmrKNfw4PuXPlltskdegdiaJ7ItkzdbKJrtophQUiczMU4HgrXagHnw/640?wx_fmt=png)

### 6.7.2> 运行操作错误  

*   针对执行中的异常，只会导致该条指令的执行失败，而不会影响事务中其他的指令
    
*   操作演示
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzksxUHnkpuIibQiaHPuKOxsTBR7j6omcMGCBr3UQcWLj7XutzuriaoIzicQ/640?wx_fmt=png)

七、Redis 实现订阅发布  

=================

7.1> 概述
-------

*   如果熟悉消息中间件，那么对发布订阅一定不陌生。发布者 Publish 一条消息，消息发送到 Channel 通道中，然后所有订阅了这个通道的订阅者 Subscriber 都会接收到这条消息。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzojvCZ8nLVcXYcboIqgsx9LZsT2to2KeVDicDjemzW3ImRKTIwaLOdUA/640?wx_fmt=png)

*   针对发布订阅，redis 提供了 9 个相关的指令，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzRvjLaQqibQe0l2CZ443hJJhedjUQ6iaZRSLvsSIpVRaycUcdLl271fiaA/640?wx_fmt=png)

7.2> SUBSCRIBE（v2.0.0）  

-------------------------

*   官方文档：
    
    http://www.redis.cn/commands/subscribe.html
    
*   指令格式：**SUBSCRIBE channel [channel ...]**
    
*   订阅给指定频道的信息。一旦客户端进入订阅状态，客户端（Jedis、lettuce 等）就只可接受订阅相关的命令：
    
    SUBSCRIBE、PSUBSCRIBE、UNSUBSCRIBE 和 PUNSUBSCRIBE 除了这些命令，其他命令一律失效。
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz7rjOy8dyjDPkAZdvQnZk13nGDtRpl4hiahggibKFYlOibODGrjm8iaH4kA/640?wx_fmt=png)

7.3> PUBLISH（v2.0.0）  

-----------------------

*   官方文档：
    
    http://www.redis.cn/commands/publish.html
    
*   指令格式：**PUBLISH channel message**
    
*   将信息 message 发送到指定的频道 channel
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzHELORLNjaOcn0Cl4rDcNicN5w8GOEAo2YBwVsTogSGzfxuMTXRYFHEA/640?wx_fmt=png)

7.4> PSUBSCRIBE（v2.0.0）  

--------------------------

*   官方文档：
    
    http://www.redis.cn/commands/psubscribe.html
    
*   指令格式：**PSUBSCRIBE pattern [pattern ...]**
    
*   订阅给定的模式 (patterns)。如果想输入普通的字符，可以在前面添加 \。支持的模式(patterns) 有:
    

*   pattern 等于 h?llo 时（？表示匹配一个字符），会订阅到 hello，hallo，hxllo 等等
    
*   pattern 等于 h*llo 时（* 表示匹配 0 或任意个字符），会订阅到 hllo，heeeello 等等
    

*   pattern 等于 h[ae]llo 时（[ae] 表示 a 或者 e），会订阅到 hello，hallo 但是不能匹配订阅 hillo
    

*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzs8S462wbLPPmFT8Ppss8kKN3CtIlOjGia2XiaQ0VSL57iaxHHSEXFluWw/640?wx_fmt=png)

7.5> PUBSUB CHANNELS
--------------------

*   官方文档：
    
    http://www.redis.cn/commands/pubsub.html
    
*   指令格式：**PUBSUB CHANNELS [pattern]**
    
*   列出指定 channel 频道中含有一个或多个订阅者 (不包括从模式接收订阅的客户端)。如果 pattern 未提供，所有的信道都被列出，否则只列出匹配上指定 pattern 模式的信道被列出.
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzGeVfKYwL9GXeOIJ76N0kyUoaqomNKtTW94jXlZDwSypO8BXQicN3fIA/640?wx_fmt=png)

7.6> PUBSUB HELP（v6.2.0）
------------------------

*   官方文档：
    
    http://www.redis.cn/commands/unwatch.html
    
*   指令格式：**PUBSUB HELP**
    

7.7> PUBSUB NUMPAT（v2.8.0）
--------------------------

*   官方文档：
    
    http://www.redis.cn/commands/pubsub.html
    
*   指令格式：**PUBSUB NUMPAT**
    
*   返回**订阅模式**的数量 (使用命令 PSUBSCRIBE 实现)。注意，这个命令返回的不是订阅模式的客户端的数量，而是客户端订阅的所有模式的数量总和。
    
*   操作演示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRziamQNkvCZ1wIq3E8UnT55OkrTWWhoMibzvmCJDvLV2bKCkTtv2KSibibOA/640?wx_fmt=png)

7.8> PUBSUB NUMSUB（v2.8.0）  

-----------------------------

*   官方文档：
    
    http://www.redis.cn/commands/pubsub.html
    
*   指令格式：**PUBSUB NUMSUB [channel-1 ... channel-N]**
    
*   列出指定信道的订阅者个数 (不包括订阅模式的客户端订阅者)
    
*   操作演示
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzVIpAnj5Yy90qiacQKfibWXsykNd3oPrJDbveibPISTT30DV7Sj9VmQp4A/640?wx_fmt=png)

7.9> UNSUBSCRIBE（v2.0.0）  

---------------------------

*   官方文档：
    
    http://www.redis.cn/commands/unsubscribe.html
    
*   指令格式：**UNSUBSCRIBE channel [channel ...]**
    
*   取消普通订阅。指示客户端退订给定的频道，若没有指定频道，则退订所有频道。如果没有频道被指定，即：一个无参数的 UNSUBSCRIBE 调用被执行，那么客户端使用 SUBSCRIBE 命令订阅的所有频道都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的频道。**该指令是为 Jedis 或 lettuce 等客户端服务的。**
    

7.10> PUNSUBSCRIBE（v2.0.0）
--------------------------

*   官方文档：
    
    http://www.redis.cn/commands/punsubscribe.html
    
*   指令格式：**PUNSUBSCRIBE [pattern [pattern ...]]**
    
*   取消模式订阅。指示客户端退订指定模式，若果没有提供模式则退出所有模式。如果没有模式被指定，即一个无参数的 PUNSUBSCRIBE 调用被执行，那么客户端使用 PSUBSCRIBE 命令订阅的所有模式都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的模式。**该指令是为 Jedis 或 lettuce 等客户端服务的。**
    

八、Redis 主从复制  

===============

8.1> 概述
-------

*   主从复制，是指将一台 Redis 服务器的数据复制到其他的 Redis 服务器。前者称为主节点（Master/Leader），后者称为从节点（Slave/Follower）；数据是从主节点复制到从节点的。其中，主节点负责写数据（当然有读的权限），从节点负责读数据（它没有写数据的权限）。**默认的配置下，每个 Redis 都是主节点**。
    
*   一个主节点可以有多个从节点，但是一个从节点只能有一个主节点，即：主从节点是 1 对 N 的关系。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz45aER80jziaOh6L2IhNjXqPoq63VBjASLMzpzhg3PO2JNyTLDc38gkA/640?wx_fmt=png)

*   主从复制的用处
    

*   **数据冗余**
    

主从复制实现了数据的备份，实际上提供了数据冗余的实现方式。

*   **故障恢复**
    

当主节点出现异常时，可以由从节点提供服务，实现快速的故障恢复，实际上提供了服务冗余的实现方式。

*   **负载均衡**
    

在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务，分担服务器的负载；

在写少读多的业务场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器是并发量。

*   **高可用**
    

哨兵配合主从复制，可以是实现 Redis 集群的高可用。

8.2> 环境搭建
---------

*   创建 redis-cluster 目录，然后复制 3 份 redis（也可以一份 redis3 份不同的配置文件，启动的时候，读取响应的配置文件），如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzVoPuAAfZh5yaYnrdgeHA55FdMktZYNQCrRkxHqMZVcibMKpkTm5rZqw/640?wx_fmt=png)

*   分别修改它们的 redis.conf 配置文件
    

```
# redis-6380/redis.conf
port 6380
pidfile /var/run/redis-6380.pid
logfile "redis-6380.log"
dbfilename dump-6380.rdb
daemonize yes

# redis-6381/redis.conf
port 6381
pidfile /var/run/redis-6381.pid
logfile "redis-6381.log"
dbfilename dump-6381.rdb
daemonize yes
# 如果不通过修改配置文件，也可以在客户端中输入“SLAVEOF 127.0.0.1 6380”即刻生效！！
# 也可以在客户端中输入“SLAVEOF NO ONE”来断开主从关系
replicaof 127.0.0.1 6380 

# redis-6382/redis.conf
port 6382
pidfile /var/run/redis-6382.pid
logfile "redis-6382.log"
dbfilename dump-6382.rdb
daemonize yes
replicaof 127.0.0.1 6380

```

*   启动这 3 个 redis 服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz5ZwIia0Qic8XhOFtnA5fMODlibHeNHb8AGiaokycnllaqzq02J5chtbqdQ/640?wx_fmt=png)

*   开启三个客户端，来连接这 3 个 redis 服务实例；**利用 ping 查看服务是否正常，并且通过 info replication 查看自己的角色**
    

*   **redis-6380**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzZzsm3VYondf5FqMykTviagnNefibudCWPiaqP3U8icrOzmppVdKEOPEMQw/640?wx_fmt=png)

*   **redis-6381**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzp0sM8KKcnmrgy1yGrOR6XTU5uu4JIZnC3W3zibcG3XoJ8Z1dLLzI9Eg/640?wx_fmt=png)

*   **redis-6382**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzA81D8Pw0Vt0eBG2K0YO2z45ytAEbiaRHKYGetd9Ngk1uKhPzJN44ibxg/640?wx_fmt=png)

8.3> 相关特性
---------

### 8.3.1> 从节点是只读的

*   我们测试一下主节点 redis-6380 的读写操作，读写都 ok
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzndNbSJYv7gAettIhFiccXRYiaTB0o5KfohfUZEjLHOBUvhcpKwQofs7A/640?wx_fmt=png)

*   我们测试一下从节点 redis-6381 的读写操作，发现不能执行写入操作，但是可以读取数据，其中 muse 是我们在 6380 主节点中添加的
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzHfDicuJ8bWhL48HJibqoIAUx2K7IhI9tW9hbxKMI3TIXcwJjpAtpF5JA/640?wx_fmt=png)

*   我们测试一下从节点 redis-6382 的读写操作，也一样是只读的
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzg670VB3bVtficvtKGNZibKsOREQ4jYVLmMvIJr3IWIicb51icdQSvO7oWA/640?wx_fmt=png)

### 8.3.2> 主节点意外宕机

*   我们关闭主节点 6380 的服务，查看从节点的对外服务是否收到影响
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzx2qibdicanYrdusic8H7H9Jr12spUiaZ6d14UwyXMsekR62FaicbGFGNb1g/640?wx_fmt=png)

*   测试两个从节点是否可以对外正常的提供服务。如下所示，我们可以看到，从节点依然可以堆外提供只读的服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzKXJHxOWKn87A9bKuJctyZsHrTA7lXbJXKUDYlc5dSbueWzUwPcMmTg/640?wx_fmt=png)

*   虽然主节点挂掉了，但是这两个从节点并不会自动的成为主节点，他们依然是从节点的角色。我们可以通过 info replication 来确认一下
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzZWxmsZkr0ianvGcrDTjWdQkJHdzfXn87MfXZo1o0c4HSkK3HkQcJChA/640?wx_fmt=png)

*   我们重新启动主节点，并且添加数据，我们来确认一下，这两个从节点会不会依然能够获得主节点同步过来的新数据
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzV5k5Hcanibu7T1r8wVSU5npFTibGdnicI4XsEQ6f1xlShhibUVLCVIxCEA/640?wx_fmt=png)

【解释】我们发现，两个从节点都可以获取到新添加的 bob。说明，只要主节点再次成功启动，主从结构依然可以自动的建立起来。

8.4> 实现原理
---------

*   Redis 的主从复制可以分为两个阶段：sync 阶段和 command propagate 阶段。
    

### 8.4.1> sync 阶段

*   当从节点启动后，会发送 sync 指令给主节点，要求全量同步数据。具体步骤如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzIrhSiaZxmMrDAgo2JFlSzsKDogtmW56toaPeicQoEWZHialjXS70lNwmw/640?wx_fmt=png)

【解释】  

*   步骤 1：Slave 启动后，连接 Master 节点并发送 sync 指令。
    
*   步骤 2：Master 节点接到 sync 指令后，会执行 BGSAVE 指令，生成 RDB 文件。此外，在 Master 节点生成 RDB 文件时，会将此后客户端执行的增删改操作都存入缓冲区。
    
*   步骤 3：文件生成后，会发送给 Slave 节点，Slave 节点接收到后，会删除所有旧的数据，然后加载 RDB 数据，实现数据全量同步操作。
    
*   步骤 4：当 Slave 数据加载完毕后，Master 会将缓冲区的指令发送给 Slave
    
*   步骤 5：由 Slave 去执行缓冲区新增的指令。
    

### 8.4.2> command propagate 阶段

*   即：命令传播阶段。
    
*   上面我们介绍了，Slave 节点通过 sync 指令请求 Master 节点全量数据的同步操作。那么，如果后续 Master 节点接收到新的增删改操作，也需要 Slave 节点接收同步的更新，那么这种就是 command propagate
    

### 8.4.3> psync 指令

*   当主从节点都正在运行的时候，出现了网络抖动，造成连接断开，那么当网络恢复，两个节点再次建立起连接的时候。从节点发送 sync 指令后，主节点依然需要重新生成 RDB，并对从节点进行全量数据的同步造成。那么这中间的耗时是非常严重的，并且传输备份文件也会对网络带宽造成很大的消耗。那么为了解决这个问题，从 Redis 2.8 开始，引入了 psync 指令来代替 sync 指令
    
*   psync 指令会根据不同的情况，来确定执行**全量重同步**还是**部分重同步**
    
*   全量重同步
    
    当从节点是第一次与主节点建立连接的时候，那么就会执行全量重同步，这个同步过程与上面我们介绍的 sync 阶段 + command propagate 阶段一样
    
*   部分重同步
    
    从节点的**复制偏移量**无法在**复制积压缓冲区**中找相应待同步的数据
    
    主节点与从节点不是第一次同步（根据 Redis 节点 ID 判断）
    
*   什么是复制偏移量？
    
    Master 节点和 Slave 节点都保存着一份赋值偏移量。
    
    当 Master 节点每次向 Slave 节点发送 n 字节数据的时候，就会在 Master 节点偏移量加上 n；而 Slave 节点每次接收到 n 个字节的时候，也会在 Slave 节点偏移量上加 n。
    
    在命令传播阶段，Slave 节点会定期的发送心跳 **REPLCONF ACK{offset}** 指令，这里的 offset 就是 Slave 节点的 offset。当 Master 节点接收到这个心跳指令后，会对比自己的 offset 和命令里的 offset，如果发现有数据丢失，那么 Master 节点就会推送丢失的那段数据给 Slave 节点。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzOSBJOwicIyribOzuXxuPJCNHBMnB0zop5RzFXTwQV5MEpKfX6ibjAzEicw/640?wx_fmt=png)

*   什么是复制积压缓冲区？
    
    复制积压缓冲区是由主节点维护的一个固定长度（默认 1MB）的队列。
    
    它存储了每个字节值与对应的复制偏移量。
    
    因为复制积压缓冲区的大小是固定的，所以它保存的是主节点近期执行的写命令。当从节点将 offset 发送给主节点后，主节点便会根据 offset 与复制积压缓冲区的大小来决定是否可以使用部分重同步。如果 offset 之后的数据仍然在复制积压缓冲区内，则执行部分重同步；否则还是执行全量重同步。
    
*   节点 ID
    
    Redis 节点服务启动之后，就会产生一个用来唯一标识 Redis 节点的 ID。
    
    当 Master 节点与 Salve 节点进行第一次连接同步的时候，Master 节点会将 ID 发送给 Slave 节点，Slave 节点接收到会，会对其进行保存。那么当主从服务之间发生了中断重连的时候，Slave 服务器会将这个 ID 发送给 Master 服务器，Master 服务器会拿自己的 ID 进行对比，如果相同，则说明主从之前是连接过的。否则，则说明是第一次建立的连接。那么，就需要全量去同步数据了。
    

九、Redis 哨兵  

=============

9.1> 概述
-------

*   我们介绍主从复制的时候发现，主节点挂掉从节点不会自动变为主节点，需要人工的去配置主节点才可以。但是这种做法费时费力，怎样能让 redis 在主节点挂掉的情况下，自己从从节点中选择新的主节点呢？这时候，就需要使用 Sentinel 哨兵了。
    
*   哨兵本质就是一个 **Redis 实例节点**。哨兵模式是一种特殊的模式，它能够后台监控主机是否故障，如果故障了，则根据投票数自动将 Slave 节点转换为新的 Master 节点。首先 Redis 提供了哨兵的命令，哨兵是一个独立的进程，会独立的运行。它的原理是：哨兵通过发送命令，等待 Redis 服务器响应，从而监控运行的多个 Redis 实例。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzMmM2yletXVamRghq7tfRow1sxDFjCqZcsNxDiaUOxuhjTQicHlpSxficg/640?wx_fmt=png)

*   哨兵的作用：
    
*   **监控（Monitoring**）
    

Sentinel 会不断地检查主节点和从节点是否运作正常。

*   **通知（Notification）**
    

当被监控的某个 Redis 服务器出现问题时，Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。

*   **自动故障迁移（Automatic failover）**
    

当一个主服务器不能正常工作时，Sentinel 会开始一次自动故障迁移操作，它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器；当客户端试图连接失效的主服务器时，集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

*   然而一个哨兵进程对 Redis 服务器进行监控，可能会出现问题。因此，我们可以使用多哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzvosP9t8DoJed35aQS3FUypYibic4zzicY8gZaTPbAWnGnd97gtNXwvMIg/640?wx_fmt=png)

*   Redis 的 Sentinel 中，关于下线（down）有两个不同的概念：
    
*   **主观下线**（Subjectively Down， 简称 SDOWN）
    

指的是单个 Sentinel 实例对服务器做出的下线判断。

*   **客观下线**（Objectively Down， 简称 ODOWN）
    

指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后，得出的服务器下线判断。

一个 Sentinel 可以通过向另一个 Sentinel 发送 SENTINEL is-master-down-by-addr 命令来询问对方是否认为给定的服务器已下线。

9.2> 环境搭建（3 哨兵 1 主 2 从）
-----------------------

*   我们来搭建如下图的 Sentinel 集群
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRztib9aYEDQKiceV922HoHeicCIf3hmgFK9ojcUHzqY7tHYZoCJ8SKEqLGQ/640?wx_fmt=png)

*   Redis 源码中包含了一个名为 sentinel.conf 的文件，这个文件是一个带有详细注释的 Sentinel 配置文件示例。运行一个 Sentinel 所需的最少配置如下所示：
    

```
# sentinel的端口号，如果配置3个Sentinel，只需要修改这个port即可，即：26380、26381、26382
port 26380
# 监视127.0.0.1:6380的主节点，且至少有2个Sentinel判断主节点失效，才可以自动故障迁移
sentinel monitor mymaster 127.0.0.1 6380 2
# 指定了Sentinel认为服务器已经断线所需的毫秒数；如果服务器在给定的毫秒数之内，没有返回Sentinel发送的PING命令的回复，或者返回一个错误，
# 那么Sentinel将这个服务器标记为主观下线（subjectively down，简称 SDOWN ）
sentinel down-after-milliseconds mymaster 60000 
# 故障迁移超时时间
sentinel failover-timeout mymaster 180000
# 指定了在执行故障转移时，最多可以有多少个从服务器同时对新的主服务器进行同步，这个数字越小，完成故障转移所需的时间就越长。
sentinel parallel-syncs mymaster 1

```

*   基于上面 Sentinel 的配置文件，我们复制 3 份，只需要修改对应的端口即可，其他配置不用变化，如下所示
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzicMbiasanzj0ANmMmAM8GwRVDlaIwwexLaxn9J8zsXXHhTtGuz9LHEnA/640?wx_fmt=png)

*   除了上面列出的【Sentinel 所需最少配置】之外，还有修改对应的端口号，如下所示：
    

*   以 sentinel-26380.conf 为例
    
*   【**说明 1**】下面我们要观察 Sentinel 是如何监控实例的，所以暂时我们可以把 daemonize 设置为 no，线上环境，我们要设置为 yes，让它在后台监控运行即可
    
*   【**说明 2**】由于我们需要控制台查看 Sentinel 监控情况，所以我们设置为 logfile ""，这样就可以从控制台输出信息了。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzAuGaCoQs1WOozsfB9wqcCw0XoBicfiap6ALEW4DKqy8LqXXyIusYict7g/640?wx_fmt=png)

*   修改完 Sentinel 配置文件后，我们来启动一下 redis 的 3 个节点（我们清除掉 redis.conf 中配置的 replicaof，采用通过 SLAVEOF 指令的方式来动态设置主节点，这样会更灵活）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzaCsICQmnVRJeggGFjnlloNr97ibUwWCmzZ8MmicnCyZECDicdNhaemyiaQ/640?wx_fmt=png)

*   启动 Sentinel 的两种方式：
    

```
redis-sentinel /path/to/sentinel.conf

```

或

```
redis-server /path/to/sentinel.conf --sentinel

```

*   启动 Sentinel 哨兵
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzwX1UIOgD8lydXnyXT1plLqeG473FKGX3bicVicndsjquABgicrkhJhe4g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzEdLCD0VeRrGSaHRjhrVNzwQaviaYWcDY9NcerfpVpVvCbckCR0TuYJw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzmCShXjzbg0D836amhBgPf01c2o2Mict4hzaicVdpkdHdDnCAo1qo0soQ/640?wx_fmt=png)

9.3> 测试主节点关闭的自动选主
-----------------

*   关闭主节点 6380
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz3ffIufrUvg7xUobfbIiaUlmCsD2rzIAkysLxC1KSYtRWcWKbn1PSD1Q/640?wx_fmt=png)

*   我们来看 Sentinel 中输出的日志信息，从日志中我们可以看到，Sentinel 将 6381 作为了新的主节点
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzia5WssX2tShBFo2WasgmxXibxPXooA4nqwtHgPCsrETVr1sUHcMwDULA/640?wx_fmt=png)

*   我们用 info replication 验证一下 6381 和 6382 的角色
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzvFOg2RvDou8Ps4supuRzg8yUBW66xZZmVFePU7zgk94gAcYYu9qZiag/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzz9tNT3LJrs0TuSPfzCjic7ibtD2ja9o5jakPWQ1FnPGEiaATABL7SeVZA/640?wx_fmt=png)

*   此时我们再次启动 6380，我们发现它的角色已经是从节点了。也就是说，即使原主节点又恢复正常了，也只能老老实实的当从节点这个角色了。（就像朱祁镇和朱祁钰的故事）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzhiaTutfNyvBicicZtHC9AibUMztlRLcEoHEjpibLzLRHtXqAggcR5IgTialw/640?wx_fmt=png)

9.4> 测试关闭一个 Sentinel，其他哨兵是否正常工作

*   我们尝试关闭 26380 这个 Sentinel，发现其他两个 Sentinel 能够发现
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzUBe8BdTXKJOMRfLKED2oU5hcWRDu4rIbQjXm0ScTeoTbGwFIibUJJbQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzyRNY4dlBZ54Qrq08k2wIAItY6NWkaiakFHSoQfn3SUl3PuF1C11zb5Q/640?wx_fmt=png)

*   关闭 Master 节点 6381，我们发现，Sentinel 哨兵工作正常，选择了 6380 称为新的 Master 节点
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzYzdOhgJVaFUsA2ernXqPgC4vjllv9qRH7aa7G9YeY0b1sbc94qgWXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzAq6iaoficQcZaQXQJRxktJdMDmyJ4b8xC9HcmcdLGMOfp43pKgubeLDg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzooLHJyKVGEiashvJm6A0V9Pgich5ssAdvib54PexLJTibFB9966lfmK5Ew/640?wx_fmt=png)

9.5> 测试只剩下一个 Sentinel
---------------------

*   因为我们配置 Sentinel 的时候，配置的是需要 2 个 Sentinel 确认主机下线才进行迁移，但是现在就剩下 1 个 Sentinel 了，是否还能执行自动的主从切换
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz1TqAjeibjzjMZdia8A5HrVFSJAWDqYvUp1iaclsbKDcozgG1puPSdHGAg/640?wx_fmt=png)

*   我们下线 26381 这个 Sentinel，只留下 26382
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz9Ae9icbIG18EkQTI6jhdGfxrQrpHjma362PmB3hueRdGIhs5mAhbZHQ/640?wx_fmt=png)

*   这时我们把刚刚关闭的 6381 启动起来，那么现在就是一主（6380）两从（6381 和 6382），然后我们下线主节点（6380）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzuTh9F7YaEJgJsRCv1J1IfZ0Kzf5iaBlseVMnoUq4Tu3KmTIRoqtVDhA/640?wx_fmt=png)

*   我们来看一下 26381 这个 Sentinel，发现没有执行选主（因为必须有个 Sentinel 认为主节点下线，才会执行选主；而现在只剩下这一个 Sentinel 了，所以无法执行选主操作）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzqgJEPb4f2bjUtzp24ldorfAlFHZpajzjwfwBztJg0zjiasBdcyDFJMg/640?wx_fmt=png)

9.6> Sentinel 对配置文件都做了什么  

---------------------------

*   我们随便打开一个 sentinel.conf 配置文件，查看最后面的几行，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzguvpCPZ2icnovvh6picX7ROluLo463v1cCsmmRiadn9oaqm4QGBzBp50A/640?wx_fmt=png)

*   我们再看看 redis.conf 配置文件，有什么变化
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzvrGIDY5ia5EdJxxd2QFDIx3NB88HNuUZJyuqGiaKMCtOPxCWT3I8vpDA/640?wx_fmt=png)

*   所以，我们发现，当 Sentinel 执行选主操作的时候，它也会去针对当前的主从情况对配置文件进行修改，比如当确定 6382 是从节点的时候，它就会往配置文件中加入 replicaof 127.0.0.1 6380，来配置主节点。
    

9.7> 实现原理
---------

*   当主节点发生异常挂掉了，Sentinel 会感知到并且执行自动的故障迁移，那它是如何感知到主节点异常的呢？下面我们来介绍一下 Sentinel 的三种定时监控任务。
    

### 9.7.1> INFO 指令获得最新节点拓扑图

*   每个 Sentinel **每隔 10 秒**就会向主从节点中发送 INFO 指令，通过该指令可以获得整个 redis 的节点拓扑图。那么这时候，如果有新的节点加入或者有节点退出集群，那么 Sentinel 就可以很快的感知到拓扑图的变化。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzL9m1PRMo2fArHicjykO5mFmC79LykEgdpHtaboN5b2adzkWKf9gD5fQ/640?wx_fmt=png)

### 9.7.2> 通过发布订阅获得 Master 节点和其他 Sentinel 的信息

*   每个 Sentinel **每隔 2 秒**会向指定频道上发布自己对 Master 节点是否正常的判断以及当前 Sentinel 节点的信息，并且通过订阅这个频道，可以获得其他 Sentinel 节点的信息和对 Master 节点是否存活的判断。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzJcaapSYX5PfdSxWcjIeBHZaCvjXSIb3CmkW86vibarPz8KkN4zPANUw/640?wx_fmt=png)

### 9.7.3> PING 指令心跳检测

*   每个 Sentinel **每隔 1 秒**会向所有节点（Sentinel 节点、Master 节点、Slave 节点）发送 PING 指令来进行心跳检测。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz1Zbe9lWtExM8qeQJ04iayBD2ngCwxPxP1MllbsJ4GzpSOgwqpTEboNg/640?wx_fmt=png)

### 9.7.4> 选举流程

*   当一个 Sentinel 判断主节点不可用的时候，会首先进行 “**主观下线**”，此时，这个 Sentinel 通过 **sentinel is-masterdown-by-addr** 指令获取其他哨兵节点对主节点的判断，如果当前哨兵节点对主节点主观下线的票数超过了我们定义的 quorum 值，则主节点被判定为 “**客观下线**”。
    
*   Leader Sentinel 节点会从原主节点的从节点中选出一个新的主节点，选举流程如下：
    

*   首先，过滤掉所有主观下线的节点
    
*   选择 slave-priority 最高的节点，如果有则返回，没有就继续下面的流程
    
*   选择出复制偏移量 offset 最大的节点，如果有则返回，没有就继续下面的流程
    
*   选择 run_id（服务器运行 ID）最小的节点
    

*   在选择完毕后，Leader Sentinel 节点会通过 **SLAVEOF NO ONE** 命令让选择出来的从节点成为主节点，然后通过 **SLAVEOF** 命令让其他的节点成为该节点的从节点。
    

十、Redis Cluster  

==================

10.1> 概述
--------

*   Redis3.0 开始引入了去中心化分片集群 Redis Cluster。
    
*   传统的 Redis 集群是基于主从复制 + 哨兵的方式来实现的。但是集群中都只有一个主节点提供写服务。
    

*   Redis Cluster 则采用多主多从的方式，支持开启多个主节点，每个主节点上可以挂载多个从节点。
    
*   Cluster 会将数据进行分片，将数据分散到多个主节点上，而每个主节点都可以对外提供读写服务。这种做法使得 Redis 突破了单机内存大小限制，扩展了集群的存储容量。并且 Redis Cluster 也具备高可用性，因为每个主节点上都至少有一个从节点，当主节点挂掉时，Redis Cluster 的故障转移机制会将某个从节点切换为主节点。
    

*   Redis Cluster 是一个去中心化的集群，每个节点都会与其他节点保持互连，使用 gossip 协议来交换彼此的信息，以及探测新加入的节点信息。并且 Redis Cluster 无需任何代理，客户端会直接与集群中的节点直连。
    

10.2> 分片方式
----------

### 10.2.1> 哈希取模

*   这种方式就类似我们使用 HashMap 时选址的方式，只要 hash 计算出来的值够散列，那么每个 key 都可以均匀的分散到 N 个节点上。
    
*   但是它存在的问题就是，如果要扩容或缩容，会导致 key 重新计算存储位置，从而导致缓存失效。
    

### 10.2.2> 一致性哈希

*   一致性哈希算法将整个哈希值空间组织成一个虚拟的圆环，其范围为 0 ~ 2^32-1，如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz8ZOs7iaYFRn78NhVbeBfAqZZ2JdBPhbsrK8iaNFDKCM0gic7T2BDCQY4A/640?wx_fmt=png)

*   一致性哈希算法的原理
    
    我们会先对 Key 计算它的 hash 值，从而确定它在环上的位置。然后从该位置沿着环顺指针地走，找到的第一个节点，便是这个 Key 应该存放的服务器节点的位置。
    
*   当我们向集群中增加或减少节点时，就无需像哈希取模算法那样，对整个集群 Key 的位置进行重新计算。一致性哈希算法将增减节点的影响限制在相邻的节点上，比如：我们在 node2 与 node4 之间增加一个节点 node5，则只有 node4 中的一部分数据会迁移到新增节点上；如果我们想要将 node4 节点置为下线状态，则 node4 节点的数据只会迁移到 node 3 中，其他节点无影响。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzTcRia2MCerGSUsyv7nlgo0mz0GZHYnR5cDxZ6xzyFCJSX7RdZnsetTg/640?wx_fmt=png)

*   一致性哈希算法的缺点
    

当节点比较少时，增删节点对单个节点的影响会很大，从而导致出现数据不均衡的情况。拿上图来举例，当我们删除任意一个节点，都会导致集群中的某一个节点的数据量由总数据的 1/4 变为 1/2。

### 10.2.3> 虚拟节点 + 一致性哈希

*   该方案在一致性哈希的基础上，引入了虚拟节点这一概念。原本是由实际节点来 “抢占” 哈希环的位置，现在则是将虚拟节点分配给实际节点，然后由虚拟节点来抢占。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzcticha9uibyfcJssP11Ahzcjqncx3CgVLM3EwMCgjl4zE5V2mt5cEJicg/640?wx_fmt=png)

*   在引入了虚拟节点这一概念后，数据到实际节点的映射关系就变成了数据到虚拟节点，再由虚拟节点到实际节点了。Redis 集群便是采用了这种方案。一个集群包含 16384 个哈希槽（hash slot）也就是 16384 个虚拟节点。譬如，我们的集群有三个节点，那么：
    

*   Master1 节点负责处理 0～5460 号 slot
    
*   Master2 节点负责处理 5461～10922 号 slot
    
*   Master3 节点负责处理 10923～16383 号 slot
    

*   当我们在集群中新增了一个节点 Master4，那么集群只需要将 Master1，Master2，Master3 中负责的一部分 hash slot 分配给 Master4 节点就可以了；
    
*   如果要移除某一个节点，也只需要将该节点负责的 hash slot 分配给其他的节点即可。这样集群便实现了良好的可扩容性。同时，由于存在 16384 个虚拟节点，那么这些 hash slot 在哈希环上可以分布均匀，从而实现负载均衡。
    

10.3> 搭建集群（3 主 3 从）
-------------------

*   由于 Redis Cluster 要求必须要至少 6 个节点，所以我们就以配置 3 主 3 从为例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzpXFkfQOBdk7rzfxYVzuMgDJeMVylbRr14dQ5KGuWMOc76VKQNHqoFA/640?wx_fmt=png)

*   修改 redis-6390.conf~redis-6395.conf 配置文件
    

```
# 配置集群节点对应的端口号(分别为6390，6391，6392，6393，6394，6395)
port 6390
# 守护进程开启
daemonize yes
# 关闭保护模式
protected-mode no
# 将集群开启
cluster-enabled yes
cluster-config-file nodes-6390.conf(分别为6390，6391，6392，6393，6394，6395)

```

*   启动集群中 redis 服务
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzZXWeQGrPo4ZWPKibwsuiaMjPbcohkF8BREibcqIsA1rkXMicH6ibPDn5RUA/640?wx_fmt=png)

*   分配主从（--cluster-replicas 1 表示创建一主一从）
    

```
./redis-cli --cluster create 127.0.0.1:6390 127.0.0.1:6391 127.0.0.1:6392 127.0.0.1:6393 127.0.0.1:6394 127.0.0.1:6395 --cluster-replicas 1

```

*   执行完毕结果如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzYIjTxMQe13k7pUXgRs4OicQTBptiaO6WL2X9aJHJMwEbWrdV7DA85Hew/640?wx_fmt=png)

*   我们来登录 6390，看看他在集群中的角色信息是什么
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzVKEXUHhSalgo7PTQ3Sa9lL4Ukjkxfm4MpG56n9uV9jbQGHEBhc7MNw/640?wx_fmt=png)

*   我们在 6390 客户端中添加一条记录，我们发现，它根据 key 值确认了 slot=12965，然后将数据存储到了 6392 这个节点上，并且客户端也切换为 6392 了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzXAAKsCZPZWv4hEN5iboaic6jduh6sQe51S8IJW5J6yR2QA1OuZGSGAbA/640?wx_fmt=png)

10.4> 部署过程中可能出现的异常  

---------------------

*   配置完集群后，可能会报如下错误，这说明 16384 个槽位没有分配完
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzBd30Asazql7MQYSmgIfppwIGAHEtzAOfPRnPpJ5VGhzGphFQcN7dicw/640?wx_fmt=png)

*   我们通过如下指令就可以进行检查和修复
    

```
redis-cli --cluster check 172.17.0.2:6379
redis-cli --cluster fix 172.17.0.2:6379 #官方修复功能

```

*   修复后的结果如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzIFCNsJibCghgL9LmFZ4FJVFEHu9lp1r0KKQfGV0rpvSyDaGj4Sj5Krw/640?wx_fmt=png)

十一、缓存常见问题及解决方案  

=================

11.1 > 缓存穿透（查不到数据）
------------------

### 11.1.1> 概述

*   当用户想要查询一个数据，发现 Redis 中不存在，也就是所谓的缓存没有命中，于是这个数据请求就会打到数据库中。结果数据库中也不存在这条数据，那么结果就是什么都没查询出来。那么当用户很多时候的查询，缓存中都没有数据，请求直接打到数据库中，这样就会给数据库造成很大的压力，缓存的作用也就几近于失效了，那么这种情况就叫做缓存穿透。
    

### 11.1.2> 解决方案

*   方案一：保存空值
    

*   当数据库中也查询不到数据时，那么将返回的空对象也缓存起来，同时设置一个过期时间，之后再访问这个数据将会从缓存中获取，从而起到保护数据库的作用。
    
*   例如：查询 userId=100 的用户信息（key=[userId]，value=[用户 json]），那么如果缓存和 DB 中都不存在，则在缓存中保存一条 key=100，value="" 的数据，那么用户再查询 userId=100 的时候，就直接可以返回空了。不需要查询 DB。
    

*   方案二：布隆过滤器
    

*   步骤 1：将数据库所有的数据加载到布隆过滤器。
    
*   步骤 2：当有请求来的时候先去布隆过滤器查询，判断查询的数据是否存在。
    
*   步骤 3：如果 Bloom Filter 判断数据不存在，那么直接返回空给客户端。
    
*   步骤 4：如果 Bloom Filter 判断数据存在，那么则查询缓存或 DB（下图以 Redis 中不存在数据为例）。
    
*   步骤 5：将 DB 中查询的结果返回给客户端（并且缓存到 Redis 中）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzmhKojGzFstcJicfZQowKOenWoiaASnjs5ULNNDa2OwL8icNltmFiaWO2fg/640?wx_fmt=png)

11.2> 缓存击穿（高并发查询某数据，且缓存过期）  

-----------------------------

### 11.2.1> 概念

*   指一个非常热点的 key，在不停的高并发请求着，那么当这个 key 在缓存中失效的一瞬间，持续对这个 key 的高并发就击穿了缓存，直接请求到了数据库，就像在一个屏障上早开了一个洞。
    
*   当热点 key 过期失效的一瞬间，高并发突然融入，会对数据库突然造成巨大的压力，严重的情况甚至会造成数据库宕机。
    

### 11.2.2> 解决方案

*   方案一：设置热点数据永不过期
    

*   从缓存层面来看，没有设置过期时间，所以不会出现热点 key 过期后所产生的缓存击穿问题。
    

*   方案二：加互斥锁
    

*   使用分布式锁，当缓存数据过期后，保证对每个热点 key 同时只有一个线程去查询后端服务，并将热点数据添加到缓存。
    

11.3 > 缓存雪崩（缓存大批量失效或 Redis 宕机）
------------------------------

### 11.3.1> 概念

*   指在某一个时间段，缓存集中过期失效，或 Redis 宕机，导致针对这批数据的查询都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。
    
*   其实缓存集中过期，倒不是最致命的，比较致命的是 Redis 发生节点宕机或断网。因为缓存集中过期后，数据库压力增大，但是随着缓存的创建，压力也会逐渐变小。但是 Redis 服务节点宕机，对数据库服务器造成的压力是不可预知的，很有可能是持续压力而最终造成数据库宕机。
    

### 11.3.2> 解决方案

*   方案一：配置 Redis 集群
    

*   通过配置 Redis 集群，提升高可用性，那么即使挂掉几个 Redis 节点，集群内的其他 Redis 节点依然可以继续对外提供服务。
    

*   方案二：限流降级
    

*   缓存失效后，通过加锁或队列来控制读取数据库且写入缓存的线程数量。
    

*   方案三：数据预热分散过期时间
    

*   在正式部署之前，先把可能被高频访问的数据预先访问一遍，这样大部分热点数据就加载到缓存中了，并且通过设置不同的过期时间，让缓存失效的时间尽量均匀，防止同一时刻大批量缓存失效。
    

11.4> 布隆过滤器
-----------

### 11.4.1> 概念介绍

*   布隆过滤器（Bloom Filter）是 1970 年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都比一般的算法要好的多，缺点是有一定的误识别率和删除困难。
    
*   布隆过滤器的特征是：
    
    它可以判断某个数据**一定不存在**，但是**无法判断一定存在**。（缺失有点拗口，但当我们介绍完它的原理，就很容易明白了）
    

### 11.4.2> 使用场景

*   原本有 10 亿个号码，现在又来了 10 万个号码，如何快速判断这 10 万个号码是否在 10 亿个号码库中？
    
*   需要爬虫的网站千千万万，对于一个新的网站 url，我们如何判断这个 url 我们是否已经爬过了？
    

*   针对上面的需求，我们一般会想到两种解决方案：
    

*   方案一：将 10 亿个号码存入数据库中，进行数据库查询，准确性有了，但是速度会比较慢。
    
*   方案二：将 10 亿号码放入内存中，比如 Redis 缓存中，这里我们算一下占用内存大小：10 亿 * 8 字节 = 8GB，通过内存查询，准确性和速度都有了，但是大约 8gb 的内存空间，挺浪费内存空间的。
    

*   那么对于类似这种，海量数据集合，如何准确快速的判断某个数据是否在大数据量集合中，并且不占用内存？那么布隆过滤器应运而生了。
    

### 11.4.3> 实现原理

#### 11.4.3.1> 布隆过滤器是什么？

*   它是一种数据结构，是由一串很长的二进制向量组成，也可以将其看成一个二进制数组。既然是二进制，那么里面存放的不是 0，就是 1，但是初始默认值都是 0。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRznoASmfdIAiaThxrlzlucib6libyj8w3q8dOTZM0SQrRzRsMGM2eddV2WA/640?wx_fmt=png)

#### 11.4.3.2> 添加数据  

*   当要插入一个元素时，将其数据分别输入 k 个哈希函数，产生 k 个哈希值。以哈希值作为位数组中的下标，将所有 k 个对应的比特置为 1。
    
*   比如，下图 hash1(x)=1，那么在第 2 个格子将 0 变为 1（数组是从 0 开始计数的），hash2(x)=6，那么将第 5 个格子置位 1，hash3(x)=16，那么将第 16 个格子置位 1，依次类推。如下图所示（只演示 hash1~hash3）：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRz5ibXyKiahF5xOlUIIcaYxZmhLicqEyNyNMibOUDyZGbRhv7uTaYAaGCGgQ/640?wx_fmt=png)

#### 11.4.3.3> 判断数据是否存在  

*   知道了如何向布隆过滤器中添加一个数据，那么新来一个数据，我们如何判断其是否存在于这个布隆过滤器中呢？
    
*   很简单，我们只需要将这个新的数据通过上面自定义的几个哈希函数，分别算出各个值，然后看其对应的地方是否都是 1，**如果存在一个不是 1 的情况，那么我们可以说，该新数据一定不存在于这个布隆过滤器中**。
    

*   那么反过来说，如果通过哈希函数算出来的值，对应的地方都是 1，那么我们能够肯定的得出：_这个数据一定存在于这个布隆过滤器中吗？_
    
*   答案是否定的，因为多个不同的数据通过 hash 函数算出来的结果是会有重复的，所以会存在某个位置是别的数据通过 hash 函数置为的 1。即：“假阳性”（false positive）如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzgyg6fuibsjdtCH3nAX4hAmEVfxQAWGUuqak7VaBxiaGZEM4fYzJ1vxQA/640?wx_fmt=png)

十二、高频 Redis 面试问题  

===================

12.1> 为什么 Redis 是单线程的
---------------------

*   官方的回复
    
    因为 Redis 是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且 CPU 不会成为瓶颈，那就顺理成章地采用单线程的方案了。Redis 每秒可以处理十万以上的请求。
    
*   采用了单线程之后，可以避免使用锁实现的复杂性和性能消耗
    
*   采用了单线程，就避免了多线程导致的 CPU 上下文切换带来的性能消耗
    

12.2> Redis 如何保证多客户端连接的时候，依然高吞吐
-------------------------------

*   redis 采用非阻塞 I/O 和 I/O 多路复用技术来保证在多连接的时候，系统的高吞吐量。
    

### 12.2.1> 非阻塞 I/O

*   当我们启动一个客户端时，客户端于服务端建立连接，并通过套接字 Socket 来处理他们之间的请求。比如我们执行了 get muse 指令，执行过程如下所示：
    

*   步骤 1：服务端要先监听客户端的请求——**listen**;
    
*   步骤 2：客户端请求发过来的时候与其建立连接——**accept**；
    
*   步骤 3：紧接着服务端需要从套接字 Socket 中读取客户端的请求——**recv**；
    
*   步骤 4：服务端对请求进行解析操作——**parse**；
    
*   步骤 5：解析出来的请求类型为 “get”，key 为 “name”，然后再根据 key 获取对应的 value；
    
*   步骤 6：将响应值返回给客户端，即：向 Socket 中写入数据——**send**；
    

*   但是，其中存在的一个问题是，Socket 默认的读写方式是阻塞的，当 Redis 通过 recv 或 send 向客户端读写数据时，如果数据一直没有到达，那么 Redis 主线程就会一直处于阻塞状态。
    
*   而 Redis 并没有采用默认的阻塞方式，它采用的是 **Socket 的非阻塞模式**，在非阻塞 I/O 模式下，读写方法不会产生阻塞，而是能读多少就读多少，能写多少写多少。当客户端一直不发送数据时，主线程也不会傻傻的一直等到，而是直接返回，去做其他的事情。
    

### 12.2.2> I/O 多路复用机制

*   Redis 采用 I/O 多路复用机制，使得 Redis 在单线程模式下依然可以高效的处理多个 I/O 流
    
*   Redis **基于 Reactor 模式**开发了自己的**文件事件处理器**（File Event Handler）。
    

*   文件事件处理器是由**套接字**，**I/O 多路复用程序**，**文件事件分派器**，**事件处理器**构成的。
    
*   I/O 多路复用程序（epoll / kqueue / select...，依据操作系统的不同，会采用不同的函数，Linux 使用的是 epoll，Mac OS 则是 kqueue）会监听多个套接字，每当一个套接字准备执行应答，读写，关闭等操作时，就会**产生对应的文件事件**，这些事件会存储到一个**事件队列**中，并由队列向文件事件分派器传送。而文件事件分派器会根据这些事件对应的类型来调用不同的事件处理器执行相应的事件回调函数。I/O 多路复用机制使得 Redis 无需一直轮询是否有请求发生，这样就避免了资源的浪费。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzyiaSRmUuhOO0UNmyl4TwGrOGjk7qu4n9Ql4zT24e7dJxia1mcqS0iaGNA/640?wx_fmt=png)

12.3> 如何从海量数据中查询除满足某一固定前缀的 key  

---------------------------------

*   针对这类前缀查询，我们有两种查询方案，分别为：keys 和 scan
    

### 12.3.1> keys

*   语法：**KEYS pattern**
    
*   在数据量不大的情况下，我们可以使用 keys 的方式查询除所有符合 pattern 的 key，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzp67zt4MPzK3C0iaUVpBibo3oAtnbyrkx5Jln5PxWSvg2Wcg1ttQppu8g/640?wx_fmt=png)

*   但是在海量数据情况下，如果使用 keys 命令执行，查询速度就会非常的慢了。
    
*   keys 命令会将所有的 Key 全部遍历一遍，其时间复杂度为 O(N)。不仅如此，在数据量极大的情况下，该命令会**阻塞 Redis I/O 多路复用的主线程**一段时间，如果我们的 Redis 正在为线上的业务提供服务，主线程一旦被阻塞，那么在此期间其他的发送向 Redis 服务端的命令都会被阻塞，从而导致 Redis 出现卡顿，引发响应超时的问题。
    

### 12.3.2> scan

*   语法：**SCAN cursor [MATCH pattern] [COUNT count] [TYPE type]**
    
*   scan 命令的时间复杂度同样为 O(N)，不过它需要迭代多次才能返回完整的数据，并且每次返回的数据**有可能会产生重复**。
    

*   cursor 为查询游标，游标从 0 开始，也从 0 结束。每次返回的数据中，都有下一次游标该传入的，我们通过这个值，再去进行下一轮的迭代。
    
*   scan 命令通过游标分布执行，**不会产生线程阻塞**，所以非常适合使用于海量数据的生产环境下。
    

*   scan 命令并不保证每次执行都会返回某个给定数量的元素，每次迭代返回的元素数量都是不确定的。所以，即便是返回了 0 条数据，只要返回的游标值不为 0，就说明需要继续迭代，**只有当返回的游标值为 0 时，才代表迭代完毕**。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzpx1dib8U3tb9Eyt0ibOJYxZiappOyuwOfw2TicngPKHYRV0nmM5ASN9GAA/640?wx_fmt=png)

12.4> 什么是 Redis Pipeline
------------------------

*   Redis 是一个基于 TCP 协议的 NoSql 数据库，而客户端与 Redis 服务端之间的交互过程可以分为以下几个简略的步骤：
    

*   步骤 1：客户端向 Redis 服务端发起一个 request 请求
    
*   步骤 2：Redis 服务端收到请求，等待处理
    

*   步骤 3：Redis 服务端处理请求
    
*   步骤 4：Redis 服务端将结果 response 响应给客户端
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzRPFuXyQUNIv7MFia0yv3zGBQHwGR7w7FEoEr6AUiapFANLbqtvWtoIDQ/640?wx_fmt=png)

*   由于 Redis 服务端需要等待上一条命令响应完成后再去执行下一条命令，如果此时需要执行大量的命令，通过传统的 Redis 请求 / 响应模型便会严重影响效率，这中间不仅仅多了 RTT（Round Time Trip）即客户端与服务端来回交互的时间，而且还会频繁调用系统 IO 来发送网络请求。为了解决这样的问题，Redis 为我们提供了 Pipeline（管道）技术。
    
*   Pipeline 允许客户端一次发送多条命令，Redis Server 则会对这些命令进行 “批处理”，执行完毕后将结果一次性发送给客户端。Pipeline 可以将多次 IO 往返时间缩减为一次，前提是 Pipeline 执行的指令之间没有依赖的相关性，如果指令之间有依赖，则还是建议分批去发送指令。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCicujI9669YWJ0fxbSsX9cRzVoUwsmFCoReNsa4duCI9vFdWa4MVoAm81bQdR9Ss4fEsvn5dRXrV7g/640?wx_fmt=png)