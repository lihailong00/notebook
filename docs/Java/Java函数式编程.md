# Java函数式编程

[toc]

## lambda表达式



## Stream

> 获取流对象：通常通过list，set转为流对象。

```java
public class StreamDemo {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("apple", "banana");
        Set<String> set = new HashSet<>(Arrays.asList("apple", "banana", "peach"));

        // 通过Collection接口的实现类调用stream方法获取stream对象
        Stream<String> stream = list.stream();
        Stream<String> stream1 = set.stream();

        // 直接获取stream对象
        Stream<Integer> integerStream = Stream.of(1, 2, 5, 2, 2);
        Stream<String> stringStream = Stream.of("lc", "lhl", "tourist");
    }
}

```





> 常见操作：sorted, filter, distinct, map, limit, collect

```java
public class StreamDemo {
    public static void main(String[] args) {
        ArrayList<Integer> arr = new ArrayList<>(Arrays.asList(1, 4, 2, 3, 4, 2, 5, 4, 6, 3));
        Stream<Integer> integerStream = arr.stream();
        Stream<String> newStream = integerStream.sorted()
                .filter(x -> x % 2 == 0)  // 过滤掉奇数
                .distinct()
                .map(x -> "新元素：" + x)  // 将每个元素转为“字符串+数字”，这里转变了元素的类型
                .limit(2);
        List<String> list = newStream.collect(Collectors.toList());
        Set<String> set = newStream.collect(Collectors.toSet());
    }
}
```



## parallelStream

1. 使用方式和Stream几乎一样。
2. 有线程安全问题，尽量不要做修改操作。
3. 处理大量数据有优势，但是处理少量数据时由于线程的切换，反而降低程序效率。

