# paxos可靠直观的解释

paxos 的工作，就是把一堆机器协同起来，让分布式系统对外表现得像是单个系统

## 背景

目标：系统可用性要需要达到 9 ~ 13 个 9
现状：硬件不可靠，例如磁盘宕机、服务器宕机、IDC 间丢包
解决方法：数据冗余，即多副本策略，一份数据存多份，当一份数据丢失时，还有其它数据可用。
带来的问题：如何保证多副本对外表现的只有一个副本？ 这就需要 paxos 这类共识算法来保证。

## 复制策略

### 主从复制

异步复制：主库写入本地磁盘后就给客户端响应成功，不需要等待数据同步到从库。缺点就是：如果主库还未将数据同步到从库就发生磁盘损坏，那么数据就丢失了。

同步复制：主库必须等所有从库都写入成功后，才能给客户端响应成功。缺点就是：任何一个从库失败，都会阻塞系统继续运行（为了保证系统一致性，主库必须按顺序执行请求）

半同步复制：主库只需要把数据复制到足够多（并不是全部）的机器上就可以给客户端响应成功。优点：1 台机器宕机也不会让整个系统停止写入。缺点：如果主库将数据 a 复制到从库 1，将数据 b 复制到从库 2，那么主库宕机后，没有一个从库拥有完整的数据，导致系统对外表现得不一致。

为了解决半同步复制中数据不一致的问题，可以采取**多数派读写**的策略：每条数据必须写入半数以上的机器，每次读取的数据都必须检查半数以上的机器是否有这条数据。任意一个机器故障，数据也不会丢失，系统仍能对外提供服务。

对于一条数据的更新，多数派读写会产生不一致的状态，例如 x=a 写入了 node-1 和 node-2，然后 x=b 写入了 node-2 和 node-3，如果读 node-1 和 node-2，x 的值就要两个，哪个才是正确的呢？

解决办法：需要引入一个全局递增的时间戳对写入进行排序，最后写入者胜利

即使有了时间戳，在客户端没有完成一次完整的多数派写入时，整个系统对外提供的信息仍然是不一致的，例如 x=a 写入了 node-1 和 node-2，然后 x=b 写入了 node-3，如果读 node-1 和 node-2，x 的值是 a，如果读 node-1 和 node-3，x 的值是 b，不符合线性一致性的定义。

> 当正常情况不可能出现不一致时，多想想异常情况，例如网络分区、节点故障、处理延迟无上限等

### 无主复制

待续。

### 多主复制

待续。

## 从多数派到 paxos

（5）如果两个客户端同时进行读改写操作，可能产生一个客户端覆盖另一个客户端的情况。解决办法：

- 加锁 --> Redis redlock 的实现
- CAS --> ABA --> paxos

（5） paxos 不解决存储不可靠和消息错误的问题，解决这些问题的 paxos 请参考 [Byzantine Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science)#Byzantine_Paxos)

- 存储必须是可靠的：没有数据丢失和错误
- 容忍：消息丢失和乱序

## paxos 协议

### 概念

- Proposer：发起 paxos 的进程
- Acceptor：存储节点，接收、处理和存储消息
- Quorum：n/2+1 个 Acceptors
- Round：1 轮包含 2 个阶段：Phase-1 & Phase-2

每 1 轮的编号（rnd）：

- 全局唯一（用于区分 Proposer）
- 单调递增（用于区分先后顺序）
- 后写胜出

在 Acceptor 也有几个概念：

- last_rnd：Acceptor 记住的最后一次进行写前读取的 Proposer，以此来决定谁可以在后面真正把一个值写到存储中
- v 是最后被写入的值
- vrnd 跟 v 是一对，记录了v 是在哪个 Round 中被写入的

> v 和 vrnd 是用于恢复一次未完成的 paxos 用的

### 基本流程

#### 阶段 1

- Proposer
  - 发送请求，带上自己的 rnd（全局单调递增）
- Acceptor
  - 如果请求中 rnd 小于 Acceptor 的 last_rnd，则拒绝请求
  - 如果请求中 rnd 大于 Acceptor 的 last_rnd，将本地 last_rnd 设置为请求的 rnd
  - 返回应答，带上自己之前的 last_rnd 和 之前已接受的 v

> 如果请求中 rnd 等于 Acceptor 的 last_rnd，说明是同一个 proposer 发的重复消息，因为 rnd 是全局唯一的

#### 阶段 2

- Proposer
  - 如果 Proposer 收到多数个应答，就可以继续运行，否则失败退出
  - 如果应答中的 last_rnd 大于发出的 rnd，则失败退出
  - 选值：
    - 如果所有应答都没有非空的 v，这表明之前没有客户端完成了写入（因为一个多数派的读一定可以看到一个多数派写）。这时， Proposer 可以写入自己的值。
    - 如果收到了某个应答中被写入了的 v 和 vrnd，那么可能其它客户端正在运行（可能结束也可能没结束）。但已经写入的值不能被修改，所以必须保持原值。于是需要将最大 vrnd 对应的 v 作为想要写入的值。
  - 发起请求，写入选定的值

- Acceptor
  - 如果请求的 rnd 跟 本地的 last_rnd 不一致，拒绝
  - 如果请求的 rnd 跟 本地的 last_rnd 一致，则将请求中的 v 和 rnd 写入本地

> 对于 Proposer，如果少于一半的应答的情况下，paxos 算法继续运行会出现什么问题？
> 非多数派读可能漏掉多数派写（即两个节点集合没有交集），修改已经确定的数据。例如，rnd=1 写入了 node-2 和 node-3，但 rnd=1 只读取了 node-1，发现之前没有其他客户端写入，就往所有节点写入了自己的值，覆盖了已经确定的值。

> 对于 Proposer，如果应答中的 last_rnd 大于发出的 rnd，paxos 算法继续运行会出现什么问题？
> 说明已经有更大 rnd 的客户端先读取了 Acceptor，本客户端的写入必定失败，即后写胜出规则。

> 将最大 vrnd 对应的 v 作为想要写入的值
> 如果小的 vrnd 对应的 v 被写入了，其他 proposer 的多数派读不可能看不到。所以，只有最大 vrnd 对应的 v 是可能要达成多数派的值。

### 改进

#### multi paxos

优化：用一次 RPC 为多个 paxos 实例运行 phase-1，平均 1 个 paxos 实例只需要 1 个 RPC

举个例子：Proposer X 可以一次为 10 个值运行 phase-1，然后分别为这 10 个运行 phase-2，那么只需要 11 次 RPC。相比 classic paxos，省下了 9 个 RPC。平均下来，一个 paxos 实例只需要 1 个 RPC。

#### fast paxos

（1）特点：

- 没冲突：1 轮 RPC 确定一个值
- 有冲突：2 轮 RPC 确定一个值

（2）想法


如果系统只有一个 Proposer，那么 phase-1 就是多余的，因为不存在并发操作。怎么实现系统只有一个 Proposer 呢？可以选主。但网络分区的情况下，系统中还是可能有两个主节点，此时还是可能有冲突。

（3）算法

- Proposer
  - 直接发送 phase-2
  - rnd 是 0，因为 0 保证一定小于 classic rnd，所以可以在出现冲突时安全回到 classic paxos
- Acceptor
  - 只有在 v 是空时才接受 fast phase-2 请求

如果发生冲突，回退到 classic paxos，用一个 rnd > 0 来运行。

> fast-paxos 的 quorum 跟 classic paxos 的 quorum 是相同的吗？
> 为了在退化成 classic paxos 不会选择不同的值， fast-paxos 达成多数派的 Acceptor 的个数必须在 classic paxos 多数派 Acceptor （n/2+1）中占大多数，即 >= 3/4n + 1

#### 其它

在 phase-2，Acceptor 可以接受 rnd >= last_rnd 的请求

### 疑问

> classic paxos 也是 phase1 选主，phase2 复制

> 一个 paxos 实例只负责一个值的确定，新的写入需要创建新的实例。至于多个实例之间怎么组成一个变量的多次修改就是上层的事情了

> 在一个新的实例下，如果 Acceptor 读出了所有未完成的 paxos 实例，就必须去恢复它。如果没有才能做自己想做的事

> 如何用这个 Paxos 去设计一个 KV database
> 在 kv 系统中，不是每个变量对应一个 paxos 实例，而是每个变量的每次变化对应一个 paxos 实例，即变量在存储上是多版本的

### 参考文章

- [On the Parallels between Paxos and Raft, and how to Port
Optimizations](http://mpaxos.com/pub/raft-paxos.pdf)