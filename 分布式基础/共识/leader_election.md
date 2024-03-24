# leader 选举

## Master

### 定义

任一时刻，仅有一个节点成为 leader 或没有任何节点成为 leader，我们把这种 leader 称作 Master

### 选举

**选举**

（1）任意节点都可以发起 BeMaster 操作将自己提升为 Master
（2）如果获知自己成为 Master，那么从 BeMaster 开始的 timeout 时间内可认为自己是 Master
（3）如果得知其他人已被选为 Master，那么必须等待 timeout 时间才能发起 BeMaster 操作

**算法的正确性**

- 使用共识算法保证，最多只有一个节点会被选举为 Master，并且选举成功后任何人看到的都是同一个节点
- Master 的单点性由租约机制保证。由于恒定 T(BeMaster) < T(Know other as master)，那么 Master 实际过期时间肯定要比非 Master 节点认为 Master 过期的时间早，从而保证 Master 任期内肯定不会出现其它节点来尝试抢占 Master

**续期**

只需要在 Master 任期内完成一次 BeMaster 操作，即可延长 Master 任期。

考虑一种情况，A, B，C 三个节点组成一个集群，节点 A 是 Master 且在不断续期。节点 C 可能因网络故障或节点故障，与节点 A 无法通信，错过了 A 的好几个任期。节点 C 认为 A 的 Master 任期过期后，即可尝试发起 BeMaster 操作，违背了算法的保证。

### 疑问

- 续期的实现？增加版本号，再执行一次 paxos 实例？
- 节点 C 发起 BeMaster 操作，递增版本号，会发现已有 Master，应该没有副作用才对？

## Raft

### 定义

任一时刻，可以有多个节点自认为是 leader，但这些 leader 的任期编号必定不同，只有任期编号最大的 leader 才能真正写入数据

### 选举

待续。

### 疑问

（1）Raft 的选举算法跟 Paxos 有关系吗？

答：没关系

在某个任期内，follower 只会投票一次，这可能会造成投票分裂的情况。而 Paxos 算法出于活性考虑，Acceptor 可以多次接受提案，这是两者最大的不同，也就是说 Raft 某个任期内的选举 != 一个 Paxos 实例（Acceptor 可以多次接受提案）

但 Raft 选举算法不会出现长时间选举不出 leader 的情况，因为 follower 收不到 leader 的信息，会转为 Candidate 发起新任期的投票，选举新 leader。在 Raft 中，不同任期的 leader 可以是不同的，也就是说 Raft 多个任期的选举 != 一个 Paxos 实例（只能对一个值达成共识）

## Multi-Paxos

### 定义

待续。


## 参考资料

- [Paxos理论介绍(3): Master选举](https://zhuanlan.zhihu.com/p/21540239)
- [使用Multi-Paxos协议的日志同步与恢复](http://oceanbase.org.cn/archives/111)