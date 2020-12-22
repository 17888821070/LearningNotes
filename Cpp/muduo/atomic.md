# Atomic

```
template<typename T>
class AtomicIntegerT : noncopyable {
public:
    AtomicIntegerT()
      : value_(0) {}

    T get() {
      // in gcc >= 4.7: __atomic_load_n(&value_, __ATOMIC_SEQ_CST)
      return __sync_val_compare_and_swap(&value_, 0, 0);
    }

    T getAndAdd(T x) {
      // in gcc >= 4.7: __atomic_fetch_add(&value_, x, __ATOMIC_SEQ_CST)
      return __sync_fetch_and_add(&value_, x);
    }

    T getAndSet(T newValue) {
      // in gcc >= 4.7: __atomic_exchange_n(&value_, newValue, __ATOMIC_SEQ_CST)
      return __sync_lock_test_and_set(&value_, newValue);
    }

    T addAndGet(T x) {
      return getAndAdd(x) + x;
    }

    T incrementAndGet() {
      return addAndGet(1);
    }

    T decrementAndGet() {
      return addAndGet(-1);
    }

    void add(T x) {
      getAndAdd(x);
    }

    void increment() {
      incrementAndGet();
    }

    void decrement() {
      decrementAndGet();
    }

private:
    volatile T value_;
};
```

gcc 从 4.1.2 提供了 __sync_* 系列的 built-in 函数，用于提供加减和逻辑运算的原子操作，因为是内置函数，所以使用的时候不需要 include 任何头文件

```c
type __sync_fetch_and_add (type *ptr, type value, ...)
type __sync_fetch_and_sub (type *ptr, type value, ...)
type __sync_fetch_and_or (type *ptr, type value, ...)
type __sync_fetch_and_and (type *ptr, type value, ...)
type __sync_fetch_and_xor (type *ptr, type value, ...)
type __sync_fetch_and_nand (type *ptr, type value, ...)
 
 
type __sync_add_and_fetch (type *ptr, type value, ...)
type __sync_sub_and_fetch (type *ptr, type value, ...)
type __sync_or_and_fetch (type *ptr, type value, ...)
type __sync_and_and_fetch (type *ptr, type value, ...)
type __sync_xor_and_fetch (type *ptr, type value, ...)
type __sync_nand_and_fetch (type *ptr, type value, ...)

// 提供原子的比较和交换，如果 *ptr == oldval，就将 newval 写入 *ptr
// 在相等并写入的情况下返回 true
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval, ...)
// 在返回操作之前的值
type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)

// 将 *ptr 设为 value 并返回 *ptr 操作之前的值
type __sync_lock_test_and_set (type *ptr, type value, ...)
// 将 *ptr 置 0
void __sync_lock_release (type *ptr, ...)

/*
type 可以是 1,2,4 或 8 字节长度的 int 类型
后面的可扩展参数 (...) 用来指出哪些变量需要 memory barrier
*/
```

gcc 4.7 之后，由于实现了 C++11，改写了原子操作接口

```cpp
type __atomic_load_n(type *ptr, int memorder);
void __atomic_store_n(type *ptr, type val, int memorder);
type __atomic_exchange_n(type *ptr, type val, int memorder);
bool __atomic_compare_exchange_n(type *ptr, type *expected, 
                type desired,bool weak, int success_memorder,
                int failure_memorder);

void __atomic_load(type *ptr, int memorder);
void __atomic_store(type *ptr, type val, int memorder);
void __atomic_exchange(type *ptr, type val, int memorder);
bool __atomic_compare_exchange(type *ptr, type *expected, 
                type desired,bool weak, int success_memorder,
                int failure_memorder);

type __atomic_add_fetch(type *ptr, type val, int memorder)
type __atomic_sub_fetch(type *ptr, type val, int memorder)
type __atomic_and_fetch(type *ptr, type val, int memorder)
type __atomic_xor_fetch(type *ptr, type val, int memorder)
type __atomic_or_fetch(type *ptr, type val, int memorder)
type __atomic_nand_fetch(type *ptr, type val, int memorder)

type __atomic_fetch_add(type *ptr, type val, int memorder)
type __atomic_fetch_sub(type *ptr, type val, int memorder)
type __atomic_fetch_and(type *ptr, type val, int memorder)
type __atomic_fetch_xor(type *ptr, type val, int memorder)
type __atomic_fetch_or(type *ptr, type val, int memorder)
type __atomic_fetch_nand(type *ptr, type val, int memorder)

bool __atomic_test_and_set (void *ptr, int memorder)
void __atomic_clear (bool *ptr, int memorder)
void __atomic_thread_fence (int memorder)

bool __atomic_always_lock_free (size_t size, void *ptr)
bool __atomic_is_lock_free (size_t size, void *ptr)
```