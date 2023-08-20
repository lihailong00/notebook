# MySQL几种log

[toc]



## BINLOG

### 是什么

用于记录数据库的**增、删、改**操作，并以二进制的形式存放在硬盘中。任何存储引擎都会记录binlog日志。

### binlog应用场景

1. 主从复制：主数据库记录 binlog，并将其发送给从数据库，从数据库根据 binlog 中的操作进行数据同步，使得从数据库的数据与主数据库保持一致。
2. 数据恢复。



### binlog刷盘

binlog刷盘是指将二进制日志数据从内存缓冲区写入磁盘的过程。

`sync_binlog`控制了 binlog 数据写入磁盘的频率。

- `0`：表示 binlog 在每次提交事务时，都只写入到操作系统缓冲区，由操作系统决定刷新到磁盘的时机（异步刷盘）。
- `1`：表示 binlog 在每次提交事务时，都将数据直接刷盘到磁盘，以确保数据持久化存储（同步刷盘）。
- `N`：其中 N 是一个大于 1 的值，表示 binlog 在每 N 个提交事务时，才将数据刷盘到磁盘。这可以在一定程度上平衡性能和数据安全性。

`1`的效果最好，但是性能较差。



### binlog日志的三种格式

> STATMENT

记录内容：SQL语句。

优点：格式简单高效。

缺点：使用随机函数、时间函数等操作会导致主从数据库不一致。



> ROW

记录内容：变化的行信息。

优点：数据复制的准确性。

缺点：产生更多的binlog数据。尤其是ALTER table操作。



> MIXED

记录内容：SQL语句或变化的行信息。

优点：集大成者。

缺点：无。



## REDOLOG

### redo log应用场景

Redo Log主要应用于**事务**。

1. 事务的持久性：Redo Log记录事务的更改操作。当数据库发生故障时，可以通过Redo Log重新执行事务的修改。
2. 减少



Redo Log包括两部分：内存中的`redo log buffer`和硬盘中的`redo log file`。

在计算机操作系统中，用户空间(user space)下的缓冲区数据一般情况下是无法直接写入磁盘的，中间必须经过操作系统内核空间(kernel space)缓冲区(OS Buffer)。因此，redo log buffer写入redo log file实际上是先写入OS Buffer，然后再通过系统调用fsync()将其刷到redo log file中。



mysql支持三种将redo log buffer写入redo log file的时机，可以通过innodb_flush_log_at_trx_commit参数配置，各参数值含义如下：

![img](https://pic4.zhimg.com/80/v2-213622cb332c35e77eea6667a471d8ef_1440w.webp)

![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/v2-41365bae03d3607ea95ff74246356a99_1440w.webp)





## Undo Log

Undo是指**回滚**，而不是没有做。

数据库事务四大特性中有一个是原子性，具体来说就是 原子性是指对数据库的一系列操作，要么全部成功，要么全部失败，不可能出现部分成功的情况。

实际上，原子性底层就是通过undo log实现的。undo log主要记录了数据的逻辑变化，比如一条INSERT语句，对应一条DELETE的undo log，对于每个UPDATE语句，对应一条相反的UPDATE的undo log，这样在**发生错误时，就能回滚到事务之前的数据状态**。
