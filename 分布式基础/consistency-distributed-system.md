# 各种consistency对比

## 线性一致性（Linearizability）

两个机器上都有变量 X，node1 将其写作 A，node2 将其写作 B，「写A」操作在「写B」开始之前完成，B 一定覆盖 A
*   对于多个观察者来说，任意一个观察者观察到 B 后，其它观察者必须只能观察到 B
*   B 的操作者被告知操作完成后，所有观察者都只能观察 B

效果：虽然是多机备份，但对外表现得只有一份数据一样

## 顺序一致性（Sequential）

两个机器上都有变量 X，node1 将其写作 A，node2 将其写作 B，「写A」操作在「写B」开始之前完成，A 可能覆盖 B，B 可能覆盖 A

*   集群所有节点必须看到一样的顺序，即全局顺序一致，要么都看到 A, B，要么都看到 B，A
*   同一台机器 node1 先写 A，或者先看到 A，然后再写 B，所有节点必须用 B 覆盖 A，即节点的局部顺序在全局顺序中得到保持。对于某个观察者来说，看到 B 后就无法再观察到 A；但是写 B 被通知完成后，还是有观察者可以观察到 A

线性一致性要求 B 的操作者被告知操作完成后，所有观察者都只能观察 B，而顺序一致性则依然可以观察到 A（之后再观察到 B）。也就是说，线性一致性要求观察者看到的值必须和操作发生的真实时间顺序一致。

## 因果一致性（Causality）

两个机器上都有变量 X，node1 将其写作 A，node2 将其写作 B，「写A」操作在「写B」开始之前完成，某些节点上 A 可能覆盖 B，某些节点上 B 可能覆盖 B

同一台机器 node1 先写 A，或者先看到 A，然后再写 B，所有节点必须用 B 覆盖 A，即节点的局部顺序在全局顺序中得到保持。对于某个观察者来说，看到 B 后就无法再观察到 A

对因果一致性来说，每个进程都有自己的历史，但所有因果相关的写操作作为一个子集，是有全序关系的，并且这些操作在所有进程上观察到的顺序都一样

因果一致性只约束有因果关系的写和读，对于不是因果相关的写入是不关心顺序的

核心：读尊重因果相关的写的顺序

## 先入先出（FIFO）

同一台机器 node1 先写 A（先看到 A 的要求消失了），然后再写 B，所有节点必须用 B 覆盖 A。对于某个观察者来说，看到 B 后就无法再观察到 A

## PRAM（Pipelined RAM）

所有写操作都在彼此独立的 pipeline 里，因此任何一个进程观察到的某个进程的写都是一样的，但并不意味着所有进程的写都有一个全局顺序

这个模型假设了节点之间不存在需要处理的依赖，即不关心写操作和后继的读操作的因果关系

## 强最终一致性（Strong Eventually）

同一台机器 node1 先写 A，然后再写 B，所有节点在某个时间点后用 B 覆盖 A。在这之前，观察者可以看到各种乱序

## 最终一致性（Eventually）

同一台机器 node1 先写 A，然后再写 B，所有节点在某个时间点有同一个值，可能是 A，也可能是 B。在这之前，观察者可以看到各种乱序

## 有没有比Linearizability更难的问题?

Consensus == Total Order Broadcast == Atomic Test-Write Register; 

## 有没有比Consensus更难的问题

Atomic Commit

1. 分布式数据库的 commit 是一个 atomic commit 问题，这个问题的解分为 blocking 和non-blocking 两种

1. 其中 non blocking Weak Atomic Commitment, consensus 和 uniform consensus 这三个问题是等价的。只要你可以解决其中任何一个，你就可以解决其他两个。

1. non blocking Atomic Commitment(NB-AC)(少了weak)的 Non triviality 要求没有 failure 发生则 correct 的 node 都 commit，而 non blocking Weak Atomic Commitment(NB-WAC)的 Non triviality 要求没有任何 node 被怀疑 failure 则 correct 的 node 都 commit。就是由于这一点，NB-AC 在异步环境无解，而 NB-WAC 可解。因为异步环境我们无法得到一个完美的 failure detector，所以我们不可能完美不犯错的 detect 有没有 failure。

1. 由于 NB-WAC 和 consensus 等价，且 NB-AC 比 NB-WAC 难，所以 NB-AC 是比 consensus 要难的。

## 最终排序

Atomic commit > Consensus > Linearizability > Sequential > Causality > FIFO > Strong Eventually > Eventually

## 参考

比如1995年的ABD算法，和2017年的SCD broadcast，都可以用来实现linearazibility，而这些算法都比consensus弱；

jepsen.io/consistency

https://www.microsoft.com/en-us/research/wp-content/uploads/2011/10/ConsistencyAndBaseballReport.pdf

参见Distributed Systems: An Algorithmic Approach 2nd Edition这本书的 Chapter16 Replicated Data Management的section 16.3 Data-centric consistency models

https://en.wikipedia.org/wiki/Atomic_broadcast#Equivalent_to_Consensus

关于consensus与transaction的关系建议看两篇论文。

1. Revisiting the relationship between non-blocking atomic commitment and consensus
2. Consensus on Transaction Commit

因果一致性的应用：微信并发评论，以及 riak 的 version vector

SOFAJRaft 线性一致读实现剖析 | SOFAJRaft 实现原理
https://zhuanlan.zhihu.com/p/71828936

线性一致性和 Raft
https://cn.pingcap.com/blog/linearizability-and-raft