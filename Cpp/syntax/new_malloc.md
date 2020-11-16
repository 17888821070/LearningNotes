# 堆内存分配

|分配|释放|类型|能否重载|
|-|-|-|-|
`malloc()`|`free()`|C 函数|不能
`new`|`delete`|C++ 表达式|不能
`::operator new()`|`::operator delete()`|C++ 函数|能
`std::allocator<T>::allocate()`|`std::allocator<T>::deallocate()`|C++ 标准库|能

## new/delete

C++ 则提供了两个关键字 `new` 和 `delete`，需要编译器支持

使用 `new` 操作符申请内存分配时无须指定内存块的大小，编译器会根据类型信息自行计算

`new` 做两件事，一是分配内存，二是调用类的构造函数；`delete` 会调用类的析构函数和释放内存；

`new` 操作符内存分配成功时，返回的是对象类型的指针，类型严格与对象匹配，无须进行类型转换，故 `new` 是符合类型安全性的操作符

`new` 的底层其实就是 `malloc()`，`delete` 的底层其实就是 `free()`

## malloc/free

C 语言提供了 `malloc()` 和 `free()` 两个系统函数，完成对堆内存的申请和释放，`malloc()/free()` 是库函数，需要头文件支持

`malloc()` 需要显式地指出所需内存的尺寸

```cpp
    // 开辟单地址空间
    int *p = new int;  // 开辟大小为 sizeof(int) 空间
    int *q = new int(5);  // 开辟大小为 sizeof(int) 的空间，并初始化为 5
    
    // 开辟数组空间
    // 一维
    int *a = new int[100]{0};  // 开辟大小为 100 的整型数组空间，并初始化为 0
    // 二维
    int (*a)[6] = new int[5][6];
    // 三维
    int (*a)[5][6] = new int[3][5][6];
    // 四维及以上以此类推
    
    int (*p)[3] = (double(*)[3])malloc(sizeof(int) * 5 * 3);
```

`malloc` 和 `free` 只是分配和释放内存

`malloc` 内存分配成功则是返回 `void *`，需要通过强制类型转换将 `void*` 指针转换成我们需要的类型

## ::operator new/::operator delete

`::operator new` 和 `::operator delete` 底层也是 `malloc()` 和 `free()`

重载后可自定义该类的动态内存分配

```cpp
class A {
private:
    int a;

public:
    // 重载 ::operator new
    static void* operator new(size_t size) {

    }
    // 重载 ::operator new[]
    static void* operator new[](size_t size) {

    }

    // 重载 ::operator delete
    static void operator delete(void * pt) {

    }
    // 重载 ::operator delete[]
    static void operator delete[](void * pt) {

    }
}

```