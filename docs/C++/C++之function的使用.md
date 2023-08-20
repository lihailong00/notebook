# C++之function的使用



[toc]

## 封装普通函数

```cpp
#include <iostream>
#include <functional>

void test(int a, int b) {
    std::cout << "test函数" << std::endl;
    std::cout << "a=" << a << " b=" << b << std::endl;
}

int main() {
    // void表示函数的返回值，(int, int)表示函数的参数列表
    std::function<void(int, int)> f = test;
    f(2, 5);
    return 0;
}
```



## 封装匿名函数

```cpp
#include <iostream>
#include <functional>

int main() {
    std::function<float(int, int)> f = [](int a, int b) -> float {
        std::cout << "匿名函数" << std::endl;
        std::cout << "a=" << a << " b=" << b << std::endl;
        return 0.1;
    };
    float ret = f(2, 5);
    std::cout << "返回值为：" << ret << std::endl;
    return 0;
}
```



## 封装类成员函数

```cpp
#include <iostream>
#include <functional>

class Person {
public:
    int printInfo(int a, float b) {
        std::cout << "类成员函数" << std::endl;
        std::cout << "a=" << a << " b=" << b << std::endl;
        return 1;
    }
};

int main() {
    // Person* 是this指针
    std::function<int(Person*, int, float)> f = &Person::printInfo;
    Person p;
    int ret = f(&p, 1, 2);
    std::cout << "返回值是：" << ret << std::endl;
    return 0;
}
```



## bind机制

如果我们想将函数对象和参数绑定在一起，形成一个新的对象。那么可以使用`std::bind()`实现。

```cpp
#include <iostream>
#include <functional>

int func(int a, int b, int c) {
    std::cout << "a=" << a << " b=" << b <<  " c=" << c << std::endl;
    return a + b + c;
}

int main() {
    auto newFunc = std::bind(func, 1, 2, 3);
    int ret = newFunc();
    std::cout << "ret=" << ret << std::endl;
    return 0;
}
```

如果我们想将函数与一部分参数绑定形成一个新的参数，而另一部分参数传入到新函数的参数列表中，我们可以使用`std::placeholders::_数字`实现，操作如下：

```cpp
#include <iostream>
#include <functional>

int func(int a, int b, int c) {
    std::cout << "a=" << a << " b=" << b <<  " c=" << c << std::endl;
    return a + b + c;
}

int main() {
    auto newFunc = std::bind(func, 1, std::placeholders::_1, std::placeholders::_2);
    int ret = newFunc(3, 4);
    std::cout << "ret=" << ret << std::endl;
    return 0;
}
```

