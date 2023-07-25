> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MTE0NTc0Ng==&mid=2247484835&idx=1&sn=a8b93f49d495b0530f9235a851461326&chksm=e911475ede66ce480b990f15a440620b94ddc464dd36743abe9fe371034f2cdb2a10ed598197&scene=178&cur_album_id=2162186087471906816#rd)

    在电商等常见的搜索业务场景中，Elasticsearch 扮演着举足轻重的作用。它对于数据的准实时搜索可以达到很高的查询效率，并且天生自带的分布式、高可用、易扩展的能力，也使其具有了十足的魅力。那么，下面就是本篇文章的大纲结构。

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaDtXth7YV7vJHbJFaMISRtthYT38KV0wiaJLlicEtPAbreoI3cc6La29A/640?wx_fmt=png)

    话不多说，下面就进入正题吧~  

一、简介  

=======

1.1> 为什么需要 es
-------------

*   当我们想要模糊查找某些数据的时候，在关系型数据库，可以使用 like '% 手机 %'这种方式，但是，如果我们在搜索的场景下，比如我想要买一款冬天穿的毛衣，我们会搜索：“毛衣  厚  冬天 男”，那么就会按照我们的搜索条件，查询出相关产品了，如果想要通过关系型数据库来实现，就会非常麻烦了。并且，在海量数据下，like 的查询性能也不高。那么，我们怎么去解决这个问题呢？关于这个问题，我们可以通过使用 Elasticsearch 来实现。
    

1.2> 什么是 es
-----------

*   首先，我们先来百度一下 Elasticsearch 到底是什么？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaX6ciaLSlEGpaTogkAAjwJwq5Dx4of91Hun2ImNWQsLw5mVO7VjQEqrg/640?wx_fmt=png)

*   ES 不是数据库，它适合于海量数据、更新频率很低的数据（ES 没有事务也不适合处理并行更改数据）。
    
*   ES 更适合处理关系相对简单稳定的数据，一些关系比较复杂的数据用 mysql 这样的关系数据库用 sql 很容易实现，但是 es 就相当的复杂了。
    

*   使用案例
    

*   维基百科使用 Elasticsearch 来进行全文搜做并高亮显示关键词，以及提供 search-as-you-type、did-you-mean 等搜索建议功能。
    
*   英国卫报使用 Elasticsearch 来处理访客日志，以便能将公众对不同文章的反应实时地反馈给各位编辑。
    
*   StackOverflow 将全文搜索与地理位置和相关信息进行结合，以提供 more-like-this 相关问题的展现。
    
*   GitHub 使用 Elasticsearch 来检索超过 1300 亿行代码。
    
*   每天，Goldman Sachs 使用它来处理 5TB 数据的索引，还有很多投行使用它来分析股票市场的变动。
    

1.3> Doug Cutting 和 Lucene 和 Hadoop
-----------------------------------

*   https://www.cnblogs.com/doit8791/p/9556821.html
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwacYP5zStmmDlAKLv4QpqpudgZnCfEtNznib6MiajGicRJRNHy91slGtXUQ/640?wx_fmt=png)

1.4> Elasticsearch 和 Solr
-------------------------

### 1.4.1> Elasticsearch 的优缺点

【简介】

*   Elasticsearch 是一个**实时的分布式**搜索和分析引擎。
    
*   Elasticsearch 是一个建立在全文搜索引擎 Apache Lucene™ 基础上的搜索引擎，可以说 Lucene 是当今最先进，最高效的全功能开源搜索引擎框架。但是 Lucene 只是一个框架，要充分利用它的功能，需要使用 JAVA，并且在程序中集成 Lucene。需要很多的学习了解，才能明白它是如何运行的，Lucene 确实非常复杂。
    

*   Elasticsearch 使用 Lucene 作为内部引擎，但是在使用它做全文搜索时，只需要使用统一开发好的 API 即可，而不需要了解其背后复杂的 Lucene 的运行原理。
    

【优点】

*   Elasticsearch 是**分布式**的（天生带有易扩展高可用的特性），不需要其他组件，分发是实时的，被叫做”Push replication”。
    
*   Elasticsearch 完全支持 Apache Lucene 的**接近实时的搜索**（新增到 ES 中的数据在 1 秒后就可以被检索到）。
    

*   处理多租户（multitenancy）不需要特殊配置，而 Solr 则需要更多的高级设置。
    
*   Elasticsearch 采用 Gateway 的概念，使得完备份更加简单。
    

*   各节点组成对等的网络结构，某些节点出现故障时会自动分配其他节点代替其进行工作。
    

【缺点】

*   **非实时性**的搜索的速度没有 Solr 快。
    
*   Elasticsearch **仅支持 json** 文件格式。
    
*   版本更新太多，比如 6.x 和 7.x 在使用上也有不少的区别。
    

### 1.4.2> Solr 的优缺点

【简介】

*   Solr（读作 “solar”）是 Apache Lucene 项目的开源企业搜索平台。其主要功能包括全文检索、命中标示、分面搜索、动态聚类、数据库集成，以及富文本（如 Word、PDF）的处理。Solr 是高度可扩展的，并提供了分布式搜索和索引复制。Solr 是最流行的企业级搜索引擎，Solr4 还增加了 NoSQL 支持。
    
*   Solr 是用 Java 编写、运行在 Servlet 容器（如 Apache Tomcat 或 Jetty）的一个独立的全文搜索服务器。Solr 采用了 Lucene Java 搜索库为核心的全文索引和搜索，并具有类似 REST 的 HTTP/XML 和 JSON 的 API。Solr 强大的外部配置功能使得无需进行 Java 编码，便可对 其进行调整以适应多种类型的应用程序。Solr 有一个插件架构，以支持更多的高级定制。
    

*   因为 2010 年 Apache Lucene 和 Apache Solr 项目合并，两个项目是由同一个 Apache 软件基金会开发团队制作实现的。提到技术或产品时，Lucene/Solr 或 Solr/Lucene 是一样的。
    

【优点】

*   Solr 有一个更大、更成熟的用户、开发和贡献者社区。
    
*   支持添加**多种格式**的索引，如：HTML、PDF、微软 Office 系列软件格式以及 JSON、XML、CSV 等纯文本格式。
    

*   Solr 比较成熟、稳定。
    
*   不考虑建索引的同时进行搜索，速度更快。
    

【缺点】

*   建立索引时，搜索效率下降，**实时索引搜索效率不高**。
    

### 1.4.3> Elasticsearch 与 Solr 的比较

*   当单纯的对已有数据进行搜索时，Solr 更快
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa34KusCmTRNWpGSZEibqPp1NuO8gLfOzt9CuZe5PpHxeyDDwiadjFCGMQ/640?wx_fmt=png)

*   当实时建立索引时，Solr 会产生 io 阻塞，查询性能较差，而 ES 具有明显的优势
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaibKicStwvTf0vh0mpNhujFc1OX8rsTrK70OtNibIO9VVP0LKMM49c6dpQ/640?wx_fmt=png)

*   随着数据量不断增加，Solr 的搜索效率会变得更低，而 Elasticsearch 却没有明显的变化
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa2mawWFBOJdGqXMZLnNB9O6HW0N3IoiadGL0epW97cVb1jzgQyWr8Ulw/640?wx_fmt=png)

*   综上所述，**Solr 的架构不适合实时搜索的应用**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa7CNZ2jWcTauYM3iaPkicnWGA2YnjiaHoE8w9T3BibeT2q9ibBWhdSVvnZ5g/640?wx_fmt=png)

【总结】

    二者安装都很简单；

*   Solr 利用 Zookeeper 进行分布式管理，而 Elasticsearch 自身带有分布式协调管理功能;
    
*   Solr 支持更多格式的数据，而 Elasticsearch 仅支持 json 文件格式；
    

*   Solr 官方提供的功能更多，而 Elasticsearch 本身更注重于核心功能，高级功能多有第三方插件提供；
    
*   Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于 Elasticsearch。
    

*   Solr 是传统搜索应用的有力解决方案，但 Elasticsearch 更适用于新兴的实时搜索应用。
    

1.5> ELK
--------

*   https://www.elastic.co/cn/what-is/elk-stack
    
*   那么，ELK 到底是什么呢？“ELK” 是三个开源项目的首字母缩写，这三个项目分别是：**Elasticsearch**、**Logstash** 和 **Kibana**。
    

*   **Elasticsearch**：是一个搜索和分析引擎。
    
*   **Logstash**：是服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如 Elasticsearch 等 “存储库” 中。
    

*   **Kibana**：则可以让用户在 Elasticsearch 中使用图形和图表对数据进行可视化。
    

*   Elastic Stack 是 ELK Stack 的更新换代产品。
    
*   官网介绍
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaOlyl1K1tWIiceG6crhE1wBVmqiaE92iaW0Q2ZjTF5NmUzJnbz75cAlTFQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaZY8BIwJocBKXdR1doaxw4f6kZaiaxVHNcIWM1PAv9YgTGRnTVOHnDUQ/640?wx_fmt=png)

二、安装  

=======

2.1> 安装 Elasticsearch
---------------------

*   不需要安装，解压运行即可
    

    https://www.elastic.co/cn/elasticsearch/

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwawGD2E2dqZwHnSqlDbaXHypqiaOI4ftnex5UOlia9FyPZGLXeURTIZ6IQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwafh1J0uSXzmicDpmquCyBhYs0lBJHlb9EokrRmhiaib1cHSFzb9KXeHxlg/640?wx_fmt=png)

  ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa9ngMQdsnBNqLfMGNqWGMfqjStts7nXFicSTyjKdRqDcZ1eFxuID95MQ/640?wx_fmt=png)

*   运行 bin/elasticsearch
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwacb8pBickbzsic8k1oz6A1kt8tjgwjHyibhX0zmaN4FjHJQpJsEMib8TuYQ/640?wx_fmt=png)

*   访问 http://localhost:9200/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwabXW8WOB9KkJaiaIAicHC34jRcYXTLXv3tNNBI9LXrCr8yU6T5PKW4Z5A/640?wx_fmt=png)

2.2> 安装 Node.js
---------------

*   下载 Node.js 并安装，官网 https://nodejs.org/en/
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwalBEyp8IRDCNhawSJQZyWaDX20BV7wXMMibWQomBACwWIAdKEjDzyW6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwalBEyp8IRDCNhawSJQZyWaDX20BV7wXMMibWQomBACwWIAdKEjDzyW6Q/640?wx_fmt=png)

*   通过 node -v 和 npm -v 查看是否成功
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaZz7KVkPXL13Gkaa2jazGrbsLNcG8cqJu3QibciayctoGNuQEmMGy8DlA/640?wx_fmt=png)

*   npm install 安装依赖出现 PhantomJS not found on PATH PhantomJS not found on PATH Downloading https://github
    

1>  手工下载 https://github.com/Medium/phantomjs/releases/download/v2.1.1/phantomjs-2.1.1-windows.zip

2> 将下载下来的 phantomjs-2.1.1-macosx.zip 放到 / var/folders/12/hwj74fs53f79l1mcj909s8wh0000gn/T/phantomjs 路径下

2.3> 安装 Elasticsearch-head
--------------------------

*   安装 elasticsearch-head 需要有 npm 支持
    

*   安装指令
    

```
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
open http://localhost:9100

```

*   官网截图
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaHT1NTjQPTOf83nBfnTHCHypVVc7xUFM3Wepmm4eVkzbQ8wMlvFVBhQ/640?wx_fmt=png)

*   启动 elasticsearch-head
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa9h8vkR38ib0hlY2EJpQKy7tFT30NK7ZhXP3MajdJKPrCiaJr4nicLUia2g/640?wx_fmt=png)

*   elasticsearch-head 界面如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaPibnyqHlUMffRHoJlGLMibNqJiaiboQ0fSWdL6Umq0xYTBoiaakgL6KfcNA/640?wx_fmt=png)

*   但是点击连接，没有反应，我们来看一下请求响应，发现是**由于 CORS 跨域问题**，导致了连接不上 ES（因为 head 是 9200 端口，而 es 是 9100 端口）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwawjPXeYybFJhdcJrq9gYiaeYtt44wjWNddbByicIAiaot5RJEnWhGHmiaZA/640?wx_fmt=png)

*   解决跨域问题。修改 es 的配置文件——**elasticsearch.yml**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaGb0IodDApibQ4Pnnol8hLAEVWL9etJaPnvfrvr22CJBUbZvHszLCVpg/640?wx_fmt=png)

*   在配置文件 elasticsearch.yml 的末尾加上如下配置：
    

```
http.cors.enabled: true
http.cors.allow-origin: "*"

```

*   重启 es，发现我们已经可以通过 head 连接到 es 了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaCDoDfv3Akf2ZYeCC5GzWcMDA9J5faEN8voxmMAuxGMpyUTmevhWbXA/640?wx_fmt=png)

2.4> 安装 Kibana
--------------

*   不需要安装，解压运行即可。
    

*   下载 Kibana，注意 **Kibana 的版本定要与 ES 版本一致** https://www.elastic.co/cn/start
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaiaOSIRCIYIfTdlF4FCT7XArfIGIibLIXkMzksve1Qic6ERFjiaQr95CQUw/640?wx_fmt=png)

*   下载之后解压，执行如下语句，开启 kibana
    
    bin/kibana
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaF6M2772iakV8GXyhdT5jib028mVSMib3VVLglcG72YxiawR0nibkGibYOYlw/640?wx_fmt=png)

*   访问 http://localhost:5601
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaiat7GmnxMSGSbqYpkEejPwRic7KIkYYoDhicJHIK2DVqgTGk6libgIoHZg/640?wx_fmt=png)

*   修改为中文显示，打开 kibana-7.15.1/config/**kibana.yml** 配置文件，按照提示修改，然后重启 kibana 即可。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaKmibMkg8tD01PKhkUSqvxOEhUoP3ibka6tiahUKq8ATiahmdbZABcLY9ww/640?wx_fmt=png)

*   访问 http://localhost:5601，我们发现，已经变为中文了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwagrVDxXpQSfGmlvb4fs7jQFRrmMwsuBcnL4ARXSrqDexWibIJxruslwA/640?wx_fmt=png)

三、ES 概念介绍  

============

3.1> index、type、document、field
------------------------------

*   ES 概念与关系型数据库概念对比
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaSmTgcibRED6TPCTfmzcVJs9F7lId5WG7Vg2icXgDVb1oRiav5xH8Hsfbg/640?wx_fmt=png)

*   索引
    
    索引是**映射类型**的容器，它是一个非常大的文档集合。索引存储了映射类型的字段和其他设置。然后他们被存储到了各个分片上。
    
*   类型
    
    类型是**文档**的逻辑容器，就像关系型数据库一样，表格是行的容器。类型对于字段的定义称为映射，比如：name 映射为字符串类型。
    
*   文档
    
    一个文档同时包含字段和对应的值，也就是同时包含 key:value，ES 是面向文档的，意味着索引和搜索数据的最小单位就是文档。
    
    文档的结构很灵活，不依赖预先定义的模式，它对于字段是非常灵活的，有时候，我们可以忽略字段或者动态的添加一个新的字段。
    

3.2> 分片、副本  

-------------

*   分片
    
    在大数据时代，单机是无法存储规模巨大的数据的。那么我们就将数据拆分成多个部分，然后存储到多台机器，构成大规模集群。那么这种数据拆分成若干个部分就叫做分片。
    
*   副本
    
    如果分片挂掉了，数据就丢失了。那么为了提高系统的可用性。我们把分片复制多个，这就是副本了。
    
    副本除了可以有备份的作用之外，还能够实现并行的读取操作，分担集群压力。
    
    但是，副本的产生，也会随之带来**数据一致性的问题**，即：有的副本写数据成功，但是有的副本写数据失败。
    
*   ES 将数据副本分为主从两部分，即：**主分片**（primary shard）和**副分片**（replica shard）
    
    主分片上的数据作为权威的数据。
    
    当写数据的时候，先写主分片，写入成功后再写副分片。
    
    恢复数据的时候，以主分片上的数据为准。
    
    当我们创建一个索引的时候，默认是 5 个分片，每个分片 1 个副本。   ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa5PCEwlVeHnRW5QILdGpA78L0vD50lrMMqzicjyicW78kEj3rEA2N4pPQ/640?wx_fmt=png)
    
*   分片是底层的**基本读写单元**。ES 利用分片将数据分发到集群内各处。**分片是数据的容器，文档保存在分片内，不会跨分片存储**。分片又被分配到集群内的各个节点里。当集群规模变化的时候，ES 会**自动**将集群中节点上的分片进行重新的分配和迁移，从而保证数据仍然**均匀分布**在集群里。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwau3Tq5ZV3csvaDH5xQiaZwickmOGt4tM7qtBsyocQDCE9l2unW0RvuEWw/640?wx_fmt=png)

*   默认的集群名称为 elasticsearch
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaiavWg64uwPiadBN49XOS5swia1YzS96Uhd3DPibltgVgzPEgkq9VvVFvfg/640?wx_fmt=png)

3.3> 倒排索引  

------------

*   在搜索引擎中**每个文件都对应一个文件 ID**，文件内容被表示为一系列**关键词的集合**（实际上在搜索引擎索引库中，关键词也已经转换为关键词 ID）。例如：“文档 1” 经过分词，提取了 20 个关键词，每个关键词都会记录它在文档中的**出现次数**和**出现位置**。
    
*   既然我们谈到了倒序索引，那么顾名思义，也会存在正序索引，那么我们下面来举个例子，对比一下这两种索引。
    
*   比如有如下 3 个文档：
    
    文档 1：今天我们一起学习 Elasticsearch
    
    文档 2：Elasticsearch 学习起来真有趣
    
    文档 3：今天大家下课一起吃烧烤怎么样
    
*   正序索引
    
    **文档的 ID ——> 关键词 N：出现的次数，出现的位置列表**
    
    文档 1：【今天】【我们】【一起】【学习】【Elasticsearch】
    
    文档 2：【Elasticsearch】【学习】【起来】【真】【有趣】
    
    文档 3：【今天】【大家】【下课】【一起】【吃】【烧烤】【怎么样】
    
*   倒序索引
    
    **关键词 N——> 文档 N 的 ID**
    
    【今天】：文档 1，文档 3
    
    【我们】：文档 1
    
    【一起】：文档 1，文档 3
    
    【学习】：文档 1，文档 2
    
    【Elasticsearch】：文档 1，文档 2
    
    ... ...
    
*   那么通过正序索引和倒序索引的对比，我们如果想要搜索关键词 “一起”，那么我们就可以迅速的知道这个关键词在文档 1 和文档 3 中存在。如果我们搜索 “我们一起”，就会迅速找到文档 1 包含关键词 “我们一起”，文档 3 值包含关键词 “我们”，那么针对这种搜索结果，文档 1 的 score 就比文档 3 要高了。
    

3.4> 字段类型  

------------

### 3.4.1> 概述

*   在创建索引的时候，我们可以不去指定字段类型，由 ES 去**自行决定**；我们也可以通过 **mappings** 的方式，指定索引中字段的类型。
    
*   我们先来看一下，ES 中支持的字段类型有哪些？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa5CHvqqSTf3KTxXB5ZmYMf3ZyZkP9ujnXbBicv2z0ia48IkWPOiaNC3Bicw/640?wx_fmt=png)

### 3.4.2> 字符串类型

#### a> string

*   string 类型在 ElasticSearch 旧版本中使用较多，从 ElasticSearch 5.x 开始不再支持 string，**由 text 和 keyword 类型替代**。
    

#### b> text

*   当一个字段是要被全文搜索的，比如 Email 内容、产品描述，应该使用 text 类型。设置 text 类型以后，字段内容会被分析，在生成倒排索引以前，字符串会被分词器分成一个一个词项。text 类型的字段不用于排序，很少用于聚合。
    

【特点】**会分词**，然后进行索引，支持模糊、精确查询但不支持聚合

#### c> keyword

*   keyword 类型适用于索引结构化的字段，比如：email 地址、主机名、状态码和标签。如果字段需要进行过滤 (比如：查找已发布博客中 status 属性为 published 的文章)、排序、聚合。keyword 类型的字段只能通过精确值搜索到。
    

【特点】**不进行分词**（分词器在 keyword 上没有作用），直接索引，支持模糊、精确查询并且支持聚合

*   如果**不指定类型**，ES 字符串将默认被同时映射成 text 和 keyword 类型，（一个字符串字段可以映射为 text 字段用于全文本搜索，也可以映射为 keyword 字段用于排序或聚合）会自动创建映射，如下是未指定类型的索引 student：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaBEK1IsUNKbT83ibjDJgrrf610hgFPicM4sbaKrHuRYvoq5TRrwPOQm1g/640?wx_fmt=png)

#### d> 实操对比 text 和 keyword

*   我们先来看一下这两个类型对文档内容如何处理的
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwareCYfeTEmGFyEsmQbibnicmqib3eb2KdcxezgU4gj7zhUUfK0vjmcicQqg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwan6u8W9sfVmicIhj0DMwUvI3IYuPsC3DAMZWmxse0xod8tudVMTiccFgg/640?wx_fmt=png)

*   我们创建一个索引，包含一个 text 类型的 name 和一个 keyword 类型的 desc。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwanayvdicicxhRmWYAEJ6ZOdk11t0pdOU7ic6UhKwaCkibDCoicAHN1U1jFMQ/640?wx_fmt=png)

*   然后向其中插入两个文档
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa7MW3WES2KJdJCOwhcT104dfRiazsW9jWN5YoI9r5ODZhZAlCMCtvJAw/640?wx_fmt=png)

*   我们来查询 text 类型的 name 字段
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaEvSEus101OlCnEQ4WnT4bhFiaCbtVtTLCb6ib6ibsk1iaKM0vCon0wIddg/640?wx_fmt=png)  

*   同样搜索 “缪斯”，在 keyword 类型的 desc 字段中，就只能查询出文档 001 了。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaGGXNfr37PbJG0ZagpBmGFgmuDQ4TibibHU3jV6qnweaMuBta1xOeyabg/640?wx_fmt=png)

3.5> ES 集群健康值颜色  

------------------

### 3.5.1> 概述

*   我们在 elasticsearch-head 界面上，会看到如下的信息。
    

    ![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa2LaufCPKdP2nbqHaKia1bJf1JjgETKrruic3Ljcmn2V0WlKichY6n7eKA/640?wx_fmt=png)

*   那么 ES 集群都包含哪些颜色呢？都代表什么含义呢？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwalutUGuxaDGJZpTbGmfXkjibaIDgj7Et723xJL70AvsjJcmrIgyv3HKA/640?wx_fmt=png)

*   Elasticsearch 集群颜色变黄色了要不要紧？
    
    Elasticsearch 集群黄色代表——分配了所有主分片，但至少缺少一个副本。没有数据丢失，因此搜索结果仍将完整。
    
    注意：您的高可用性在某种程度上会受到影响。如果更多分片消失，您可能会丢失数据。将黄色视为应该提示调查的警告。
    
*   我们安装的 Elasticsearch 集群为什么是黄色的？
    
    由于只有一个节点，因此群集无法放置副本，因此处于黄色状态。
    

【解决方案】

*   方案 1：可以将副本数降低为 0 个
    
*   方案 2：将第二个节点添加到群集，以便可以将主分片和副本分片安全地放在不同的节点上。 
    

### 3.5.2> 集群健康状态如何排查？  

*   查看集群状态
    
    **GET _cluster/health**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwak8TctYQ8gxYEfWL5kwC5kTKkGeyG2yKm9z9ZV8dHrOZEpW24v4RmlQ/640?wx_fmt=png)

*   查看分片状态
    
    **GET _cat/shards?v**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa8GhSWP92j4wZpIcmn3ch1st4s8jDhUQRw11ycry5SbkAL1CqD6ic9Sg/640?wx_fmt=png)

*   查看集群中不同节点索引的状态
    
    **GET _cat/shards?h=index,shard,prirep,state,unassigned.reason**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaPPgMnfARw27Q0ic3HrHLDTticoB30RGU4YOjk6oB8J3OJ1uxxmHVzH9g/640?wx_fmt=png)

*   查看 unsigned 的原因
    
    **GET /_cluster/allocation/explain**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwawlaHrGeFazhDW5BZQW2Q3h1glUM9tAjeaGQGoVGIdsia30LahKevYGA/640?wx_fmt=png)

【解释】

*   cannot allocate because allocation is not permitted to any of the nodes：无法分配，因为不允许分配给任何节点。
    

四、分词器  

========

4.1> 概述
-------

*   分词器的作用就像它的命名一样，能够一段文字拆分成多个词，比如："清华大学"，如果拆分为 "清","华","大","学"，那这种分词就失去了词汇的意义了。这里如果涉及中文分词的场景，建议使用 ik 分词器。
    

4.2> ES 内置分词器
-------------

### 4.2.1> Standard Analyzer（默认）

*   standard 是默认的分析器。它提供了基于语法的标记化（基于 Unicode 文本分割算法），适用于大多数语言。
    
*   【分词方式】区分中英文，英文按照空格切分同时大写转小写；中文按照单个词分词。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaTJS8VNPpOPwrGXoGd3WDcPjJRmtS82wq4nSE4EnjrjO0yNjHtzZICA/640?wx_fmt=png)

### 4.2.2> Simple Analyzer

*   simple 分析器当它遇到只要不是字母的字符，就将文本解析成 term，而且所有的 term 都是小写的。
    
*   【分词方式】先按照空格分词，英文大写转小写，不是英文不分词。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa8oO5zws1Gb98ljRFI8EcGNibP9UfMEZlr5UpZ4mMiaeZtC11KicPatibQw/640?wx_fmt=png)

### 4.2.3> Whitespace Analyzer

*   【分词方式】按空格分词，英文不区分大小写，中文不再分词
    

4.3> 安装  

----------

*   下载地址
    
    https://github.com/medcl/elasticsearch-analysis-ik/releases
    
*   解压放到 elasticsearch-7.15.1/plugins/ik 目录下
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaLCGG4134XViajpjEmtCe7WjWaPzqHv4evEHRXlFEpc23XdRibEF93l6Q/640?wx_fmt=png)

*   重启 Elasticsearch，从日志中看到了 ik 分词器已经被加载 
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaspDF81icGicDbHlMjWTZjC7GLIuAfKTw6DmIfthhfBFvmlZ2rK8Xc4XA/640?wx_fmt=png)

*   我们通过./elasticsearch-plugin list 指令来查看 Elasticsearch 中加载的插件有哪些
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaHKRAT0sDZ4Oa4WRbqAFhMUKzicBSUTGsa7sau72LGGt8xFHExTnezuA/640?wx_fmt=png)

4.4> 使用 ik 分词器  

-----------------

### 4.4.1> 使用 ik_smart

*   会做最粗粒度的拆分，比如：会将 “中华人民共和国国歌” 拆分为“中华人民共和国”，“国歌”，特点是，不会有重复的数据，适合 Phrase Query。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwayueh34A0CmQrOc7Moe8febicS5Cut1ay6vEFcsyXKDgKa3iaHialHxyZQ/640?wx_fmt=png)

### 4.4.2> 使用 ik_max_word

*   会将文本做最细粒度的拆分，比如会将 “中华人民共和国国歌” 拆分为“中华人民共和国”,“中华人民”，“中华”，“华人”，“人民共和国”，“人民”，“人”，“民”，“共和国”，“共和”，“和”，“国国”，“国歌”，会穷尽各种可能的组合，适合 Term Query；
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaZ7okCmhtcYTBXplx3FAsnlCiatiawSGdjxdUibZzXo7TfSSv4pHuv15zA/640?wx_fmt=png)

### 4.4.3> 自定义分词词汇

*   我们尝试分词一部经典的电影《夏洛特烦恼》，它正确的分词应该是 “夏洛”“特”“烦恼”。但是 ik 分词器不知道有“夏洛” 这个词，所以无论是 ik_smart 还是 ik_max_word，都无法把 “夏洛” 这个词拆分出来。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa22oW1EmKOicVxiaFK4bMfSGAwwia89mKxk4mFsLXqg4y0CiaEXUEDBakCA/640?wx_fmt=png)

*   要解决这个问题，我们就需要自己添加 "夏洛" 这个词到 ik 分词器的字典中。我们打开 elasticsearch-7.15.1/plugins/ik/config 下的 IKAnalyzer.cfg.xml 配置文件。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa6weKUPfheLLFWIxeZAsyfnOvUUnVqmEJs9DGpC6kKrd5evHC1tkg3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa3MiaN7e6qVjvqRmronkzEtajch03NKVoDMWz8pS8PuWCkwSGvchFHFw/640?wx_fmt=png)

*   我们创建 muse.dic，将 “夏洛” 加到字典中，并配置 ext_dict 标签，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaUKUeYZqgRh9mpbRKmu63ahgZscm7sv8Bbw7NAT62n7jiaXfhsJ4L1Ag/640?wx_fmt=png)

*   重启 ES，再次执行 ik_max_word 分词操作，发现已经有 “夏洛” 的词汇了。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwat8L6Wrjfc7Jjt04hnWttWJk4DnhUXTBXzy3VLibFfaMQjFjg1vmLiclw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaibo4D4yK4dHmrOtalajciaiaBLNLREGlcM1wpibfPd3wcrolumJwCrkLIg/640?wx_fmt=png)

五、实操  

=======

5.1> ES 请求规范
------------

*   ES 支持基于 RESTful 和 API 的方式进行命令请求，官方推荐使用 RESTful 风格的请求方式
    

*   Rest 风格支持（使用 HTTP 请求方式动词来表示对资源的操作）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwabibumfBTulvFfkmrPHstzrA2mficoGpvo860dHCwzpVOOEE9zbKX268A/640?wx_fmt=png)

*   ES 基于 REST 指令
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaGyoH8icKViaLolClpmpgqhV0IVLcdZS3UPWGiciaiahmRezs70QmXsZ3m8Q/640?wx_fmt=png)

【注意】如果不指定 TYPE_NAME 的值，则默认为_doc。

5.2> 创建索引  

------------

### 5.2.1> 不指定字段类型映射

*   第一种方式：创建无字段索引
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaIYphgHnyuCy9CicSGNDfoQwHmibkazI4ceXWddbVxiczp7RLOTgHtfYYA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaT9qCCZB6G05eXLKd9WlmdvyIXb8sYgqBNdicpSXxvRibbDjEKwUnlUXg/640?wx_fmt=png)

*   第二种方式：指定一个不存在的索引来创建文档（执行指令之前，是没有索引 student 的；创建文档会在 5.3 中详细介绍）
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa13a0Al8WZxHxfzYmWLjtoZKUd36GO8ohgn7HLTK1plHyxItzUcpMhg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaNKeyzMjuu0Aiam8qWLp1LNoHSRIJg0gFib6PMhIxnpPg4IsK2iav9ibhWQ/640?wx_fmt=png)

### 5.2.2> 指定字段类型映射

*   通过 mappings 进行指定字段类型，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwahDl4f8wp55aCQSUxfTGSrFS6KxysVibW5sXjmKD0Mld0DxibOp3WSVvQ/640?wx_fmt=png)

*   去 elasticsearch-head 上查看索引信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaYNlZ3lWhVEgjZ6kT5icTZPoYiaw05icHTWVGIUp6ia7EhwWcibhKZdfkYKA/640?wx_fmt=png)

5.3> 创建文档  

------------

### 5.2.1> 指定文档 id

*   我们采用 **PUT** 的请求方式，新增一个索引名称 = student，索引类型 = type1，文档 id=001 的文档，里面的内容为学生 muse 的信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa13a0Al8WZxHxfzYmWLjtoZKUd36GO8ohgn7HLTK1plHyxItzUcpMhg/640?wx_fmt=png)

*   采用 **POST** 的请求方式，新增一个索引名称 = student，索引类型 = type1，文档 id=002 的文档，里面的内容为学生 bob 的信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwam5DjXqZjD641HfEEsia9oobRHuzk4LYGKZicDrIfibq7AAhrNt8RJfJ4Q/640?wx_fmt=png)

*   我们查看一下新增的索引和文档
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwao55E08nohEUwGpsm6hO61LljhicQqc7Oxib9tickw66b4nHzMQuUVJGqA/640?wx_fmt=png)

### 5.2.2> 随机文档 id

*   当我们尝试**使用 PUT** 创建文档的时候，提示报错。**只允许 POST 方式**去创建随机文档 id 的文档
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaRHiahD8IpjKRXZAaMGCYsT5Lbtb0QGaJSzMtIymbvSoviadXrHNRtOTA/640?wx_fmt=png)

*   我们把 PUT **修改为 POST**，再次执行请求，创建文档成功
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaiayVqEiaqrxsGiafd3nysjXKu4Wrqc6d1HHNMtlpEOUZMvrYRU1lth56Q/640?wx_fmt=png)

*   我们查看索引 student 中的所有文档内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaULsbq5S4415rZS3tEab7ZiahKicW4RQqq8l1WZ9UaknDVahL3ibfRfAYw/640?wx_fmt=png)

5.4> 查询信息  

------------

### 5.4.1> 查询索引信息

*   通过 http://localhost:9200/[INDEX_NAME] 指令，查询索引相关信息，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa10Dv8jU1MukkHP4bem2PvSXaN6ribeVqyVEE6Wfjg5jbZ62g1MMjDHA/640?wx_fmt=png)

### 5.4.2> 查询文档信息

*   我们通过
    
    http://localhost:9200/[INDEX_NAME]/[TYPE_NAME]/[DOC_ID] 指令，
    
    查询文档相关信息，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwayuHVzB8qmOH20sYNtic3MUp420sSwiajibJUXrrlURZ647hoM07W5tHcQ/640?wx_fmt=png)

### 5.4.3> 不同分词器下的条件搜索

#### a> 默认分词器进行搜索

*   我们来看 student 索引
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaJNhbYTfS62KMTKBqcoveiauiboGjnpKONfrMMlOlDwnzcoJHoDicQ373A/640?wx_fmt=png)

*   查找 的值
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa5xoxVmBCA6Skp7hkB9hf34TaeuWAMK6vKKLu9OjhfQviaTANibIVowRA/640?wx_fmt=png)

*   当我们试图查找 ，发现查找不到
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaq1GZj260Hdt342JYkNibuO60BuqZO4jrdR8GNicaeGku6eO3Xv30kkkg/640?wx_fmt=png)

*   那如果我们想要通过查找条件为 “muse” 来查找 muse001 的用户怎么办呢？我们可以使用上面介绍的 ik 分词器试一试。我们知道，默认的**分词器是 standard**，那么它对 muse001 的分词和 **ik_max_word 的分词**结果是怎么样?
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwamwrgDvmgiagZoeljdJrPWdjRJK5oBK01cNwIfuVSMG3jtVAyhfLVamQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa1e6zkpJNB1oSUww9yWJE8ia59pNGmI7LqpxNQR99FlQENuSezWqxsow/640?wx_fmt=png)

#### b> 自定义分词器进行搜索

*   我们创建两个索引，一个索引为 “user_standard”，它 name 使用 standard 分词器；一个索引为 “user_ik”，它 name 使用 ik_max_word 分词器；
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaghGiaNSYrm7GFoAibtygftaAqdIytN9KhHicOfUDUiaT27JRKgKjCftsKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa2zricXZpV068lvQaElbFYKPHRAzcMZM7FSprIHKAPEnIuic9Mdsz1Hgw/640?wx_fmt=png)

*   然后分别给索引 user_standard 和 user_ik 添加文档
    
    内容为：，age=10。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaHxtcCtKllqHyZHUUfVbOJHA2zLXiaytMCGlDcNsKIQE0LIJQicEbkBEg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa44iah4AjMT7L8y52UibibhI7eiaVg9nSZZkB9gq3NyNYOET9hRLhAr3LZw/640?wx_fmt=png)

*   以 “muse” 为查询条件，搜索索引为 user_standard， 的文档，发现没有搜索到。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaCgNkOINWt7Tr4ictzYE6yG9SR5R5qK5CRdMIrR1ic1ZzgAuxWGLk0SvA/640?wx_fmt=png)

*   以 “muse” 为查询条件，搜索索引为 user_ik， 的文档，发现搜索到了。所以，不同的分词器，会影响不同的搜索结果
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwarh5zrEe53iaia3XyD1ibIdX01ib5VicLk43TymhxZbqVwJMN0ibfQyneEO4g/640?wx_fmt=png)

### 5.4.4> 复杂条件搜索

*   上面 5.4.3 中，我们查询使用了`GET /student/type1/_search?q=name:muse001`，那么，我们也可以使用如下的方式执行**等效的**查询效果：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa9kKFscWY02Youw0icGibNBdT2cKhsM7nPJyaibgicqVaj2p2ibentSibyljQ/640?wx_fmt=png)

【解释】

*   其中 hits 会列举出查询出来的文档，其中的_score 代表匹配度，这个值越高，说明匹配度越高。
    

#### a> 准备工作

*   我们创建一个 book 索引，里面添加关于书籍信息的文档，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwajaAVcib8lTakSwUMx4uoOG1ibujmCbvPicBQu6WABnTMR3Y1B3HbbTMMg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa2QT6hRoPtImHKeurbaOjOkV5B29OfiaoM3l4g99czA0mtbP1VX724Cg/640?wx_fmt=png)

#### b> 指定需要展示的列

*   当我们只想查询展示 name 和 tag 这两个字段时，我们可以使用_source，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwavs51ELnslsq9FvBtMFico3o6lzdhXQq5td5YfCozftaPktHePDXhPQw/640?wx_fmt=png)

#### c> 多个内容搜索

*   我们只需要根据空格分割即可。比如我们要查询 tag 标记，包含：“英文”、“经典”，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaC6ncRUDHTbia2ILWcZoCbDicz8P9judZkewsXlNoK1ydwBhJbbOtzfoQ/640?wx_fmt=png)

#### d> 对查询结果进行排序

*   针对结果排序，我们使用 “sort” 即可支持 desc 和 asc ，我们来演示针对 price 进行降序排序，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaxlx5rPhUH9hrHE3QnU0oMxgO04brQfg1KCldpeZHSIB8lLbIStic7iaQ/640?wx_fmt=png)

#### e> 分页查询

*   利用 “from”（从第几个数据开始，从 0 开始）和 “size”（获取多少条数据）来实现分页。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwahuVibFcghlvpuguBcBWCNU3V87ku9ZfJw2LQv9AhF34tmUnhgUlXeYg/640?wx_fmt=png)

#### f> bool 查询

*   【and 操作】我们现在想要查询名称包含 “Java” 并且价格为 100 块钱的书籍。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaBvRYntrHico43uxUdbyf0h3ZPBIMuWHmb2MMaoQNviaapQ9Yc8Pk31ibw/640?wx_fmt=png)

*   【or 操作】我们现在想要查询名称包含”Java” 或者价格为 100 块钱的书籍。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwak7Zo77MMXsIYeapxKx8WplJ1ztSiaKCibfyf6MpaqJDGeIoCxofBUiafw/640?wx_fmt=png)

*   【非操作】我们来查询名字里没有 “Java” 的书籍
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaceeictPYEGVf9vgbibLBDnFhSlM0Z6NVLWbPYqPpFAACpsqyibu6G2O6g/640?wx_fmt=png)

*   【结果过滤】我们来查询所有书名中有 “Java” 且价格在 80~100 之间的书
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaq9Mia2yKs32SWR4jCIjliaicOYhiaEuiaDS6ul5fXTl8AUsicw9TA9GQCc4Q/640?wx_fmt=png)

#### g> term 精确查找

*   我们可以利用 term 进行精确查找，因为它是直接**通过倒排索引指定的词条**进行精确查找的，**不会对【搜索词】进行分词**。也就是说，如果我们的搜索词写得不够 “精确”，那就很难搜索到东西了。
    
*   而我们上面例子使用的 match，它是先**对【搜索词】进行分词**，然后**使用分词器解析文档**，然后再进行查询。所以，term 查询会比 match 的方式查找更快。
    

*   举例，我们要查询 “编程思想”，通过 match 可以查询到，但是通过 term 就查不到了
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwau7aMVxHOpiaMkF5S1UFU6Lb56D7PZcyP2f0ZyOtvDic4yPSeibGSYI0aQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaA9m7cwXOHOw4S9x6CXZ1P8Gwp5IhCgm6yOnMVK2wUbbib0d9y25fzqQ/640?wx_fmt=png)

*   因为 term 不对搜索词进行分词操作，而分词器 standard 针对中文的拆分方式是一个字一拆，所以，如果我们只查一个字的话，term 就可以查询到文档了。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa8Os6Q73DkwxXibFWvIiazkKnFxT1jibdOOBX8ZDpeibkbazLWwq2kXz1Hw/640?wx_fmt=png)

#### f> 多个 term 精确查找

*   我们要精确的去查找书名是 java 或者价格是 100 的书有哪些
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaJRFynGMleL9pHtL3sYTnOLmdfHEnbynXG1HnYMXIYicpMUDVGpYgRgg/640?wx_fmt=png)

### 5.4.5> 高亮查询

*   我们在网上操作搜索的时候，会发现搜索的词被高亮了。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaNb9koKWPibElpElvlEKocQxMJO9gfDV9CGTqfRysgic69pticBvqScElg/640?wx_fmt=png)

*   我们来模拟，搜索 java 的书籍，如何让 java 高亮，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaoQApRdYO5F84hr8ZDtB1GLSic4CMDKFppDtDfxl5Q6FQaiavO9kGEnVQ/640?wx_fmt=png)

*   我们也可以将默认高亮标签 <em>...</em > 修改为其他字符。比如，我们把它修改为 < muse>...</muse>
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaq9qsX3KYunqJ0ux7s4XibnaWp2l5OAaFWPqwsbZmYT7ul8v8ORFFNZw/640?wx_fmt=png)

### 5.4.6> 通过_cat 查询 ES 信息

*   我们可以通过_cat 来查看 ES 相关信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwar7B7wJpfibiaLtwgImLZiaY5sgovUhAUZzJYMDPWkuHPWjicNTKkde2R8w/640?wx_fmt=png)

*   例如，通过 indices 查看索引信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwarhaaCLggyT26VzX0sUyAgPJjBmAHIF39tNicrx54j3tgthMqQRmKLwQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaHjpDicbA1Qre34GtTBT4GvNluj1QgAosUvyescFIz8f1lSibK6DpdRmA/640?wx_fmt=png)

*   例如，通过 allocation 显示每个节点的分片数量和相关容量。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa5PtmMQLmPIucV5lNj3maXCXVODDFvDmkrlEJrME0PMLjxa2ESDN4LA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaXc5kliaNg4hG142ibYYsu2ZmibggBIpmicjibNibrvhuibKicShGVu4kTR81mQ/640?wx_fmt=png)

5.5> 修改数据  

------------

### 5.5.1> 覆盖修改

*   通过 **PUT/POST 请求**，可以对文档进行覆盖修改
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaBtwF9qQkyJ1I93MUHKTVx4icJib7UvS5xxBcLOmwkP0ELIy7kn3eo5FA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaIicicCVgcLWnQmnXQYRVwKfnygPVTyJ7l7UHQageEVIwSvAGIksdfqWw/640?wx_fmt=png)

*   但是假入我只想修改 name，而没有把其他字段也加到请求中，就会产生覆盖的情况
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwafaxPmD9AiafKdKDS2nib6ZKoCIqDe15a4IFdxVN8UMAhuV5uqDcvnEvA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwarbrH76P1Rc9IjDtoffUbGbWvvVTkKcMlXIhxGricBDBBiaJafClbWzBQ/640?wx_fmt=png)

### 5.5.2> 非覆盖修改

*   通过 **POST 请求 +"/_update"**，可以对文档进行非覆盖修改。我们先把 001 文档恢复成原样：
    

```
PUT student/type1/001
{
  "name": "muse",
  "age": 20,
  "city": "北京"
}

```

*   执行 POST+"/_update" 请求，将 "muse" 修改为 “muse001”，这里我们直接用缺省其他字段的方式，来验证是不是文档内容被覆盖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaP51n2Qhf9orFMRoxiayXaut1MD9AKcsn1K1D7IicyNKibkC3QtjwU8I1Q/640?wx_fmt=png)

*   通过上面的截图，我们发现，使用【覆盖修改】的请求格式是有问题的。正确的应该如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaYBpJOOmBKp0jBBibIRiaLn4yw6e7ibZdcP9h91DRLqTDWfoNQcL2NAPZQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa8L2NVCRM6iaqlbGrM1OeG6mVZzKsE2yTSvGnPY3xHQeMwVM4Q0QM7yw/640?wx_fmt=png)

【解释】

*   我们可以通过 elasticsearch-head 看到，name 更新为 “muse002” 了，并且 age 和 city 没有被覆盖。
    
*   那我们尝试一下，用 GET 请求可以不？
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaBqicOLSmQnVAkr821caSH0r9nZnicb2JjpLvLrictfiaRbXInFjz3DF2dA/640?wx_fmt=png)

【解释】

*   我们发现，用 GET 方式请求就会报错了，提示这种更新方式只能使用 POST。
    

5.6> 删除数据  

------------

### 5.6.1> 删除文档

*   我们通过
    
    **DELETE http://localhost:9200/[INDEX_NAME]/[TYPE_NAME]/[DOC_ID]**
    
    来执行删除文档操作
    
*   如下是删除前 student 索引里的文档
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaKf44FicSqemgFJLeW0McTGib5C8p0r6bxB82jvPoymcepbgKwicDSibHiaw/640?wx_fmt=png)

*   现在我们要删除文档 id（_id）为 001 的这个文档，执行如下指令
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaaiaubRsLyN2oOsvQowGFibMmSAcnLkkHjMQvxQicREjwtENicZnV5Kttxg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaDZoX9PiblhmmRE7Ch7EfKYbwPLT8hODEZow9T4AarVibUBVe3WBsqteQ/640?wx_fmt=png)

### 5.6.2> 删除索引

*   我们通过 **DELETE http://localhost:9200/[INDEX_NAME]** 来执行删除索引操作
    
*   现在我们要删除名称为 student 的这个索引，执行如下指令
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwa9UnUcb11WsLTC82Dek10V4aq8pSlWGHgxfMreaU1Yfw61d9ibibWBWEA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaARcLZuUtzuVDfqoVZpiaIiaDT7o04wELzt6ruIahN9RWDbWAW8vhUptg/640?wx_fmt=png)

六、与 SpringBoot 进行集成开发  

========================

*   创建 SpringBoot 项目，引入 Elasticsearch 依赖
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaOrHXYKWxG1Pf7A1Pd0riaCWKGQDbXv8jP5pdiaLIVILSr9K5l6357FnQ/640?wx_fmt=png)

*   我们可以看到，SpringBoot2.6.0 默认的 Elasticsearch 版本是 7.15.2，
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaBiaV8CLnDfzCF6kt7moxamng1rLxKPwLbYvsOqET1iaW9NrTpfQ4xC5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaL0IHI7G3tUzLHMNQ4ggs2YjxTdlDaPDfhcD7R8Tc0icIVMkicgPhu7aw/640?wx_fmt=png)

*   我们本地使用的是 7.15.1，所以，我们修改一下依赖版本。在 pom 中加入 version，指定 7.15.1
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwaJuUNn9kBChoEzn53sddZFoM707t9RrQsLLgpa021SOXU7ic87MEfgxw/640?wx_fmt=png)

*   ESClentConfig.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwakO3PajrH6ZOq6jUGXvXhJUfEvX9NrcvslGic4QBKICHU4z30vTM4XDw/640?wx_fmt=png)

*   Book.java
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOC8W9h5lFRaoDTFZLHboZHwahxkFLh7D5M4AcjicPQRs4krU8w1ywSkWXIMk54UzHW29DS1HRmbt50Q/640?wx_fmt=png)

*   ElasticsearchDemoApplicationTests.java
    

```
@Slf4j
@SpringBootTest
class ElasticsearchDemoApplicationTests {
    private final static String INDEX_NAME = "springboot-demo";
    @Resource
    private RestHighLevelClient restHighLevelClient;
    private static Gson gson = new Gson();
    /**
     * 创建索引 CreateIndexRequest
     */
    @Test
    void createIndex() throws Throwable {
        CreateIndexRequest request = new CreateIndexRequest(INDEX_NAME);
        restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
    }
    /**
     * 获取索引 GetIndexRequest
     */
    @Test
    void getIndex() throws Throwable {
        GetIndexRequest request = new GetIndexRequest(INDEX_NAME);
        GetIndexResponse response = restHighLevelClient.indices().get(request, RequestOptions.DEFAULT);
        log.info("mappings={} \n settings={}", gson.toJson(response.getMappings()),
                gson.toJson(response.getSettings()));
    }
    /**
     * 判断索引是否存在 GetIndexRequest
     */
    @Test
    void existIndex() throws Throwable {
        GetIndexRequest request = new GetIndexRequest(INDEX_NAME);
        boolean exists = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
        log.info("exists={}", exists);
    }
    /**
     * 删除索引 DeleteIndexRequest
     */
    @Test
    void deleteIndex() throws Throwable {
        DeleteIndexRequest request = new DeleteIndexRequest(INDEX_NAME);
        AcknowledgedResponse response = restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);
        log.info("isAcknowledged={}" + response.isAcknowledged());
    }
    /**
     * 创建文档 IndexRequest
     */
    @Test
    void createDoc() throws Throwable {
        IndexRequest request = new IndexRequest(INDEX_NAME);
        // 方式一：采用对象转Gson的方式创建文档
        Book book = new Book("Java编程思想", 100, Lists.newArrayList("Java", "经典", "入门", "语言"));
        request.id("001");
        request.timeout(TimeValue.timeValueSeconds(5));
        request.source(gson.toJson(book), Requests.INDEX_CONTENT_TYPE);
        IndexResponse response = restHighLevelClient.index(request, RequestOptions.DEFAULT);
        log.info("response={}", response);
        // 方式二：采用逐一传参的方式创建文档
        request.id("002");
        request.timeout(TimeValue.timeValueSeconds(5));
        request.source("name", "Spring Boot编程思想", "price", 110, "tags", Lists.newArrayList("SpringBoot",
                "Spring", "新书"));
        response = restHighLevelClient.index(request, RequestOptions.DEFAULT);
        log.info("response={}", response);
        // 方式三：批量创建文档
        List<Book> books = Lists.newArrayList(new Book("a", 1, Lists.newArrayList("a1")),
                new Book("b", 2, Lists.newArrayList("b2")),
                new Book("c", 3, Lists.newArrayList("c3")),
                new Book("d", 4, Lists.newArrayList("d4")));
        BulkRequest bulkRequest = new BulkRequest(INDEX_NAME);
        AtomicInteger id = new AtomicInteger(1);
        books.stream().forEach(b -> bulkRequest.add(
                new IndexRequest(INDEX_NAME)
                        .id(String.valueOf(id.getAndIncrement())) // 指定文档id，不指定则取默认值
                        .source(gson.toJson(b), Requests.INDEX_CONTENT_TYPE)));
        BulkResponse bulkResponse = restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        log.info("bulkInsert is success={}", !bulkResponse.hasFailures());
    }
    /**
     * 获取文档 GetRequest
     */
    @Test
    void getDoc() throws Throwable {
        GetRequest request = new GetRequest(INDEX_NAME, "001");
        boolean exists = restHighLevelClient.exists(request, RequestOptions.DEFAULT);
        log.info("exists={}", exists);
        GetResponse response = restHighLevelClient.get(request, RequestOptions.DEFAULT);
        log.info("response={} \n source={}", gson.toJson(response), response.getSourceAsString());
    }
    /**
     * 更新文档 UpdateRequest
     */
    @Test
    void updateDoc() throws Throwable {
        // 方法一：直接修改响应属性
        UpdateRequest request = new UpdateRequest(INDEX_NAME, "001");
        request.doc("name", "Java编程思想最新版");
        UpdateResponse response = restHighLevelClient.update(request, RequestOptions.DEFAULT);
        log.info("status={}", response.status());
        // 方式二：Gson方式更新
        Book book = new Book("Spring Boot编程思想", 110, Lists.newArrayList("SpringBoot", "Spring",
                "新书")); // 这个book对象，应该从ES中获取；为了方便，此处我就直接new了
        book.setName("Spring Boot编程思想最新版");
        request = new UpdateRequest(INDEX_NAME, "002");
        request.doc(gson.toJson(book), Requests.INDEX_CONTENT_TYPE);
        response = restHighLevelClient.update(request, RequestOptions.DEFAULT);
        log.info("status={}", response.status());
    }
    /**
     * 搜索文档 SearchRequest
     */
    @Test
    void searchDoc() throws Throwable {
        SearchRequest request = new SearchRequest(INDEX_NAME);
        SearchSourceBuilder builder = new SearchSourceBuilder();
        builder.timeout(TimeValue.timeValueSeconds(10));
        builder.query(QueryBuilders.matchQuery("name", "思想"));
        // builder.query(QueryBuilders.termQuery("name", "思想"));
        builder.from(0);
        builder.size(10);
        builder.highlighter(new HighlightBuilder().field("name").preTags("<muse>").postTags("</muse>")); // 高亮
        request.source(builder);
        SearchResponse response = restHighLevelClient.search(request, RequestOptions.DEFAULT);
        Arrays.stream(response.getHits().getHits()).forEach(System.out::println);
    }
    /**
     * 删除文档 DeleteRequest
     */
    @Test
    void deleteDoc() throws Throwable {
        DeleteRequest request = new DeleteRequest(INDEX_NAME, "001");
        DeleteResponse response = restHighLevelClient.delete(request, RequestOptions.DEFAULT);
        log.info("status={}", response.status());
    }
}

```