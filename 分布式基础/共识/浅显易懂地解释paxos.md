## 选值策略

场景 2-3-2 Pj 的提案 Proposal-j 最终会被法定集合 Q-j 接受，即 v 的值被决定为 b，且 Proposer-i.proposal-id > Proposer-j.proposal-id，必须要让 Proposer-i.v == Proposer-j.v，否则 v 就会被决定为另一个值，违背算法的安全性要求。

要让 Proposer-i.v == Proposer-j.v，Pi 需要先主动询问进程集合。

Acceptor 不知道哪个 Porposal 会被通过，它只知道自己接受了哪些 Porposal，所以回复自己知道的所有 Proposal 就行了

Pi 接收到了法定集合 Q-i 知道的所有提案，记这个提案集合为 K-i，那么 Pi 应该从中选择哪个提案的值作为提案 Proposer-i 的值呢？

假设这个被选中的提案是 Proposer-m，我们希望 Proposer-m 是 Proposer-j，实际上只需要 Proposer-m.v == Proposer-j.v 就能满足要求。反过来看，K-i 中可能存在提案 Proposal-f 且 Proposal-f.v != Proposer-j.v

我们不妨假设存在一个选择策略 CL，CL 能够让选择出的提案 Proposer-m.v == Proposer-j.v。然后我们分析下 Proposal-f 有什么特征，因为这个特征的反面可能就是选择策略的具体形式。

Proposal-f 能够被提出，说明一定存在一个法定集合 Q-f，它里面的每个进程都接受了 PreProposal-f。

Q-f 和 Q-j 必定存在一个公共进程，记作 Ps，Ps 既接受了 PreProposal-f，也接受了 Proposal-j，但先后顺序未知，但只有两种可能：

（1）Ps 先接受 PreProposal-f
（2）Ps 先接受 Proposal-j

PreProposal-f.proposal-id 和 Proposal-j.proposal-id 大小关系也未知，不妨先假设 PreProposal-f.proposal-id > Proposal-j.proposal-id

对于情形 1，Ps 先接受 PreProposal-f，会导致后续无法再接受 Proposal-j，即 Proposal-j 未达成一致，与假设不符合。

对于情形 2，Ps 先接受 Proposal-j 后接受 PreProposal-f。由于假设了策略 CL 存在，那么 Proposal-f.v == Proposal-j.v，与 Proposal-f 的假设矛盾。

当假设 PreProposal-f.proposal-id > Proposal-j.proposal-id，情形 1 和 2 都得出了矛盾。另外，提案的编号各不相同，所以必定 PreProposal-f.proposal-id < Proposal-j.proposal-id，即 Proposal-f.proposal-id < Proposal-j.proposal-id

结论：假设提案 Proposal-j 会被通过，策略 CL 存在。对于任意一个 proposal_id 更大的提案 Proposal-i，它预提案阶段获得的所有提案 K-i，其中存在 Proposal-f.v != Proposal-m.v，那么 Proposal-f.proposal-id < Proposal-j.proposal-id

也就是说，Proposal-m.proposal-id >= Proposal-j.proposal-id，Proposal-m.v == Proposal-j.v。我们只需要选择 proposal-id 最大的那个提案的值，就能令 Proposal-i.v == Proposal-m.v。这就是策略 CL 的具体形式。

## 形式证明

CP1：如果一个提案 Proposal-j 最终会被通过，那么任意一个提案 Proposal-i，如果 Proposal-i.id > Proposal-j.id，那么 Proposal-i.v == Proposal-j.v

辅助条件：选值策略是从回复预提案的多数派集合的所有提案中选择编号最大的那个提案。

**第一种证明方法（归约法）**：首先假设 Proposal-i.v != Proposal-j.v，如果能得出矛盾，就能证明 CP1.

记大多数进程的集合为 Q
记预提案阶段 Q 中所有进程回复的所有提案集合为 K，其中编号最大的提案为 MaxProposal(K)

如果一个提案 Proposal-j 最终会被通过，那么必定存在一个集合 Q-j，其中每个进程都接受了 Proposal-j

回复 Proposal-i 预提案的进程集合为 Q-i

由于两个多数派的交集必定不为空，所以一定有一个进程 Pk 既接受了 Proposal-j，又回复了 Proposal-i，但顺序有两种：
- Pk 先接受了 Proposal-j
- Pk 先回复了 Proposal-i

如果 Pk 先回复了 Proposal-i，且 Proposal-i.id > Proposal-j.id，那么 Pk 不可能接受 roposal-j，这种情况违背了前提条件。

所以 Pk 一定是先接受了 Proposal-j，后回复了 Proposal-i。也就是说，Q-i 回复的所有提案 K-i 中一定包含了 Proposal-j。

Proposal-i 的值来自 K-i 中编号最大提案的值，记这个提案为 Proposal-m, m < i. 由于 Proposal-i.v != Proposal-j.v 且 Proposal-i.v == Proposal-m.v，所以 Proposal-m != Proposal-j

至此，我们可以得到一个结论，基于一个提案 Proposal-j 最终会被通过的前提下，如果存在一个提案 Proposal-i，Proposal-i.id > Proposal-j.id 且 Proposal-i.v != Proposal-j.v，那么必然存在一个提案 Proposal-m，Proposal-m.id > Proposal-j.id 且 Proposal-m.v != Proposal-j.v，

记如果存在一个提案 Proposal-i，Proposal-i.id > Proposal-j.id 且 Proposal-i.v != Proposal-j.v 为 CF(j, i)，那么上述结论可以简化为

基于一个提案 Proposal-j 最终会被通过的前提下，如果 CF(j, i) 存在，必然 CF(j, m) 存在，且 j < m < i. 

这个过程可以无限递归下去，但区间范围是不断缩小的，最终会递归到一个区间 CF(j, j+1) 存在，但找不到一个区间 CF(j, y) 存在，此时 j < y < j+1。与上述结论矛盾。由于 CF(j, m) 不存在，所以 CF(j, i) 不存在，即假设不成立。

**第二种证明方法**

假设存在一个提案的非空集合 T，集合中的任意一个提案 Proposal-k.id > Proposal-j.id 且 Proposal-k.v != Proposal-k.v。假设提案 Proposal-i 是集合 T 中编号最小的提案。因为集合非空，所以提案 Proposal-i 必定存在。

类似上述证明方法，我们知道必然存在一个提案 Proposal-m，Proposal-m.id > Proposal-j.id 且 Proposal-m.v != Proposal-j.v，j < m < i，即 Proposal-m 也属于集合 T，但 m < i，与 Proposal-i 是集合 T 中编号最小的提案矛盾。

也就是说，集合 T 不存在，即任意一个提案 Proposal-k.id > Proposal-j.id，必定 Proposal-k.v == Proposal-k.v

## 常见问题

xxxxx

## 参考文章

- [如何浅显易懂地解说 Paxos 的算法？](https://www.zhihu.com/question/19787937)
- [微信 PaxosStore：深入浅出 Paxos 算法协议](https://mp.weixin.qq.com/s/aJoXSQo9-zmukN2RsiZ3_g)
- [朴素 Paxos 算法理论推导与证明](https://mp.weixin.qq.com/s/eeJXS5rBA9mXpSJaTNjF-Q)
- [从 Paxos 到 Multi-Paxos (一)](https://zhuanlan.zhihu.com/p/432800857)
- [从 Paxos 到 Multi-Paxos（二）](https://zhuanlan.zhihu.com/p/477462091)