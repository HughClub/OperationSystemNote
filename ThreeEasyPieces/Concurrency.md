# Concurrency

## Thread

>   经典观点是一个程序只有一个执行点（一个程序计数器，用来存放要执行的指令）
>
>   但多线程（multi-threaded）程序会有多个执行点（多个程序计数器，每个都用于取指令和执行）

多线程程序的地址空间布局上 有多个栈

### Atomic

>   作为一个不可分割的单元

同步原语(synchronization primitive):要求硬件提供一些有用的指令 可以在这些指令上构建一个通用的集合

### terminology

-   **临界区(critical section)**是访问共享资源的一段代码，资源通常是一个变量或数据结构
-   **竞态条件(race condition)**出现在多个执行线程大致同时进入临界区时，它们都试图更新共享的数据结构，导致了令人惊讶的（也许是不希望的）结果
-   **不确定性(indeterminate)**程序由一个或多个竞态条件组成，程序的输出因运行而异，具体取决于哪些线程在何时运行。这导致结果不是确定的(deterministic)，而我们通常期望计算机系统给出确定的结果。 
-   为了避免这些问题，线程应该使用某种**互斥(mutual exclusion)**原语。这样做可以保证只有一个线程进入临界区，从而避免出现竞态，并产生确定的程序输出。 

## lock

锁的评价

-   mutual exclusion:锁是否有效 能够阻止多个线程进入临界区
-   fairness:当锁可用时，是否每一个竞争线程有公平的机会抢到锁
-   performance:使用锁之后增加的时间开销



>   需要硬件支持

硬件原语

-   `test_and_set`:`exchange`
-   `compare_and_exchange`:exchange if equals expected



>   两阶段锁(two-phase lock)

-   第一阶段使用自旋锁:使用固定的自旋计数
-   第二阶段使用睡眠



### Pthreads

一些 POSIX API

`<pthread.h>`

编译参数 -pthread` or `-l pthread`

#### mutex

```C
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);

// init
// method one
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
// method two
pthread_mutex_init(&lock, NULL); // return 0 if succeed

// destroy
pthread_mutex_destroy();

int pthread_mutex_trylock(pthread_mutex_t *mutex);  
int pthread_mutex_timedlock(pthread_mutex_t *mutex, 
                           struct timespec *abs_timeout); 
```

#### condition variables

```C
/**
 * 假定互斥量已lock
 * unlock 互斥量 并让进程休眠(atomic)
 */
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
/**
 * 唤醒线程 lock 互斥量
*/
int pthread_cond_signal(pthread_cond_t *cond); 

// init
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;  
pthread_cond_t  cond = PTHREAD_COND_INITIALIZER; 
```

### Imp

一种分类

-   乐观锁:总认为资源和数据不会被别人所修改 所以读取不会上锁
    -   Version
    -   CAS
-   悲观锁:总认为最坏的情况可能会出现(Java)
    -   `Synchronized`
    -   `ReentrantLock`

[参考链接](https://mp.weixin.qq.com/s?__biz=MzkwMDE1MzkwNQ==&mid=2247496062&idx=1&sn=c04e0b83f38c45d06538ebac69529ee1&source=41#wechat_redirect)

#### Spinlock

自旋锁:未获取锁就一直循环等待(busy-wait)

![spinlock](https://img2018.cnblogs.com/blog/1515111/201910/1515111-20191015194619321-127153615.jpg)

避免了操作系统进程调度和线程切换

[简单的Java实现](https://www.cnblogs.com/cxuanBlog/p/11679883.html)

```java
public class SimpleSpinLock {

    private AtomicBoolean available = new AtomicBoolean(false);

    public void lock(){
        // 循环检测尝试获取锁
        while (!tryLock()){
            // doSomething...
        }
    }
    public boolean tryLock(){
        // 尝试获取锁，成功返回true，失败返回false
        return available.compareAndSet(false,true);
    }

    public void unLock(){
        if(!available.compareAndSet(true,false)){
            throw new RuntimeException("释放锁失败");
        }
    }
}
```

**无法保证多线程竞争的公平性**

## Condition

>   有效且简单的做法:在使用条件变量发送信号时持有锁

对于条件变量使用`while` 能解决假唤醒问题

### covering  condition

广播唤醒的条件变量

## Semaphore

```C
sem_init(sem_t,...);
sem_wait(); // P
sem_post(); // V
```

信号量的值

-   为正:还剩资源数
-   为负:等待资源数



实现mutex

`sem_init(&sem,0,1);`

## 并发问题

1.  死锁:四个条件
    1.  互斥
    2.  持有并等待
    3.  非抢占
    4.  循环等待
2.  非死锁
    1.  违反原子性(atomicity violation)缺陷
    2.  错误顺序(order violation)缺陷
    3.  可以通过**锁的添加**来实现修改

### 违反原子性

>   违反了多次内存访问中预期的可串行性
>
>   即代码段本意是原子的 但在执行中并没有强制实现原子性



### 违反顺序缺陷

>   两个内存访问的预期顺序被打破了
>
>   即 A 应该在 B 之前执行 但是实际运行中却不是这个顺序

### 死锁预防

#### 循环等待

最常用:让代码不会**循环等待**

全序或偏序获取锁



```C
// 通过锁地址强制锁的顺序
void do_something(mutex_t *m1, mutex_t *m2);

// examples
do_something(L1, L2); // thread 1
do_something(L2, L1); // thread 2

void do_something(mutex_t *m1, mutex_t *m2){
    // ...
    if (m1 > m2) {  // grab locks in high-to-low address order 
      pthread_mutex_lock(m1);
      pthread_mutex_lock(m2);
    } else {
      pthread_mutex_lock(m2);
      pthread_mutex_lock(m1);
    }
	// ...
}
```

#### 等待并持有

>   以通过原子地抢锁来避免

```C
lock(prevention); 
lock(L1); 
lock(L2); 
... 
unlock(prevention);
```

不适用于**封装**

#### 非抢占

>   `trylock`

#### 互斥

wait-free

>   像是CAS

```C
void AtomicIncrement(int *value, int amount) { 
	do { 
     int old = *value; 
	} while (CompareAndSwap(value, old, old + amount) == 0); 
}
```

### 死锁避免

1.  调度
2.  检查和恢复:数据库多见

## Event-based

>   关键问题：不用线程 如何构建并发服务器 

### Event loop

```C
while (1) { 
    events = getEvents();  
    for (e in events) 
        processEvent(e); 
}
```

重要API:`select/poll`

>   似乎这本书写的时候 还没有 `epoll`



>   用异步IO解决可能的阻塞系统调用

但这样执行回调也有一些麻烦 比如读写分离 需要使用到之前的fd进行写入

