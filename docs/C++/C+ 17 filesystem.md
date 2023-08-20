

```cpp
/* 功能解释：
 * 1. 去指定目录创建一系列文件
 * 2. 去指定目录删除一系列文件
 * 3. 显示指定目录下的文件信息
 * 4. 读取指定路径的文件数据
 * 5. 向指定文件写入数据
 * */

#include <iostream>
#include <fstream>
#include <filesystem>
using namespace std;
namespace fs = std::filesystem;

// 去指定目录创建多个文件
void createFiles(fs::path& targetPath) {
    if (!fs::is_directory(targetPath)) {
        cerr << "文件目录不存在！" << endl;
        return;
    }

    // 在指定目录下创建文件

    // 1. 获取当前目录下的配置文件 file_create_config.txt
    fs::path configCreatePath = fs::current_path();

    // 提示用户
    string remind;
    cout << "请在当前项目目录下的file_create_config.txt文件中填写待添加的文件名。填写完毕后输入yes：";
    while (cin >> remind, remind != "yes");

    // 跳转到上一级目录，并通过 / 拼接路径
    configCreatePath = configCreatePath.parent_path() / "file_create_config.txt";

    // 2.从当前目录的filein.txt文件中读取需要创建的文件名
    ifstream ifs;
    ifs.open(configCreatePath);

    // 如果没有权限，则会打开失败
    if (!ifs.is_open()) {
        cerr << "配置文件读取失败！" << endl;
        return;
    }

    string buf;
    ofstream ofs;
    while (ifs >> buf) {
        // 从配置文件中读取文件名，并在指定目录创建这些文件
        ofs.open(targetPath / buf);
        ofs.close();
        cout << "成功添加文件：" << buf << endl;
    }
    ifs.close();
}

// 去指定目录下删除多个文件
void deleteFiles(fs::path& targetPath) {
    if (!fs::is_directory(targetPath)) {
        cerr << "文件目录不存在！" << endl;
        return;
    }

    // 在指定目录下删除文件

    // 1. 获取当前目录下的配置文件 file_delete_config.txt
    fs::path configCreatePath = fs::current_path();

    // 提示用户
    string remind;
    cout << "请在当前项目目录下的file_delete_config.txt文件中填写待删除的文件名。填写完毕后输入yes。";
    while (cin >> remind, remind != "yes");

    // 跳转到上一级目录，并通过 / 拼接路径
    configCreatePath = configCreatePath.parent_path() / "file_delete_config.txt";

    // 2.从当前目录的filein.txt文件中读取需要创建的文件名
    ifstream ifs;
    ifs.open(configCreatePath);

    // 如果没有权限，则会打开失败
    if (!ifs.is_open()) {
        cerr << "配置文件读取失败！" << endl;
        return;
    }

    string buf;
    int count = 0;
    while (ifs >> buf) {
        // 从配置文件中读取文件名，并在指定目录删除这些文件
        if (fs::remove(targetPath / buf)) {
            cout << "成功删除文件：" << buf << endl;
            count++;
        }
    }
    ifs.close();
    cout << "共删除" << count << "个文件！" << endl;
}

// 显示当前目录下的文件
void showFiles(fs::path& targetPath) {
    if (!fs::is_directory(targetPath)) {
        cerr << "文件目录不存在！" << endl;
        return;
    }

    // 获取指定目录的内容
    fs::directory_iterator iter(targetPath);
    for (const auto& it : iter) {
        if (it.is_directory()) {
            cout << it.path().filename() << endl;
        }
        else {
            cout << it.path().filename() << "\t" << it.file_size() << " Byte" << endl;
        }
    }
}

void write2File(fs::path& targetPath) {
    ofstream ofs;
    ofs.open(targetPath);
    if (!ofs.is_open()) {
        cerr << "文件不存在" << endl;
        return;
    }
    string buf;
    cout << "请向文件中写入一段数据：";
    cin >> buf;
    ofs << buf;
    cout << "写入成功！" << endl;
    ofs.close();
}

void readFromFile(fs::path& targetPath) {
    ifstream ifs;
    ifs.open(targetPath);
    if (!ifs.is_open()) {
        cerr << "文件打开失败！" << endl;
        return;
    }
    string buf;
    while (getline(ifs, buf)) {
        cout << buf << endl;
    }
}

int main() {
    while (true) {
        cout << "=================================" << endl;
        cout << "欢迎来到文件管理系统" << endl;
        cout << "1.去指定目录下创建多个文件" << endl;
        cout << "2.查看目录下的所有文件信息" << endl;
        cout << "3.去指定目录下删除多个文件" << endl;
        cout << "4.读取指定路径的文件数据" << endl;
        cout << "5.写入指定路径的文件" << endl;
        cout << "0.退出系统" << endl;
        cout << "=================================" << endl;
        cout << "请输入：";
        string op;
        cin >> op;

        if (op == "1") {
            fs::path targetPath;
            cout << "请输入指定目录:";
            cin >> targetPath;
            createFiles(targetPath);
        }
        else if (op == "2") {
            fs::path targetPath;
            cout << "请输入指定目录:";
            cin >> targetPath;
            showFiles(targetPath);
        }
        else if (op == "3") {
            fs::path targetPath;
            cout << "请输入指定目录:";
            cin >> targetPath;
            deleteFiles(targetPath);
        }
        else if (op == "4") {
            fs::path targetPath;
            cout << "请输入指定文件:";
            cin >> targetPath;
            write2File(targetPath);
        }
        else if (op == "5") {
            fs::path targetPath;
            cout << "请输入指定文件:";
            cin >> targetPath;
            readFromFile(targetPath);
        }
        else if (op == "0") {
            break;
        }
        else {
            cout << "错误输入！请重新输入。" << endl;
        }
        system("pause");
        system("cls");
    }
    return 0;
}
```

