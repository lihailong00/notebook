# C++移动语义

[toc]

## 什么是移动语义

移动语义是指在不拷贝对象的情况下将对象的资源从一个地方移动到另一个地方的过程。



## 右值引用

引用分为左值引用和右值引用，引用的目的都是为了减少内存开销优化内存的一种方法。

右值引用就是将那些临时阐述的变了或对象“抢”过来，作为长生命周期的对象。

```cpp
#include <iostream>

void func(int&& x) {
    std::cout << "x=" << x << std::endl;
}

int main() {
    func(10);
    /*
    报错，右值引用不能接收“正常”的变量
    int a = 10;
    func(a);
    */
    return 0;
}
```





## std::move()

通常把“将亡”对象的值“低成本”的赋值给新的对象。

有些情况使用std::move没有效果。目前我也不太懂什么时候使用该函数。我的建议是想要将一个值转移给另一个值的时候使用，如果clion警告，再删除这个函数也不迟。

```cpp
#include <iostream>
#include <string>

void func(std::string& str) {
    std::string tmp = "hello, world!";
    str = std::move(tmp);
}

int main() {
    std::string str;
    func(str);
    std::cout << "str=" << str << std::endl;
    std::string str2 = std::move(str);
    std::cout << "move之后，str=" << str << std::endl;
    std::cout << "move之后，str2=" << str2 << std::endl;
    return 0;
}
```

以下这种情况，std::move就没作用。

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = std::move(a);

    std::cout << "a=" << a << std::endl;
    std::cout << "b=" << b << std::endl;

    return 0;
}
```

