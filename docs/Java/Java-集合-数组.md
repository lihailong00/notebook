# Java-集合-数组

[toc]



## 概览

所有源码基于**JDK1.8**。

数组是一种非常简单的数据结构。我们主要讨论三个数组类：ArrayList，CopyOnWriteArrayList和Vector。讨论过程中，需要注意以下几点：

1. 扩容方式：当数组装满后，不能直接在尾部添加元素（想想为什么），而是需要重新找一片新的内存，然后将原先的数组拷贝到新的位置。
2. 并发安全：当多个线程同时操作一个数组对象时，如何保证线程安全问题。
3. fast-fail和fast-safe机制：使用迭代器访问Java集合时，都可能产生这两类问题。

总体来看，**ArrayList是线程不安全的；CopyOnWriteArrayList是线程安全的；Vector也是线程安全的，不过效率较低，其原理大致就是给ArrayList中的方法加上synchronized。**



## 添加元素

> ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
}
```

讲解：先不管函数`ensureCapacityInternal`。add函数的作用是将元素放入数组，并且将尾指针后移一位。



> Vector

```java
public class Vector<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    /**
     * Appends the specified element to the end of this Vector.
     *
     * @param e element to be appended to this Vector
     * @return {@code true} (as specified by {@link Collection#add})
     * @since 1.2
     */
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
}
```

讲解：和ArrayList的实现几乎一样，只是多了`synchronized`修饰函数。



> CopyOnWriteArrayList

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 获取数组对象和数组长度
            Object[] elements = getArray();
            int len = elements.length;

            // 将原数组中每个元素的引用都复制一遍，放到一个新的数组中，并且给新数组加入1个元素
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;

            // 将新的数组赋值给原来的数组，这步操作耗时极短
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }

    /**
     * Gets the array.  Non-private so as to also be accessible
     * from CopyOnWriteArraySet class.
     */
    final Object[] getArray() {
        return array;
    }

    /**
     * Sets the array.
     */
    final void setArray(Object[] a) {
        array = a;
    }
}
```

讲解：

- 并发安全方面

  - add函数中使用锁`lock`，保证了多线程环境下添加元素不会产生线程安全问题。

    

- 写时复制（Copy On Write）与fail-safe

  - 增加元素时，现将**原来数组中的所有引用**拷贝一份并放入新数组（可搜索<u>`Arrays.copyOf()`是深拷贝还是浅拷贝</u>）。新数组中添加元素后，再将新数组的引用指向原数组。这样看来，**每次添加元素，都会创建1个新数组和复制2次数组！非常耗费资源！**

    

  - 采用大费周章的方式修改数组，是为了实现**fail-safe**机制。

    

  - 获取`CopyOnWriteArrayList`对象的迭代器对象`iter`时，`iter`对象中有一个数组引用`snapshot`指向当前数组`array`，即便之后数组增删元素，`iter`还是操作原来的数组，此时就会产生**数据一致性**问题。

    ```java
    public class CopyOnWriteArrayList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
        /** The array, accessed only via getArray/setArray. */
        private transient volatile Object[] array;
        
        /**
         * 获取迭代器对象
         */
        public Iterator<E> iterator() {
            return new COWIterator<E>(getArray(), 0);
        }
    
        /**
         * 获取当前数组对象
         */
        final Object[] getArray() {
            return array;
        }
    
        /**
         * 迭代器内部类
         */
        static final class COWIterator<E> implements ListIterator<E> {
            /** Snapshot of the array */
            private final Object[] snapshot;
            /** Index of element to be returned by subsequent call to next.  */
            private int cursor;
    
            private COWIterator(Object[] elements, int initialCursor) {
                cursor = initialCursor;
                snapshot = elements;
            }
        }
    }
    ```

    

- 使用场景

  - 适合于并发编程时**读多写少**的场景。



