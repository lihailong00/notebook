# MySQL之MVCC

[toc]



## 概念

MVCC：Multi-Version Concurrency Control（多版本并发控制）。

用于实现**数据库事务并发控制**的一种策略，用于处理多个事务**同时对数据库进行读写操作**时的冲突和隔离问题。

