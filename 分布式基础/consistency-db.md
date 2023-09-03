# 数据库一致性

## Serializability Consistency

如果一串事务的执行结果跟这些事务按某种顺序一个个执行相同，我们就说它符合 Serializability

因为串行执行是实现 Serializability 的最简单办法，所以很多人误以为 Serializability 就是串行执行。例如两个事务 T1 和 T2，如果它们没有操作相同的数据，就可以并发执行。但从客户端看来，T1 和 T2 就像是串行执行
