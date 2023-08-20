# MySQL性能分析工具

[toc]

## 数据准备

直接用，不用管语法。

```sql
# 表一
CREATE TABLE s1 (
id INT AUTO_INCREMENT,
key1 VARCHAR(100),
key2 INT,
key3 VARCHAR(100),
key_part1 VARCHAR(100),
key_part2 VARCHAR(100),
key_part3 VARCHAR(100),
common_field VARCHAR(100),
PRIMARY KEY (id),
INDEX idx_key1 (key1),
UNIQUE INDEX idx_key2 (key2),
INDEX idx_key3 (key3),
INDEX idx_key_part(key_part1, key_part2, key_part3)
) ENGINE=INNODB CHARSET=utf8;

# 表二
CREATE TABLE s2 (
id INT AUTO_INCREMENT,
key1 VARCHAR(100),
key2 INT,
key3 VARCHAR(100),
key_part1 VARCHAR(100),
key_part2 VARCHAR(100),
key_part3 VARCHAR(100),
common_field VARCHAR(100),
PRIMARY KEY (id),
INDEX idx_key1 (key1),
UNIQUE INDEX idx_key2 (key2),
INDEX idx_key3 (key3),
INDEX idx_key_part(key_part1, key_part2, key_part3)
) ENGINE=INNODB CHARSET=utf8;

# 设置参数log_bin_trust_function_creators
set global log_bin_trust_function_creators=1; # 不加global只是当前窗口有效。

# 创建函数
DELIMITER //
CREATE FUNCTION rand_string1(n INT)
RETURNS VARCHAR(255) #该函数会返回一个字符串
BEGIN
DECLARE chars_str VARCHAR(100) DEFAULT
'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
DECLARE return_str VARCHAR(255) DEFAULT '';
DECLARE i INT DEFAULT 0;
WHILE i < n DO
SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
SET i = i + 1;
END WHILE;
RETURN return_str;
END //
DELIMITER ;

# 创建往s1表中插入数据的存储过程
DELIMITER //
CREATE PROCEDURE insert_s1 (IN min_num INT (10),IN max_num INT (10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit = 0;
REPEAT
SET i = i + 1;
INSERT INTO s1 VALUES(
(min_num + i),
rand_string1(6),
(min_num + 30 * i + 5),
rand_string1(6),
rand_string1(10),
rand_string1(5),
rand_string1(10),
rand_string1(10));
UNTIL i = max_num
END REPEAT;
COMMIT;
END //
DELIMITER ;

# 创建往s2表中插入数据的存储过程
DELIMITER //
CREATE PROCEDURE insert_s2 (IN min_num INT (10),IN max_num INT (10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit = 0;
REPEAT
SET i = i + 1;
INSERT INTO s2 VALUES(
(min_num + i),
rand_string1(6),
(min_num + 30 * i + 5),
rand_string1(6),
rand_string1(10),
rand_string1(5),
rand_string1(10),
rand_string1(10));
UNTIL i = max_num
END REPEAT;
COMMIT;
END //
DELIMITER ;

# s1表数据的添加1万条记录：
CALL insert_s1(10001,10000);

# s2表数据的添加1万条记录：
CALL insert_s2(10001,10000);

```



## EXPLAIN各列的作用

### table

不论查询语句有多复杂，里边包含了多少个表 ，到最后也是需要对每个表进行单表访问 的，所以MySQL规定EXPLAIN语句输出的每条记录都对应着某个单表的访问方法，该条记录的table列代表着该表的表名（有时不是真实的表名字，可能是简称）。

案例：

```sql
EXPLAIN SELECT * FROM s1;
# s1是驱动表，s2是被驱动表
EXPLAIN SELECT * FROM s1 INNER JOIN s2;
```



### id

在一个大的查询语句中每个SELECT关键字都对应一个唯一的id，通常出现了几个select关键字就对应几个id。

- id如果相同，可以认为是一组，从上往下顺序执行。
- 在所有组中，id值越大，优先级越高，越先执行。
- **一个id号代表一趟独立的查询，一个SQL的查询趟数越少越好。**

```sql
 EXPLAIN SELECT * FROM s1 WHERE key1 = 'a';
 
 #多表查询
 EXPLAIN SELECT * FROM s1 INNER JOIN s2;
 
 
 EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2) OR key3 = 'a';
 
 # 查询优化器可能对涉及子查询的查询语句进行重写,转变为多表查询的操作
 EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key2 FROM s2 WHERE common_field = 'a');
 
 
 # Union去重-->UNION取并集并去重时产生一个临时表
 EXPLAIN SELECT * FROM s1 UNION SELECT * FROM s2;
 
 
 # UNION ALL不用去重，只有两个id/行
 EXPLAIN SELECT * FROM s1  UNION ALL SELECT * FROM s2;
 
```



### select_type

一条大的查询语句里边可以包含若干个SELECT关键字，每个SELECT关键字代表着一个小的查询语句。只要知道了某个小查询的select_type属性，就知道了这个小查询在整个大查询中扮演了一个什么角色。

个人感觉不太重要，需要的时候继续补充吧。



### type☆

执行计划的一条记录就代表着MySQL对某个表的执行查询时的访问方法，其中的type列就表明了这个访问方法是啥，是较为重要的一个指标。

性能排序：`system` > `const` > `eq_rep` > `ref` > `ref_or_null` > `index_merge` > `unique_subquery` > `index_subquery` > `range` > `index` > `all`。

1. `system`：innodb引擎中没有这种访问方法。
2. 



1. 创建一张表

   ```sql
   CREATE TABLE `test` (
     `id` int(4) NOT NULL AUTO_INCREMENT,
     `a` varchar(10) NOT NULL,
     `b` varchar(10) NOT NULL,
     `c` varchar(10) NOT NULL,
     `d` varchar(10) NOT NULL,
     `e` varchar(10) NOT NULL,
     PRIMARY KEY (`id`),
     UNIQUE KEY `idx_a_b_c` (`a`,`b`,`c`) USING BTREE
   ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
   ```

2. 插入一条数据：

   ```
   id=1 a=2 b=2 c=2 d=2 e=2
   ```

3. 在SQL语句前加上`explain`即可查看执行过程：

   ```sql
   explain select * from test where id = 1;
   ```

   可以看到运行结果为：

   | id   | select_type | table | type  | possible_keys | filtered | Extra       |
   | ---- | ----------- | ----- | ----- | ------------- | -------- | ----------- |
   | 1    | SIMPLE      | test  | const | PRIMARY       | 100.0    | Using where |

   **重点是type类型的值！**



## type的类型







## 其他

explain只会查询SQL语句执行过程，并不会真正执行SQL语句。