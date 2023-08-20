# C++强制转换



## reinterpret_cast

等同于强制转换，见下案例：

```cpp
#include <iostream>

int main() {
    // 显式强制转换
    int a = 10;
    int* p = (int*)a;
    
    // 将括号中的a强制转换成int*
    int* p2 = reinterpret_cast<int*>(a);
    return 0;
}
```



