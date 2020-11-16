# 预处理指令

预处理指令是在编译器进行编译之前进行的操作

预处理指令以符号 `#` 开头，且必须是该行除了任何空白字符外的第一个字符

预处理过程扫描源代码，对其进行初步的转换，产生新的源代码提供给编译器

## `#include`

包含一个源代码文件

## `#define/undef`

宏定义、取消宏定义

`#define` 存在一些不足，C++ 强调使用const来定义常量

预处理过程会把源代码中出现的宏标识符替换成宏定义时的值，但仅仅是进行标识符的替换

## `#if/elif/endif`

如果给定条件为真，则编译下面代码

如果前面的 `#if` 给定条件不为真，当前条件为真，则编译下面代码

结束一个 `#if……#else` 条件编译块

## `#ifdef/ifndef`

如果宏已经定义，则编译下面代码

如果宏没有定义，则编译下面代码

## `#error`

停止编译并显示错误信息


## `pragma pack(n)`

对齐的时候就会把元素的大小和 n 进行比较，取较小的那个来对齐

合法 n 的数值分别为 1、2、4、8、16

```cpp
struct test1 {
    int a;
    char b;
    short c;
    char d;
}
sizeof(test1) == 12

#pragma pack(2)
struct test2 {
    int a;
    char b;
    short c;
    char d;
}
#pragma pack() // 取消对齐
sizeof(test2) == 10

struct test2 {
    char a;
    int b;
    short c;
}
sizeof(test2) == 12

#pragma pack(2)
struct test3 {
    char a;
    int b;
    short c;
}
#pragma pack() // 取消对齐
sizeof(test3) == 8
```

`#pragma pack(show)` 显示当前内存对齐的字节数

单纯使用 `#pragma pack(push)` 会将当前的对齐字节数压入栈顶，并设置这个值为新的对齐字节数， 就是说不会改变这个值

而使用 `#pragma pack(push, n)` 会将当前的对齐字节数压入栈顶，并设置 n 为新的对齐字节数

单纯使用 `#pragma pack(pop)` 会弹出栈顶对齐字节数，并设置其为新的内存对齐字节数

使用 `#pragma pack(pop, n)` 情况就不同了，他会弹出栈顶并直接丢弃，设置 n 为其新的内存对齐字节数