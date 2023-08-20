# Java常用集合

[toc]



## List

> ArrayList、CopyOnWriteArrayList和Vector（被淘汰了）

1. **关于添加元素。ArrayList是线程不安全的，CopyOnWriteArrayList和Vector是线程安全的。**举一反三，remove函数也是相同的道理。

   ArrayList的源码

   ```java
   /**
    * Appends the specified element to the end of this list.
    *
    * @param e element to be appended to this list
    * @return {@code true} (as specified by {@link Collection#add})
    */
   public boolean add(E e) {
       modCount++;
       add(e, elementData, size);
       return true;
   }
   ```

   

   Vector的源码

   ```java
   /**
    * Appends the specified element to the end of this Vector.
    *
    * @param e element to be appended to this Vector
    * @return {@code true} (as specified by {@link Collection#add})
    * @since 1.2
    */
   public synchronized boolean add(E e) {
       modCount++;
       add(e, elementData, elementCount);
       return true;
   }
   ```

   

   CopyOnWriteArrayList源码：

   ```java
   /**
    * Appends the specified element to the end of this list.
    *
    * @param e element to be appended to this list
    * @return {@code true} (as specified by {@link Collection#add})
    */
   public boolean add(E e) {
       synchronized (lock) {
           Object[] es = getArray();  // 获取数组对象
           int len = es.length;  // 获取数组原来的长度
           es = Arrays.copyOf(es, len + 1);  // 将原数组拷贝到新数组，且新数组容量+1
           es[len] = e;  // 将添加的元素加入新数组
           setArray(es);  // 用新数组替换原数组
           return true;
       }
   }
   ```

   





2. 关于数组扩容。`grow()`是关于数组扩容的函数。

   ArrayList

   ```java
   /**
    * Increases the capacity to ensure that it can hold at least the
    * number of elements specified by the minimum capacity argument.
    *
    * @param minCapacity the desired minimum capacity
    * @throws OutOfMemoryError if minCapacity is less than zero
    */
   private Object[] grow(int minCapacity) {
       int oldCapacity = elementData.length;
       if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
           int newCapacity = ArraysSupport.newLength(oldCapacity,
                   minCapacity - oldCapacity, /* 至少需要增长的空间 */
                   oldCapacity >> 1           /* 首选增长空间，增长1.5倍 */);
           return elementData = Arrays.copyOf(elementData, newCapacity);
       } else {
           // 数组为空的情况
           // DEFAULT_CAPACITY 的值为10
           return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
       }
   }
   ```

   可以看到，如果数组容量为0，则扩容时会创建一个大小为10的数组。如果数组中有元素，那么默认为扩容1.5倍，如果没有这么大的空间，则扩容到刚刚够用就行。

   

   Vector

   ```java
   /**
    * Increases the capacity to ensure that it can hold at least the
    * number of elements specified by the minimum capacity argument.
    *
    * @param minCapacity the desired minimum capacity
    * @throws OutOfMemoryError if minCapacity is less than zero
    */
   private Object[] grow(int minCapacity) {
       int oldCapacity = elementData.length;
       int newCapacity = ArraysSupport.newLength(oldCapacity,
               minCapacity - oldCapacity, /* minimum growth */
               capacityIncrement > 0 ? capacityIncrement : oldCapacity
                                          /* 如果初始化Vector时没有指定capacityIncrement，则扩容oldCapacity，也即扩容1倍。如果指定扩容量，则扩容capacityIncrement。 */);
       return elementData = Arrays.copyOf(elementData, newCapacity);
   }
   ```



## Map

> HashMap、ConcurrentHashMap和Hashtable（被淘汰了）

