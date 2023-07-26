

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZDJN7YXSRms4CVj2ogO45dx4MN1b439xrLoUnIcHJ8f2AWUPzfjY2Bg/640?wx_fmt=png)

四、Seata AT 模式源码解析
=================

4.1> 概述
-------

*   AT 事务流程分为两个阶段，一阶段的主要流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZFapbEULnPE43OLAZmbYoz1U2nicFKUf5ldLb42K25Qqj0AiaH1p6ay7A/640?wx_fmt=png)

*   上面第一阶段执行完毕后。TC 在收到全局事务提交 / 回滚指令后发起二阶段处理：
    

*   如果是全局事务提交，则 TC 通知多个 RM 异步地清理本地的事务日志。
    
*   如果是全局事务回滚，则 TC 通知每个 RM 回滚数据。
    

*   我们可以看到第二阶段中，针对于事务回滚的操作，是**基于事务日志来实现**的。那么下面我们来介绍一下事务日志表
    

4.2> 事务日志表结构分析
--------------

*   事务日志表——undo_log，表结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZzoy0Bg16hDkMtjjzWWatqwWTibjzDib9RQ51LjOjSjTt54vAN1HKteGA/640?wx_fmt=png)

*   undo_log 中的 **rollback_info** 字段是整个表中的核心字段，里面保存着完整的 undoLog 信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZLGTkpuwyrYhlUgUticceoNa0SiabEpZlPBiaZxRQZD42Piavcsib4Dc9oiaQ/640?wx_fmt=png)

*   undoLog 表中 rollback_info 中存储的内容，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZhFrHUWcacsje5cUHObvibfQttZLfzzf0xelvicMgh7bUZibndSHyGDicdg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZZPTNoyoqMGibd4pjmtHAic07kLbxzUdrx9976cLZibaKPaoRwUnagFibZg/640?wx_fmt=png)

【解释】

*   从上图中，我们可以看到 json 中包含两个部分，分别是：**beforeImage** 和 **afterImage**。
    
*   beforeImage 就是 “写” 操作**之前**的数据备份，我们称之为：**前镜像**。
    
*   afterImage 就是 “写” 操作**之后**的数据备份，我们称之为：**后镜像**。
    
*   针对于 **insert** 操作来说，**beforeImage 为空**，afterImage 就是新插入行的数据。
    
*   针对于 **delete** 操作来说，beforeImage 就是删除前的数据，**afterImage 为空**。
    
*   针对于 **update** 操作来说，beforeImage 就是更新前的数据，afterImage 是更新后的数据。
    

*   我们从上面 json 内容中，也可以看到，无论是 beforeImage 还是 afterImage，都包含了以下 3 层的结构：
    

*   一个表的信息：**io.seata.rm.datasource.sql.struct.TableRecords**
    
*   一行记录的信息：**io.seata.rm.datasource.sql.struct.Row**
    
*   一行记录中每个字段的信息：**io.seata.rm.datasource.sql.struct.Field**
    

*   下面我们针对上面这 3 个类进行源码介绍。
    

### 4.2.1> TableRecords

*   我们来看一下 Json 中 TableRecords 的内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZxodUhCvmvNwsvCgqIQTQ6YZV72DB9PEVTajF6BTSQpwXHbJPLFglfQ/640?wx_fmt=png)

*   TableRecords 源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZNh6uX7LiahrUXa4wQ1qibm0aTqjicedeF05iaDMAzTAlzKnFaoyW7lYyKw/640?wx_fmt=png)

【解释】在上面这 3 个参数中，**tableMeta 表元数据是最重要的**，在 AT 模式的很多处理环节中都要用到它。

#### a> TableMeta

##### a-1> TableMeta 结构

*   TableMeta 源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZkgKN53Aic4ws8b0zh37V0udQticZaeia8uepDoWWrxicVGDq1nibWdLt1cw/640?wx_fmt=png)

【解释】

*   列元数据（**ColumnMeta**）包含了很多的列信息，例如：列名、列数据类型、列是否允许为空、列是否为自增字段等。
    
*   索引元数据（**IndexMeta**）包括：索引名称、索引类型、索引所包含的列等等。
    

##### a-2> 获取 TableMeta

*   表元数据获取步骤如下所示：
    

*   1> 从缓存中获取 TableMeta。
    
*   2> 从数据库执行查询并生成 TableMeta。
    

##### a-3> 从缓存中获取 TableMeta

*   **AbstractTableMetaCache** 类的 **getTableMeta(...)** 方法用于实现表元数据的缓存操作，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZkpepXajeZ7HMlFbiac2j48icA9ooyicnqVyxAHbJMT4mEbG2yuNn66wng/640?wx_fmt=png)

*   通过上图中第一个红框的 getCacheKey 方法获得**缓存的 key 值**，具体逻辑如下图源码所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZAxxMSZFl3oGU22YCyuEERA1kwLyGAV3MsXmAdWGUQ4ahtCDa1LKvLQ/640?wx_fmt=png)

【解释】getCacheKey 方法里面逻辑比较简单，主要就是拼装 resourceId 和特殊处理后的 tableName

##### a-4> 从数据库执行查询并生成 TableMeta

*   如果无法从缓存中获得表的元数据信息（TableMeta），则需要调用 **AbstractTableMetaCache 的 fetchSchema(...)** 方法从 DB 中获取，而该方法是抽象方法，具体由其子类实现，子类如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ0bE45TtLC6dwPyTYkF2mPdzvrQ8BODSfZyoGSHNd7EozhpG3MK6phA/640?wx_fmt=png)

*   由于我们采用的是 MySQL 数据库，所以我们来看一下 **MysqlTableMetaCache 的 fetchSchema(...)** 方法，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZFerX6vCx3M16gibdianMMExHvyLGOEicia2udibs2VzVonezH90d4soLmdA/640?wx_fmt=png)

*   上图中第二个红框的 resultSetMetaToSchema 方法用于**组装 TableMeta**，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZJuwdLk4VdHdVaibKQ9VKbfupxFQticq4xGBbFVjWZ1Okw99ToHLj27Ag/640?wx_fmt=png)

*   遍历所有列，创建列元数据（**ColumnMeta**）并维护到 TableMeta 的逻辑如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZsEfABIhTGSMRlhicEsAVLkeOdfJEwZqjGHTlEzm8cKm7j0dx36YZU6Q/640?wx_fmt=png)

*   遍历所有列，创建索引元数据（**IndexMeta**）并维护到 TableMeta 的逻辑如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ9OJl6q4eR8a0C2LrPxVIYb2lnMQgeeshknuFtCwQ09zlicCiajqI88UA/640?wx_fmt=png)

#### b> Row

##### b-1> Row 结构

*   我们来看一下 Json 中 Row 的内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZxdDJzOonhHc9GegNKYjg21yRicUiaeibHGBwgaLIe8raFft8xJGlcicibrg/640?wx_fmt=png)

*   一个 Row 对象表示表中的**一行记录**，里面只包含字段列表 List<Field>。Row 源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ5N02LEia70tUib943JNpPHTnicA1Y1wZtZB2EmLJAPc8s9espvPlAcCQw/640?wx_fmt=png)

##### b-2> Field

*   我们来看一下 Json 中 Field 的内容
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZLYJicINYnXZIsibOfA5RoJIdVCrTrEoNhictx9yr6VfK0d3oEg10ELS0g/640?wx_fmt=png)

*   一个 Field 对象表示**一行记录的一个字段**，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZVXJnHzzZmXiac9QsaSYYeD6p4SLuxGb7POvRX0Rfiang2pdDaeevsPzg/640?wx_fmt=png)

【解释】

*   keyType 字段的取值范围为：**KeyType.NULL** 和 **KeyType.PRIMARY_KEY**。
    
*   type 字段的取值范围为：**java.sql.Types** 类中所定义的数据库字段类型。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ1vGOguVwgrHhJ95G4jmBuEBPCUwwbQ4owFMRhPfu57rLIjn9s3AV3Q/640?wx_fmt=png)

### 4.2.2> 总结

*   通过上面详细介绍的 undoLog 表中 rollback_info 列中的 TableRecords 结构内容，我们再看一下事务日志的前镜像内容表达的含义：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZib9Mw5TwOz0a8EgPsFHzBJibaW5OCSxS5WnFlEYfJkp6KF8nPEibZIgQw/640?wx_fmt=png)

【解释】

*   事务日志记录的内容，是针对 **update** 操作语句
    
*   【前镜像】是针对表 t_stock，主键 id=1，count=**992**
    
*   【后镜像】是针对表 t_stock，主键 id=1，count=**990**
    
*   执行语句：**update t_stock set count=990 where id = 1**;
    

*   如果发生回滚，则可以从后镜像中得到业务 SQL 语句当时插入行的详细数据，判断当时的数据是否与当前数据一致。如果一致，则可以安全地完成该业务 SQL 语句的回滚。**事务日志的相关处理逻辑，通过事务日志管理器 UndoLogManager 接口完成**。下面我们来详细介绍一下这部分内容：
    

4.3> 事务日志管理器 UndoLogManager
---------------------------

*   事务日志管理器定义了如下 **5 个**方法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZWxAXqIn0sMPPcG0o35pp0yFKaAZZ3TicUwNbkzlFIUoY5ouuEeyy5CA/640?wx_fmt=png)

*   Seata 实现了基于 **MySQL**、**Oracle** 和 **Postgresql** 这三个数据的事务日志管理器，该接口对应的实现类如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZmxOMB3ic9AHP3LuoZibPcxic4t1bgCuQscvbThV2picov5ibGLlfRMKBksQ/640?wx_fmt=png)

*   以 MySQLUndoLogManager 为例，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZD3Hobs7yYSOtTQCeJpr51RpEtItrOZakh3wcf0dpJAGU4Giayq4YQog/640?wx_fmt=png)

4.4> Seata 的数据源代理
-----------------

*   数据源代理是 AT 模式的核心组件。
    
*   数据源代理的功能包括：
    
    在 SQL 语句执行前后、事务 commit 或者 rollback 执行前后，进行一些**与 Seata 分布式事务相关的操作**。例如：分支事务注册、分支状态回报、全局锁查询、事务日志插入等等。
    
*   Seata 对 java.sql 库中的 **DataSource**、**Connection**、**Statement** 和 **PreparedStatement** 进行二次包装，生成了**四个代理类（Proxy）**，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZT9Supibk3icYw3V2hkUaicPK7Dct1b1dcDVNecLYMYP2J8zkxfSMNOy8Q/640?wx_fmt=png)

### 4.4.1> DataSourceProxy

*   数据源代理类（DataSourceProxy）**代理的数据源就是业务数据库**。其类结构如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZcDWoxnSMeqWqow0rtxEib9y4ZgBJjBaCOKvtqrV7KpgZQ3krrIcbWvQ/640?wx_fmt=png)

【解释】由于 DataSourceProxy 是 DataSource 接口的一个实现，这使得 DataSourceProxy 类**可以分析要执行的 SQL 语句**，以及**生成对应的回滚 SQL 语句**。

*   AbstractDataSourceProxy 类的源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZcbxeIj17pXQJhCEvQXML71XaD7lfJs3hEAHy5nZQhLvf49QaibfXk3g/640?wx_fmt=png)

【解释】

*   可以看到 AbstractDataSourceProxy 提供了一个接收原始数据源 DataSource 的构造函数，并保存到 targetDataSource 中。其他方法**直接调用原始数据源的相应方法**。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZE9TEfRnwuAlvLzZyd9nnmHSvS4rcrzVWCSrKkNaQXhLIFDtvwNs4rg/640?wx_fmt=png)

*   AbstractDataSourceProxy 类并未实现分布式事务的具体内容，而是由 **DataSourceProxy** 类的构造方法实现的，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZSaUGp13wBBXmc9H63g50ib8cjZ4iaWPiaLwtB6wicoBXcOrbbaPT13yeiaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZHSmoydWxWEPPS0uDCDZxQabccqxq76bG8byV5SY197XNMr1LDfNoug/640?wx_fmt=png)

【解释】

*   在上方代码中，DataSourceProxy 的构造函数将原始数据源保存为 targetDataSource，然后**调用 init() 方法执行初始化**操作。
    
*   在 init() 方法中，先用原始数据源创建一个连接；然后通过这个连接获得 **URL 地址**、**数据库类型**、**用户名称**等信息。
    
*   最后**把本数据源代理注册到资源管理器 ResourceManager 中**。由于 **DataSourceProxy 本身就是一个资源**，可以由 ResourceManager 管理。
    
*   如果开启了表元数据检查（默认不开启），则默认每 1 分钟执行一次刷新操作。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZlnXOrzs4M54sh17icnZgNDyAfFQ9uuxdU1pjuKOibUb6mO07A5FT4rKQ/640?wx_fmt=png)

*   由于 **ResourceManager 是 Seata 的一个重要组件**，所以下面内容我们来分析资源管理器的源码内容。
    

### 4.4.2> ResourceManager 资源管理器

*   资源管理器 ResourceManager 相关接口如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZK6PzvlhkHdLiabNmUsPEs2L7TcRhuI6pm7jOlVhPNEDYrqaFLbibibylQ/640?wx_fmt=png)

【解释】

*   AT 模式的资源管理类——**DataSourceManager**
    
*   TCC 模式的资源管理类——**TCCResourceManager**
    
*   Saga 模式的资源管理类——**SagaResourceManager**
    
*   XA 模式的资源管理类——**ResourceManagerXA**
    

*   我们通过上图中源码可以发现，ResourceManager 继承了 **ResourceManagerInbound** 和 **ResourceManagerOutbound** 这两个接口。这两个接口分别提供了 “对内” 和“对外”的两类操作。而所谓的 “对内” 和“对外”的视角出发点，就是资源管理器（RM）。
    

*   ResourceManagerInbound 接口定义如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZvCuG1yZmBiaXYSvBMQ6144AkTJ7l4yjOmsCgUxffcDeZeKz9CWrXM5A/640?wx_fmt=png)

【解释】该接口用来**接收 TC** 发送来的请求，其中包含了：二阶段的分支事务**提交请求**、二阶段的分支事务**回滚请求**。

*   ResourceManagerOutbound 接口定义如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ9WwxmUwMHQfkWtibs4JGt3exH98cTrbBhp0DvazRba0M1KLN0CqYr9g/640?wx_fmt=png)

【解释】接口用于 RM 主动**发送到 TC** 的事务处理请求，例如：**分支事务注册**、**事务状态上报**、**查询全局锁**。

*   ResourceManager 提供了对资源的注册、取消注册、获取所有资源等方法，其接口定义如下：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZdwjLZaSCFIUH3HUezEDX8UiaAJtnH5tSQHxhXxUG7TXXhOic0Nzg89Nw/640?wx_fmt=png)

*   下面我们回顾一下，当我们介绍 DataSourceProxy 的时候，将数据源代理注册到 RM 是通过调用 **DefaultResourceManager.get().registerResource(this)** 方法来实现的，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZjZeYKxf6YNSXNaVkCyauujouQicdoA9KA0lbASDJ8ib0L60w4kRzVwfg/640?wx_fmt=png)

*   那么我们就针对 DefaultResourceManager 方法是如何实现 **registerResource(Resource resource)** 方法，在这个方法内，实际执行了 **3 个部分**内容，下图中已经用红框标注了，如下是具体的源码实现：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZntxATHw8UAnvh8jeGI9sg8vdrZlXP4nfTMw3GGmEyicwbe3NgMN6z5w/640?wx_fmt=png)

*   **第一部分：获得分支类型 BranchType**
    

在方法 resource.getBranchType() 中，resource 就是 **DataSourceProxy** 实例，它返回的**分支类型是 AT**，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZxQ9v9fW7m6Gh2WnJ3lT9EzIJdjiaF009ibxOBTIjp5h8bgkias9iadbXcA/640?wx_fmt=png)

*   **第二部分：通过分支类型 BranchType 获得相应的资源管理器 ResourceManager**
    

在 getResourceManager(...) 方法中，用于获取资源管理器 ResourceManager 对象，通过入参 branchType=AT，我们可以获得 **AT 的资源管理器为 DataSourceManager**。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZia55muCvIYJzaE4B5gYfGW1aiawQYQNiaVz5GZxs5BHiaPia9iaEyZqfH9EA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ7ZaXuYaLAWITf86JHKgUiaQQg0wRKpR37C8uGS8J5JtSnTW0hsvVBFw/640?wx_fmt=png)

那 **resourceManagers 是在哪个地方初始化的呢**？我们来看一下 ResourceManager 的构造方法

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZxdA7CUpVbdpqn9MEw20IiapV5rq4ic2OeqHnvBfwnYDbc3tWc0rZYxLw/640?wx_fmt=png)

下面我们来看一下 **EnhancedServiceLoader.loadAll(...)** 方法，它通过 SPI 机制加载 **META-INF/services/io.seata.core.model.ResourceManager** 中配置的 ResourceManager 的具体实现类，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZnfAHvGgIM5dPibhoPTXkEdpjX8oykN71bYMYOL9QJtB73F77IMQAVFA/640?wx_fmt=png)

*   **第三部分：注册资源 Resource**
    

方法 **DataSourceManager.registerResource(Resource resource)** 用于注册资源，当 TC 收到资源注册请求后，会把客户端连接与 **resourceGroupId 和 resourceId 在内存中建立对应关系**。在推进二阶段提交或二阶段回滚操作时，可以根据 resourceGroupId 和 resourceId 找到相应的客户端连接并发送请求。这种机制保证了二阶段操作的高可用性。其源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ01bB35pGQPw1yCvJhcibOfCWzxcDPqzSibkj9nUjflN7uYZI311RRiavg/640?wx_fmt=png)

4.4.3> ConnectionProxy

*   上面介绍了数据源代理 DataSourceProxy，那么还需要通过它的 **getConnection()** 方法来创建数据库的连接代理 ConnectionProxy，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZQ7tFGsPia8FlxfvmS6ica4jcLsPL7OxQ8JZvfgsGPvyXLCdnRDbkXYqw/640?wx_fmt=png)

*   当我们构造好连接代理 ConnectionProxy 之后，那么业务代码拿到的数据库连接就是 ConnectionProxy 实例，所以在执行本地事务提交的时候，实际执行的是 **ConnectionProxy 类的 commit() 方法**，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZSot8wiapSAEicKtdib3ndnyFOtrNgUYZVcDcomwOpOUa1t6ltErliaJfZQ/640?wx_fmt=png)

【解释】如上面截图中标注的**步骤 1** 和**步骤 2**，进行详细的源码解析。

#### a> LOCK_RETRY_POLICY.execute()——锁冲突重试

*   在 commit() 方法中，通过 **LOCK_RETRY_POLICY.execute()** 方法增加了**锁冲突重试机制**（LOCK_RETRY_POLICY = new **LockRetryPolicy()**）。其中，LockRetryPolicy 是 **ConnectionProxy 的内部类**。那么，我们来看一下 LockRetryPolicy 的 execute() 方法执行了哪些内容：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZdkzuzfSib8BdNVhQfULTODHuKkibslnarUic07hlia61icnkKFA1dvnrAfw/640?wx_fmt=png)

【解释】

*   如果配置 **client.rm.lock.retryPolicyBranchRollbackOnConflict=false**，则会调用重试操作，即：doRetryOnLockConflict() 方法，当发生锁冲突的时候，会抛 LockConflictException 异常，抓取异常后，会调用 sleep 方法执行睡眠，之后再循环调用 callable.call();
    
*   如果没有配置 **client.rm.lock.retryPolicyBranchRollbackOnConflict**，或者配置该属性为 true，则只执行一次 callable.call()，不执行重试操作。
    

*   LockRetryController.sleep() 方法源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZMZokiafn65D8Grk8mCOa4v6gvYoiaXkQVyI2NsCUicibvX8ucu3anKnA5Q/640?wx_fmt=png)

【解释】每次执行 sleep 方法，都会促使 lockRetryTime 减 1，如果值小于 0，则不执行 sleep 操作了，直接抛出异常。

*   其中，加锁重试次数（**lockRetryTimes**）和加锁重试间隔（**lockRetryInternal**）是通过在 LockRetryController 的构造函数中被初始化的，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZQbuRCNehuxicfRl6CBmo1MEhWsgeictoWKpNtrYUaYVibMzO7E1kX7GrQ/640?wx_fmt=png)

*   构造函数中调用的 **getLockRetryInternal()** 方法和 **getLockRetryTimes()** 方法实现方式类似，都是先获得定制参数，如果获得不到，则获取全局配置。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZUtAMqKFmP9qKeB7jbaTHlZJ0JxWlXUl9WFCdE8nqOg8BZDHMKibobzA/640?wx_fmt=png)

#### b> doCommit()——提交本地事务

*   ConnectionProxy 的 doCommit() 方法逻辑如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZnKLSqwot1ibrpLdIqeEKB6oibczSsP2mqF1AjoekVo1juGE4XrTlagfg/640?wx_fmt=png)

##### b-1> processGlobalTransactionCommit()

*   该方法是用于分支事务提交逻辑，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ6HTia5ED0jFPdG60TeQoCp9aiah2Zxoiaoo4PrL5p565DpRP50v6rsqDw/640?wx_fmt=png)

【解释】processGlobalTransactionCommit() 方法中包含 **3 个**重要的步骤（我们随后会分别解析一下），分别为：

*   1> **register()** 向 TC 注册分支事务
    
*   2> **flushUndoLogs(...)** 保存事务日志
    
*   3> **report(...)** 向 TC 上报分支事务状态
    

###### Step1> register()

*   register() 方法用于**向 TC 注册分支事务**，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZjkbOEwia6DOwCibKuTAiaGAMgZRVErnibbUFI2udaAicTAygzMylv5oR28A/640?wx_fmt=png)

DefaultResourceManager 的 branchRegister(...) 方法用于**分支注册操作**，源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZH8GUUz8lhaMHuPaua3dGr4U5M8VMYa0a32RCo4RmnrML5t0xYhXhTA/640?wx_fmt=png)

AbstractResourceManager 的 branchRegister(...) 方法逻辑如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZf20sDiar0fb0RiagcYVn3t5DGkfL9nE7SiasSwtMha4lF9E8obVRHYKtg/640?wx_fmt=png)

###### Step2> flushUndoLogs(...)

*   flushUndoLogs(...) 用于保存事务日志，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZgMxou6JWumrjWyoptfZZ2G60o7H2ZlUpdVdibznYajQrJbLvGUibbMRw/640?wx_fmt=png)

【解释】

*   首先，从上面截图中的源码中可以看到，**xid、branchId 和 undoItems 都是从连接上下文（connectionContext）中获取到的**。
    
*   其次，会将日志内容转换为字节数组，采用什么方式转换，可以通过 “client.undo.logSerialization” 进行配置，默认是 JacksonUndoLogParser。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZkODdk2WcArwChDibicpwaOGw5BtFyG6kNzjZwFpfJzMylVCDhLshqXyQ/640?wx_fmt=png)

*   第三，如果日志需要被压缩的话，则进行事务日志内容压缩操作，是否需要被压缩的逻辑在 needCompress 方法中，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ7fiaQTWmXKqH0mapH0MHzW24rUsNfUJTiauvYIDx1y8HQNyWaCA2rcqg/640?wx_fmt=png)

*   最后，调用 insertUndoLogWithNormal() 方法来插入事务日志，该方法是一个抽象方法，具体实现是由不同的数据库来决定的。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZH5r3btIYJXFHejreMat7SyEKFm9QvM1QjEJLq6CibOIuFt4Vfz33Ykw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZSOJDRoJjx2CRK0ibKm4IaCAdwAZSVM1nNQvu3En4ZnSlgPqBzwS2tLw/640?wx_fmt=png)

*   一般来说，我们常用的数据库时 MySQL，所以我们来看一下 MySQLUndoLogManager 的 insertUndoLogWithNormal() 方法源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZkELeLYttwKicF9CFB24wyIZhMbWeSPuZibLBJduVgWFZf9Jw0bFD8zLQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZsy1chMfPEEKEonh2e2Sy9Akck0ZvWdzIp5LSvQn5T4IILA0oTAyJTQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZqoMCACm8McGtiakicOUv9Kj3mxxc1Zgqicw7S0wNYzIoHDgwDbVzQgnaA/640?wx_fmt=png)

###### Step3> report(...)

*   report(...) 用于向 TC 上报分支事务状态，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZAIBHO16QHFjNnlJdYFkAj5fbAibenicjcgGYicsILh4xkpoaXyCcTY3sQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZKDUKhcuRvcdzZmchKL3wrKS3LJicGdoKClo8LC3CqtSpM1iaJWpibiantw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZIiaA88Ntiabd8QUAnIXqmesc0llWOyzdhp12uDAd6vW4Klx9xI6Yxzpg/640?wx_fmt=png)

##### b-2> processLocalCommitWithGlobalLocks()

*   当需要查询全局锁的时候，就会执行这部分的代码。那么，什么时候去查询全局锁呢？我们知道，在 AT 模式中，在第一阶段加上 Seata 全局锁后，提交本地事务，然后释放数据库锁。这就造成了一个问题——在一个分布式事务完成后，数据修改已经入库了，但是它可能还处于一个未结束的分布式事务中，即：它修改的数据对分布式事务来说是中间数据，有可能还会回滚回去。这个时候，另一个分布式事务查询它刚修改的行，就会读到中间数据，即：发生了分布式事务的脏读。
    
*   但是，这种中间状态的情况并不会长时间持续，一般来说，很快就结束了。所以，如果在实际应用中，允许这种中间状态的产生，则可以不去查询全局锁了。因为毕竟查询全局锁是有一定的性能开销的。
    
*   下面是 processLocalCommitWithGlobalLocks() 方法的源码注释：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZFXGIjiaoHAFzgLwXOprjGpZ67ULqD1voAd9GsE5koiaYNXRfP59AOQtg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZjIJpx2FZs5ZXByIWfub44cib1BnwtKDwJClRG0Gsia3ibMSX2Xl9udT3A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZVMCLibUP2OuS189qaoTuEsluEwJRzM06ekjSAR0LEgJ8uNqLXGhC2uQ/640?wx_fmt=png)

*   其中，根据不同的 branchType，来实现不同的 lockQuery 方法，下面是 AT 模式
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZPfwSlgrOibYjmt2SeUvUv3QAfLcKOeeqhp4fgEAggqTzU1kHLh0ialhw/640?wx_fmt=png)

*   **除了 AT 模式之外，其他的模式都是不支持全局锁的**，即：直接返回 “无全局锁”
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZjxZWkWl56kTAlGCaJf4lKVfhNJ2bG0xRIAaoJPYRG2zcYic15y7yiazg/640?wx_fmt=png)

##### b-3> 通过 SPI 获得响应的实现类

*   我们以获得 UndoLogParser 为例子，看一下如何通过 SPI 机制获得 UndoLogParser 实现的子类
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZwt6MQmfO1TaDr1yQ3I37f5roy9jXtub1S99Xt8oZIcSKwgaiaC3ljKA/640?wx_fmt=png)

*   UndoLogParserFactory.getInstance() 方法如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZw2zfVZHdpufdDHc5j2k0AaV7v17CsnIKNcNqDm6IUqs4FeiaS6Crq4g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZM708qVdghG7xJIiccSjlLABiazJ88Bqma9E1m8gzn1oWWqQg7JtHKBwA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZG3d4T3OKY9xQ2UBCIGbTt6IPVEjHaE7ZO7A7lGCgPduZCmpVUCH5dw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZDUxE9ZWwdJiaftnicywnTrfcRp490nqhu3LlEJm3dSOkS8g3ibwyDWwoQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZRsFCEY9Ict8084PcyXR7R9ZOXRZicRC3cDId438eknkScHde2O73Hvw/640?wx_fmt=png)

【解释】随着代码的深入跟踪，我们看到最后是从 nameToDefinitionsMap 通过查询 key 是 activateName 来获得 UndoLogParser 的实现子类。

*   那 nameToDefinitionsMap 是从哪里初始化的呢？我们来看 **EnhancedServiceLoader.getUnloadedExtensionDefinition()**，在该方法内执行了向 nameToDefinitionsMap 中赋值的操作，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZaWQ6hr0BA2cibicpuc2O2typSfdAiaibppib9cAibKY0blhp3IwI1TxotoJA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZoP2bhm5egDXYic3Dibibib1MDe56LmPM4ibgVQWFAMic60K5ribzWsW3ibaxTw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZkODdk2WcArwChDibicpwaOGw5BtFyG6kNzjZwFpfJzMylVCDhLshqXyQ/640?wx_fmt=png)

【解释】从上面的源码中我们可以看到，如果我们要获取一个 UndoLogParser 的某个子类的时候，可以通过指定 serviceName 的方式，例如：serviceName=“jackson”，那么，再根据 **io.seata.rm.datasource.undo.UndoLogParser** 文件中配置的所有 UndoLogParser 实现类去查找。根据每个实现类的 **@LoadLevel 注解中的 name 属性**，来进行匹配。

### 4.4.4> StatementProxy 和 PreparedStatementProxy

*   Statement 与 PreparedStatement 的区别
    

*   Statement 用于执行**静态 SQL** 语句，PreparedStatement 用于执行预**编译 SQL** 语句的。
    
*   在使用 PreparedStatement 对象执行 SQL 语句时，SQL 语句会首先被数据库解析和编译，然后被放到**命令缓冲区**中。每次执行同一个 PreparedStatement 对象，SQL 语句会被解析一次，但**不会被再次编译**。在缓冲区中有预编译的命令，并且可以重用。
    
*   PreparedStatement 对象可以减少编译次数，提高数据库的性能，并且有**更好的安全性**。
    
*   同样向数据库中插入 N 条记录，PreparedStatement 对象会比 Statement **插入效率高**很多。
    

*   在 **AbstractConnectionProxy** 中提供了创建 StatementProxy 和 PreparedStatementProxy 的方法，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZKCmKQciakicggjBO9kG6554sRiagux0c8Q9PoQhMIHWO92EUhBFKSOjMg/640?wx_fmt=png)

*   PreparedStatementProxy 的类关系图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZaM4gWKMd3LYROzXpTmdZqq8G5zrCqyVuJ254VR021QGBW6SRDcQ7sw/640?wx_fmt=png)

#### a> AbstractStatementProxy

*   AbstractStatementProxy 中包含 3 个变量，分别为：**connectionProxy**、**targetStatement** 和 **targetSQL**。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZL236CROGxJGHCuBHiazYEx9heicT6COrOUficQQvtuTpSYydoQ9pTNphw/640?wx_fmt=png)

该类中很多方法都是**直接调用 targetStatement 实现的**，代理类没有做额外的工作，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ4DMAYXo5UNm2s6MOjMF6RbCzaFW6PCsnlajFz3yZ9kESM6AQmpic7xg/640?wx_fmt=png)

#### b> StatementProxy

*   我看来看 AbstractStatementProxy 的实现类 StatementProxy 的源码，我们会发现，这些方法真正实现都是通过执行模板类 **ExecuteTemplate 的 execute() 方法**来实现的。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZbfmJ3YAcy7xvlgPcBXdmczmJ7zYIwtMTPlKpJtadRMux4Pibw11oWbg/640?wx_fmt=png)

#### c> ExecuteTemplate 执行模板类

*   ExecuteTemplate 本身只提供了 **execute(...)** 方法，源码注释如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZkdEcul3RxibfeH0uzZ396dpLosMk9c1vYIDJTZbwaB26icRvPUTChOCw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZCuVyvUpHcQ1oIrN584yCiaptc4U5rQwibm6xlq6Vxo8RJVoXtxmxiaB0A/640?wx_fmt=png)

【解释】

*   首先，判断这个 SQL 语句不在分布式事务中，并且也没有查询 Seata 全局锁的要求，则不需要将其纳入 Seata 框架下进行处理。只需要**执行原始的 statement** 即可。
    
*   如果这个 SQL 语句在分布式事务中，则将其纳入 Seata 框架进行处理，并且根据不同的 SQL 语句类型选用不用的**执行器 Executor** 来执行。
    
*   Seata 框架处理的 SQL 语句包括：**insert**、**update**、**delete**、**select...for update**。
    

##### c-1> SQLRecognizer

*   SQL 识别器的接口源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZuls3VskAba7TYX6x8SmKotQEjedBepF0jVURyZJc0qibcMRKsI4BJCQ/640?wx_fmt=png)

*   抽象类 **BaseRecognizer** 实现了 SQLRecognizer 接口，而针对不同的数据库，都有其对应的一组实现类，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZdzKj0dlLPXO71ahUI4yKE6mhqmPhO7B1ibBPZIThEKRURnhCV8JkuyA/640?wx_fmt=png)

*   那么以我们常用的 MySQL 数据库为例，看一下 MySQL 相关接口的类图结构：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZSTCJ37QOWBb2ibRFHYUlODsx0RHibPlf4p2cQ3ricWa6VzGVDdn4nupLQ/640?wx_fmt=png)

##### c-2> SQLVisitorFactory.get(...)

*   如果 sql 识别器集合是空的，则通过 SQLVisitorFactory.get(...) 去获取，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZWGVkzIgRNAnHicVpmQLAUnnqe7wMIFIia9BCLzthaP065dMcxeT1pheA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZncicUehZYeO8cjEN0wtGMmreVFCs4NicJzQcaJ5HrDTic0KM3E9ibBxNKQ/640?wx_fmt=png)

*   SQLRecognizerFactory 是 SQL 识别工厂。它是用于**创建 SQLRecognizer 对象**的。实现类如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZLqSF7TptDaUEzSq9sXdsRTf0YCLKBD85RTG8VFqibA4oSia1FdTx3JeA/640?wx_fmt=png)

【解释】

*   这些识别器都要借助于开源 Druid 库生成的**抽象语法树 AST**，由 com.alibaba.druid.sql.**SQLUtils.parseStatements()** 方法生成，该方法会把传入的 **SQL 语句解析成 SQLStatement 对象集合**，**每一个 SQLStatement 对象代表一条完整的 SQL 语句**。
    
*   Seata AT 模式使用了 Druid 的解析器解析 SQL 语句。
    

##### c-3> Executor

*   SQL 执行器 Executor 接口类图如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZYTzo77kV2BSCLPJlA6Cm70IFaomBkqp2noicRMonrQ76GluVy5p05KA/640?wx_fmt=png)

*   Executor 接口仅提供了一个方法，即：**execute(Object... args)**
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZw4hiaxuEHoq3sm0HsKq99ibGMyEibZmG1Utr3ibgAfW5q58jqBzIIZWjXg/640?wx_fmt=png)

*   BaseTransactionalExecutor 实现了 Executor 的 execute(...) 方法，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZnjcUftcVeSSsA1wBnIV6asbzBJrpXu8icTroXOMNEVDmm1WeSFeibZZw/640?wx_fmt=png)

【解释】

*   如果处于分布式事务中，则会**绑定全局事务 id**。
    
*   如果需要查询 Seata 全局锁，则在连接上下文中设置需要查询 Seata 全局锁的标识。
    
*   最后执行 **doExecute() 方法**，该方法由具体的子类去实现。
    

*   其中，**AbstractDMLBaseExecutor** 实现了 BaseTransactionalExecutor 的 doExecute() 方法，具体实现如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZI1ZmwoMg3Us4M0dKnQzRrQjwkYIhgAV8oQUGABmKLR9cJoI2Yic7zSg/640?wx_fmt=png)

【解释】

*   首先，通过 statementProxy 获得了连接代理 connectionProxy。
    
*   其次，通过 connectionProxy 的 getAutoCommit() 方法可以判断出本次连接是否是事务自动提交的。
    
*   如果 **autocommit=1**，则执行 **executeAutoCommitTrue(args)**。
    

*   如果 **autocommit=0**，则执行 **executeAutoCommitFalse(args)**。
    

*   下面我们来看一下 AbstractDMLBaseExecutor 的 **executeAutoCommitTrue(args)** 方法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZVFV7m9OS4dNBuODdAh5S4D5kt5Bq5GTnAX5PsegZe0YEyGtbr8NY5A/640?wx_fmt=png)

【解释】从上面代码中的逻辑我们可以看出来，针对于已经开启自动提交的连接，会先关闭自动提交，然后调用的**实际执行逻辑是 executeAutoCommitFalse 方法**，执行完毕后，会手动进行提交操作，并且最后开启自动提交。

*   下面我们来看一下 AbstractDMLBaseExecutor 的 executeAutoCommitFalse(args) 方法
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZqUra0boBFAgL8Fibaic6krs9IGfYg5llgACABJmic9Hbhs37ibdicbicEcpQ/640?wx_fmt=png)

【解释】目前**只有 MySQL 数据库支持多个主键**。对于其他数据库，如果表存在多主键，则不允许使用 AT 模式，但是可以使用 TCC、Saga 或 XA 模式。

*   下面我们会针对这 4 个步骤进行解析
    

###### **Step1> beforeImage**

*   对于不同的 SQL 类型，匹配不同的执行器 Executor，那么对于 beforeImage() 方法的实现也是不同的。对于 insert 语句来说，前镜像是空的。而 update 和 delete 语句在前镜像实现上是类似的，下面以 **update 语句**为例：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZicWMhadCI0H7aoA0ia7IhSRTEvztWD4jBHwvWD79lkd3blS2mYowIgqg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZIZzKmHibuPcJBphMguExibYvKgIlBibpRKDBic3akKibtmrxjQMyq8jgTsA/640?wx_fmt=png)

【解释】

*   getTableMeta() 方法已经在 4.2.1 中介绍过了，此处就不介绍了。
    
*   前面也介绍过，前镜像和后镜像都被存放在 TableRecords 对象中。
    

*   **UpdateExecutor 的 buildBeforeImageSQL()** 用于构建一条查询**前镜像记录的 SQL 语句**，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZjB8licZm9L5yaYNdLm0MmvItqzCQsT3oyewuQ0y4OxlVYpz9xQR51Nw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZZ0omsXVoypkJ0qcIElLYEk6Hibvs0oO299BazHrDlj5DP3nkaxeRHqg/640?wx_fmt=png)

【解释】

*   通过上面的源码我们发现，拼装的逻辑主要是通过 update 的 **SQL 识别器**，即：SQLUpdateRecognizer 来实现的。
    
*   最终拼装一条 “**select ... from ... where ... order by ... limit ... for update**” 的 SQL 语句，并把参数放在 paramAppenderList 列表中。
    

*   获得到了 selectSQL 语句后，随后会执行 buildTableRecords() 方法，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZrBtSKjqKGwj59wet5loFkazgWQGEgYzyaAM0FiczHdmnxnqayJDxBCA/640?wx_fmt=png)

【解释】

*   首先：基于拼装的 selectSQL 来创建一个 PreparedStatement 实例对象。
    
*   其次：遍历 paramAppenderList，对预编译 SQL 赋值参数。
    
*   然后：执行 ps.executeQuery() 获得结果集 ResultSet。
    
*   最后：通过 TableRecords.buildRecords(...) 方法将 ResultSet 转换为 TableRecords。
    

###### **Step2> execute**

*   步骤 2 中，就是利用目标 statement 执行原始 SQL 语句，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZWRQpaoicNLVSWSD2YfgBs9vE0bZbU5dVia3v7iaibB69yU0Q14UAUqvhdA/640?wx_fmt=png)

###### **Step3> afterImage**

*   步骤 3 中，利用 afterImage 生成后镜像，它与 beforeImage 方法一样，都是抽象的方法，具体实现都延迟到子类中了。如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZPDO2JNnVkYv5iaRGEHYBdIxVtge1IRO1MHAicibfs9ianiaHf1iajgEwIXMw/640?wx_fmt=png)

*   如果是 delete 语句，则后镜像是空的。其中 update 和 Insert 在构造方式比较相似，我们还是以 UpdateExecutor 为例，看一下是怎么处理的后镜像。
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZSJv1DPOfY6j2rq0pMzJicicPiajvPkVcYz9XpRG4bB4Eicm8o7r3fUGlhQ/640?wx_fmt=png)

*   buildAfterImageSQL(...) 方法用于构造后镜像 SQL，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZlrP43omVGnhl3onLxiad8Lt0ORItP7ub6KiaSjoBZlgBj5fPEoxPVfaw/640?wx_fmt=png)

###### **Step4> prepareUndoLog**

*   第四步骤是准备事务日志，即：BaseTransactionalExecutor.prepareUndoLog(...)
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZuNNVjgUy4sPFzDYzyOT2loibAZkHrSv2OO8zd4dT4vgsN6UBvgYSXwg/640?wx_fmt=png)

其中，将合并后的事务日志保存到连接上下文 ConnectionContext 中

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZBTMlFnFFACLLS2HjVFoYZ1E3QQfTGjZufTY0JQpicKfP8223DTSOMibw/640?wx_fmt=png)

【解释】

*   如果 beforeImage 和 afterImage 都为空，则直接返回。因为没有事务日志需要保存。
    
*   检查 update 语句的 beforeImage 和 afterImage 行数是否相同，如果不相同，则直接报错。对于 update 来说，前镜像和后镜像保存的是相同行，所以行数应该是相同的，只是行的内容不同。
    
*   通过调用 buildLockKey() 方法构建 Seata 行锁数据
    

*   如果是 **delete** 语句，则使用 **beforeImage**；
    
*   如果是 **insert 或 update** 语句，则使用 **afterImage**；
    

*   一个本地事务中可能包含多条 SQL 语句，每条 SQL 语句都可能生成 Seata 行锁数据，需要在构建完成本条 SQL 语句的行锁数据后将这些行锁数据**合并成一个大字符串**。
    
*   执行 buildUndoItem() 方法把 beforeImage 和 afterImage 构建为 undoLog，把新构建的 undoLog 与用该本地事务中别的 SQL 语句已经构建的 undoLog 合并在一起。
    

4.5> AT 模式的两阶段提交
----------------

### 4.5.1> 一阶段处理

*   一阶段处理流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZqbKRDzB4zcbX985uhYRU0HxiahxzvwjETFcWWQhhPE9D7icuGccu9YqQ/640?wx_fmt=png)

【解释】

*   首先：在一阶段中，Seata 会先**拦截业务 SQL 语句**，**解析 SQL 语句**的语义，**提取表元数据**，找到 SQL 语句**要更新的业务数据**。
    
*   其次：在业务数据被更新前将其保存成**前镜像**。
    
*   然后：**执行 SQL 语句**更新业务数据；在业务数据更新后，将其保存成**后镜像**，并**生成 Seata 事务锁数据**，**构建事务日志并且插入事务日志表**。
    

*   以 update tb_user set name='muse' where id=1; 为例，整个流程如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZzhkiaOoevq4ngymFqib3LDwTKLibEGQc6Eh3cniaWyI0tvphQea5r4yTKQ/640?wx_fmt=png)

【解释】在本地事务提交之前，通过 TC 注册分支事务。

*   如果发生全局锁冲突，则回滚本地事务，在休眠一段时间之后重新开启数据库本地事务。
    
*   如果没有全局锁冲突，则注册分支事务成功。
    

*   一阶段的主要工作是生成 SQLRecognizer、Executor、beforeImage、afterImage 等。AT 模式客户端的**主要开销都在一阶段中**。
    

### 4.5.2> 二阶段的提交处理

*   如果全局事务是提交状态，则 TC 会先进行 “**放锁**” 操作，然后释放各个分支事务在一阶段加的全局锁，并**推进二阶段提交**。
    
*   RM 在接收到分支事务二阶段提交指令后，**只需要删除保存的事务日志数据**，完成数据清理即可，因为 SQL 语句在一阶段中已经提交至数据库中了。
    
*   为了提升性能，RM 会**立即返回** TC 处理成功，并通过**异步线程批量删除**在二阶段中提交的分支事务的日志数据。
    
*   在 AT 模式中，是通过 **DataSourceManager.branchCommit()** 方法来完成分支事务的二阶段提交的，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ8uGwh0VD0o2XOOU23WU7qCkIGAjeEzwFE3tM69mDRxOa1YRmt6Ov3Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZoSmuxyIDjD7RestoOiax9SSKStogUBmATYhNhZ3CibUeAfrm09egOic0A/640?wx_fmt=png)

【解释】创建完二阶段上下文后，将二阶段上下文添加到 CommitQueue 中，随即就返回二阶段已提交状态了。此阶段采取的是异步方式，而非同步阻塞的方式。

*   下面，我们来着重看一下将二阶段上下文添加到 CommitQueue 的代码逻辑，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZbrRGHuicWg5x8E9kx2uekVAfVRDCLmNYvZQlzlM7WPLIbXJBccMgq8Q/640?wx_fmt=png)

scheduledExecutor 是每秒执行一次的定时任务线程池，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ7hrpAtAQNzNAaz1BvPtVSOPY0NI4uVHq1tW1vMxKgb6YiaAQdPnWXcw/640?wx_fmt=png)

【解释】调用 offer 方法将 Phase2Context 提交到提交队列中。如果返回 false，则说明 commitQueue 已经满了。

*   如果 commitQueue 已经满了，则会采取异步执行 doBranchCommitSafely() 方法，执行如下代码：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZhibK4EGG56HOW61VKOVp0AkIClYcLQiaM2sdg4aCYDbPtpsoRdPaeQ3A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZkJd3lia3n0vB2Zj0JMtNxvbrw24vrO72ibaeoOMWzLesgvCmcHzDPgzg/640?wx_fmt=png)

【解释】

*   该方法的入参是已经按照资源 id 汇总的二阶上下文集合，即：contexts 中所有的二阶上下文的 resourceId 都是第一个入参值。
    
*   由于 Phase2Context 集合大小不确定，为了方式列表过大，拼装的 SQL 语句过长，所以默认采取每 1000 条记录为一组的方式进行分批切割操作。
    
*   然后针对每个批次进行批量删除 undoLog 操作，即：deleteUndoLog。
    

*   如下是批量删除 undoLog 的逻辑代码 deleteUndoLog() 的源码注释：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZu9OVXErfZ3hz39bzT9aJG04DJbsMVlCynSS6sH40pEG67NgcTutV5A/640?wx_fmt=png)

*   统计完事务集合 **xids** 和分支集合 **branchIds** 后，传递给 batchDeleteUndoLog 方法，源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZz6YiatiaM2h1XfG74xh1pbFbND0f3XTR4kSHD6Em2dlhcNAyBibAqT9yg/640?wx_fmt=png)

*   将事务 id 集合的 size 和分支 id 集合的 size 都传递给 toBatchDeleteUndoLogSql 方法的目的，就是用于拼装 size 个‘?’，作为预编译的占位符。拼装后的 SQL 类似于：**DELETE FROM undo_log WHERE branch_id IN (?,?,?...) AND xid IN (?,?,?...)；**源码如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZJicRBEYUicbvIFXiaRzaez2ibp9a7Ap6uECuxIPicGWGFzvzsicNZBIL4oEA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZOPn0ztSf1JW4rF0uohmNxDTewQ66cBXFpyE090H5ZeicyonCWia2x0IQ/640?wx_fmt=png)

### 4.5.3> 二阶段的回滚处理

#### 4.5.3.1> 概述

*   如果全局事务是回滚状态，那么 TC 会触发二阶段回滚操作。
    
*   RM 在收到分支二阶段回滚指令后，会**回滚一阶段已经执行的业务 SQL 语句，还原业务数据**。
    
*   二阶段回滚的基本思想是：**用前镜像还原业务数据**。如下图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZhOdFC9zevBDEiaVQsX2icN1iaa1aCf7efAJA9LwMuLw6GibAmvDWicDTFMg/640?wx_fmt=png)

【解释】在还原前，要先校验 “脏写”，即：**对比后镜像和数据库中的当前值**。

*   如果数据完全一致，则说明没有 “脏写”，可以还原数据。
    
*   如果数据不一致，则说明有 “脏写”，需要转人工处理。
    

*   要分析一下分支事务回滚逻辑，我们先来看一下 **DataSourceManager** 的 **branchRollback()** 方法，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZRgKdfImNbpiaGyL1NOEgHNrr9csxnZEQIA2eGAqKqLC6Xeubx6E846A/640?wx_fmt=png)

*   真正执行分支事务回滚操作的就是在 **AbstractUndoLogManager** 的 **undo(...)** 方法中
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZpWhX5EiaGeHY70qpfhkXkNSJicJ0ylxScF6XibVdLRVcFI371GzSNXnJA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZn6h1R1JFSnFFX5ibVD0ibLzYvFKxiax5TQQH2eMauiaNgPibuqfW6xC6K9g/640?wx_fmt=png)

【解释】

*   undo() 方法核心逻辑：使用 undo_log 数据来补偿分支事务在一阶段中锁增加、删除、修改的数据。
    
*   在上方代码中，先查找在 undo_log 表中本分支事务 ID 和 XID 所对应的记录，并把查到记录的 rollback_info 字段内容转化为 BranchUndoLog 对象，然后循环处理 BranchUndoLog 包含的所有 SQLUndoLog 对象。
    
*   对于每个 SQLUndoLog 对象，都使用 Undo 执行器完成回滚。在所有 SQLUndoLog 对象对应的回滚都完成后，删除分支事务对应的 undo_log，并提交本地事务。
    

#### 4.5.3.2> Undo 执行器

*   对于每个 SQL 语句进行回滚，是通过 AbstractUndoExecutor 的 executeOn() 方法，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZ0QdDk0OSEwwMAiarKof9pW5E9ibUicJuiaNZ2zzMbSOtTJjbAniaFanddFw/640?wx_fmt=png)

##### **a> dataValidationAndGoOn**

*   脏写检查的目的是防止错误的数据补偿。具体逻辑在 AbstractUndoExecutor 的 dataValidationAndGoOn() 方法：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZHNgUdfh6hV7B1Cz9SU6UOvJ6q2KIENgQtO4lTRlibGenYElMiafmjWpw/640?wx_fmt=png)

##### **b> buildUndoSQL**

*   buildUndoSQL() 是一个抽象方法，对于不同的数据库都有对应的实现，并且根据不同的数据库操作也有 insert、delete 和 update 这 3 种实现类，如下所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZuvIt3FAhiaJJIOA1MejwDcmztEZnf8NH2rDgp2CPmmYb4UicWkI2yrbQ/640?wx_fmt=png)

###### **b-1> MySQL 的 insert 回滚执行器**

*   构建基于 MySQL 数据库的 **insert 语句的回滚 SQL**，我们要参照 **MySQLUndoInsertExecutor** 的 **buildUndoSQL()** 方法：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZzZpqk1DxEFhptNicY3iby4uFXsg5CLx6ClT089ePLKAqqyqsTss5qFfA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZOur0RAaCXdMhubBT5G8scllibSxPngtGo1jSlnOk5uOkW4YXJ0mm1VA/640?wx_fmt=png)

【解释】如果业务 SQL 语句是 insert 语句，则它的回滚语句就是 delete 语句，删掉在一阶段中插入的行。

###### **b-2> MySQL 的 delete 回滚执行器**

*   构建基于 MySQL 数据库的 **delete 语句的回滚 SQL**，我们要参照 **MySQLUndoDeleteExecutor** 的 **buildUndoSQL()** 方法：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZeSAnoTaRmzC7PYkdHUv2Pr1N8MKDMW9UvXnxSgntibk1O2uaDcR84yw/640?wx_fmt=png)

【解释】

*   在 buildUndoSQL() 方法中，以 SQLUndoLog 对象的前镜像作为数据来源（即：在一阶段中删除的行）来构建 insert 语句。
    
*   如果业务 SQL 语句为 delete 语句，则它的回滚语句就是 insert 语句，把在一阶段中删除的行重新插入进去。
    

###### **b-3> MySQL 的 update 回滚执行器**

*   构建基于 MySQL 数据库的 **update 语句的回滚 SQL**，我们要参照 **MySQLUndoUpdateExecutor** 的 **buildUndoSQL()** 方法：
    

![](https://mmbiz.qpic.cn/mmbiz_png/AZHyCoMMOCibhySWjicewK8zB5mU8ibETdZk6yMCYBFiccQMGkTHMwBqKceQ2FpRCeiciaBxCFkMYPKSUhEaibyQpJV8g/640?wx_fmt=png)

【解释】

*   在 buildUndoSQL() 方法中，以 SQLUndoLog 对象的前镜像作为数据来源（即：在一阶段中更新的行在更新之前的值）来构建 update 语句。
    
*   如果业务 SQL 语句为 update 语句，则它的回滚语句就是 update 语句，把在一阶段中更新的行的值恢复回去。
    

*   在 RM 成功完成分支事务的二阶段回滚后，TC 会对该分支事务在一阶段中加的 Seata 全局锁进行 “放锁” 操作。