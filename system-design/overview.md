## 概述

### 关于这门课程

*   **系统设计的新视角**：从第一性原理去理解系统设计，而不是记忆各种问题的解法。问题有各种各样的变形，不可能单靠固定的解法就能解决所有问题。必须掌握问题的本质，以不变应万变

*   **深度和广度**：给出了适当的理由，说明为什么我们使用某些组件；还涉及可伸缩性、可用性、可维护性、一致性和容错性

*   **迭代提升**：好的系统设计是迭代出来的。推荐至少两次迭代，第一次是在 80% 的时间内尽最大努力提出一个设计；第二是在剩余的 20% 的时间内去提升它

**交互式学习**：对于重要概率，通过问题和测验来加强理解

### 谁应该学习这门课

系统设计是给那些想在职业生涯中取得进步的软件工程师准备的



## SQL or NoSQL

NoSQL 的优点：

* 数据是非结构化的，或者不需要任何关系型数据（没有联表查询（join），条件查询的需要）
* 只需要序列化或反序列化数据（例如 JSON, XML）
* 大容量：需要存储大量的数据
* 高性能：应用要求非常低的延迟

## 垂直扩展和水平扩展

（1）垂直扩展（vertical scaling/scale up）

定义：通过增加单台计算机的资源（如CPU、内存、存储或网卡）来增加系统的性能

优点：无需改变应用程序的架构就能提升性能

缺点：硬件性能的有上限；高性能机器的成本高；可用性低

（2）水平扩展（horizontal scaling/scale out）

定义：、将软件运行在多个用网络连接的机器上，在逻辑上视为单体

优点：
    * 高性能：并行处理可以缩短执行时间；负载均衡将工作均匀地分配给各个节点，提高整个系统的性能
    * 可扩展性：通过增加服务器数量来处理更多的请求和存储更多的数据
    * 高可用性：如果一个服务器发生故障，其他服务器可以接管其工作，保证系统的持续运行
    * 成本效益：相较于购买高性能服务器，购买和维护多台中低端服务器可能成本更低

缺点：
    * 复杂性：需要管理和协调多台服务器的工作，这可能会带来额外的复杂性。例如，数据的一致性和同步问题
    * 软件支持：一些旧的或者非分布式的系统可能需要进行大量的修改才能在分布式环境中运行
    * 网络通信：增加服务器数量会增加网络通信的复杂性，可能会导致网络瓶颈，降低系统性能

当流量比较低时，垂直扩展是一个好的选择，它的主要优势是简单。但是由于它的几个缺点，大规模系统更倾向于水平扩展。

## 负载均衡器（Load balancer）

解决的问题：

* 如果用户直连服务器，那么服务器宕机时，用户将不能访问网站
* 如果许多用户同时连接到服务器并且达到服务器的负载上限时，服务器响应就会很慢甚至断开连接

负载均衡器：均匀地将流量分散到多个 web 服务器

模型变为：user --> load balancer --> server

加上负载均衡器后，可以提高 web 层的可用性：

* 如果某个服务器下线，负载均衡器可以将流量切到其它机器
* 如果流量增长地非常快，可以增加更多的服务器，负载均衡器自动将流量路由到这些新机器

为了更好的安全性，负载均衡器与服务器使用私有 IP 相连接

## 数据复制（Database replication）

主从结构：一主多从，主机支持写入操作，需要将 insert, delete, update 等操作发往主机执行；从机只支持读取操作，由于大多数场景是读多写少的，所以从机一般会有多个

优点：

* 更好的性能：多个从机允许更多的读请求被并行处理
* 高可靠性：即使因台风或地震等自然灾害导致数据库损坏，数据也不会丢失，因为已经被复制到多台机器了
* 高可用：把数据复制到多个地区后，即使某个数据库下线了，你还可以继续访问其它数据库

如何提升了系统的可用性：

* 如果唯一的一台从机下线了，可以将读操作定向到主机。等故障被发现后，会启动新的从机，再将流量导向新从机
* 如果主机下线了，某个从机将被提升为新的主机，同时新增一台从机。在生成环境中，这个选择新主机的操作比较复杂，要考虑新主机的数据是否完整

## 缓存（Cache）

### 缓存的作用

缓存是一个临时存储区域，保存了昂贵响应的结果或者频繁访问的数据，以便于后续的读取请求能够更快地被响应

### 分类

（1）Cache Aside

（2）Write Through

实现：

* 所有写操作

优点：

* 数据被同时写入缓存和存储系统，减少了数据丢失的风险

缺点：

* 影响写操作的性能

（3）Write Behind

实现：

* 当数据被写入缓存时，就给客户端返回成功
* 数据在后台异步写入存储系统，例如定期更新或者数据卸载时

优点：

* 提高了写操作的性能

缺点：

* 增加了数据丢失的风险。如果缓存在数据持久化前就宕机了，数据就丢失了

## 内容分发网络（Content delivery network, CDN）

待续