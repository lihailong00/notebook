```cpp
#include <stdio.h>
// 网络通信需要包含的头文件，需要加载的文件
#include <WinSock2.h>
#pragma comment(lib, "WS2_32.lib")

void error_die(const char* str) {
    perror(str);
    exit(1);
}

/*
* 实现网络的初始化：
* 返回值：服务器的套接字
* 参数：port，表示端口号。如果port为0，那么自动分配一个可用的端口。
*/

int startup(unsigned short * port) {
    WSADATA data;  // 存储winsocket初始化后得到的数据
    int ret = WSAStartup(
        MAKEWORD(1, 1),  // 1.1版本的协议
        &data);

    if (ret != 0) {
        error_die("WSAStartup die!");
    }

    int server_socket = socket(PF_INET,  // 套接字的类型（包括文件套接字和网络套接字，这里使用网络套接字）
        SOCK_STREAM,  // 数据流
        IPPROTO_TCP);
    
    if (server_socket == -1) {
        // 打印错误提示，并结束程序
        error_die("socket die!");
    }

    // 设置端口可复用
    int opt = 1;
    ret = setsockopt(server_socket, 
        SOL_SOCKET, 
        SO_REUSEADDR, 
        (const char*)&opt, 
        sizeof(opt));

    if (ret == -1) {
        error_die("set socket opt");
    }

    // 配置服务器端的网络地址
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // htons是将机器中的整型转换成“网络字节序”（大端模式）
    // h: host  to: to  n: net  s: unsigned short  l: unsigned long
    server_addr.sin_port = htons(*port);  // 开放的端口
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 所有ip的客户端都可以访问

    // 绑定套接字和地址
    // （不用太在意类型转换）
    ret = bind(server_socket, 
        (const struct sockaddr*)&server_addr, 
        sizeof(server_addr));

    // 动态分配一个端口
    int nameLen = sizeof(server_addr);
    if (*port == 0) {
        ret = getsockname(server_socket, (struct sockaddr*)&server_addr, &nameLen);
        if (ret < 0) {
            error_die("fail distributing port");
        }
        
        *port = server_addr.sin_port;
    }
    
    if (ret == -1) {
        error_die("bind error");
    }

    // 创建监听队列
    ret = listen(server_socket, 5);
    if (ret < 0) {
        error_die("listen error!");
    }
    return server_socket;
}

int main() {
    unsigned short port = 0;
    int server_sock = startup(&port);
    printf("httpd服务器已经启动，正在监听%d端口", port);
    return 0;
}
```





```cpp
#include <stdio.h>
// 网络通信需要包含的头文件，需要加载的文件
#include <WinSock2.h>
#pragma comment(lib, "WS2_32.lib")

void error_die(const char* str) {
    perror(str);
    exit(1);
}

/*
* 实现网络的初始化：
* 返回值：服务器的套接字
* 参数：port，表示端口号。如果port为0，那么自动分配一个可用的端口。
*/

int startup(unsigned short * port) {
    WSADATA data;  // 存储winsocket初始化后得到的数据
    int ret = WSAStartup(
        MAKEWORD(1, 1),  // 1.1版本的协议
        &data);

    if (ret != 0) {
        error_die("WSAStartup die!");
    }

    int server_socket = socket(PF_INET,  // 套接字的类型（包括文件套接字和网络套接字，这里使用网络套接字）
        SOCK_STREAM,  // 数据流
        IPPROTO_TCP);
    
    if (server_socket == -1) {
        // 打印错误提示，并结束程序
        error_die("socket die!");
    }

    // 设置端口可复用
    int opt = 1;
    ret = setsockopt(server_socket, 
        SOL_SOCKET, 
        SO_REUSEADDR, 
        (const char*)&opt, 
        sizeof(opt));

    if (ret == -1) {
        error_die("set socket opt");
    }

    // 配置服务器端的网络地址
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // htons是将机器中的整型转换成“网络字节序”（大端模式）
    // h: host  to: to  n: net  s: unsigned short  l: unsigned long
    server_addr.sin_port = htons(*port);  // 开放的端口
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 所有ip的客户端都可以访问

    // 绑定套接字和地址
    // （不用太在意类型转换）
    ret = bind(server_socket, 
        (const struct sockaddr*)&server_addr, 
        sizeof(server_addr));

    // 动态分配一个端口
    int name_len = sizeof(server_addr);
    if (*port == 0) {
        ret = getsockname(server_socket, (struct sockaddr*)&server_addr, &name_len);
        if (ret < 0) {
            error_die("fail distributing port");
        }
        
        *port = server_addr.sin_port;
    }
    
    if (ret == -1) {
        error_die("bind error");
    }

    // 创建监听队列
    ret = listen(server_socket, 5);
    if (ret < 0) {
        error_die("listen error!");
    }
    return server_socket;
}

// 处理用户请求的线程函数
DWORD WINAPI accept_request(LPVOID arg) {
    
    return 0;
}

int main() {
    unsigned short port = 0;
    int server_sock = startup(&port);
    printf("httpd服务器已经启动，正在监听%d端口", port);

    struct sockaddr_in client_addr;

    int client_addr_len = sizeof(client_addr);
    while (true) {
        // 阻塞式等待客户端访问此服务器
        
        // server_sock套接字用于接收访问
        // client_sock套接字用于处理客户端的请求
        int client_sock = accept(server_sock, 
            (struct sockaddr*)&client_addr, 
            &client_addr_len);
        
        if (client_sock == -1) {
            error_die("accept");
        }
        
        // 使用client_sock和用户交互
        // 创建一个新的线程
        DWORD thread_id = 0;
        // 第四个参数作为第三个回调函数的参数
        CreateThread(0, 0, accept_request, (void*)client_sock, 0, &thread_id);

    }
    closesocket(server_sock);
    return 0;
}
```



没法自动分配端口？！

```
#include <stdio.h>
// 网络通信需要包含的头文件，需要加载的文件
#include <WinSock2.h>
#pragma comment(lib, "WS2_32.lib")

#define PRINTF(str) printf("[%s - %d]"#str"=%s", __func__, __LINE__, str);

void error_die(const char* str) {
    perror(str);
    exit(1);
}

/*
* 实现网络的初始化：
* 返回值：服务器的套接字
* 参数：port，表示端口号。如果port为0，那么自动分配一个可用的端口。
*/

int startup(unsigned short * port) {
    WSADATA data;  // 存储winsocket初始化后得到的数据
    int ret = WSAStartup(
        MAKEWORD(1, 1),  // 1.1版本的协议
        &data);

    if (ret != 0) {
        error_die("WSAStartup die!");
    }

    int server_socket = socket(PF_INET,  // 套接字的类型（包括文件套接字和网络套接字，这里使用网络套接字）
        SOCK_STREAM,  // 数据流
        IPPROTO_TCP);
    
    if (server_socket == -1) {
        // 打印错误提示，并结束程序
        error_die("socket die!");
    }

    // 设置端口可复用
    int opt = 1;
    ret = setsockopt(server_socket, 
        SOL_SOCKET, 
        SO_REUSEADDR, 
        (const char*)&opt, 
        sizeof(opt));

    if (ret == -1) {
        error_die("set socket opt");
    }

    // 配置服务器端的网络地址
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // htons是将机器中的整型转换成“网络字节序”（大端模式）
    // h: host  to: to  n: net  s: unsigned short  l: unsigned long
    server_addr.sin_port = htons(*port);  // 开放的端口
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 所有ip的客户端都可以访问

    // 绑定套接字和地址
    // （不用太在意类型转换）
    ret = bind(server_socket, 
        (const struct sockaddr*)&server_addr, 
        sizeof(server_addr));

    // 动态分配一个端口
    int name_len = sizeof(server_addr);
    if (*port == 0) {
        ret = getsockname(server_socket, 
            (struct sockaddr*)&server_addr, &name_len);
        if (ret < 0) {
            error_die("fail distributing port");
        }
        printf("\n端口号：%d\n", server_addr.sin_port);
        *port = server_addr.sin_port;
    }
    
    if (ret == -1) {
        error_die("bind error");
    }

    // 创建监听队列
    ret = listen(server_socket, 5);
    if (ret < 0) {
        error_die("listen error!");
    }
    return server_socket;
}

// 从指定的客户端套接字，读取一行数据，并保存到buff中，返回读取的字符数
int get_line(int sock, char* buff, int size) {
    char c = 0;
    int i = 0;
    // 如果获取 \r\n，则只存储 \n
    // 如果获取\n，直接存储 \n
    while (i < size - 1 && c != '\n') {
        int n = recv(sock, &c, 1, 0);
        if (n > 0) {
            if (c == '\r') {
                // MSG_PEEK: 瞄一眼下一个字符
                n = recv(sock, &c, 1, MSG_PEEK);
                if (n > 0 && n == '\n') {
                    recv(sock, &c, 1, 0);
                }
                else {
                    c = '\n';
                }
            }
            buff[i++] = c;
        }
    }
    buff[i] = 0;
    return 0;
}

// 处理用户请求的线程函数
DWORD WINAPI accept_request(LPVOID arg) {
    char buff[1024];
    
    int client = (SOCKET)arg;

    // 读取一行数据
    int num_char = get_line(client, buff, sizeof(buff));
    
    PRINTF(buff);

    return 0;
}

int main() {
    unsigned short port = 1800;
    int server_sock = startup(&port);
    printf("httpd服务器已经启动，正在监听%d端口", port);

    struct sockaddr_in client_addr;

    int client_addr_len = sizeof(client_addr);
    while (true) {
        // 阻塞式等待客户端访问此服务器
        
        // server_sock套接字用于接收访问
        // client_sock套接字用于处理客户端的请求
        // 两个套接字都在服务器中
        int client_sock = accept(server_sock, 
            (struct sockaddr*)&client_addr, 
            &client_addr_len);
        printf("hello world");
        if (client_sock == -1) {
            error_die("accept");
        }
        
        // 使用client_sock和用户交互
        // 创建一个新的线程
        DWORD thread_id = 0;
        // 第四个参数作为第三个回调函数的参数
        CreateThread(0, 0, accept_request, (void*)client_sock, 0, &thread_id);
    }
    closesocket(server_sock);
    return 0;
}
```

