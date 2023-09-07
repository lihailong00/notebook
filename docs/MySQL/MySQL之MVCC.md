# MySQL之MVCC（TODO）

[toc]



## 概览

MVCC：Multi-Version Concurrency Control（多版本并发控制），这里特指InnoDB引擎在**RC**或**RR**隔离级别下多个事务**读写并发**时的冲突和隔离问题。

MVCC的核心思想是维护数据的**UndoLog版本链**，并采用**读快照**、**写实时**的策略解决读写冲突，同时基于**ReadView**机制决定快照读的版本。

MVCC实现需要数据库中的**行记录的3个隐式字段**，**undo log日志**、**readView**。



当前读：读取最新的版本。对应SQL：`select ... for update`,  `select ... lock in share mode`,  `update`,  `insert`,  `delete`。

快照读：读取历史版本。对应SQL：`select ...`。



## UndoLog版本链

行记录的3个隐式字段：

- `row_id`：数据表没定义主键时，自动生成`row_id`作为主键。

- `trx_id`：记录了新增/最近修改这条行记录的事务ID，事务ID自增。

- `roll_pointer`：指向上一个版本的数据，如下图：

  当事务100（trx_id=100）执行了 `insert into t_user values(1,'张三',20); `之后： ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c50b48b021ca4b3c8930b82ea698e97a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

  当事务102（trx_id=102）执行了 `update t_user set name='李四' where id=1; `之后： ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dab3839ba4ba4edc82984922ad224995~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

  当事务103（trx_id=103）执行了 `update t_user set name='王五' where id=1; `之后：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d1470d871c541c08fc706e172e39da2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)



## ReadView

ReadView机制中，有4个重要的变量：

- `m_ids`：活跃事务（未提交事务）ID。
- `min_trx_id`：`m_ids`中最小事务ID。
- `max_trx_id`：下一个要分配的事务ID（不是`m_ids`中最大的事务id + 1）。
- `creator_trx_id`：生成ReadView的事务ID。

RC隔离级别下，每一次快照读都会生成一个最新的ReadView。

RR隔离级别下，只有第一次快照读才会生成一个最新的ReadView。

详细算法暂未总结。





