# MySQL——锁（TODO）

[toc]



## 分类

按照锁的性质分类：共享锁和独占锁。

按照锁的范围分类：全局锁、表级锁、行级锁。





## 共享锁

前提：下述不同事务获取的锁的范围一致。

1. 多个事务可以同时获取共享锁，用于读取数据。
2. 存在事务获取共享锁时，其他事务不能获取独占锁。



## 独占锁

前提：下述不同事务获取的锁的范围一致。

一个事务获取独占锁时，其他事务不能获取共享锁和排他锁。



共享锁和独占锁之间的关系：

| 当前请求\请求锁类型 | S（共享锁） | X（独占锁） |
| ------------------- | ----------- | ----------- |
| S（共享锁）         | 兼容        | 冲突        |
| X（独占锁）         | 冲突        | 冲突        |



## 全局锁

用于全库备份，案例如下：

```sql
# 加全表锁
flush tables with read lock;
# 无法执行修改操作
update user set username = 'lihailong' where id = 1;
# 可以执行查询操作
select * from user;
# 解锁
unlock tables;
# 可以执行查询操作
update user set username = 'lihailong' where id = 1;
```



## 表级锁

**元数据锁**和**意向锁**是比较重要的表锁。



1. 元数据锁（MDL）

   系统自动控制，避免**DML和DDL冲突**。当一张表**增删改查**时，加MDL读锁。当一张表结构发生变更时，加MDL写锁。



2. 意向锁

   - 意向锁的作用是：**协调行锁与表锁**。

   - 当事务A对一行数据加上**独占锁**时，同时会对表加上意向独占锁（IX）；当事务A对一行数据加上共享锁时，同时会对表加上意向共享锁（IS）。

   - IS与表共享锁兼容，与表独占锁互斥。

     ```sql
     # 1
     begin;
     # 1
     select * from user where id = 1 in share mode;
     # 2
     lock tables user read;  # 正常执行，IS与表共享锁兼容
     # 2
     lock tables user write;  # 阻塞，IS与表独占锁互斥
     ```

   - IX与表共享锁和表独占锁都互斥。

     ```sql
     # 1
     begin;
     # 1
     select * from user for update;
     # 2
     lock tables user read;  # 阻塞，IX与表共享锁互斥
     # 2
     lock tables user write;  # 阻塞，IX与表排他锁互斥
     ```





## 行级锁

1. 概览

   - 主要应用于InnoDB存储引擎。MyISAM不支持行级锁。

   - 常见的行级锁有：行锁、间隙锁、临键锁。

   - 不同SQL语句对应的锁类型：

     | SQL                           | 锁类型 | 说明                                    |
     | ----------------------------- | ------ | --------------------------------------- |
     | INSERT/UPDATE/DELETE          | 独占锁 | 自动加锁                                |
     | SELECT                        | 不加锁 |                                         |
     | SELECT ... LOCK IN SHARE MODE | 共享锁 | 手动在SELCT语句后添加LOCK IN SHARE MODE |
     | SELECT ... FOR UPDATE         | 独占锁 | 手动在SELCT语句后添加FOR UPDATE         |

   - 间隙锁和临键锁主要解决了**幻读**问题。

   

2. 行锁：锁定一个行记录。适用于RC隔离级别。



3. 间隙锁

   - 间隙锁由系统创建，锁定查询范围中的**间隙**（不是锁行记录）。适用于**RR及以上**的隔离级别。

   - 间隙锁的作用是**防止**其他事务向该间隙**插入行记录**。但是其他事务仍然可以给该间隙加锁。**间隙锁与间隙锁可以叠加**。

   - 假定有如下数据：

     | id（unique index） |
     | ------------------ |
     | 1                  |
     | 5                  |
     | 10                 |

     ```sql
     # 1
     begin;
     # 2
     begin;
     # 1
     select * from user where id = 3 for update;  # 优化为间隙锁，锁id=15～id=30之间的间隙
     # 2
     select * from user where id = 3 for update;  # 正常执行，间隙锁与间隙锁不会冲突
     # 2
     insert into user values(2);  # 阻塞，锁id=5左边的一个间隙
     # 2
     insert into user values(12);  # 正常执行
     ```

     



> 临键锁

系统创建，锁定查询范围中的**索引行**和**间隙**。适用于RR隔离级别。



总结：

RR级别下，如果查询走唯一（或主键）索引，那么扫描范围内的节点和区间都要加锁。

RR级别下，如果查询走普通索引，扫描范围为`[a, b]`，那么加锁范围拓展到`[A, B]`，其中`A < a && b < B && A是a左侧最接近的节点 && B是b右侧最接近的节点`。

如下图：

![演示](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/5e336859958535ed43bf51424af9e4b9.png?q-sign-algorithm=sha1&q-ak=AKIDxfG3y7YOzOxW_NparZXiHsiCtmCup7yA_JMbf_P2-_DklCTYzFsHtGuarOheQBqn&q-sign-time=1692781122;1692784722&q-key-time=1692781122;1692784722&q-header-list=host&q-url-param-list=ci-process&q-signature=e7b4e058ac8b8ba52397bc92d1db42fb605dc17a&x-cos-security-token=aDxBxBM7gMq4ROD9P9a8g3JhTa9I3pCa4b1592cded01420b3154f7069442eff2H2yzq2slN464vHeZdxHeM5jvi29bmJTuYCdaIaO-hDlJlzN45glGLoFP_bh3RUReiCHT-3tlL2YbsrsWEL4wyPfbdIR5GYkpNZ2r-TuQJAInoLP-511llwTMEyCGVWqHn_yed2eSUJ2gLsaMHjC9BPjG8uxZSclDhF0Lc4ZKf7Lpb6lFlt53bH6iHGKfU_XUBdL3tCdVOMpdIorHW80ScQ&ci-process=originImage)







## 重点

1. 如果**操作不走索引**，那么**行锁会升级为表锁**！

```sql
# 1
begin;
# 2
begin;
# 1
select * from user where age = 20 for update;  # 假定age属性没有索引
# 2
update user set age = 40 where age = 30;  # 阻塞，因为锁了整张表
```





## 实战案例

分别分析以下案例的执行过程。

1. 假定有事务1和事务2和以下数据。

   user表：

   | id   | username   |
   | ---- | ---------- |
   | 1    | Longcoding |
   | 2    | Xiaoming   |

   

```sql
# 1
begin;
# 2
begin;
# 1
update user set username = 'lc' where id = 1;
# 2
select username from user where id = 1;  # 显示Longcoding而不是lc
```



2. 假定有事务1和事务2和以下数据

user表：

| id   | username   |
| ---- | ---------- |
| 1    | Longcoding |
| 2    | Xiaoming   |

```sql
# 1
begin;
# 1
select * from user;
# 2
alter table user add column score int;  # 事务2会被阻塞
# 1
commit;  # 释放元数据锁，事务2立即执行
```

同样的场景：

```sql
# 1
begin;
# 2
alter table user add column score int;  # 事务2正常执行
# 因为事务1没执行dml语句，因此不会给表加上元数据锁
```





select object_schema, object_name, index_name, lock_type, lock_mode, lock_data from performance_schema.data_locks;
