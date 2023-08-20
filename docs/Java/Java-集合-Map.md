# Java-集合-Map

[toc]

## HashMap

### 数据结构 :high_brightness:

最重要的一句话：**数组+链表+红黑树**。



HashMap用于存放键值对`<K,V>`数据。我们存入的`<K,V>`键值对被封装成`Node<K,V>`存入`HashMap`的`Node<K,V>`数组（**哈希桶**）中。存入的`<K,V>`数据的`K`会经过`hash`算法得到一个数字作为哈希桶的下标。具体代码如下：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



当出现`hash`冲突时，采用**链地址法**解决哈希冲突。

当哈希冲突较为频繁时，链的长度就会变长，查找效率就会显著降低。此时有两种解决方案：

- 链转为**红黑树**，提高查询效率。
- 哈希桶扩容，降低哈希冲突。



HashMap的结构大致如下：

![HashMap 数据结构草图](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/171348d1ada3138e~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)





## 扩容机制

1. 首先需要知道`HashMap`中有如下参数：

- `int threshold`：`threshold = capacity * loadFactor`，其中`capacity=table.length`。
- `Node<K, V> table`：也即哈希桶。

​	默认情况下，`table`的大小为`16`，`loadFactor`的大小为0.75。



2. 可人为指定`table`的大小：假定传入`x`，则传出`y`，其中`y>=x`且`y`是2的整数次幂。

- `HashMap<Integer, Integer> map = new HashMap<>(8)`：`table.length=8`，`threshold=6`。

- `HashMap<Integer, Integer> map = new HashMap<>(10)`：`table.length=16`，`threshold=12`。

- 补充：传入`10`，得到`16`的函数是这样实现的，非常巧妙！

  ```java
  static final int tableSizeFor(int cap) {
      int n = cap - 1;
      n |= n >>> 1;
      n |= n >>> 2;
      n |= n >>> 4;
      n |= n >>> 8;
      n |= n >>> 16;
      return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
  }
  ```

  



3. 当`hashmap`对象中的元素个数 ≥ `threshold`时，触发扩容机制。每次扩充2倍。

   易错：假定`Node<K,V> table`的大小为16，`threshold`的大小为`12`，当我们插入12个元素时，即便这12个元素经过`hash`运算后都得到相同的下标，此时`Node<K,V> table`也要扩容2倍。

   

4. 扩容过程中，会将`hashmap`中的所有对象都重新做一次`hash`运算，然后放入新的下标处。

   举个例子：`Node<K,V>[] table`长度为16时，`key`为18的`Node<K,V>`放在下标2处；`Node<K,V>[] table`长度为32时，`key`为18的`Node<K,V>`放在下标18处。

   









### 有序性

`HashMap`不保证顺序性，但是循环遍历时，输出的顺序不会变化。



### 并发问题

`HashMap`是线程不安全的，多线程编程下建议使用`ConcurrentHashMap`。



## ConcurrentHashMap

`ConcurrentHashMap`与`HashMap`的实现几乎一样。

`ConcurrentHashMap`是线程安全的，它通过对哈希桶中的**一个元素**加锁（**CAS+synchronized**）保证并发安全，同时提高并发量。



## TreeMap

采用红黑树存储数据，可以看做是`HashMap`的简化版。

`TreeMap`中的元素是有序的。
