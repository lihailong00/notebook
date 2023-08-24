# MySQL之MVCC

[toc]



## 概览

MVCC：Multi-Version Concurrency Control（多版本并发控制），用于实现**数据库事务并发控制**的一种策略，用于处理多个事务**同时对数据库进行读写操作**时的冲突和隔离问题。

MVCC的实现思路是维护数据的版本链，读快照、写实时，从而解决读写冲突。技术难点是快照读时，应该读取版本链上的哪一版数据。

MVCC实现需要数据库中的3个**隐式字段**，**undo log日志**、**readView**。



当前读：读取最新的版本。select ... for update, select ... lock in share mode, update, insert, delete

快照读：



## MVCC实现原理

3个隐式字段：

DB_TRX_ID：最近修改这行记录的事务ID。

DB_ROLL_PTR：回滚指针，在Undo Log版本链中，指向上一个版本的数据。

DB_ROW_ID：隐藏主键，如果表没有主键，将会生成该隐式字段。



undolog：

insert、update、delete的时候，生成undo log，便于回滚。

insert时，undo log日志只在当前事务需要，当前事务提交时，立即删除该记录。

update、delete时，产生的undo log日志不仅在当前事务需要，其他事务快照读时也需要，所以当前事务commit后不会被立即删除。

![undolog版本链](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/%E6%88%AA%E5%B1%8F2023-08-23%2023.35.48.png?q-sign-algorithm=sha1&q-ak=AKIDEcfOnMNBCwZFFWcrhhD8V9VT6iz44j4q2f0GxAvUhwmcghNm-CrRvLxcPtmtrb4A&q-sign-time=1692805070;1692808670&q-key-time=1692805070;1692808670&q-header-list=host&q-url-param-list=ci-process&q-signature=9b6afdb1b41438ead53ca96511338eb6580f3758&x-cos-security-token=HVhosrkse2Yh1D7G8zElywoIt4mvcfWad6c84a171b9f1f288cc3321bcee907efYsm_oTGk30XoqLERnLyaYkUTMYSMdqh6gZYPUcy6Sv0KUbF7df6-RNdxK_Nmm4ZU0-TYdtCzVOTF5veJZ_Lv1hYHn_W0mAGNsI7kBn3bScfRYGbEtXuKtlku_3CJurOWimv2C5JgOIe3lPs6dr8C8tBP7IryPAcUxvlg7d5tKYTMT7C_GW8CNeV-FEsViJVA&ci-process=originImage)
