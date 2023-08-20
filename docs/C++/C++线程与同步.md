# C++线程与同步

[toc]



## lock_guard与mutex实现同步

> lock_gard锁的是当前所处的代码块。它的原理是创建lock_guard类型的变量时在默认构造函数中获得锁，当前代码块执行完毕后在析构函数中释放锁。

```cpp
#include <iostream>
#include <thread>
#include <mutex>

int gData = 0;
std::mutex mtx;

void func() {
    for (int i = 0; i < 1000000; i++) {
        std::lock_guard<std::mutex> lg(mtx);
        gData++;
    }
}

int main() {
    std::thread t1(func);
    std::thread t2(func);
    std::thread t3(func);
    t1.join();
    t2.join();
    t3.join();
    std::cout << "gData=" << gData << std::endl;
    return 0;
}
```



## 条件变量

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <string>
#include <condition_variable>

std::string str;
std::condition_variable cv;
std::mutex mtx;
bool isSet = false;

void task1() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    str = "hello, world!";
    isSet = true;
    // 如果使用notify_all函数，则能让所有与该条件变量相关的阻塞线程继续执行。
    // 如果使用notify_one函数，则随机让一个与该条件变量相关的阻塞线程继续执行。
    cv.notify_all();
}

void task2() {
    // 条件变量配合unique_lock使用
    std::unique_lock<std::mutex> ul(mtx);
    
    while (!isSet) { cv.wait(ul); }
    
    std::cout << "str=" << str << std::endl;
}

int main() {
    std::thread t1(task1);
    std::thread t2(task2);
    std::thread t22(task2);
    // 阻塞t1线程，直到线程结束
    t1.join();
    t2.join();
    t22.join();
    return 0;
}
```



## wait函数

上述案例中`while (!isSet) { cv.wait(ul); }`也可以被这样代替：

```cpp
cv.wait(ul, []() { return isSet; });
```

上述代码的意思是：

当wait函数第二个参数的返回值如果为false，则阻塞在此处，并释放和ul相关的锁。



## join函数和detach函数

线程使用完毕后，必须调用join函数或者detach函数，否则会报错。（不要纠结报错原因）

> 线程join函数后，阻塞该线程，直到线程被释放。
>
> 易错：join函数并不是表示释放线程。

```cpp
#include <iostream>
#include <thread>

void task1() {
    std::cout << "hello, world" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::atexit([]() -> void {
        std::cout << "t1线程将要结束！" << std::endl;
    });
}

int main() {
    std::thread t1(task1);
    t1.join();
    std::cout << "主线程结束！" << std::endl;
    return 0;
}
```



## 线程函数传参

```cpp
#include <iostream>
#include <thread>

void func(int num, std::string str) {
    std::cout << "num=" << num << " str=" << str << std::endl;
}

int main() {
    // 参数直接跟在后面即可
    std::thread t(func, 10, "hello,world!");
    t.join();
    return 0;
}
```



## promise和future

`std::promise` 和 `std::future` 是一对关联的对象，其中 `std::promise` 对象用于在一个线程中生成一个结果，而 `std::future` 对象则用于在另一个线程中获取这个结果。这样，就可以在一个线程中异步地执行某些计算，并在另一个线程中等待并获取计算的结果。

```cpp
#include <iostream>
#include <future>
#include <thread>

void task(int a, int b, std::promise<int>& ret) {
    // promise设定值以后，future才能获取到该值
    ret.set_value(a * 2 + b * 3);
}

int main() {
    std::promise<int> p;
    // 绑定future和promise
    std::future<int> f = p.get_future();
    // 在另一个线程中执行运算任务
    std::thread t(task, 10, 20, std::ref(p));
    // promise设定值以前，future会阻塞在此处
    // promise设定值以后，future才能get到值
    // get函数只能调用一次
    std::cout << "result is " << f.get() << std::endl;
    t.join();
    return 0;
}
```

