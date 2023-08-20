

# typename的使用

[toc]



## template的参数

这种使用场景非常常见，typename和class可以互换。

```cpp
#include <iostream>

template<typename T>
T sum(T&& a, T&& b) {
    return a + b;
}

int main() {
    std::cout << sum<int>(2, 3);
    return 0;
}
```



## 标识嵌套从属类型名称

假定我们有以下一个类模板：

```cpp
template<typename T>
class MyClass {
    T::iterator* iter;
};
```

按照常理，T::iterator是一个类型。但是如果我们用下面这个类来实例化上述类模板：

```cpp
class MyType {
    static int iterator;
};
```

那么编译器就会认为T::iterator * iter 是两数相乘。



总之，如果一个类模板中出现了依赖模板参数的类型，需要使用typename指定出来，让编译器将后面的字段当做类型。例如：

```cpp
template<typename T>
class MyClass {
    // 依赖 T
    typename T::iterator* iter;
};
```

需要掌握这种使用方法。