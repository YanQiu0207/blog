#  Paxos

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

## 参考文章

- [如何浅显易懂地解说 Paxos 的算法？](https://www.zhihu.com/question/19787937)
- [微信 PaxosStore：深入浅出 Paxos 算法协议](https://mp.weixin.qq.com/s/aJoXSQo9-zmukN2RsiZ3_g)
- [朴素 Paxos 算法理论推导与证明](https://mp.weixin.qq.com/s/eeJXS5rBA9mXpSJaTNjF-Q)
- [从 Paxos 到 Multi-Paxos (一)](https://zhuanlan.zhihu.com/p/432800857)
- [从 Paxos 到 Multi-Paxos（二）](https://zhuanlan.zhihu.com/p/477462091)