# TrueTime

## 背景

由于时间漂移，多节点时钟不一致，导致无法直接比较时间戳。

## 实现

利用 TrueTime，可以直接比较事务的时间戳来判断事务发生的先后顺序。同时要支持全球部署。

### 整体架构

![truetime](https://raw.githubusercontent.com/YanQiu0207/image/main/truetime.PNG)

* 每个 data center 具有若干个 time master 进程，每台机器上运行一个 time slave 进程。
* time master 提供精确的时间，大多数 time master 配备 GPS 时钟，少数 time master 配备原子钟。
* time slave 每隔 30 秒从 time master 同步时间。当需要校准 local clock 时，从多个 time master 获取时间信息，通过算法识别并丢弃异常时间信息，并将多个时间信息归并到一个统一的时间。

> 为什么 time master 有些配备 GPS 时钟，有些配备原子钟？

因为这两种硬件的故障原因是正交的，同时使用可以提高硬件可靠性。GPS 时钟的故障原因有天线或接收器故障，电磁干扰和电子欺骗等；原子钟可能由于频率问题造成时钟漂移，这两种原因不相交。

> time slave 的时间误差是多少？

TrueTime 的数据来源是节点的 local clock，所以误差来源有：

1. 到 time master 的网络延迟，大概是 1 ms
2. local clock 的漂移。按照最坏的漂移速度 (200 us/s) 来计算，由于每隔 30 s 的同步一次，误差大概是 0~6ms （可能是提前，也可能是落后），通常是一个关于时间的锯齿形函数。

所以，TrueTime 总的误差是 1~7 ms 内变化。因此，大多数情况下，平均误差是 4 ms 左右。

> TrueTime 与 NTP 的关键区别是什么？

时间误差小并且误差是有上限的。

### 底层原理

#### Marzullo 算法

如何保证任意机器的时间误差有上限？

如何保证所有事务的时间戳具有可比性（基于同一个参考系）？

保证绝对时间一定落在了 TT.now() 返回的区间之中。基于这个保证，使得全球所有服务器分配的时间戳都是基于同一参考系，具有可比性。

#### Commit Wait

在 TrueTime 的基础上，引入一个 Commit Wait 的限制，保证了 External Consistency，即：如果事务 T2 在事务 T1 提交后才开始执行，那么事务 T2 的提交时间戳肯定大于事务 T1 的提交时间戳。

对于一个事务$T_i$，定义开始事件为$e_i^{start}$，定义提交事件为$e_i^{commit}$，定义提交时间戳为$s_i$，那么对一致性的要求就变成了：

$$
t_{abs}(e_1^{commit}) < t_{abs}(e_2^{commit}) \implies s_1 < s_2
$$

关键在于 Commit Wait，等待 T1 的时间戳成为过去时才提交事务，即事务提交的绝对时间大于选择的事务时间戳。

证明过程如下：

![proof](https://raw.githubusercontent.com/YanQiu0207/image/main/proof.PNG)

* commit wait 规则：事务1的提交时间戳 s1 小于事务1的真实提交时间
* assumption：事务1提交后，事务2才开始
* causality：事务2先开始，再发送请求到服务器
* start：节点为事务选择的时间戳，一定大于等于当前的绝对时间，即大于等于 `TT.now().lastest`
* transitivity：事务1的提交时间戳小于事务2的提交时间戳

read-write 事务提交步骤如下：

![20220711224819](https://raw.githubusercontent.com/YanQiu0207/image/main/20220711224819.png)

> Commit Wait 消耗的等待时间至少是平均误差的 2 倍？

关键点在于 **TT.now() 返回的是一个时间区间，不是一个时间点**。它保证真实时间在这个区间。当你为某个事务选择提交时间戳时，获取的时间区间是 [t1-e, t1+e]，选择的时间戳是 t1+e。根据 Commit Wait 规则，你需要等到某个时间点的区间 [t2-e, t2+e] 大于 t1+e时才能提交事务，即 t2-e > t1+e，调整下就是 t2-t1 > 2e，也就是说，需要的等待时间至少是平均误差的 2 倍

### 快照读

基于 External Consistency，Spanner 可以感知操作的先后顺序。给定一个时间戳 t，Spanner 能够识别哪些是历史数据，并提供一致性快照。

### 误差

对误差测量要求非常高，如果实际误差大于测量误差，那么绝对时间可能飘到了区间外面，后续操作无法保证 External Consistency.

误差越大，延迟越高，吞吐量越小。

自 Spanner 在 OSDI12 发表论文后，Spanner 一直在努力减少误差，并提高测量误差的技术

## 速记

1. 保证任意机器的时间戳差距不超过一个上限值 E
2. 当 coordinator 选好时间戳后，会等待一个 E 的时间，再告诉参与这个事务的分区执行提交操作。等待的原因是保证提交时所有参与此事务的机器的本地时间都超过了将要提交事务的时间戳，所以这些分区之后发生的任何事务的时间戳，都必定大于之前提交事务的时间戳。

## 参考

- [简单解释 Spanner 的 TrueTime 在分布式事务中的作用](https://zhuanlan.zhihu.com/p/44254954)
- [关于 Spanner 中的 TrueTime 和 Linearizability](https://zhuanlan.zhihu.com/p/35473260)
- [True Time 算法的实现与思考](https://github.com/luohaha/MyBlog/issues/5)
- [分布式事务实现机制对比](https://zhuanlan.zhihu.com/p/334742480)