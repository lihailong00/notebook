# MySQL基础

[toc]



## MySQL系统框架

1. 连接池。

   ```mysql
   -- 最大连接数
   show VARIABLES LIKE 'max_connections';
   -- 单次最大数据报文
   show VARIABLES LIKE 'max_allowed_packet';
   ```

   

2. SQL解析器：解析SQL。
3. SQL优化器：优化SQL语句。
4. 存储引擎：处理SQL执行计划，实施读写操作。
   1. innoDB：默认。
   2. MyISAM：不常用。
   3. ......



## MySQL数据写入原理

![截屏2023-08-06 22.04.17](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/%E6%88%AA%E5%B1%8F2023-08-06%2022.04.17.png)



1. 记录Undo Log。

2. 将修改的数据暂时写入内存的`Buffer Pool`中。

3. 为防止`Buffer Pool`中的数据异常丢失，特意用`Redo Log`记录变更数据。而`Redo Log Buffer`是`Redo Log`临时写入内存的缓存。

   `Redo Log`刷盘策略：`innodb_flush_log_at_trx_commit`。

   1. 值为1（默认）：每次提交事务前，将信息写入`Redo Log Buffer`，同时将信息添加到操作系统的内存中，并立即刷盘。
   2. 值为0：每次提交事务前，将信息写入`Redo Log Buffer`，每隔1s，放入系统内存并刷盘一次。
   3. 值为2：每次提交事务前，将信息写入`Redo Log Buffer`，同时将信息添加到操作系统的内存中，每隔1s，刷盘一次。

4. 刷盘操作。

5. `Redo Log`写入的同时，进行`Binlog`的刷盘。`Binlog`的作用是历史数据查询、数据库备份和恢复、主从复制等功能。



## MySQL存储结构

`InnoDB`引擎下，新建表时会`*.ibd`文件，用于记录表结构、数据、索引等。具体结构如下：

![截屏2023-08-06 22.39.19](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/%E6%88%AA%E5%B1%8F2023-08-06%2022.39.19.png)

其中页是基本单位。



## SQL执行原理

![截屏2023-08-07 00.01.23](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/%E6%88%AA%E5%B1%8F2023-08-07%2000.01.23.png)



1. 连接驱动必须一次性将SQL语句发送给连接池。可设置`max_allowed_packet`传输数据包大小上限。

2. 查询缓存在MySQL8.0中被废弃。废弃的原因如下：

   1. 缓存命中率低。
   2. 建立新的无用的缓存。
   3. 数据与缓存同步需要一定代价。

3. SQL解释器通过词法分析、语法分析SQL语句。

4. 预处理器会拆分请求，先提交SQL模版语句，再提交参数并执行。多次重复的语句只需提交一次模版，后续修改参数即可。

5. SQL优化器用于优化SQL，常用指令有：

   ```mysql
   -- 打开优化成本分析轨迹
   SHOW VARIABLES LIKE 'optimizer_trace';
   SET query_cache_type = 'enabled=on';
   -- 查阅后一定要记得关闭
   SET query_cache_type = 'enabled=off';
   ```

   注意：无法计算子查询和包含UNION的查询。

6. 优化后的SQL传输给SQL执行器。

7. 优化器执行时，每一张表都有一个handler实例。

8. 通过TCP协议将数据返回给MySQL客户端。