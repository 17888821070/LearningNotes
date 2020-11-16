# 原子操作

原子操作是不可打断的最低粒度操作，是线程安全的、更高效的线程同步手段；使用原子变量就不需要使用互斥量来保护数据，用起来更简洁

C++11 中所有的原子类都是不允许拷贝、不允许移动的

## `std::atomic_flag`

提供了标志的管理，标志有三种状态：clear、set 和未初始化状态
- clear：可以理解成 bool 类型的 false
- set：可理解为 bool 类型的 true

### 实例化

缺省情况下 `atomic_flag` 处于未初始化状态，除非初始化时使用了 `ATOMIC_FLAG_INIT` 宏，则此时 `atomic_flag` 处于 clear 状态

### `std::atomic_flag::clear()`

把 `atomic_flag` 置为 clear 状态

```cpp
void clear(memory_order m = memory_order_seq_cst) volatile noexcept;
void clear(memory_order m = memory_order_seq_cst) noexcept;
```

### `std::atomic_flag::test_and_set()`

检测 flag 是否处于 `set` 状态。如果不是，则将其设置为 `set` 状态，并返回 false；否则返回 true

保证多线程环境下只被设置一次

```cpp
bool test_and_set(std::memory_order order = std::memory_order_seq_cst) volatile noexcept;
bool test_and_set(std::memory_order order = std::memory_order_seq_cst) noexcept;
```

![](https://upload-images.jianshu.io/upload_images/6687014-40e5b28ef720dad9.png?imageMogr2/auto-orient/strip|imageView2/2/w/291/format/webp)

|`memory_order` 取值|意义|
|:-|:-:|
|`memory_order_relaxed`|宽松模型，不对执行顺序做保证|
|`memory_order_consume`|当前线程中，满足 happens-before 原则。当前线程中该原子的所有后续操作，必须在本条操作完成之后执行|
|`memory_order_acquire`|当前线程中，读操作满足 happens-before 原则。所有后续的读操作必须在本操作完成后执行|
|`memory_order_release`|当前线程中,写操作满足 happens-before 原则。所有后续的写操作必须在本操作完成后执行|
|`memory_order_acq_rel`|当前线程中，同时满足 memory_order_acquire 和memory_order_release|
|`memory_order_seq_cst`|最强约束。全部读写都按顺序执行|

```cpp
#include <atomic>    // atomic_flag
#include <iostream>  
#include <list>      
#include <thread>    

void race(std::atomic_flag &af, int id, int n) 
{
    for (int i = 0; i < n; i++) 
    {
    }
    // 第一个完成计数的打印：Win
    if (!af.test_and_set()) 
    {
        printf("%s[%d] win!!!\n", __FUNCTION__, id);
    }
}

int main() 
{
    std::atomic_flag af = ATOMIC_FLAG_INIT;

    std::list<std::thread> lstThread;
    for (int i = 0; i < 10; i++) 
    {
        lstThread.emplace_back(race, std::ref(af), i + 1, 5000 * 10000);
    }

    for (std::thread &thr : lstThread) 
    {
        thr.join();
    }

    return 0;
}
```

## `std::atomic<T>`

`std::atomic<T>` 定义了一些 `atomic` 应该具有的通用操作

### `std::atomic<T>` 构造函数

```
// 将原子对象放在未初始化的状态中。一个未初始化的原子对象可以通过调用 atomicinit 来初始化
atomic() noexcept = default;

// 用 desired 初始化对象，初始化不是原子性的
constexpr atomic( T desired ) noexcept;
```

### `operator =`

```cpp
T operator=( T desired ) noexcept; 
T operator=( T desired ) volatile noexcept;
```

用 val 替换存储的值，该操作是原子性的；与大多数赋值运算符不同，原子类型的赋值运算符不会返回对它们的左参数的引用，返回一个存储值的副本


### `std::atomic::is_lock_free()`

判断 `atomic<T>` 中的 T 对象是否为 `lock free`，若是返回 true

`lock free` (锁无关)指多个线程并发访问 T 不会出现数据竞争，任何线程在任何时刻都可以不受限制的访问 T，并不会导致线程阻塞

```cpp
bool is_lock_free() const noexcept;
bool is_lock_free() const volatile noexcept;
```

事实上，该函数可以做为一个静态函数，所有指定相同类型 T 的 `atomic` 实例的 `is_lock_free` 函数都会返回相同值

### `std::atomic::store()`

操作是原子的，按照同步所指定的内存顺序

```cpp
void store(T desr, memory_order m = memory_order_seq_cst) noexcept;
void store(T desr, memory_order m = memory_order_seq_cst) volatile noexcept;
```

### `std::atomic::load()`

读取，加载并返回变量的值，操作是原子的，按照同步所指定的内存顺序

```cpp
T load(memory_order m = memory_order_seq_cst) const volatile noexcept;
T load(memory_order m = memory_order_seq_cst) const noexcept;
```

### `std::atomic::exchange()`

交换，赋值后返回变量赋值前的值，整个操作是原子性的(一个原子的读-修改-写操作)

```cpp
T exchange(T desr, memory_order m = memory_order_seq_cst) volatile noexcept;
T exchange(T desr, memory_order m = memory_order_seq_cst) noexcept;
```

### `std::atomic::compare_exchange_weak()`

`compare_exchange_weak` 操作是原子的，排它的，其它线程如果想要读取或修改该原子对象时，会等待先该操作完成

```cpp
bool compare_exchange_weak(T& expect, T desr, memory_order s, memory_order f) volatile noexcept;
bool compare_exchange_weak(T& expect, T desr, memory_order s, memory_order f) noexcept;
bool compare_exchange_weak(T& expect, T desr, memory_order m = memory_order_seq_cst) volatile noexcept;
bool compare_exchange_weak(T& expect, T desr, memory_order m = memory_order_seq_cst) noexcept;
```

![](https://upload-images.jianshu.io/upload_images/6687014-7b0794e93556901f.png?imageMogr2/auto-orient/strip|imageView2/2/w/321/format/webp)

与 `compare_exchange_strong` 版本不同， `compare_exchange_strong` 版允许返回伪 false，即使原子对象所封装的值与 expect 的物理内容相同，也仍然返回 false

### `std::atomic::compare_exchange_strong()`

进行 compare 时，与 `std::atomic::compare_exchange_weak()` 一样，都是比较的物理内容

与 `std::atomic::compare_exchange_weak()` 不同的是，`std::atomic::compare_exchange_strong()` 不会返回伪 false，即：原子对象所封装的值如果与 expect 在物理内容上相同则一定会返回 true

```cpp
bool compare_exchange_strong(T& expect, T desr, memory_order s, memory_order f) volatile noexcept;
bool compare_exchange_strong(T& expect, T desr, memory_order s, memory_order f) noexcept;
bool compare_exchange_strong(T& expect, T desr, memory_order m = memory_order_seq_cst) volatile noexcept;
bool compare_exchange_strong(T& expc, T desr, memory_order m = memory_order_seq_cst) noexcept;
```

### `std::atomic::fetch_add()` 、`std::atomic::fetch_sub()`、`std::atomic::fetch_and()`、`std::atomic::fetch_or()`、`std::atomic::fetch_xor()`

将 arg 加或者减去到包含的值并返回在操作之前的值，整个操作是原子的(一个原子的读-修改-写操作)

```cpp
T fetch_add(T arg, std::memory_order order = std::memory_order_seq_cst);
T fetch_sub(T arg, std::memory_order order = std::memory_order_seq_cst);
T fetch_and(T arg, std::memory_order order = std::memory_order_seq_cst);
T fetch_or(T arg, std::memory_order order = std::memory_order_seq_cst);
T fetch_xor(T arg, std::memory_order order = std::memory_order_seq_cst);
```