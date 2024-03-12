
#### 1、基础概念

&emsp;&emsp;由于线程共享进程的内存空间，因此任何线程对内存内数据的操作都可能对其他线程产生影响，因此多线程的同步与互斥机制是十分重要的。  
&emsp;&emsp;线程本身只占用少量的系统资源，其内存空间也只拥有堆栈区与线程控制块（Thread Control Block，简称TCB），因此对线程的调度需要的系统开销会小得多，能够更高效地提高任务的并发度。

#### 2、线程的资源

##### 一个进程中的多个线程共享以下资源： 
 
&emsp;&emsp;**静态数据（例如全局变量等）**  
&emsp;&emsp;打开文件的文件描述符  
&emsp;&emsp;信号处理函数  
&emsp;&emsp;执行的命令、当前工作目录、用户ID（UID）、用户组ID（GID） 
 
##### 每个线程私有的资源有：  

&emsp;&emsp;线程标识符（简称线程号，TID）、程序计数器（PC）与相关寄存器、执行状态与属性、错误号errno、信号掩码与优先级  
&emsp;&emsp;**堆栈区（局部变量、函数返回地址等）**

----------

## 线程的创建、控制与删除

&emsp;&emsp;使用NPTL线程库可以实现  
&emsp;&emsp;所需头文件：  

```cpp
#include<pthread.h>
pthread_create()//创建
pthread_exit()  //退出
pthread_join()  //阻塞等待线程退出
pthread_cancel()//一个线程杀死另外一个线程
```

#### 1、线程的创建

```cpp
int pthread_create(
        pthread_t *thread,      //线程的标识符（ID号）
        pthread_attr_t *attr,   //线程属性设置, NULL缺省
        void *(*routine)(void *),   //线程执行的函数
        void *arg                   //线程执行的函数的参数
        );//成功：返回0 失败：返回错误码

```
#### 2、线程退出函数（主动）
```cpp
void pthread_exit(
        void *retval//线程结束时的返回值,可以通过pthread_join()接收
        )；
```

#### 3、等待指定的线程退出

```cpp
int pthread_join(
        pthread_t thread, 
        void **thread_result//用户定义的指针，当不为NULL时用来接收等待线程结束时的返回值，即pthread_exit()函数内的retval值
        );//成功：返回0 失败：返回错误码
        

```
#### 4、删除指定线程      一个线程去结束另一个线程

```cpp
int pthread_cancel(
        pthread_t thread
        );
```
&emsp;&emsp;被取消的线程可以（使用pthread_setcancel()函数或pthread_setcanceltype()函数）设置自己的取消状态：  
&emsp;&emsp;被取消线程接收到另一个线程的取消请求后，是接受还是忽略？  
&emsp;&emsp;如果接受，是立即结束操作还是等待某个函数调用？

#### 5、获取当前线程标识符

```cpp
pthread_t pthread_self(void)//
```

----------

## 线程的同步与互斥。

### 互斥

通常情况下我们会定义一个pthread_mutex_t类型的全局变量来表示互斥锁。

```cpp
#include<pthread.h>
pthread_mutex_init();   //初始化互斥锁
pthread_mutex_lock();   //互斥锁上锁（阻塞）
pthread_mutex_trylock();//互斥锁判断上锁（非阻塞）
pthread_mutex_unlock(); //互斥锁解锁
pthread_mutex_destroy();//删除互斥锁
```

```cpp
int pthread_mutex_init(
        pthread_mutex_t *mutex,     //互斥锁类型指针
        const pthread_mutexattr_t *mutexattr//互斥锁的属性，NULL缺省
        );
```

```cpp
int pthread_mutex_lock()(pthread_mutex_t *mutex)    //上锁，若不成功则等待
int pthread_mutex_trylock()(pthread_mutex_t *mutex)    //上锁，若不成功则返回失败
int pthread_mutex_unlock()(pthread_mutex_t *mutex)    //解锁
int pthread_mutex_destroy()(pthread_mutex_t *mutex)    //删除互斥锁

```

### 同步——信号量semaphore

> * 只有1个信号量-----------实现互斥
有2个及以上的信号量-----实现同步
> * 信号量semaphore：全局变量

####1、PV操作

```cpp
P操作（申请资源）：Sem-1，（Sem可用资源的个数）
    若>=0，则本进程继续，
    若< 0，则阻塞
    
V操作（释放资源）：Sem+1，
    若> 0，则继续（表示资源s还有，即没有线程因为资源s被阻塞，所以不需用唤醒，继续运行）
    若<=0，则唤醒队列中一个因为P操作而阻塞的进程，然后继续本进程
```

#### 2、函数

```cpp
#include<semaphore.h>

1、初始化信号量
sem_t *sem;        //信号量对象
int sem_init(
        &sem, 
        0, 
        unisgned int value//信号量初始化的值，即资源个数
        )；
```
```cpp
//函数返回值：成功：0、失败：-1

int sem_wait(sem_t *sem)    //P操作，若不成功则等待
int sem_trywait(sem_t *sem)    //P操作，若不成功则返回错误，并不会阻塞线程
int sem_post(sem_t *sem)    //V操作，并释放一个阻塞的线程

int sem_getvalue(sem_t *sem)//获取信号量的值
int sem_destroy(sem_t *sem)    //删除信号量

```
