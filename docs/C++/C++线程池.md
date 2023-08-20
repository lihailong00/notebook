```cpp
#ifndef THREADPOOL_H
#define THREADPOOL_H

#include <deque>
#include <pthread.h>
#include <iostream>
#include <cstring>
#include <unistd.h>

// 每次添加的线程数量
const int ADD_NUM = 2;

// 任务结构体
// 任务的实例就是函数
class Task {
public:
    // 每一个任务都是一个函数
    void (*function)(void* arg);
    // 函数的参数
    void* arg;
};

// 线程池结构体
class ThreadPool {
public:
    // 任务队列。（可以考虑维护一个环形队列）
    // deque可能会导致内存泄漏
    std::deque<Task*> taskQueue;

    int queueCapacity;

    // 管理者线程id
    pthread_t managerId;

    // 工作线程id
    std::deque<pthread_t> workerIds;

    // 最小线程数
    int minThreadNum;

    // 最大线程数
    int maxThreadNum;

    // 工作线程数
    int busyThreadNum;

    // 存活线程数
    int aliveThreadNum;

    // 需要杀死的线程数
    int exitThreadNum;

    // 锁整个线程池
    pthread_mutex_t mutexPool;

    // 给busyThreadNum添加一个锁
    pthread_mutex_t mutexBusy;

    // 是否销毁该线程池。0表示不销毁，1表示销毁
    int shutdown;

    // 判断线程池是否满了
    pthread_cond_t notFull;
    // 判断线程池是否空了
    pthread_cond_t notEmpty;
};

// 给线程池添加任务
void threadPoolAdd(ThreadPool* pool, void(*func)(void*), void* arg) {
    pthread_mutex_lock(&pool->mutexPool);
    while (pool->taskQueue.size() == pool->queueCapacity && !pool->shutdown) {
        // 任务队列已满，阻塞生产者线程
        pthread_cond_wait(&pool->notFull, &pool->mutexPool);
    }
    // 当阻塞解除时，如果线程池已关闭，则停止
    if (pool->shutdown) {
        pthread_mutex_unlock(&pool->mutexPool);
        return;
    }

    // 添加任务
    Task* task = new Task;
    task->function = func;
    task->arg = arg;
    pool->taskQueue.push_back(task);

    pthread_cond_signal(&pool->notEmpty);

    pthread_mutex_unlock(&pool->mutexPool);
}

// 获取线程池中工作的函数
int getBusyThreadNum(ThreadPool* pool) {
    pthread_mutex_lock(&pool->mutexBusy);
    int busyThreadNum = pool->busyThreadNum;
    pthread_mutex_unlock(&pool->mutexBusy);
    return busyThreadNum;
}

// 获取线程池中存在的线程
int getAliveThreadNum(ThreadPool* pool) {
    pthread_mutex_lock(&pool->mutexPool);
    int aliveThreadNum = pool->aliveThreadNum;
    pthread_mutex_unlock(&pool->mutexPool);
    return aliveThreadNum;
}

// 销毁线程池
// 返回0表示销毁成功，返回-1表示销毁失败
int destroyThreadPool(ThreadPool* pool) {
    if (pool == nullptr) {
        return -1;
    }

    // 关闭线程池
    pool->shutdown = 1;

    // 销毁管理者线程
    pthread_join(pool->managerId, nullptr);

    // 唤醒阻塞的消费者线程
    for (int i = 0; i < pool->aliveThreadNum; i++) {
        pthread_cond_signal(&pool->notEmpty);
        // 子线程中会自动销毁
    }

    delete pool;
    pool = nullptr;
    pthread_mutex_destroy(&pool->mutexPool);
    pthread_mutex_destroy(&pool->mutexBusy);
    pthread_cond_destroy(&pool->notEmpty);
    pthread_cond_destroy(&pool->notFull);
    return 0;
}

// 自定义线程退出
void* threadExit(ThreadPool* pool) {
    // 获取当前线程的线程标识符。
    pthread_t tid = pthread_self();
    for (int i = 0; i < pool->maxThreadNum; i++) {
        if (pool->workerIds[i] == tid) {
            pool->workerIds[i] = 0;
            std::cout << "退出线程，线程编号为：" << tid << std::endl;
            break;
        }
    }
    pthread_exit(nullptr);
}

void* worker(void* arg) {
    ThreadPool* pool = (ThreadPool*)arg;
    while (true) {
        pthread_mutex_lock(&pool->mutexPool);
        // 判断当前任务队列是否为空
        while (pool->taskQueue.empty() && !pool->shutdown) {
            // 阻塞工作线程
            // 当任务队列不为空时，唤醒此处
            pthread_cond_wait(&pool->notEmpty, &pool->mutexPool);

            // 判断是否需要销毁线程
            if (pool->exitThreadNum > 0) {
                pool->exitThreadNum--;
                pool->aliveThreadNum--;
                pthread_mutex_unlock(&pool->mutexPool);
                threadExit(pool);
            }
        }

        // 判断线程池是否被关闭
        if (pool->shutdown) {
            pthread_mutex_unlock(&pool->mutexPool);
            threadExit(pool);
        }

        // 从任务队列中取出任务
        // 可以考虑用环形队列优化
        Task task;
        task.function = pool->taskQueue[0]->function;
        task.arg = pool->taskQueue[0]->arg;
        pool->taskQueue.pop_front();

        // 解锁
        // 唤醒生产者的条件变量
        pthread_cond_signal(&pool->notFull);
        pthread_mutex_unlock(&pool->mutexPool);

        std::cout << "线程开始工作！" << std::endl;


        pthread_mutex_lock(&pool->mutexBusy);
        pool->busyThreadNum++;
        pthread_mutex_unlock(&pool->mutexBusy);

        // 调用函数
        task.function(task.arg);
        delete task.arg;
        task.arg = nullptr;

        std::cout << "线程结束工作！" << std::endl;

        pthread_mutex_lock(&pool->mutexBusy);
        pool->busyThreadNum--;
        pthread_mutex_unlock(&pool->mutexBusy);

    }
}

void* manager(void* arg) {
    ThreadPool* pool = (ThreadPool*)arg;

    while (!pool->shutdown) {
        // 按照某种频率调节线程池中的线程数量
        sleep(1);

        // 取出线程池中任务的数量和当前线程的数量
        pthread_mutex_lock(&pool->mutexPool);
        int queueSize = pool->taskQueue.size();
        int aliveThreadNum = pool->aliveThreadNum;
        pthread_mutex_unlock(&pool->mutexPool);

        // 取出正在工作的线程数量
        pthread_mutex_lock(&pool->mutexBusy);
        int busyThreadNum = pool->busyThreadNum;
        pthread_mutex_unlock(&pool->mutexBusy);

        // 添加线程
        // 存活线程的个数 < 当前任务的个数 && 存活的线程数 < 最大线程数
        if (aliveThreadNum < queueSize && aliveThreadNum < pool->maxThreadNum) {

            pthread_mutex_lock(&pool->mutexPool);

            // counter记录添加的线程个数
            int counter = 0;

            for (int i = 0; counter < ADD_NUM
                            && i < pool->maxThreadNum
                            && pool->aliveThreadNum < pool->maxThreadNum; i++) {
                if (pool->workerIds[i] == 0) {
                    pthread_create(&pool->workerIds[counter], nullptr, worker, pool);
                    counter++;

                    pool->aliveThreadNum++;
                }
            }

            pthread_mutex_unlock(&pool->mutexPool);
        }

        // 销毁线程
        // 自定义算法：线程存活数 > 工作的线程 * 2  && 线程存活数 > 最小线程数
        if (pool->aliveThreadNum > pool->busyThreadNum * 2
            && pool->aliveThreadNum > pool->minThreadNum) {

            pthread_mutex_lock(&pool->mutexPool);
            pool->exitThreadNum = ADD_NUM;

            pthread_mutex_unlock(&pool->mutexPool);

            // 让工作的线程“自杀”
            for (int i = 0; i < ADD_NUM; i++) {
                pthread_cond_signal(&pool->notEmpty);
            }
        }
    }
}


// 创建线程池
ThreadPool* createThreadPool(
        int minThreadNum,
        int maxThreadNum,
        int queueCapacity) {
    ThreadPool* pool = new ThreadPool;

    pool->workerIds.resize(maxThreadNum);

    pool->minThreadNum = minThreadNum;
    pool->maxThreadNum = maxThreadNum;
    pool->busyThreadNum = 0;
    pool->aliveThreadNum = minThreadNum;
    pool->exitThreadNum = 0;

    // 初始化锁和条件变量
    // 条件变量和锁通常一起使用
    if (pthread_mutex_init(&pool->mutexPool, nullptr) != 0 ||
        pthread_mutex_init(&pool->mutexBusy, nullptr) != 0 ||
        pthread_cond_init(&pool->notEmpty, nullptr) != 0 ||
        pthread_cond_init(&pool->notFull, nullptr) != 0) {
        std::cout << "初始化失败！" << std::endl;
    }

    // 任务队列
    pool->queueCapacity = queueCapacity;

    // 
    pool->shutdown = 0;

    // 创建线程
    // manager是函数， 后面的是manager函数的参数
    pthread_create(&pool->managerId, nullptr, manager, pool);

    for (int i = 0; i < minThreadNum; i++) {
        pthread_create(&pool->workerIds[i], nullptr, worker, pool);
    }

    return pool;
}



#endif // !THREADPOOL_H

/*
问题：条件变量是什么？
答案：https://www.cnblogs.com/harlanc/p/8596211.html

问题：互斥锁和条件变量一起使用的场景？
答案：

问题：简述该案例的生产者-消费者模型？

问题：调用pthread_cond_wait(&条件变量, &互斥锁)函数会发生什么？
答案：
    1. 线程阻塞在此处(A处)，并且释放该锁。
    2. 当其他地方(B处)调用pthread_cond_signal(&条件变量)函数并使用相同的条件变量时，会唤醒A处。
    3. A处会尝试获取锁，如果获取成功，则继续往下执行；否则一直阻塞。

问题：该项目的特点？
答案：
    1. 生产者产生一个任务后，唤醒消费者；消费者消耗一个任务后，唤醒生产者。
    
*/
```

