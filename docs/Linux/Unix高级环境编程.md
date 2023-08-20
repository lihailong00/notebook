# Unix高级环境编程

[toc]



## 文件

问题：文件的基本操作有哪些？

答案：

```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <iostream>
using namespace std;

void opt1() {
        // 文件打开方式：
        // O_RDONLY     以只读的形式打开
        // O_WRONLY     以只写的形式打开
        // O_RDWR       以可读可写的形式打开
        int fd = open("./t1.txt", O_RDWR);
        if (fd == -1) {
                cout << "文件打开失败！" << endl;
        }
        else {
                cout << "成功打开文件！！！" << endl;
        }
}

void opt2() {
        // O_CREAT      如果文件不存在，创建文件，通过mode指定文件的权限
        int fd = open("./t2.txt", O_CREAT);
        if (fd == -1) {
                cout << "文件打开失败！" << endl;
        }
        else {
                cout << "成功打开文件！！！" << endl;
        }
}

// 思考：为什么用opt3创建的文件一会儿是红底，一会儿是黄底
void opt3() {
        // O_CREAT 和 O_EXCL 一起使用时，如果文件不存在，则创建文件；否则报错
        int fd = open("./t3.txt", O_CREAT | O_EXCL);
        if (fd == -1) {
                cout << "文件打开失败" << endl;
        }
        else {
                cout << "成功打开文件！！！" << endl;
        }
}

void opt4() {
        // O_TRUNC      如果有文件，则清空并打开文件。否则返回错误！
        int fd = open("./t4.txt", O_TRUNC);
        if (fd == -1) {
                cout << "文件打开失败" << endl;
        }
        else {
                cout << "成功打开文件！！！" << endl;
        }
}

void opt5() {
        // O_APPEND     如果有文件，打开文件。否则返回错误！
        int fd = open("./t4.txt", O_APPEND);
        if (fd == -1) {
                cout << "文件打开失败" << endl;
        }
        else {
                cout << "成功打开文件！！！" << endl;
        }
}

int main() {
        return 0;
}
```



问题：pid和PCB是什么？什么是文件描述符表项？什么是文件描述符（fd）？struct file对象记录了哪些信息？

答案：

每个进程有自己的pid和PCB；

进程中的PCB中记录了该进程使用计算机资源的情况；

进程每打开一个文件，内核会产生一个struct file类型的对象，并将该对象记录到进程PCB文件的文件描述符表项中；

文件描述表项的下标就是文件描述符；

struct file对象的成员字段记录了文件的状态、读写位置、路径等信息；



问题：task_stuct，files_stuct，files的关系是什么？

答案：

[参考文档](https://zhuanlan.zhihu.com/p/364617329)

![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/0D023A61309918C0F31E21D644EF2B65.png)

> task_stuct是一个进程的PCB，管理该进程相关的所有资源。

> files_stuct是用于管理该进程打开的所有文件，被打开的文件指针存放在fd_array静态数组和fdtable动态数组中。

> fd就是fies_stuct的下标。



问题：常见文件描述符0,1,2？

答案：

0：标准输入	键盘

1：标准输出	显示器

2：标准错误输出	显示器



问题：文件有哪些类型？

答案：

`-`：普通文件

`d`：文件夹文件

`c`：字符设备文件

`b`：块设备文件

`p`：管道文件

`s`：socket文件

`l`：软链接文件

建议到`/dev`目录下查看文件。



问题：文件有哪三种访问权限？有哪三种不同类型的用户？



问题：umask是什么？如何查看和修改进程的umask掩码？

答案：

1. 通常root用户的umask掩码是0022，普通用户是0002。第一个0表示八进制。后面三位数分别针对三种用户：自己，组内成员，其他人。
2. umask的作用：对于0002，创建的文件权限是**其他人不可写**；对于0022，**创建文件的权限是组内成员和其他人不可写**。

3. 查看进程的umask掩码：`umask`。
4. 修改进程的umask掩码：`umask 0022`



问题：实现一个cp功能？

答案：

```cpp
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <iostream>

 // 文件读取
// #include <unistd.h>
//
// ssize_t read(int fd, void* buf, size_t count);
//
// fd: 指定的文件描述符
// buf: 文件中读取到的数据放到该内存地址
// count: 指定最多读取的字节数
//      成功 返回读取到的字节数
//      失败 返回-1
//      0 代表读取到末尾
//
//文件写入
// #include <unistd.h>
//
// ssize_t write(int fd, const void* buf, size_t count);
//
// fd: 指定的文件描述符
// buf: 该内存地址中的数据读入fd对应的文件中
// count: 指定了写入文件的最大字节数
//      成功: 返回实际写入的字节数
//      失败: -1
//
//
// #include <sys/types.h>
// #include <unistd.h>
// off_t lseek(int fd, off_t offset, int whence)


int cp_file(int src_fd, int dst_fd) {
        int total = 0;
        // 从源文件中读取数据到buf
        int len_r, len_w;
        char buf[4096];
        while ((len_r = read(src_fd, buf, 4096)) > 0) {
                char* p = buf;
                while (len_r > 0) {
                        len_w = write(dst_fd, p, len_r);
                        len_r -= len_w;
                        total += len_w;
                        p += len_w;
                }
        }
        return total;
}

// 通过命令行传递文件名字
// argv[1] 源文件名字
// argv[2] 目标文件名次
int main(int argc, char* argv[]) {
        // 以只读的方式打开文件
        int src_fd = open(argv[1], O_RDONLY);

        // 以只写的方式打开文件
        //      如果文件不存在，创建文件，指定文件的权限为0644
        //      如果文件存在，将文件的内容情况

        int dst_fd = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0777);

        if (src_fd == -1 || dst_fd == -1) {
                std::cout << "文件打开失败！" << std::endl;
        }
        else {
                cp_file(src_fd, dst_fd);
        }
        // 关闭文件描述符
        close(src_fd);
        close(dst_fd);
        return 0;
}
```





## 进程

问题：什么是进程？

答案：进程是运行程序的实例。（还是比较抽象~）



问题：查看进程的常见指令？

答案：

```
pstree		用户级进程树，可以查看父子进程和兄弟进程关系
ps -aux		系统进程情况
top			进程实施情况
```



问题：进程的状态？

答案：

```
O：就绪
R：运行
S：可唤醒睡眠
D：不可唤醒睡眠
T：暂停，接收到SIGSTOP信号转入暂停，接收到嗷SIGCONT信号转入运行
X：死亡
Z：僵尸进程，已停止运行，但父进程尚未获取其状态
<：高优先级
N：低优先级
L：
s：
I：
+：
```





问题：进程的创建？

答案：

函数定义：

```cpp
#include <unistd.h>
pid_t fork(void);
/*
功能：创建一个子进程
返回值：
	父进程中：
		创建失败，返回-1
		创建成功，返回子进程的pid
	子进程中：返回0
*/
```

实际案例：

```cpp
#include <stdio.h>
#include <unistd.h>

int main() {
        // 在父进程中创建一个子进程
        pid_t pid = fork();  // 这一步执行完后，子进程中也会复制一份相同的代码
        if (pid == -1) {
                printf("子进程创建失败~\n");
        }
        else if (pid == 0) {
                printf("子进程执行！\n");
        }
        else {
                printf("父进程执行！\n");
        }
        printf("父子进程都会执行！\n");
        return 0;
}

/*
执行结果：
父进程执行！
父子进程都会执行！
子进程执行！
父子进程都会执行！
*/
```





问题：子进程创建成功后，会受到父进程的“制约”吗？

答案：子进程创建成功后，就是独立的资源调度单位，谁先调度取决于操作系统。



问题：子进程和父进程共享PCB吗？如果共享，当子进程有写操作时，如何避免影响父进程的PCB呢？

答案：共享；如果子进程有写操作时，会从父进程中复制一部分资源放到子进程中，这部分资源不被共享。



问题：进程如何终止？return和exit两种退出方式的退出状态码有何不同？

答案：

1. return、exit
2. 退出状态码的取值范围是0~255；return x 的退出状态码是x；exit(x)的退出状态码是x & 0377。

代码如下：

```c
#include <stdlib.h>
#include <stdio.h>

int main() {
	getchar();  // 让代码执行到这一步停下来
	exit(3);  // 返回 3&0377 给父进程
}
/*
然后重新开一个bash2，用pstree查看该进程的父进程。得知bash是该进程的父进程。接下来直接输入一个换行符。在bash中输入echo $?即可得到exit传递给父进程的参数。
*/
```





问题：什么是遗言函数？

答案：使用exit或return结束进程之前需要执行的函数。使用atexit函数和on_exit函数注册遗言函数。

> 函数定义

```c
#include <stdlib.h>
int atexit(void(*function)(void));
/*
功能：向进程注册遗言函数function
参数：function：遗言函数的地址
返回值：成功返回0，失败返回非0

注册几次遗言函数，就会调用几次遗言函数，但是注册顺序和调用顺序相反。
子进程继承父进程的遗言函数
*/

int on_exit(void(*function)(int, void*), void* arg);
/*
功能：想进程注册遗言函数function
参数：
	function：遗言函数地址
	arg：传递给遗言函数function的参数
返回值：
	成功 0
	失败 非0
*/
```

> 实战

atexit函数

```c

#include <stdlib.h>
#include <cstdio>
#include <unistd.h>

void bye1() {
        printf("我死啦1~\n");
}

void bye2() {
        printf("我死啦2~\n");
}

int main() {
        // 向进程注册遗言函数
        atexit(bye1);
        atexit(bye2);
        // 注册遗言函数之后再创建子进程
        pid_t pid = fork();
        if (pid == -1) {
                printf("子进程创建失败~");
        }
        return 0;
}
```

on_exit函数

```c

#include <stdlib.h>
#include <cstdio>
#include <unistd.h>

void bye1() {
        printf("我死啦1~\n");
}

void bye2() {
        printf("我死啦2~\n");
}

int main() {
        // 向进程注册遗言函数
        atexit(bye1);
        atexit(bye2);
        // 注册遗言函数之后再创建子进程
        pid_t pid = fork();
        if (pid == -1) {
                printf("子进程创建失败~");
        }
        return 0;
}
```



问题：父进程如何回收子进程的资源？

答案：

父进程回收子进程资源之前，要保证子进程执行完毕；否则父进程需要阻塞，直到子进程执行完毕。

```c
pid_t wait(int* status);
/* 
功能：阻塞等待子进程结束，
参数：用于存放子进程的退出状态码
返回值：
	成功：终止子进程的id
	失败：-1
*/
```



实战：

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
	// 创建子进程
	pid_t pid = fork();
	if (pid == -1) {
		printf("子进程创建失败");
	}
	else if (pid == 0) {
		printf("子进程执行，pid=%d\n", getpid());
		// 让子进程暂时阻塞在此处，方便发送信号
		getchar();
		exit(-1);
	}
	else {
		int code = 0;  // code是子进程的退出状态码
		printf("父进程开始执行！\n");
		// wait函数使得父进程阻塞，等待回收子进程资源
		wait(&code);
		printf("code=%d\n", code);
		if (WIFEXITED(code)) {  // 子进程正常终止
			printf("子进程正常的退出状态码是：%d\n", WEXITSTATUS(code));
		}
		if (WIFSIGNALED(code)) {  // 子进程被信号打断
			printf("子进程被信号 %d 打断\n", WTERMSIG(code));
		}
		printf("父进程继续执行！\n");
	}

	return 0;
}
```





```c
pid_t waitpid(pid_t pid, int* status, int options);
/*
功能：阻塞顶戴进程结束，然后挥手子进程的资源
参数：
	pid：指定要回收的子进程id
	status：用于存储子进程的退出状态码

关于pid的取值：
	pid < -1	？？？
	pid = -1	等待任意子进程
	pid = 0		等待和父进程同组的子进程
	pid > 0		pid的值指定了要扥带的子进程的pid
*/
```



问题：wait和waitpid的区别？

wait等待的是第一个子进程，waitpid可指定等待某个子进程。



问题：如何检查某个进程终止的原因？

答案：

获取到该进程向其父进程发送的**退出状态码**（想想怎么获取），然后调用下面的宏即可。

```
WIFEXITED(status)：如果进程正常结束，返回真
WEXITSTATUS(status)：获取进程的退出状态码（我比较困惑，明明status就是退出状态码  呀）
WIFSIGNALED(status)：如果进程被信号打断，返回真
WTERMSIG(status)：获取打断进程的信号的编号

给进程发送信号：kill -信号编号 进程的pid
```





## 进程之间通信

无名管道：

管道是内核空间中的一块内存空间。

进程之间通信必须借助管道。

管道分为无名管道和有名管道。

无名管道用于父子或兄弟进程之间。使用`pipe(2)`创建无名管道。



``` 
#include <unistd.h>
int pipe(int pipefd[2]);

// 功能：创建一个单工的无名管道
// pipefd[0] 指向管道读端，pipefd[1]指向管道写端     
```





有名管道：

有名管道实际上是一种文件。

> fifo.cpp 创建管道文件

```cpp
#include <iostream>
#include <sys/types.h>
#include <sys/stat.h>

int main(int argc, char* argv[]) {
    // 创建管道类型的文件
    // 第一个参数是文件名，第二个参数是文件的权限（可能会被掩码修改）
    int ff = mkfifo(argv[1], 0644);
    
    if (ff == -1) {
        std::cerr << "文件创建失败~" << std::endl;
    }

    std::cout << "管道文件" << argv[1] << "成功创建！" << std::endl;

    return 0;
}
```



> pa.cpp 向管道文件写入数据

```cpp
#include <iostream>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

int main(int argc, char* argv[]) {
    // 以 写 的方式打开管道文件
    int fd = open(argv[1], O_WRONLY);

    if (fd == -1) {
        std::cerr << "文件打开失败~" << std::endl;
    }

    // 向 管道文件 中写入数据
    const char* msg = "this is a msg...\n";
    write(fd, msg, strlen(msg));

    close(fd);
    
    return 0;
}
```



> pb.cpp 从管道文件读取数据

```cpp
#include <iostream>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

int main(int argc, char* argv[]) {
    int fd = open(argv[1], O_RDONLY);

    if (fd == -1) {
        std::cerr << "文件打开失败~" << std::endl;
    }

    // 从 管道文件 中读取数据
    char buf[1 << 13] = { 0 };
    int len = read(fd, buf, 1 << 13);

    write(1, buf, len);

    close(fd);

    return 0;
}
```





## 信号

问题：信号是什么？

答案：管他是什么，只要知道我们可以给进程发信号，进程收到信号后会做一些事，信号什么时候传递到是不确定的。



问题：系统有哪些信号？

答案：`kill -l`可以查看。



问题：常见信号？

```
kill 2		ctrl + c
kill 3		ctrl + \
kill 9		杀死
kill 11		用户自己使用
kill 12		用户自己使用
kill 14		时钟信号
kill 17		子进程终止时给父进程发送的信号
```



问题：信号的生命周期？

答案：

```
1. 信号被生成，并被发送给系统内核
2. 系统内核存储信号，直到可以处理它
3. 一旦有空闲，内核会按照以下三种方式处理信号
	a) 忽略信号：什么也不做。注意SIGKILL和SIGSTOP信号不能被忽略
	b) 捕获信号：内核暂停收到信号的进程，并跳转到事先注册好的信号处理函数（通常由程序员制定该函数），函数执行完毕后，重新跳回捕获信号的地方。SIGKILL和SIGSTOP信号不能被捕获。
	c) 默认信号：不同信号通常由不同的默认操作，通常是终止收到信号的进程，也有些默认操作是对信号视而不见。
```



问题：如何向进程注册信号处理函数？

答案：





