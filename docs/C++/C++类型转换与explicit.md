# C++类型转换与explicit



[toc]



## C++类型转换

### 隐式类型转换

有时候为了方便，编译器会将变量类型做隐式转换，例如：

```cpp
#include <iostream>
using namespace std;

int main() {
	int a = 10;
	double b = 20.0;
	cout << a + b << endl;
	return 0;
}
```



### 强制类型转换

```cpp
#include <iostream>
using namespace std;

int main() {
	int num = 97;
	cout << (char)num << endl;
	return 0;
}
```



### 类型转换构造函数

这个比较反直觉，但我们可以认为编译器为了通过编译而不择手段。对于`p = 2`，如果没法通过，编译器则会尝试`Person p(2)`能否通过。构造函数“莫名其妙”地提供了一个“转换”的功能。

实际上，`p = 2`**隐式**调用了`Person(int age)`这个函数。

```cpp
#include <iostream>
using namespace std;

class Person {
public:
	int age;
	Person() {
	}
    // 转换构造函数
	Person(int age) {
		this->age = age;
	}
};

int main() {
	Person p;
    p = 2;
	cout << p.age << endl;
	return 0;
}
```



## explicit关键字

由于构造函数会莫名其妙提供转换功能，而我们不希望出现这种情况。此时只需要在这个构造函数之前添加explicit关键字即可。

**建议在构造函数之前尽量使用explicit关键字**。（出自《Effective C++》）

```cpp
#include <iostream>
using namespace std;

class Person {
public:
	int age;
	Person() {
	}
    // 指定这个构造函数不能提供隐式转换的功能，只能显式调用
	explicit Person(int age) {
		this->age = age;
	}
};

int main() {
	Person p;
    // 编译报错
	p = 2;
	return 0;
}
```

