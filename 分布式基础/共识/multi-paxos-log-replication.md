## 使用 Multi-Paxos 协议的日志同步与恢复

### Leader 的产生

（1）为什么需要 Leader ?
在 Leader 的有效期内，所有提案都只能由 leader 发起。由于没有了并发冲突，日志同步可以省略 logID 阶段和 prepare 阶段。唯一的 leader 产生 logID，然后直接发送 accept 请求，得到多数派确认即日志同步成功。

注意，Multi-Paxos 允许出现多个**自认为是 leader 的 server**来并发生成日志，此时退化为 Basic-Paxos

（2）选举方法
领导者选举（Leader Elect）：经过一轮 Basic-Paxos，成功得到多数派 accept 的 proposer 即成为 leader。

对于 Leader Elect 过程，我们并不关心决议的具体内容。从 Basic-Paxos 的角度看，无论执行多少次 Leader Elect，都是同一个 Paxos Instance 对已经形成的决议反复进行投票而已。我们最关注的是最近一次形成决议的 proposer 和 proposalID。得到多数派 accept 的 proposer 将成为 Leader，而它本次使用的 proposalID 将成为任期内对所有日志进行投票时将要使用的 proposalID

（3）如何避免多个 leader
如果多个 server 并发执行 Leader Elect，可能出现两个 server 先后执行 Leader Elect 成功，都认为自己是 Leader 的情况。因此，proposer 在成为 leader 前，要先以 proposalID 写一条日志（称为 StartWorking 日志），得到多数派确认后，再提供服务。因为经过两轮成功的 Leader Elect，集群中多数派持久化的 proposalID 必定是后面那轮的 proposalID。这样前面那个 leader 执行 accept 时，由于 proposalID1 < proposalID2，必定无法得到多数派的 accept，从而知道自己不应该是 leader。

### confirm 日志的优化

（1）背景

日志内容的读取也需要执行一轮 paxos 过程，将 accept 阶段成功持久化的日志内容返回。需要注意，选择日志内容的逻辑变化：proposer 收集到多数派的响应后，如果响应都为空，那么向所有 acceptor 发出日志内容为空的 accept 请求；否则选择 proposalID 最大的日志内容发出 accept 请求，后续收到多数派的成功回复后，才能可以返回日志内容作为读取结果。

为什么读取也需要执行 paxos 过程？针对每一个 logID，读取出的内容后应该永远不变。因此，如果读取时某个 logID 还未在多数派上写入成功，需要在返回结果前，将这个结果在多数派机器上持久化成功

（2）问题

在实际工程做数据恢复时，对每条数据都执行一轮 paxos 过程的代价过大

（3）解决方案

引入 confirm 机制：Proposer 在 accept 阶段收到多数派机器的成功回复后，写一条 confirm 日志，然后同步给所有的 accptor。等 acceptor 回放日志时，发现有对应的 confirm 日志，就不需要执行 paxos 过程，可直接使用

为了性能考虑，confirm 日志会延迟、批量同步给 acceptor，所以仍然可能出现日志已经形成多数派备份，但是没有对应的confirm日志的情况，对于这些日志，需要在恢复过程中进行重确认。

在实际的工程实践中，可以使用基于 logID 的滑动窗口机制来限制 confirm 日志与对应的原始日志的距离，以简化日志回放与查询逻辑

### 新任leader对日志的重确认

在恢复过程中，如果原始日志有相应的 confirm 日志，那么可以直接回放，否则需要进行重确认

重确认有两种情况：

*   一是有原始日志，但无 confirm 日志
*   二是无原始日志，因为当前 leader 在上一个 leader 的任期内错过了一些日志同步，而这些日志在其他机器上形成了多数派。由于 logID 连续递增，被错过的日志成了连续递增序列中的空洞，需要通过重确认来补全

重确认方法：

*   向集群内其它机器发送请求，查询每个机器的最大 logID，从多数派的应答中选择最大的 logID 作为重确认的结束位置
*   对于每条的日志的重确认，需要执行一轮完整的 paxos 过程。不管日志是否曾经形成过多数派，都重试尝试持久化，我们称之为**最大 commit 原则**。之所以要这么做，是因为我们无法区分出未形成多数派备份的日志。可能日志之前已形成过多数派备份，但由于宕机，导致机器数量不足；也可能日志真的就是没形成过多数派，并且肯定也是没有应答客户端的

重确认日志时，要以当前 leader 的 proposalID 作为 paxos 的 proposalID。因此回放日志时，对于 logID 相同的多条日志，要以 proposalID 最大的为准

### 幽灵日志的复现

幽灵复现问题：上次查询不到的日志又像幽灵一样出现了

解决办法：生成日志时，leader 以当前的 leader proposalID 作为 generateID，和日志保存在一起。按 logID 回放日志时，因为 leader 在开始服务前一定会写一条 startworking 日志，所以如果出现 generateID 相对前一条日志变小的情况，说明这是一条幽灵复现日志，要忽略掉