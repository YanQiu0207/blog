## 摘要

**目的**：提高粗粒度锁和可靠的小容量存储

**设计重点**：可用性和可靠性，而不是高性能

## 介绍

目的：允许客户同步它们的活动，以及对它们环境的基础信息达成一致（*不好理解*）

主要目标：可靠性、可用性、易于理解
次要目标：吞吐量、存储容量

客户接口：类似于读写整个文件的简单文件系统，增加了建议锁和事件通知（例如修改事件）

帮开发者处理系统间的粗粒度同步，特别是处理从一组等效服务器选举一个领导者这样的问题，例如

- 选举一个 master
- 存储少量的元数据
- 服务注册和服务发现

## 设计

### 根本原因

为什么提供一个中心化的锁服务而不是一个 Paxos 库？

首先，将一个服务改造成高可用的，使用锁服务可以很容易保持原有的代码结构和通信模式。

> 主节点的所有操作都要带上一个序列号，目标服务器可以根据这个序列号拒绝旧主节点的请求

其次，需要一种机制来公布结果，例如谁是主节点，或者分区在哪个服务器上。不使用命名服务的原因是可以减少服务器数量和协议的一致性特性是共享的（*不理解*）

> 使用一致的客户端缓存而不是基于时间的缓存是因为缓存时间不好确定，

第三：对于开发者来说，基于锁的接口更好理解。

最后：可以减少应用服务器数量，即使只有一台，也能继续工作。而分布式一致性算法使用多数派来做决策，所以需要多个副本来达到高可用。

两个关键设计决策：

- 选择了锁服务，而不是共识算法库或共识服务
- 提供小文件读写以允许主节点公布身份或者其它业务，而不是构建和维护另外一个服务

不期望锁用于细粒度场景，即持有时间很短（秒甚至更少），相反，我们期望锁用于粗粒度场景，即持有时间较长（小时或天）

粗粒度锁：

- 对锁服务器施加的负载小得多
- 获取频率很低，所以短暂地不可用对客户端的延迟影响很小

>  On the other hand, the transfer of a lock from client to client may require costly recovery procedures, so one would not wish a fail-over of a lock server to cause locks to be lost. Thus, it is good for coarsegrained locks to survive lock server failures, there is little concern about the overhead of doing so, and such locks allow many clients to be adequately served by a modest number of lock servers with somewhat lower availability

细粒度锁：

- 可用性非常重要，即使是锁服务器的短暂不可用也可能导致许多客户机停止运行
- 性能和随意添加新服务器的能力非常重要，锁服务的事务速率随着所有客户机的事务速率而增长
- 在锁服务器故障期间不维护锁来减少锁的开销。因为锁的持有时间很短，问题并不大

> It can be advantageous to reduce the overhead of locking by not maintaining locks across lock server failure, and the time penalty for dropping locks every so often is not severe because locks are held for short periods. (Clients must be prepared to lose locks during network partitions, so the loss of locks on lock server fail-over introduces no new recovery paths.)

如何利用锁服务来实现应用专属的细粒度锁？

### 系统架构

## 总结