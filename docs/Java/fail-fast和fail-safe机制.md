# fail-fast和fail-safe机制

[toc]

## 是什么

在 Java 中，`fail-fast` 和 `fail-safe` 机制通常用于**处理集合类的异常情况**。在使用**迭代器**遍历集合时，如果集合发生变化（增/删操作），就会发生意外问题。



【补充】使用迭代器遍历集合时，不建议对集合进行修改，因为这会让结构产生混乱（具体原因网上搜索）。因此，在某一时刻获取集合的迭代器对象，迭代器对象就应该只对此刻的集合进行操作。如果迭代器操作的过程中集合结构发生变化，则要么立即抛出异常（`fail-fast`机制），要么拷贝一个新的集合，对新的集合进行修改，让迭代器总是操作原来的数组。



`java.util`包下的大多数数据结构都是`fail-fast`机制。当使用迭代器遍历集合时，如果集合发送变化（比如增删元素），则会抛出`ConcurrentModificationException`异常。特殊情况下，如果使用迭代器的`remove()`删除元素，则不会报错（这不重要）。



`java.util.concurrent`包下的大多数数据结构都是`fail-safe`机制。



## 如何实现fail-fast

`fail-fast`的核心是**集合增删元素后，之前的迭代器不能使用**。



显然，我们可以在集合对象中维护一个成员变量`modCount`，当集合结构发生一次改变（增/删元素），`modCount`便加1。同时我们获取集合的迭代器对象时，应在迭代器对象中创建`expectedModCount`，用于记录当前集合的`modCount`。之后迭代器访问集合时，如果发现`modCount != expectedModCount`，就应该抛出错误。下面是`ArrayList`源码实现。



```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // modCount记录该数组被修改了多少次
    protected transient int modCount = 0;
    
    public Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        // 创建迭代器对象时，记录数组当前ModCount
        int expectedModCount = modCount;

        Itr() {}
        
        // 通过迭代器对象获取数组的下一个元素
        public E next() {
            // 检查数组是否被修改
            checkForComodification();
            
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
}
```



## 为什么要有fail-safe

有时候我们不希望迭代器操作集合时，由于集合结构的改变而频繁抛出异常。此时我们就可以



## 如何实现fail-safe

最简单的思路是：每创建一个迭代器对象`iter`，便将集合中的元素全部拷贝一份到`iter`中，命名为`newElements`，迭代器只操作`newElements`对象。之后无论集合如何改变，都不会影响`newElements`对象，因而迭代器仍能正常使用。这样的缺点是：修改后的数据不能及时通知`iter`对象，导致出现**数据一致性问题**。



上述思路基于“读时复制”，仍然存在优化的空间。加入我们在两个不同的时刻分别获取一个迭代器对象，且两个时刻之间集合没有增删元素，那么我们可以让这两个迭代器对象中的`newElements`引用都指向同一个集合对象。



Java源码中的实现思路是：迭代器对象`iter`在操作集合的过程中，应该将集合保存到`snapshot`。之后`iter`的操作都针对`snapshot`。如果集合增删元素，则将原先集合中的所有引用拷贝一份到新的集合中，并对新集合进行增删操作。之后即便集合发生改变，也不会影响`snapshot`的结构，因而迭代器仍能正常使用。此时之前的原集合被`snapshot`引用。下面是部分源码：



> 创建迭代器对象时，直接让迭代器中的snapshot数组对象引用集合对象array。

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



> 修改集合时，创建一个新数组，并将原数组中的引用拷贝到新数组中，同时在新数组中添加元素。

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



## 思考问题

1. `CopyOnWriteArrayList`添加元素时，是将原集合中的所有对象拷贝到新数组中，还是将原集合中的所有对象**引用**拷贝到新数组中？
