# Serializability

**Serializability Consistency**：事务执行的结果跟事务按某种顺序一个个执行一样

注意：很多人误认为要实现串行一致性，事务只能一个个执行。实际上，如果两个事务没有操作相同的数据，是允许并行的。此时我们并不关心它们的先后顺序，哪种执行顺序都能接受

Serial Schedule：两个事务顺序执行，之间没有重叠。这种情况符合 SC

Non-Serial Schedule：两个事务执行时间上有重叠

Equivalent Schedules：对于任意数据库起始状态，若两个 schedules 分别执行所到达的数据库最终状态相同，则称这两个 schedules 等价

Serializable Schedule：如果一个 schedule 与事务的某个串行执行顺序的效果一致，则称该 schedule 为 Serializable Schedule

当两个 operation 满足以下条件时，我们认为它们是 conflicting operations：

* 来自不同的事务的相邻 operation
* 对同一个对象操作
* 两个 operation 至少有一个是 write 操作

可以穷举 conflicting operations 的所有情况：

* Read-Write Conflicts: Unrepeatable Reads
* Write-Read Conflicts: Reading Uncommited Data
* Write-Write Conflicts: Overwriting Uncommited Data

如果能够通过交换两个事务中**连续**的 non-conflicting operations，最后把这个 schedule 变成一个 Serial Schedule，那么我们称这个 Schedule 是 Conflict Serializable。两个 Schedule 互相为 Conflict Equivalent Schedules

Conflict Equivalent Serializability：不改变因果（conflict）关系且保持单个进程的 program order 不变的前提下，把没有因果关系的操作进行排序，得到一个串行化的 schedule。因为没有改变因果关系，所以前后两个 schedule 相同

View Equivalent Serializability：https://zhuanlan.zhihu.com/p/563646516?utm_id=0

Schedules S1 and S2 are view equivalent if:
→ If T1 reads initial value of A in S1, then T1 also reads initial value of A in S2.
→ If T1 reads value of A written by T2 in S1, then T1 also reads value of A written by T2 in S2 .
→ If T1 writes final value of A in S1, then T1 also writes final value of A in S2.

总结：Serial Schedule和部分Non-Serial Schedule是符合Serializability Consistency的。

Strict Serializability Consistency:

* Serializability Consistency
* Linearizability: 读取到最近的写入

如果两个事务有先后关系，例如 T1 结束后 T2 才开始执行，那么用户一定是先看到 T1，再看到 T2

如果两个事务是并行，那么这两个事务可以通过 Conflict/View 做等价串行变化。不管这两个事务是怎样的执行顺序，所有进程都要看到同样的历史

参考资料

* [入门指南](https://zhuanlan.zhihu.com/p/637960746)
* [cmu15-445](https://15445.courses.cs.cmu.edu/fall2021/schedule.html)
* [Concurrency Control Theory](https://zhuanlan.zhihu.com/p/563646516?utm_id=0)
* [分布式系统一致性的发展历史 (五)](https://danielw.cn/history-of-distributed-systems-5)
