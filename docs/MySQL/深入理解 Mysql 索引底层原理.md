# 深入理解 Mysql 索引底层原理



[参考文章](https://zhuanlan.zhihu.com/p/113917726)

提问：

1. 索引是什么？

   答：索引相当于书本的目录，方便我们快速查找数据。深入点讲，索引是**数据结构**。

2. 哈希表做索引的好处是什么？mysql为什么不用哈希表做索引？

   答：好处是可以用O(1)的时间复杂度查询数据。MySQL不用哈希表做索引是因为不能做到高效范围查找。

3. AVL树做索引的好处是什么？mysql为什么不用哈希表做索引？

   答：好处是用O(NlogN)的时间复杂度查询数据，并支持范围查找、数据排序。但是还是有优化的空间。这里需要了解一些关于磁盘IO的知识。

   ​	关于磁盘IO：

   ​	a. 数据库查询数据的瓶颈是磁盘IO，如果使用的是 AVL 树，我们每一个树节点只存储了一个数据，我们一次磁盘 IO 只能取出来一个节点上的数据加载到内存里，那比如查询 id=7 这个数据我们就要进行磁盘 IO 三次，这是多么消耗时间的。所以我们设计数据库索引时需要首先考虑怎么尽可能减少磁盘 IO 的次数。

   ​	b. 磁盘 IO 有个有个特点，就是从磁盘读取 1B 数据和 1KB 数据所消耗的时间是基本一样的，我们就可以根据这个思路，我们可以在一个树节点上尽可能多地存储数据，一次磁盘 IO 就多加载点数据到内存，这就是 B 树，B+树的的设计原理了。

   

4. B-树做索引的好处是什么？mysql为什么不用哈希表做索引？

   答：好处是用O(logH)的时间复杂度查询数据(H < N)。但是还有优化空间：每个节点不存实际数据，只存索引，这样每个节点就能存更多数据，树的层数进一步降低。所有数据存放在叶子节点。

5. mysql为什么用B+树做索引？

   答：见第4点。

6. 聚集索引和非聚集索引是什么？

   答：把数据和索引分开，这叫非聚集索引方式；把数据和索引合在一起，这叫聚集索引方式。

7. innodb和myisam引擎的区别？（围绕非聚集索引方式和聚集索引方式）

   答：

    1. 上层的区别：myisam查询性能更好，但不支持事务。

    2. 底层原理：建表时 InnoDB 就会自动建立好主键 ID 索引树，这也是为什么 Mysql 在建表时要求必须指定主键的原因(如果没有指明，数据库会自动添加一个看不见的主键)。当我们为表里的一个字段user_name加索引时InnoDB 就会建立 user_name 索引 B+树，节点里存的是 user_name 这个 KEY，叶子节点存储的数据的是主键 KEY。注意，**叶子存储的是主键 KEY！**拿到主键 KEY 后，InnoDB 才会去主键索引树里根据刚在 user_name 索引树找到的主键 KEY 查找到对应的数据。

       问题来了，为什么 InnoDB 只在主键索引树的叶子节点存储了具体数据，但是其他索引树却不存具体数据呢，而要多此一举先找到主键，再在主键索引树找到对应的数据呢?

       其实很简单，因为 InnoDB 需要节省存储空间。一个表里可能有很多个索引，InnoDB 都会给每个加了索引的字段生成索引树，如果每个字段的索引树都存储了具体数据，那么这个表的索引数据文件就变得非常巨大（数据极度冗余了）。从节约磁盘空间的角度来说，真的没有必要每个字段索引树都存具体数据，通过这种看似“多此一举”的步骤，在牺牲较少查询的性能下节省了巨大的磁盘空间，这是非常有值得的。

   