# 信号量

POSIX 信号量是一个 `sem_t` 类型的变量

POSIX 有两种信号量的实现机制：无名信号量和命名信号量；无名信号量只可以在共享内存的情况下，比如实现进程中各个线程之间的互斥和同步；命名信号量通常用于不共享内存的情况下，比如进程间通信

根据信号量取值的不同，POSIX信号量还可以分为：二值信号量和计数信号量；二进制信号量的值只有 0 和 1，和互斥量类似，若资源被锁住则信号量的值为 0，若资源可用则信号量的值为 1；计数信号量的值在 0 到一个大于 1 的限制值之间，该计数表示可用的资源的个数

有名信号量和无名信号量的差异在于创建和销毁的形式上，其他工作方式一样

## 无名信号量操作

### init

`sem_init()` 用于创建无名信号量

该函数初始化由 sem 指向的信号对象，设置它的共享选项，并给它一个初始的整数值

```cpp
int sem_init(sem_t *sem, int pshared, unsigned int value);

/*
pshared 控制信号量的类型，如果其值为 0，就表示信号量是当前进程的局部信号量，否则信号量就可以在多个进程间共享

value 为 sem 的初始值

调用成功返回 0，失败返回 -1
*/
```

### destory

`sem_destory()` 用于对用完的信号量进行清理

```cpp
int sem_destroy(sem_t *sem);
```

### getvalue

`sem_getvalue()` 返回当前信号量的值

```cpp
int sem_getvalue(sem_t *restrict, int *restrict);

/*
当前信号量的值通过 restrict 参数返回

如果当前信号量已经上锁，那么返回值为 0 或为负数，其绝对值就是等待该信号量解锁的线程数


*/
```

## 命名信号量操作

### open

`sem_open()` 用于创建或打开一个命名信号量

```cpp
sem_t *sem_open(const char *name, int oflag);

sem_t *sem_open(const char *name, int oflag,mode_t mode, unsigned int value);

/*
name 是一个标识信号量的字符串

oflag 用来确定是创建信号量还是连接已有的信号量，可以为 0，O_CREAT 或 O_EXCL

0 表示打开一个已存在的信号量

O_CREAT 表示如果信号量不存在就创建一个信号量，如果存在则打开并返回，此时 mode 和 value 都需要指定

O_CREAT|O_EXCL 表示如果信号量存在则返回错误

mode 用于创建信号量时指定信号量的权限位

value 表示创建信号量时，信号量的初始值
*/
```

### close

`sem_close()` 用于关闭命名信号量

```cpp
int sem_close(sem_t *);
```

单个程序可以用 `sem_close()` 函数关闭命名信号量，但是这样做并不能将信号量从系统中删除，因为命名信号量在单个程序执行之外是具有持久性的

当进程调用 `_exit()`、`exit()`、`exec()`或从 `main()` 返回时，进程打开的命名信号量同样会被关闭

### unlink

`sem_unlink()` 用于在所有进程关闭了命名信号量之后，将信号量从系统中删除

```cpp
int sem_unlink(const char *name);
```

## 共同操作

### wait

`sem_wait()` 为信号量值减一操作，总共有三个函数

```cpp
int sem_wait(sem_t *sem);
/* 
若 sem 小于 0 ，则线程阻塞于信号量 sem ，直到 sem 大于 0，否则信号量值减 1
*/

int sem_trywait(sem_t *sem);
/*
不阻塞线程，如果 sem 小于 0，直接返回一个错误
*/

int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
/*
第二个参数表示阻塞时间
如果 sem 小于 0 ，则会阻塞，参数指定阻塞时间长度
如果指定的阻塞时间到了，但是 sem 仍然小于 0 ，则会返回一个错误 
*/

// 成功，返回 0 
// 出错，返回 -1
```

### post

`sem_post()` 为信号量值加一操作

```cpp
int sem_post(sem_t *sem);

// 成功，返回 0 
// 出错，返回 -1
```