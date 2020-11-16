# 多线程

对于操作系统来说，一个任务就是一个进程 （Process)，在一个进程内部，要同时干多件事，就需要同时运行多个“子任务”，我们把进程内的这些“子任务”称为线程 （Thread）

每个进程至少要干一件事，所以，一个进程至少有一个线程。多个线程可以同时执行，多线程的执行方式和多进程是一样的，也是由操作系统在多个线程之间快速切换，让每个线程都短暂地交替运行，看起来就像同时执行一样。当然，真正地同时执行多线程需要多核 CPU 才可能实现

多任务的实现有3种方式：

 - 多进程模式
 - 多线程模式
 - 多进程+多线程模式

同时执行多个任务通常各个任务之间并不是没有关联的，而是需要相互通信和协调。所以，多进程和多线程的程序的复杂度要远远高于单进程单线程的程序


## 多进程并发

独立的进程可以通过常规的进程间通信机制进行通信，如管道、信号、消息队列、共享内存、存储映射 I/O、信号量、套接字等

缺点：进程间通信较为复杂，速度相对线程间的通信更慢；启动进程的开销比线程大，使用的系统资源也更多

优点：进程间通信的机制相对于线程更加安全；能够很容易的将一台机器上的多进程程序部署在不同的机器上

## 多线程并发

线程很像轻量级的进程，但是一个进程中的所有线程都共享相同的地址空间，线程间的大部分数据都可以共享。线程间的通信一般都通过共享内存来实现

缺点：共享数据太过于灵活，为了维护正确的共享，代码写起来比较复杂；无法部署在分布式系统上

优点：由于可以共享数据，多线程间的通信开销比进程小的多；线程启动的比进程快，占用的资源更少

---
## 系统提供的C库线程

`#include <process.h>` 使用 VisualC++ 运行期库函数 `_beginthreadex()`，退出也应该使用`_endthreadex()`

```cpp
unsigned long _beginthreadex(void *security, unsigned stack_size, unsigned int(__stdcall *start_address)( void *),
void * arglist, unsigned initflag, unsigned * thrdaddr)

/*
void *security：线程的安全属性，如果为NULL则为默认安全属性
unsigned stack_size：用来指定线程堆栈的大小，如果为0，则线程堆栈小和创建他的线程的相同，一般用0
unsigned(__stdcall * start_address)(void*)：指定线程函数，也就是线程调用执行的函数地址
void *arglist：传递给线程的参数列表 如果多于一个的话，使用结构，然
然后传给结构的指针
unsigned initflag：线程初始状态，0:立即运行；CREATE_SUSPEND：suspended（悬挂）
unsigned *thrdaddr：用于记录线程ID的地址
*/

void _endthreadex(unsigned status)
//停止线程返回status中指定的代码
```

```cpp
#include <iostream>
#include <process.h>

using namespace std;

struct ParamStruct
{
	int count;
};

unsigned int _stdcall Fun(void *param)
{
	ParamStruct* theParam = (ParamStruct*) param;
	cout<<theParam->count<<endl;
	return 0;
}

int _tmain(int argc, _TCHAR* argv[])
{
	ParamStruct param;
	param.count = 0;

	while(1)
	{
		cout<<"this is a main function!"<<endl;
		_beginthreadex(NULL, 0, Fun, &param, 0, NULL);
		param.count += 1;
		if (param.count > 100)
		{
			break;
		}
	}
	return 0;
}
```

---

## C++11 多线程

标准库中提供了多线程库，使用时头文件：`#include <thread>` 类名：` std::thread`

```cpp
#include <iostream>
#include <thread>

void function_1() {
    std::cout << "I'm function_1()" << std::endl;
}

int main() {
    std::thread t1(function_1);
    // do other things
    t1.join();
    return 0;
}

/*
1. 构建一个 std::thread 对象 t1，构造的时候传递了一个参数，这个参数是一个函数，这个函数就是这个线程的入口函数，函数执行完了，整个线程也就执行完了

2. 线程创建成功后，就会立即启动，并没有一个类似 start() 的函数来显式的启动线程

3. 一旦线程开始运行，就需要在线程对象销毁之前显式的决定是要等待它完成 (join)，或者分离它让它自行运行 (detach)

4. Linux 环境下，pthread 不是 linux 下的默认的库，也就是在链接的时候，无法找到 phread 库中哥函数的入口地址，于是链接会失败，附加要加 -lpthread 参数即可解决
*/

```

线程对象和对象内部管理的线程的生命周期并不一样，如果线程执行的快，可能内部的线程已经结束了，但是线程对象还活着，也有可能线程对象已经被析构了，内部的线程还在运行

线程被分离之后，即使该线程对象被析构了，线程还是能够在后台运行，只是由于对象被析构了，主线程不能够通过对象名与这个线程进行通信

```cpp
void function_1() {
    // 延时 500ms 为了保证 test() 运行结束之后才打印
    std::this_thread::sleep_for(std::chrono::milliseconds(500)); 
    std::cout << "I'm function_1()" << std::endl;
}

void test() {
    std::thread t1(function_1);
    t1.detach();
    // t1.join();
    std::cout << "test() finished" << std::endl;
}

int main() {
    test();
    // 让主线程晚于子线程结束
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));  // 延时1s
    return 0;
}

// 使用 t1.detach()时
// test() finished
// I'm function_1()

// 使用 t1.join()时
// I'm function_1()
// test() finished

/*
1. 由于线程入口函数内部有个 500ms 的延时，所以在还没有打印的时候，test() 已经执行完成了，t1 已经被析构了，但是它负责的那个线程还是能够运行，这就是 detach() 的作用

2. 如果去掉 main 函数中的 1s 延时，会发现什么都没有打印，因为主线程执行的太快，整个程序已经结束了，那个后台线程被 C++ 运行时库回收了

3. 如果将 t1.detach() 换成 t1.join()，test() 会在 t1 线程执行结束之后，才会执行结束

join()：主线程会一直阻塞着，直到子线程完成，join() 函数的另一个任务是回收该线程中使用的资源

detach()：线程放在后台运行，所有权和控制权被转交给 C++ 运行时库，以确保与线程相关联的资源在线程退出后能被正确的回收；线程被分离之后，即使该线程对象被析构了，线程还是能够在后台运行，只是由于对象被析构了，主线程不能够通过对象名与这个线程进行通信

一旦一个线程被分离了，就不能够再被 join 了；可以使用 joinable() 函数判断一个线程对象能否调用join()
*/
```

### 启动线程

创建一个 `std::thread` 对象，就会启动一个线程，并使用该 `std::thread` 对象来管理该线程，其构造函数需要的是可调用（callable）类型，只要是有函数调用类型的实例都是可以的，可传入类型：函数、 lambda 表达式、重载了 `()` 运算符的类的实例

```cpp
// 1. 传入函数
void do_task();
std::thread(do_task);

// 2. lambda 表达式
for (int i = 0; i < 4; i++)
{
    thread t([i]{
        cout << i << endl;
    });
    t.detach();
}

// 3. 重载了()运算符的类的实例
class Task
{
public:
    void operator()(int i)
    {
        cout << i << endl;
    }
};

int main()
{
    
    for (uint8_t i = 0; i < 4; i++)
    {
        Task task;
        thread t(task, i);
        t.detach(); 
    }
}

// 4. 类中的方法
class Test
{
public:
    void Func(int a)
    {
        a = 1;
    }
}

int main()
{
    Test test;
    std::thread t(&Test::Func, &test, 10);
}

// thread 构造函数无法推导重载函数

/*
当线程启动后，一定要在和线程相关联的 thread 销毁前，确定以何种方式等待线程执行结束

detach 方式，启动的线程自主在后台运行，当前的代码继续往下执行，不等待新线程结束

join 方式，阻塞当前代码，等待启动的线程完成，才会继续往下执行

需要注意：创建新线程的作用域结束后，有可能线程仍然在执行，这时局部变量随着作用域的完成都已销毁，如果线程继续使用局部变量的引用或者指针，会出现意想不到的错误，并且这种错误很难排查
以 detach 的方式执行线程时，要将线程访问的局部数据复制到线程的空间（使用值传递），一定要确保线程没有使用局部变量的引用或者指针，除非你能肯定该线程会在局部作用域结束前执行结束
使用 join 方式的话就不会出现这种问题，它会在作用域结束前完成退出
*/
```

```cpp
/*
当决定以 detach 方式让线程在后台运行时，可以在创建 thread 的实例后立即调用 detach，这样线程就会和 thread 的实例分离，即使出现了异常，thread 的实例被销毁仍然能保证线程在后台运行

线程以 join 方式运行时，需要在主线程的合适位置调用 join 方法，如果调用 join 前出现了异常，thread 被销毁，线程就会被异常所终结

为了避免异常将线程终结，或者由于某些原因，例如线程访问了局部变量，就要保证线程一定要在函数退出前完成，就要保证要在函数退出前调用join
*/

void func() {
    thread t([]{
        cout << "hello C++ 11" << endl;
    });

    try
    {
        do_something_else();
    }
    catch (...)
    {
        t.join();
        throw;
    }
    t.join();
}
// 上面代码能够保证在正常或者异常的情况下，都会调用 join 方法，这样线程一定会在函数 func 退出前完成

// 一种比较好的方法是资源获取即初始化（RAII)，该方法提供一个类，在析构函数中调用 join
class thread_guard
{
    thread &t;
public :
    explicit thread_guard(thread& _t) :
        t(_t){}

    ~thread_guard()
    {
        if (t.joinable())
            t.join();
    }

    thread_guard(const thread_guard&) = delete;
    thread_guard& operator=(const thread_guard&) = delete;
};

void func(){

    thread t([]{
        cout << "Hello thread" <<endl ;
    });

    thread_guard g(t);
}
```

### 向线程传递参数

当需要向线程传递参数时，可以直接通过 `std::thread` 的构造函数依次传入参数即可

需要注意的是，默认的会将传递的参数以拷贝的方式复制到线程空间，即使参数的类型是引用

```cpp
void func(int a,const string str);
thread t(func,3,string("hello"));

class _tagNode
{
public:
    int a;
    int b;
};

void func(_tagNode &node)
{
    node.a = 10;
    node.b = 20;
}

void f()
{
    _tagNode node;

    thread t(func, std::ref(node));
    t.join();

    cout << node.a << endl ;
    cout << node.b << endl ;
}
// 在线程内，将对象的字段 a 和 b 设置为新的值，但是在线程调用结束后，这两个字段的值并不会改变
// 引用的实际上是局部变量 node 的一个拷贝，而不是 node 本身
// 在将对象传入线程的时候，调用std::ref，将 node 的引用传入线程，而不是一个拷贝 thread t(func,std::ref(node))
```

### 线程暂停

`std::thread` 没有直接提供暂停函数从外部让线程暂停，但可以使用 `std::this_thread::sleep_for` 和 `std::this_thread::sleep_until` 在线程内部停顿一会

```cpp
#include <thread>
#include <iostream>
#include <chrono>

using namespace std::chrono;

void pausable() {
    // sleep 500毫秒
    std::this_thread::sleep_for(milliseconds(500));
    // sleep 到指定时间点
    std::this_thread::sleep_until(system_clock::now() + milliseconds(500));
}

int main() {
    std::thread thread(pausable);
    thread.join();

    return 0;
}
```

### 线程停止

一般情况下当线程函数执行完成后，线程自然停止；但 `std::thread` 实例析构时如果线程还在运行，会造成线程异常终止，可能会造成资源的泄露，尽量在析构前 join 一下，以确保线程成功结束

### 转移线程所有权

thread 是可移动的 (movable) 的，但不可复制 (copyable)，可以通过 `move()` 来改变线程的所有权，灵活的决定线程在什么时候 join 或者 detach

```cpp
thread t1(f1);
thread t3(move(t1));
// 意味着 thread 可以作为函数的返回类型，或者作为参数传递给函数，能够更为方便的管理线程
```

`std::thread` 被设计为只能由一个实例来维护线程状态，以及对线程进行操作。因此当发生赋值操作时，会发生线程所有权转移
```cpp
std::thread a(foo);
std::thread b;
b = a;

// 原来由 a 管理的线程改为由 b 管理，b 不再指向任何线程(相当于执行了 detach 操作)
// 如果 b 原本指向了一个线程，那么这个线程会被终止掉

// 赋值函数
thread& operator=(thread&& _Other) noexcept
{	// move from _Other
    return (_Move_thread(_Other));
}
```

### 获取线程 ID

每个线程都有一个 id，但此处的 `get_id()` 与系统分配给线程的 ID 并不一是同一个东西

通过 thread 的实例调用 `get_id()` 直接获取

在当前线程上调用 `this_thread::get_id()` 获取

想取得系统分配的线程ID，可以调用 `native_handle()` 函数

### 让出时间片

`std::this_thread::yield()` 将当前线程所抢到的 CPU 时间片让给其他线程，等到其他线程使用完时间片后, 再由操作系统调度, 当前线程再和其他线程一起开始抢 CPU 时间片


## 互斥量

### mutex

用于提供对共享变量的互斥访问 `#include <mutex>`

名称|用途
:-:|:-:
std::mutex|最基本也是最常用的互斥类
std::recursive_mutex|允许同一个线程对互斥量多次上锁（即递归上锁），来获得对互斥量对象的多层所有权
std::timed_mutex|除具备 mutex 功能外，还提供了带时限请求锁定的功能
std::recursive_timed_mutex|同一线程可递归的 timed_mutex

与 `std::thread` 一样，`mutex` 相关类不支持拷贝构造、不支持赋值，同时 `mutex` 类也不支持 `move` 语义(move 构造、move 赋值)



### mutex 通用操作
`lock`、`try_lock`、`unlock` 四个 `mutex` 都支持这些操作，但是不同类在行为上有些微区别

- `lock` 锁住 `mutex`
- 如果互斥量没有被锁住，则调用线程将该 `mutex` 锁住，直到调用 `unlock` 之前，该线程一直拥有该锁

- 如果 `mutex` 已被其它线程 `lock`，则调用线程将被阻塞，直到其它线程 `unlock` 该 `mutex`

- 如果当前 `mutex` 已经被当前线程锁住，则 `std::mutex` 产生死锁，而 `recursive` 系列则成功返回


### `try_lock` 尝试锁住 `mutex`

- 如果互斥量没有被锁住，则调用线程将该 `mutex` 锁住（返回 `true`），直到该线程调用 `unlock` 释放

- 如果 `mutex` 已被其它线程 `lock`，则调用线程将失败，并返回 `false`，但不会被阻塞

- 如果当前 `mutex` 已经被当前线程锁住，则 `std::mutex` 死锁，而 `recursive` 系列则成功返回 `true`

### `unlock` 解锁 `mutex`

释放对 `mutex` 的所有权

对于 `recursive` 系列 `mutex`，`unlock` 次数需要与 `lock` 次数相同才可以完全解锁

```cpp
#include <iostream>
#include <thread>
#include <mutex>

void inc(std::mutex &mutex, int loop, int &counter) {
    for (int i = 0; i < loop; i++) {
        mutex.lock();
        ++counter;
        mutex.unlock();
    }
}
int main() {
    std::thread threads[5];
    std::mutex mutex;
    int counter = 0;

    for (std::thread &thr: threads) {
        thr = std::thread(inc, std::ref(mutex), 1000, std::ref(counter));
    }
    for (std::thread &thr: threads) {
        thr.join();
    }

    // 输出：5000，如果inc中调用的是try_lock，则此处可能会<5000
    std::cout << counter << std::endl;

    return 0;
}
```


### 仅用于 timed 系列操作
`try_lock_for`、`try_lock_until` 仅用于 `timed` 系列的 mutex（`std::timed_mutex`,`std::recursive_timed_mutex`），函数最多会等待指定的时间，如果仍未获得锁，则返回 `false`。除超时设定外，这两个函数与 `try_lock` 行为一致

```cpp
// 等待指定时长
template <class Rep, class Period>
    try_lock_for(const chrono::duration<Rep, Period>& rel_time);
// 等待到指定时间
template <class Clock, class Duration>
    try_lock_until(const chrono::time_point<Clock, Duration>& abs_time);
```

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

void run500ms(std::timed_mutex &mutex) {
    auto _500ms = std::chrono::milliseconds(500);
    if (mutex.try_lock_for(_500ms)) {
        std::cout << "获得了锁" << std::endl;
    } else {
        std::cout << "未获得锁" << std::endl;
    }
}
int main() {
    std::timed_mutex mutex;

    mutex.lock();
    std::thread thread(run500ms, std::ref(mutex));
    thread.join();
    mutex.unlock();

    return 0;
}
```


## 易用性类
在常规 `mutex` 的基础上，还提供了一些易用性的类

### lock_guard

`lock_guard` 利用了 C++ RAII 的特性，在构造函数中上锁，析构函数中解锁，从而保证了一个已锁的互斥量总是会被正确的解锁

```cpp
template <class Mutex> class lock_guard

/*
Mutex 代表互斥量，可以是 std::mutex、std::timed_mutex、std::recursive_mutex、std::recursive_timed_mutex 中的任何一个，也可以是 std::unique_lock，都提供了 lock 和 unlock 的能力

lock_guard 仅用于上锁、解锁，不对 mutex 承担供任何生周期的管理，因此在使用的时候，请确保 lock_guard 管理的 mutex 一直有效
*/

lock_guard(lock_guard const&) = delete;
lock_guard& operator=(lock_guard const&) = delete;
```

```cpp
std::mutex my_lock;

void add(int &num, int &sum)
{
    while(true)
    {
        std::lock_guard<std::mutex> lock(my_lock);  
        if (num < 100)
        { 
            //运行条件
            num += 1;
            sum += num;
        }   
        else 
        {  
            //退出条件
            break;
        }   
    }   
}

int main()
{
    int sum = 0;
    int num = 0;
    std::vector<std::thread> ver;   //保存线程的vector
    for(int i = 0; i < 20; ++i)
    {
        std::thread t = std::thread(add, std::ref(num), std::ref(sum));
        ver.emplace_back(std::move(t)); //保存线程
    }   

    std::for_each(ver.begin(), ver.end(), std::mem_fn(&std::thread::join)); //join
    std::cout << sum << std::endl;
}
```

### unique_lock

`lock_guard` 提供了简单上锁、解锁操作，但当我们需要更灵活的操作时便无能为力了

`unique_lock` 拥有对 `Mutex` 的所有权，一但初始化了 `unique_lock`，其就接管了该 `mutex`，在 `unique_lock` 结束生命周期前（析构前），其它地方就不要再直接使用该 `mutex` 了

```cpp
template <class Mutex>
class unique_lock
{
public:
    typedef Mutex mutex_type;
    // 空unique_lock对象
    unique_lock() noexcept;
    // 管理m, 并调用m.lock进行上锁，如果m已被其它线程锁定，由该构造了函数会阻塞。
    explicit unique_lock(mutex_type& m);
    // 仅管理m，构造函数中不对m上锁。可以在初始化后调用lock, try_lock, try_lock_xxx系列进行上锁。
    unique_lock(mutex_type& m, defer_lock_t) noexcept;
    // 管理m, 并调用m.try_lock，上锁不成功不会阻塞当前线程
    unique_lock(mutex_type& m, try_to_lock_t);
    // 管理m, 该函数假设m已经被当前线程锁定，不再尝试上锁。
    unique_lock(mutex_type& m, adopt_lock_t);
    // 管理m, 并调用m.try_lock_unitil函数进行加锁
    template <class Clock, class Duration>
        unique_lock(mutex_type& m, const chrono::time_point<Clock, Duration>& abs_time);
    // 管理m，并调用m.try_lock_for函数进行加锁
    template <class Rep, class Period>
        unique_lock(mutex_type& m, const chrono::duration<Rep, Period>& rel_time);
    // 析构，如果此前成功加锁(或通过adopt_lock_t进行构造)，并且对mutex拥有所有权，则解锁mutex
    ~unique_lock();

    // 禁止拷贝操作
    unique_lock(unique_lock const&) = delete;
    unique_lock& operator=(unique_lock const&) = delete;

    // 支持move语义
    unique_lock(unique_lock&& u) noexcept;
    unique_lock& operator=(unique_lock&& u) noexcept;

    void lock();
    bool try_lock();

    template <class Rep, class Period>
        bool try_lock_for(const chrono::duration<Rep, Period>& rel_time);
    template <class Clock, class Duration>
        bool try_lock_until(const chrono::time_point<Clock, Duration>& abs_time);

    // 显示式解锁，该函数调用后，除非再次调用lock系列函数进行上锁，否则析构中不再进行解锁
    void unlock();

    // 与另一个unique_lock交换所有权
    void swap(unique_lock& u) noexcept;
    // 返回当前管理的mutex对象的指针，并释放所有权
    mutex_type* release() noexcept;

    // 当前实例是否获得了锁
    bool owns_lock() const noexcept;
    // 同owns_lock
    explicit operator bool () const noexcept;
    // 返回mutex指针，便于开发人员进行更灵活的操作
    // 注意：此时mutex的所有权仍归unique_lock所有，因此不要对mutex进行加锁、解锁操作
    mutex_type* mutex() const noexcept;
};
```

```cpp
std::mutex mtx;           // mutex for critical section
std::once_flag flag;

void print_block (int n, char c)
 {
    //unique_lock有多组构造函数, 这里std::defer_lock不设置锁状态
    std::unique_lock<std::mutex> my_lock (mtx, std::defer_lock);
    //尝试加锁, 如果加锁成功则执行
    //(适合定时执行一个job的场景, 一个线程执行就可以, 可以用更新时间戳辅助)
    if(my_lock.try_lock())
    {
        for (int i=0; i<n; ++i)
            std::cout << c;
        std::cout << '\n';
    }
}

void run_one(int &n)
{
    std::call_once(flag, [&n]{n=n+1;}); //只执行一次, 适合延迟加载; 多线程static变量情况
}

int main () 
{
    std::vector<std::thread> ver;
    int num = 0;
    for (auto i = 0; i < 10; ++i)
    {
        ver.emplace_back(print_block,50,'*');
        ver.emplace_back(run_one, std::ref(num));
    }

    for (auto &t : ver)
    {
        t.join();
    }
    std::cout << num << std::endl;
    return 0;
}
```

### std::call_once

保证 `call_once` 调用的函数只被执行一次，需要与 `std::once_flag` 配合使用

`std::once_flag` 被设计为对外封闭的，即外部没有任何渠道可以改变 `once_flag` 的值，仅可以通过`std::call_once` 函数修改

```cpp
#include <iostream>
#include <thread>
#include <mutex>

void initialize() {
    std::cout << __FUNCTION__ << std::endl;
}

std::once_flag of;
void my_thread() {
    std::call_once(of, initialize);
}

int main() {
    std::thread threads[10];
    for (std::thread &thr: threads) {
        thr = std::thread(my_thread);
    }
    for (std::thread &thr: threads) {
        thr.join();
    }
    return 0;
}
// 仅输出一次：initialize
```

### std::try_lock

当有多个 mutex 需要执行 `try_lock` 时，该函数提供了简便的操作，`try_lock` 会按参数从左到右的顺序，对 mutex 顺次执行 `try_lock` 操作,当其中某个 `mutex.try_lock()` 失败(返回 `false` 或抛出异常)时，已成功锁定的 mutex 都将被解锁
该函数成功时返回-1， 否则返回失败 mutex 的索引，索引从 0 开始计数

```cpp
template <class L1, class L2, class... L3>
  int try_lock(L1&, L2&, L3&...);
```

### std::lock
批量上锁方式，采用死锁算法来锁定给定的mutex列表，避免死锁，上锁顺序是不确定的
该函数保证: 如果成功，则所有mutex全部上锁，如果失败，则全部解锁

```cpp
template <class L1, class L2, class... L3>
  void lock(L1&, L2&, L3&...);
```


## 条件变量

线程同步是指线程间需要按照预定的先后次序顺序进行的行为，C++11 对线程同步提供了条件变量 `#include <condition_variable>`

### std::condition_variable

条件变量提供了两类操作：`wait` 和 `notify`

- `wait` 是线程的等待动作，直到其它线程将其唤醒后，才会继续往下执行，只接受锁  `unique_lock<mutex>`
```cpp
// std::condition_variable 提供了两种 wait() 函数
 
void wait (unique_lock<mutex>& lck);

// 只有当 pred 条件为 false 时调用 wait() 才会阻塞当前线程
// 并且在收到其他线程的通知后只有当 pred 为 true 时才会被解除阻塞
template <class Predicate>
void wait (unique_lock<mutex>& lck, Predicate pred);


std::mutex mutex;
std::condition_variable cv;

// 条件变量与临界区有关，用来获取和释放一个锁，因此通常会和mutex联用
std::unique_lock lock(mutex);  // 获得锁

// 当前线程调用 wait() 后将被阻塞
cv.wait(lock);  // 调用该函数前，当前线程应该已经对unique_lock<mutex> lck完成了加锁
// 线程阻塞时，会释放lock，然后在cv上等待，直到其它线程通过cv.notify_xxx来唤醒当前线程
// 释放锁后使得其他被阻塞在锁竞争上的线程得以继续执行
// cv被唤醒后会再次对lock进行上锁，然后wait函数才会返回
// wait返回后可以安全的使用mutex保护的临界区内的数据，此时mutex仍为上锁状态
```

- `notify` 唤醒 `wait` 在该条件变量上的线程，有 `notify_one` 唤醒一个等待的线程（多个线程阻塞时唤醒某个不确定的线程）和 `notify_all` 唤醒所有等待的线程
```cpp
std::mutex mutex;
std::condition_variable cv;

std::unique_lock lock(mutex);
// 所有等待在cv变量上的线程都会被唤醒。但直到lock释放了mutex，被唤醒的线程才会从wait返回。
cv.notify_all(lock)
```

- `std::condition_variable_any` 的 `wait()` 函数可以接受任务 lockable 参数

- `std::notify_all_at_thread_exit` 当调用该函数的线程退出时，所有在 cond 条件变量上等待的线程都会收到通知
