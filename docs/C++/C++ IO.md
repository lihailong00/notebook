# C++ IO

[toc]



## 文件IO

参考如下代码即可。以下案例仅涉及到日常使用的IO，如果要关注更多细节，还需要谷歌。

```cpp
/* 一些个人理解
 * 1. << 指向的方向就是数据传输的方向。参考如下案例：
 *      a) ifs >> buf   数据从ifs对象中流入内存buf
 *      b) ofs << buf   数据从buf流入ofs对象中
 * 2. ifstream和ofstream的区别
 *      a) 我们将内存放在“内部”，文件放在“外部”（因为文件存放在硬盘上）
 *      b) i 表示 “入”，也就是从外到内，也即 写入内存（或读取文件）
 *      c) o 表示 “出”，也就是从内到外，也即 从内存取出（或写入文件）
 * */

#include <iostream>
#include <fstream>
#include <string>
using namespace std;

void outMemory() {
    ofstream ofs;
    string filePath = "D:\\mydoc\\codes\\C++\\clion\\example.txt";

    // 如果文件不存在，则创建该文件；否则直接打开该文件
    // ios::out     清空文件内容，重新写入数（据默认值，可以不写）
    // ios::app     在文件末尾追加内容
    ofs.open(filePath, ios::out);

    // 检测文件是否打开成功
    // 如果1. 目录不存在 2. 磁盘已满 3. 没有权限 则会打开失败
    if (!ofs.is_open()) {
        cerr << "文件打开失败！" << endl;
        return;
    }

    cout << "文件打开成功！开始向文件中写入数据。" << endl;

    // 向文件中写入数据
    ofs << "hello, world!" << endl;

    ofs.close();
}

void inMemory() {
    ifstream ifs;
    string filePath = "D:\\mydoc\\codes\\C++\\clion\\example.txt";

    ifs.open(filePath);

    if (!ifs.is_open()) {
        cerr << "文件打开失败！" << endl;
        return;
    }

    // 将文件中的数据写入内存，再打印到显示器上
    string buf;
    // getline(ifs, buf)    按行读取
    // ifs >> buf   空格、tab、换行符 截断
    while (getline(ifs, buf)) {
        cout << buf << endl;
    }

    ifs.close();
}

int main() {
    outMemory();
    inMemory();
    return 0;
}
```





## stringIO

写算法题的时候偶尔会遇到这种操作。如果实际工程需要用到该函数，很有必要查找相关资料。

```cpp
#include <iostream>
#include <sstream>
using namespace std;

void SS() {
    string str = "aa bbb c ddd ee f g";
    // 将string加载到stringstream流中
    stringstream ss(str);
    
    string buf;
    // 按照 空格、tab、换行符 截取字符
    while (ss >> buf) {
        cout << "读出数据：" << buf << endl;
    }
    
    str = "aa\nbbb\nc\nddd\nee f\ng";
    stringstream ss2(str);
    // 按照 换行符 截取字符
    while (getline(ss2, buf)) {
        cout << buf;
    }
}

int main() {
    SS();
    return 0;
}
```



由于C++没有split函数，因此我们可以借助stringstream实现split功能。仅在写算法题时使用！实际工程中不要这么使用！

```cpp
#include <iostream>
#include <sstream>
using namespace std;

void split() {
    string str = "helloAworldAthisAisAfunny!";
    stringstream ss(str);
    string buf;
    // 自定义分隔符 A
    while(getline(ss, buf, 'A')) {
        cout << buf << endl;
    }
}

int main() {
    split();
    return 0;
}
```

