# 混合逻辑时钟（HLC）

## 原理

对于分布式系统中的每个事件，HLC 分配了一个混合时间戳。这个时间戳包括 wall time 和 logic time 两部分，

*   wall time 是该节点感知的当前系统最大物理时间，而不是该节点此时此刻的物理时间
*   logic time 用于比较同一个 wall time 发生的多个事件之间的先后顺序

HLC 满足以下三个性质？

HLC 依赖各节点物理时间的 clock skew 有上界，不会无限增长。

HLC 实现中实际为每个事件准备了三个维度的计量：
*   pt.j：事件 j 发生时的物理时钟值 
*   l.j：事件 j 发生时节点所感知到的系统最大物理时钟值
*   c.j：事件 j 的逻辑时钟部分，这里 c.j 是用于几个事件在同一个物理时钟值内发生时比较之间的因果关系

本质上，HLC 要维护 <l.j, c.j> 这个二维向量不回退

（1）发生本地事件或发送事件时，假设事件为 j，那么

```python
if pt.j <= l.j:
    c.j = c.j + 1
else:
    c.j = 0; l.j = pt.j

h.j = (l.j, c.j)
```

（2）假设消息为 m，接收事件为 j，那么

l.m：消息 m 携带的物理时间
c.m：消息 m 携带的逻辑时钟
l.j：事件 j 发生时节点感知的最大物理时间
pt.j：事件 j 发生时节点物理时间
c.j：事件 j 发生时逻辑时钟

```python
if l.j == l.m == pt.j:
    #如果三个时间相等
    c.j = max(c.j, c.m) + 1
else if pt.j <= l.j and l.m <= l.j:
    #如果节点感知的最大物理时间最大
    c.j = c.j + 1
else if pt.j <= l.m and l.j <= l.m:
    #如果消息携带的物理时间最大
    c.j = c.m + 1; l.j = l.m
else:
    #如果本地时间最大
    c.j = 0; l.j = pt.j
```

## 应用

### 基于 HLC 的分布式事务时钟算法

max_ts: 节点已知的系统最大物理时间
local_phys_ts: 节点获取的物理时钟，单位为毫秒

时钟操作：

*   ClockCurrent(): max_ts = max{max_ts, local_phys_ts}
*   ClockUpdate(ts): max_ts = max{max_ts, ts}
*   ClockAdvance(): max_ts = max{max_ts, local_phys_ts} + 1

事务过程：

![20230814005145-2023-08-14](https://raw.githubusercontent.com/YanQiu0207/image/main/20230814005145-2023-08-14.png)

1. 版本时间戳递增
2. 全局一致性正确性证明

总结：HLC 可以保证跨 session 之间的快照隔离（内部一致性），以及同一个 CN 的强外部一致性，跨 CN 的弱外部一致性：通过引入先进的 PTP 协议，也可以在一个 IDC 保证外部强一致性

### 何登成



### mvcc 与 hlc

### 参考

知乎

Park 数据库小白
贺大伟 我思想的戒指适合戴在我的手指上
triump 数据库内核开发
虎啾啾
张沟狂人 程序猿-数据库内核


[CockRoachDB分布式事务 - HLC和MVCC的相映成趣](https://zhuanlan.zhihu.com/p/265226466)
[为什么 CockroachDB 的跨地域性能远超同行](https://zhuanlan.zhihu.com/p/600284531)
[详解CockroachDB事务处理系统](https://zhuanlan.zhihu.com/p/26908120)
[CockroachDB 事务处理](https://zhuanlan.zhihu.com/p/543497168)
[CockroachDB架构浅析](https://zhuanlan.zhihu.com/p/332764423)

[分布式系统一致性的发展历史](https://danielw.cn/history-of-distributed-systems-1)
[Linearizability, Serializability and Strict Serializability](https://zhuanlan.zhihu.com/p/26893940)
[A Critique of ANSI SQL Isolation Levels](https://zhuanlan.zhihu.com/p/187597966)

[CockroachDB一致性讨论](https://zhuanlan.zhihu.com/p/462398795)
[从CockroachDB理解分布式数据库系统的一致性](https://zhuanlan.zhihu.com/p/405887567)
[CockroachDB's consistency model](https://www.cockroachlabs.com/blog/consistency-model/)

[CockroachDB 和 Spanner 的分歧点](https://zhuanlan.zhihu.com/p/571608056)
