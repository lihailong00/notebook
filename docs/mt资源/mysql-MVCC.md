# 1.事务 & 隔离级别

事务定义：逻辑上的一组操作，要么全部执行成功，要么都不执行

**特性**

A atomicity 原子性：事务是最小的执行单位，要么全部执行成功，要么都不执行

C consistency 一致性：事务执行前后，状态正确，数据保持一致（例如转账前后的总金额是一致的）

I Isolation 隔离性：事务执行中间状态别的事务是不可见的

D durability 持久性：提交后，数据库的数据是持久的，即使数据库发生故障也不应该对其有任何影响

![img](https://km.sankuai.com/api/file/cdn/433043522/434757583?contentType=1&isNewContent=false&isNewContent=false)



**隔离级别和出现的问题**

MySQL默认RR隔离级别（但是通过next-keyLock其实也能避免幻读）

|              | 脏读                                                         | 不可重复读                                                   | 幻读                                                         |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| RU           | √                                                            | √                                                            | √                                                            |
| RC           | ×                                                            | √                                                            | √                                                            |
| RR           | ×                                                            | ×                                                            | √                                                            |
| SERIALIZABLE | ×                                                            | ×                                                            | ×                                                            |
|              | ![img](https://km.sankuai.com/api/file/cdn/433043522/434735682?contentType=1&isNewContent=false&isNewContent=false) | ![img](https://km.sankuai.com/api/file/cdn/433043522/434747747?contentType=1&isNewContent=false&isNewContent=false) | ![img](https://km.sankuai.com/api/file/cdn/433043522/434662255?contentType=1&isNewContent=false&isNewContent=false) |





# 2.常见锁分类

## 1）锁类型（粒度）

行锁和表锁

表锁：力度最大，整张表加锁，**资源消耗少**，**并发度低**（一般是DDL处理时使用），服务器实现

行锁（基于索引实现的，索引命中数据），开销大，**锁力度小，并发度高，**存储引擎实现：

record locks：单行记录上锁（二级索引+聚簇索引）

gap locks：间隙锁，锁定一个范围 不包括record

next-key locks：record+gap锁定一个范围，包含记录（]

Insert Intention Locks：插入意向锁，由 **INSERT** 操作产生的一种**间隙锁（插入意向锁+行锁，可并发）**

| 要获取/已获取    | record | gap    | next-key | Insert Intention |
| :--------------- | :----- | :----- | :------- | :--------------- |
| record           | 不兼容 | 兼容   | 不兼容   | 兼容             |
| gap              | 兼容   | 兼容   | 兼容     | 兼容             |
| next-key         | 不兼容 | 兼容   | 不兼容   | 兼容             |
| Insert Intention | 兼容   | 不兼容 | 不兼容   | 兼容             |

**行表示已有的锁，第一列表示要加的锁**。

- 插入意向锁不影响其他事务加其他任何锁。也就是说，一个事务已经获取了插入意向锁，对其他事务是没有任何影响的；
- 插入意向锁与间隙锁和 Next-key 锁冲突。也就是说，一个事务想要获取插入意向锁，如果有其他事务已经加了间隙锁或 Next-key 锁，则会阻塞。

- 间隙锁不和其他锁（不包括插入意向锁）冲突；
- 记录锁和记录锁冲突，Next-key 锁和 Next-key 锁冲突，记录锁和 Next-key 锁冲突；



## 2）锁模式（读/写）

读锁也就共享锁，S锁

写锁也叫排它锁，X锁

读意向锁，IS

写意向锁，IX

AUTO_INC，自增锁：当表的主键为自增列时，多个并发事务通过自增锁来保证自增主键不会出现重复等现象。



为什么要有意向锁？

提升效率，A行锁，B要进行表锁，一行一行效率太低。**意向锁是表锁**，在锁表之前，需要先获取IX或者IS

|      | IS     | S      | IX     | X      |
| :--- | :----- | :----- | :----- | :----- |
| IS   | 兼容   | 兼容   | 兼容   | 不兼容 |
| S    | 兼容   | 兼容   | 不兼容 | 不兼容 |
| IX   | 兼容   | 不兼容 | 兼容   | 不兼容 |
| X    | 不兼容 | 不兼容 | 不兼容 | 不兼容 |

注：意向锁和意向锁之间不会冲突



# 3.当前读 & 快照读

快照读，读取的是记录的可见版本 (有可能是历史版本)，不用加锁

当前读，读取的是记录的最新版本，并且，当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录。



**快照读，简单的select操作（MVCC+undo）**

select * from table where ?

**当前读，特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁（next-key lock）**

select * from table where? lock **in share mode**		S锁

select * from table where? **for update**		X锁

update    X锁

delete    X锁

insert    X锁



# 4.MVCC

- multi-version concurrency control多版本并发控制 
- Lock based concurrency control基于锁的并发控制



MySQL InnoDB实现隔离级别的一种具体方式，用于实现RC和RR。RC总是读取最新的数据行，要求很低，无需使用 MVCC。可串行化隔离级别需要对所有读取的行都加锁，单纯使用 MVCC 无法实现。



## 4.1 undo & redo

### 4.1.1.redo log

在InnoDB存储引擎中，事务日志通过重做日志文件（redo log）和日志缓冲（InnoDB log buffer）实现。

预写日志WAL（write ahead logging），先写日志再写数据：当事务执行时，会往InnoDB的日志缓冲中插入事务日志；当事务提交时，必须先将日志缓冲写入磁盘。

### 4.1.2.undo log

重做日志记录了事务的行为，可以方便的通过其进行“重做”。但事务有时还需要撤销，这时就需要undo。undo与redo正好相反**，undo是先写数据后记录undo log**；事务对数据库进行修改时，会产生redo和undo，当有错误或rollback时，可以根据undo log将数据恢复到修改前的样子

undo并非对数据的物理恢复，而是**逻辑恢复**，例如插入了10万条数据，使表空间增加，undo后表空间并不会收缩；对于每条insert，undo是做相应的delete，对于update则是做了相反的update。

### 4.1.3.redo与undo

假设有A、B两个数据，值分别为1、2，开启事务分别对其进行修改A → 3，B → 4，在提交，过程如下：

- 事务开始
- 记录A=3到redo log
- 修改A=3
- 记录A=1到undo log
- 记录B=4到redo log
- 修改B=4
- 记录B=2到undo log
- 将redo log写入磁盘
- 事务提交



## 4.2. MVCC读场景

| session 1                         | session 2                    |
| :-------------------------------- | :--------------------------- |
| select a from test; return a = 10 |                              |
| start transaction;                |                              |
| update test set a = 20;           |                              |
|                                   | start transaction;           |
|                                   | select a from test; return ? |
| commit;                           |                              |
|                                   | select a from test; return ? |



session 1修改了一条记录，没有提交；与此同时，session 2 来查询这条记录，这时候返回记录应该是多少呢？

session 1 提交之后 session 2 查询出来的又应该是多少呢？



不同隔离级别情况不同，情况如下:

- RU：都是修改后的结果，即 a = 20
- RC：commit 前，a =10 , commit后 a = 20
- RR/SERIALIZABLE，都是修改前的结果，即 a = 10



思考一个问题：怎么实现session2是怎么做到查询出来的结果还是10，而不是20呢？



## 4.3 事务对某行记录的更新过程：

### 4.3.1.初始数据行

![img](https://km.sankuai.com/api/file/433043522/437310739)

F1～F6是某行列的名字，1～6是其对应的数据。每一行都有2个隐藏列DATA_TRX_ID和DATA_ROLL_PTR(如果没有定义主键，则还有个隐藏主键列)，假如这条数据是刚INSERT的，可以认为ID为1，其他两个字段为空。

1. DATA_TRX_ID占6 字节，表示这一行数据最后插入或修改的事务id。
2. DATA_ROLL_PTR则表示指向该行回滚段的指针，该行上所有旧的版本，在undo中都通过链表的形式组织，而该值，正式指向undo中该行的历史记录链表
3. DB_ROW_ID 行ID占7字节，像自增主键一样随着插入新数据自增。如果表中不存主键 或者 唯一索引，那么数据库 就会采用DB_ROW_ID生成聚簇索引。否则DB_ROW_ID不会出现在索引中。

4.3.2.事务1更改该行的各字段的值

![img](https://km.sankuai.com/api/file/433043522/437350854)

当事务1更改该行的值时，会进行如下操作：

- 用排他锁锁定该行
- 记录redo log
- 修改当前行的值，填写事务编号，使回滚指针指向undo log中的修改前的行

- 把该行修改前的值Copy到undo log，即上图中下面的行

### 4.3.3.事务2修改该行的值

![img](https://km.sankuai.com/api/file/433043522/437407070)

与事务1相同，此时undo log，中有有两行记录，并且通过回滚指针连在一起。

因此，如果undo log一直不删除，则会通过当前记录的回滚指针回溯到该行创建时的初始内容；在Innodb中存在purge线程，它会查询那些比现在最老的活动事务还早的undo log，并删除它们，从而保证undo log文件不至于无限增长。





## 4.4InnoDB MVCC实现原理

每行数据隐藏三个列：DB_TRX_ID（最近事务号）DB_ROLL_PTR（回滚指针）DB_ROW_ID（行ID），回滚指针指向undo



通过read view判断行记录是否可见，具体的判断流程如下:

- RR隔离级别下，**在每个事务开始的时候，会将当前系统中的所有的活跃事务拷贝到一个列表中(read view)**
- RC隔离级别下，**在事务中的每个语句开始时，**会将当前系统中的所有的活跃事务拷贝到一个列表中(read view) 
- 在read view中最早的事务ID为**tmin**，最晚的事务ID为**tmax**，selectNode对应的事务ID为**tcreate**
- 当读到一行时，该行上的DB_TRX_ID为tid0【undo指针上指的】，是否可见的逻辑判断如下图

![img](https://km.sankuai.com/api/file/cdn/433043522/434139385?contentType=1&isNewContent=false&isNewContent=false)

举例：回看下上面的例子

![img](https://km.sankuai.com/api/file/433043522/437310771)

| session 1                         | session 2                    | read view                                                    |                                                              |
| :-------------------------------- | :--------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| **RC**                            | **RR**                       |                                                              |                                                              |
| select a from test; return a = 10 |                              |                                                              |                                                              |
| start transaction;                |                              |                                                              |                                                              |
| update test set a = 20;           |                              |                                                              |                                                              |
|                                   | start transaction;           | 1, 2                                                         | 1, 2                                                         |
|                                   | select a from test; return ? | 1, 2tid0(当前a=20 DB_TRX_ID=1)> tmin(1) no->通过回滚指针向上找tid0 = NULL (tmin=1,tmax=2,tcreate=2 不在readview）故 return 10 | 1, 2tid0(当前a=20 DB_TRX_ID=1)> tmin(1) no->通过回滚指针向上找tid0 = NULL (tmin=1,tmax=2,tcreate=2 不在readview）故 return 10 |
| commit;                           |                              | 1, 2                                                         | 1, 2                                                         |
|                                   | select a from test; return ? | 2tid0 （当前a = 20的版本 记录的DB_TRX_ID = 1）< tmin（2）故可见 return 20 | 1, 2tid0(当前a=20 DB_TRX_ID=1)> tmin(1) no->通过回滚指针向上找tid0 = NULL (tmin=1,tmax=2,tcreate=2 不在readview）故 return 10 |



## 4.5.MVCC解决了什么问题

- MVCC使得数据库读不会对数据加锁，普通的SELECT请求不会加锁，提高了数据库的并发处理能力；
- 借助MVCC，数据库可以实现RC，RR等隔离级别，用户可以查看当前数据的前一个或者前几个历史版本。保证了ACID中的I特性（隔离性）。



# 5.加锁分析

SQL1：select * from t1 where id = 10;

**SQL2：delete from t1 where id = 10;**



**分别加什么锁？**



前提一：id列是不是主键？

前提二：当前系统的隔离级别是什么？

前提三：id列是唯一索引吗？

前提四：id列上有索引吗？



## 5.1RC隔离级别

### 5.1.1主键+RC

X锁，聚簇索引的id=10

![img](https://km.sankuai.com/api/file/cdn/433043522/437402008?contentType=1&isNewContent=false&isNewContent=false)



### 5.1.2 唯一索引+RC

id列是唯一索引，那么SQL需要加两个X锁，一个对应于id unique索引上的id = 10的记录，另一把锁对应于聚簇索引上的[name=’d’,id=10]的记录。

![img](https://km.sankuai.com/api/file/cdn/433043522/433662024?contentType=1&isNewContent=false&isNewContent=false)



为什么聚簇索引要上锁？

防止主键并发更新，违背了同一记录上的更新/删除需要串行执行的约束



### 5.1.3.非唯一索引 +RC

那么对应的所有满足SQL查询条件的记录，都会被加锁。同时，这些记录在主键索引上的记录，也会被加锁。

![img](https://km.sankuai.com/api/file/cdn/433043522/433662025?contentType=1&isNewContent=false&isNewContent=false)



### 5.1.4.无索引 +RC

若id列上没有索引，SQL会走聚簇索引的全扫描进行过滤，由于过滤是由MySQL Server层面进行的。因此每条记录，无论是否满足条件，都会被加上X锁。但是，为了效率考量，MySQL做了优化，对于不满足条件的记录，会在判断后放锁，最终持有的，是满足条件的记录上的锁，但是不满足条件的记录上的加锁/放锁动作不会省略。同时，优化也违背了2PL的约束。

其实就是全表扫，然后扫到的记录加X锁。

![img](https://km.sankuai.com/api/file/cdn/433043522/433662026?contentType=1&isNewContent=false&isNewContent=false)



## 5.2 RR隔离级别

**delete from t1 where id = 10;**



### 5.2.1主键+RR

同RC，聚簇索引的id=10



### 5.2.2唯一索引+RR

同RC，两把锁，unique索引上的id = 10的记录，另一把锁对应于聚簇索引上的[name=’d’,id=10]的记录。



### 5.2.3.非唯一索引 +RR

索引gap+行锁(next-key)，聚簇索引行锁，**防止幻读**，连续两次当前读，结果一样(例如：select * from t1 where id = 10 for update;)

RR隔离级别下，id列上有一个非唯一索引，对应SQL：delete from t1 where id = 10; 

首先，通过id索引定位到第一条满足查询条件的记录，加记录上的X锁，**加GAP上的GAP锁**，然后加**主键聚簇索引上的记录X锁**，然后返回；然后读取下一条，重复进行。直至进行到第一条不满足条件的记录[11,f]，此时，不需要加记录X锁，但是仍旧需要加GAP锁，最后返回结束。

![img](https://km.sankuai.com/api/file/cdn/433043522/434129621?contentType=1&isNewContent=false&isNewContent=false)



### 5.2.4.无索引 +RR

全表扫，GAP+行锁

![img](https://km.sankuai.com/api/file/cdn/433043522/433662027?contentType=1&isNewContent=false&isNewContent=false)



### **5.3. Serializable**

SQL2：delete from t1 where id = 10; Serializable隔离级别与Repeatable Read隔离级别完全一致。

SQL1：select * from t1 where id = 10; 在RC，RR隔离级别下，都是快照读，不加锁。Serializable SQL1会加读锁，也就是说快照读不复存在，MVCC并发控制降级为Lock-Based CC。



**结论：**在MySQL/InnoDB中，所谓的读不加锁，并不适用于所有的情况，而是隔离级别相关的。Serializable隔离级别，读不加锁就不再成立，所有的读操作，都是当前读。





# 6.总结 

InnoDB 不同隔离级别实现方式

快照读通过MVCC+undoLog 当前读通过加next-key lock实现

针对当前读，RC隔离级别对读取到的记录加锁（记录锁），存在幻读

针对当前读，RR隔离级别对读取到的记录加锁（记录锁），同时范围加锁，不存在幻读

序列化，MVCC并发控制退化成基于锁的并发控制。不区分当前读和快照读，所有读都是当前读，读加S，写加X





资料

https://www.aneasystone.com/archives/2017/11/solving-dead-locks-two.html

[1. RDS关于MySQL锁机制知识普及](https://km.sankuai.com/page/231603432)

[MySQL MVCC原理](https://km.sankuai.com/page/228145202)
