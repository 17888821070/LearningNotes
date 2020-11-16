# std::async

## std::promise

`std::promise` 是一个模板类 `template<typename R> class promise`，其泛型参数 R 为 `std::promise` 对象保存的值的类型，R 可以是 `void` 类型

作用是在一个线程 t1 中保存一个类型 `typename R` 的值，可供相绑定的`std::future` 对象在另一线程 t2 中获取

`std::promise` 允许 `move` 语义(右值构造，右值赋值)，但不允许拷贝(拷贝构造、赋值)

### 赋值

通过成员函数 `set_value` 可以设置 `std::promise` 中保存的值，该值最终会被与之关联的 `std::future::get` 读取到

`set_value` 只能被调用一次，多次调用会抛出 `std::future_error` 异常

事实上 `std::promise::set_xxx` 函数会改变 `std::promise` 的状态为 ready，再次调用时发现状态已要是 ready 了，则抛出异常

与 `std::promise` 关联的 `std::future` 是通过 `std::promise::get_future` 获取到的，自己构造出来的无效

一个 `std::promise` 实例只能与一个 `std::future` 关联共享状态，当在同一个 `std::promise` 上反复调用 `get_future` 会抛出 `future_error` 异常

```cpp
#include <iostream> 
#include <thread>   
#include <string>   
#include <future>   // std::promise, std::future
#include <chrono>   
using namespace std::chrono;

void read(std::future<std::string> *future)
{
    // future会一直阻塞，直到有值到来
    std::cout << future->get() << std::endl;
}

int main() 
{
    // promise 相当于生产者
    std::promise<std::string> promise;
    // future 相当于消费者, 右值构造
    std::future<std::string> future = promise.get_future();
    // 另一线程中通过future来读取promise的值
    std::thread thread(read, &future);
    // 让read等一会儿:)
    std::this_thread::sleep_for(seconds(1));
    // 
    promise.set_value("hello future");
    // 等待线程执行完成
    thread.join();

    return 0;
}
// hello future
```

如果 `std::promise` 直到销毁时都未设置过任何值，则 `std::promise` 会在析构时自动设置为 `std::future_error`，这会造成 `std::future.get` 抛出 `std::future_error` 异常

```cpp
void read(std::future<int> future)
{
    try 
    {
        future.get();
    } catch(std::future_error &e)
    {
        std::cerr << e.code() << "\n" << e.what() << std::endl;
    }
}

int main()
{
    std::thread thread;
    {
        // 如果promise不设置任何值
        // 则在promise析构时会自动设置为future_error
        // 这会造成future.get抛出该异常
        std::promise<int> promise;
        thread = std::thread(read, promise.get_future());
    }
    thread.join();

    return 0;
}
```

### 异常设置
通过 `std::promise::set_exception` 函数可以设置自定义异常，该异常最终会被传递到 `std::future`，并在其 `get` 函数中被抛出

```cpp
void catch_error(std::future<void> &future)
{
    try 
    {
        future.get();
    } catch (std::logic_error &e) 
    {
        std::cerr << "logic_error: " << e.what() << std::endl;
    }
}

int main() 
{
    std::promise<void> promise;
    std::future<void> future = promise.get_future();

    std::thread thread(catch_error, std::ref(future));
    // 自定义异常需要使用make_exception_ptr转换一下
    promise.set_exception(
        std::make_exception_ptr(std::logic_error("caught")));
    
    thread.join();
    return 0;
}
// 输出：logic_error: caught
```
`std::promise` 虽然支持自定义异常，但它并不直接接受异常对象，自定义异常可以通过 `std::make_exception_ptr` 函数转化为 `std::exception_ptr`

### `std::promise<void>`

`std::promise<void>` 是合法的。此时 `std::promise.set_value` 不接受任何参数，仅用于通知关联的 `std::future.get()` 解除阻塞

### 线程退出

`std::promise::set_value_at_thread_exit` 线程退出时，`std::future` 收到通过该函数设置的值

`std::promise::set_exception_at_thread_exit` 线程退出时，`std::future` 则抛出该函数指定的异常

---

## std::package_task

`std::packaged_task` 允许传入一个函数，并将函数计算的结果传递给 `std::future`，包括函数运行时产生的异常

`std::packaged_task` 支持 `move`，但不支持拷贝

`std::packaged_task` 封装的函数的计算结果会通过与之联系的 `std::future::get` 获取

`std::package_task` 除了可以通过可调用对象构造外，还支持缺省构造(无参构造)

```cpp
template <class R, class... ArgTypes>
class packaged_task<R(ArgTypes...)>
// R 是一个函数或可调用对象
// ArgTypes 是 R 的形参


#include <thread> 
#include <future>   // std::packaged_task, std::future
#include <iostream>

int sum(int a, int b) 
{
    return a + b;
}

int main() 
{
    std::packaged_task<int(int,int)> task(sum);
    std::future<int> future = task.get_future();

    // std::promise一样，std::packaged_task支持move，但不支持拷贝
    // std::thread的第一个参数不止是函数，还可以是一个可调用对象，即支持operator()(Args...)操作
    std::thread t(std::move(task), 1, 2);
    // 等待异步计算结果
    std::cout << "1 + 2 => " << future.get() << std::endl;

    t.join();
    return 0;
}
/// 输出: 1 + 2 => 3
```

### 有效状态

判断一个 `std::packaged_task` 是否可使用，可通过其成员函数 `valid` 来判断

当通过缺省构造初始化时，由于其未设置任何可调用对象或函数，`valid` 会返回 `false`

只有当 `std::packaged_task` 设置了有效的函数或可调用对象，`valid` 才返回 `true`

```cpp
int main() 
{
    std::packaged_task<void()> task; // 缺省构造、默认构造
    std::cout << std::boolalpha << task.valid() << std::endl; // false

    std::packaged_task<void()> task2(std::move(task)); // 右值构造
    std::cout << std::boolalpha << task.valid() << std::endl; // false

    task = std::packaged_task<void()>([](){});  // 右值赋值, 可调用对象
    std::cout << std::boolalpha << task.valid() << std::endl; // true

    return 0;
}
```

### `std::packaged_task::operator()(ArgTypes...)`

该函数会调用 `std::packaged_task` 对象所封装可调用对象 R，但其函数原型与 R 稍有不同

```cpp
void operator()(ArgTypes... )
```

`operator()` 的返回值是 `void`，即无返回值，因为 `std::packaged_task` 的设计主要是用来进行异步调用，因此 `R(ArgTypes...)` 的计算结果是通过 `std::future::get` 来获取的

```cpp
int main() 
{
    std::packaged_task<void()> convert([](){
        throw std::logic_error("will catch in future");
    });
    std::future<void> future = convert.get_future();

    convert(); // 异常不会在此处抛出

    try 
    {
        future.get();
    } catch(std::logic_error &e) 
    {
        std::cerr << typeid(e).name() << ": " << e.what() << std::endl;
    }

    return 0;
}
/// Clang下输出: St11logic_error: will catch in future
```

### 线程退出

`std::packaged_task::make_ready_at_thread_exit` 函数接收的参数与 `operator()(_ArgTypes...)` 一样，行为也一样，但不会将计算结果立刻反馈给 `std::future`，而是在其执行时所在的线程结束后，`std::future::get` 才会取得结果

### `std::packaged_task::reset`

`std::promise` 仅可以执行一次 `set_value` 或 `set_exception` 函数，但 `std::packagged_task` 可以执行多次，其奥秘就是 `reset` 函数

调用 `reset` 后，需要重新 `get_future` ，以便获取下次 `operator()` 执行的结果

由于是重新构造了 `promise` ，因此 `reset` 操作并不会影响之前调用的 `make_ready_at_thread_exit` 结果，也即之前的定制的行为在线程退出时仍会发生

---

## std::future

使用 `std::future` 的 `get` 方法来获取其它线程的结果

```cpp
enum class future_status
{
    ready,
    timeout,
    deferred
};

template <class R>
class future
{
public:
    // retrieving the value
    R get();
    // functions to check state
    bool valid() const noexcept;

    void wait() const;
    template <class Rep, class Period>
    future_status wait_for(const chrono::duration<Rep, Period>& rel_time) const;
    template <class Clock, class Duration>
    future_status wait_until(const chrono::time_point<Clock, Duration>& abs_time) const;

    shared_future<R> share() noexcept;
};
```

### `get()` 

该函数会一直阻塞，直到获取到结果或异步任务抛出异常

### `share()`

`std::future` 允许 `move`，但是不允许拷贝

`std::shared_future` 允许 `move`，允许拷贝，并且具有和 `std::future` 同样的成员函数

当调用 `share` 后，`std::future` 对象就不再和任何共享状态关联，其 `valid` 函数也会变为 `false`

### `wait()`

等待直到数据就绪。数据就绪时，通过 `get` 函数，无等待即可获得数据

### `wait_for` 和 `wait_until`

`wait_for` 、`wait_until` 主要是用来进行超时等待的

返回值有 3 种状态：
1. `ready` - 数据已就绪，可以通过get获取了
2. `meout` - 超时，数据还未准备好
3. `deferred` - 这个和 `std::async` 相关，表明无需 `wait`，异步函数将在 `get` 时执行

### `valid()`

判断当前 `std::future` 实例是否有效

`std::future` 主要是用来获取异步任务结果的，作为消费方出现，单独构建出来的实例没意义，因此其 `valid` 为 `false`

当与其它生产方(Provider)通过共享状态关联后，`valid` 才会变得有效， `std::future` 才会发挥实际的作用

C++11 中有下面几种 Provider，从这些 Provider 可获得有效的 `std::future` 实例：

1. `std::async`
2. `std::promise::get_future`
3. `std::packaged_task::get_future`

---

## 共享状态

共享状态其本质就是单生产者-单消费者的多线程并发模型

---

## std::async

`std::async` 用于直接创建异步任务，实际上就是创建一个线程执行相应任务，基本上可以代替 `std::thread` 的所有事情，异步任务返回的结果保存在 `std::future` 中

`std::async` 有两种重载形式

```cpp
#define FR typename result_of<typename decay<F>::type(typename decay<Args>::type...)>::type
// 用来推断函数 F 的返回值类型

// 不含执行策略
template <class F, class... Args>
future<FR> async(F&& f, Args&&... args);
// 含执行策略
template <class F, class... Args>
future<FR> async(launch policy, F&& f, Args&&... args);

// 执行策略定义了async执行F(函数或可调用求对象)的方式，是一个枚举值
enum class launch 
{
    // 保证异步行为，F将在单独的线程中执行
    // 在调用 async 时就开始创建线程
    async = 1,

    // 当其它线程调用std::future::get时，将调用非异步形式, 即F在get函数内执行
    // 延迟加载方式创建线程，直到调用 future 的 get 或 wait 时才创建线程
    deferred = 2,

    // F的执行时机由std::async来决定
    any = async | deferred
};

// 不含加载策略的版本，使用的是 std::launch::any
```

```cpp
#include <iostream>
#include <future>   // std::async, std::future
#include <chrono>   
using namespace std::chrono;

int main() 
{
    auto print = [](char c) 
    {
        for (int i = 0; i < 10; i++) 
        {
            std::cout << c;
            std::cout.flush();
            std::this_thread::sleep_for(milliseconds(1));
        }
    };

    // 不同launch策略的效果
    std::launch policies[] = {std::launch::async, std::launch::deferred};
    const char *names[] = {"async   ", "deferred"};
    for (int i = 0; i < sizeof(policies)/sizeof(policies[0]); i++) 
    {
        std::cout << names[i] << ": ";
        std::cout.flush();
        std::future<void> f1 = std::async(policies[i], print, '+');
        std::future<void> f2 = std::async(policies[i], print, '-');
        f1.get();
        f2.get();
        std::cout << std::endl;
    }
    return 0;
}

// async   : +-+-+-+--+-++-+--+-+
// deferred: ++++++++++----------
```









