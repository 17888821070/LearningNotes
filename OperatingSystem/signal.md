# 信号

每个信号都有一个编号和一个宏定义名称，所有的信号都由操作系统来发

[![BACGcR.png](https://s1.ax1x.com/2020/10/23/BACGcR.png)](https://imgchr.com/i/BACGcR)

## 分类

前 32 种信号为不可靠信号，后 32 种为可靠信号

- 可靠信号：实时信号，支持排队，信号不会丢失，发多少次就可以收到多少次

- 不可靠信号：非实时信号，不支持排队，信号可能会丢失，比如发送多次相同的信号，进程只能收到一次

## 产生方式

- 用户输入：比如在终端上按下组合键 ctrl + c，产生 SIGINT 信号

- 硬件异常：CPU 检测到内存非法访问等异常，通知内核生成相应信号，并发送给发生事件的进程

- 系统调用：`kill()`，`raise()`，`sigqueue()`，`alarm()`，`setitimer()`，`abort()` 等


1. ctrl + c 产生 SIGINT 信号

2. ctrl + - 产生 SIGQUIT 信号

## 注册和注销

### 注册

在进程 PCB 的 task_struct 结构体中有一个未决信号的成员变量 struct sigpending pending，每个信号在进程中注册都会把信号值加入到进程的未决信号集

- 每个信号在进程中注册都会把信号值加入到进程的未决信号集

- 实时信号发送给进程时，不管该信号是否在进程中注册过，都会再次注册，故信号不会丢失

- 非实时信号发送给进程时，如果该信息已经在进程中注册过，不会再次注册，故信号会丢失

### 注销

- 非实时信号：不可重复注册，最多只有一个 sigqueue 结构，当该结构被释放后把该信号从进程未决信号集中删除，则信号注销完毕

- 实时信号：可重复注册，可能存在多个 sigqueue 结构；当该信号的所有 sigqueue 处理完毕后，把该信号从进程未决信号集中删除，则信号注销完毕

## 处理方法

- 忽略此信号：大多数信号都可使用这种方式进行处理，但有两种信号却决不能被忽略，分别是：SIGKILL 和 SIGSTOP

- 直接执行进程对于该信号的默认动作 ：对大多数信号的系统默认动作是终止该进程

- 捕捉信号：执行自定义动作，需要通知内核在某种信号发生时，调用一个用户函数 handler，不能捕捉 SIGKILL 和 SIGSTOP 信号


### signal() 函数

不支持信号传递信息，主要用于非实时信号安装

```cpp
#include <signal.h>
typedef void( *sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
// 绑定失败返回 SIG_ERR

// 信号标号为signum


void handler(int sig) {
    printf("get a sig,num is %d\n",sig);
}
 
int main() {
    signal(2,handler);
    while(1) {
        sleep(1);
        printf("hello\n");
    }
    return 0;
}
```

### sigaction() 函数

支持信号传递信息，可用于所有信号安装

```cpp
/*
struct sigaction {
    void (*sa_handler)(int);  
    void (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t sa_mask;
    int sa_flags;
    void (*sa_restorer(void));
}
*/

int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact)
// signum：要操作的 signal 信号
// act：设置对 signal 信号的新处理方式
// oldact：原来对信号的处理方式
// 返回值：0 表示成功，-1 表示有错误发生
```