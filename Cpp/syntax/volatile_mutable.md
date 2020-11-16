# volatile

## 定义
遇到 `volatile` 修饰的变量，编译器对访问该变量的代码就不进行优化，从而可以提供对特殊地址的稳定访问

当要求使用 `volatile` 声明的变量的值的时候，系统总是重新从它所在的内存读取数据，即使它前面的指令刚刚从该处读取过数据，并且读取的数据立刻被保存

`volatile` 表示变量是随时可能发生变化的，每次使用它的时候必须从变量地址中读取

`for(int i =0; i<100000;i++)` 这个语句是用来测试空循环的速度的，但是编译器肯定会把它优化掉，根本就不执行；但是如果写成 `for(volatile int i=0;i<100000;i++)` 则会执行了

```cpp
/*
p 就只是一个指向 int 类型的指针
a = *p 和 b = *p 两句就只需要从内存中读取一次就够了，因为从内存中读取一次之后，CPU 的寄存器中就已经有了这个值，把这个值直接复用就可以了
因此编译器就会做优化，把两次访存的操作优化成一次
但这样的前提是代码里没有改变 p 指向内存地址的值，这个值没有发生改变
*/
int *p = /* ... */;
int a, b;
a = *p;
b = *p;

// 每次都从地址取值，保证 a、b 数据的正确性
volatile int *p = /* ... */;
int a, b;
a = *p;
b = *p;
```

一个参数既可以既是 `const` 同时是 `volatile`

指针也可以被 `volatile` 修饰，与 `const` 相同

可以把一个非 `volatile int` 赋给 `volatile int`，但是不能把非 `volatile` 对象赋给一个 `volatile` 对象

数据只在内存中读写，直接操作它既不能保证操作是 atomic 的，也不能保证 Memory Order

## volatile 类

类是 `volatile` 则里面的成员都是 `volatile` 的

## volatile 成员函数

成员函数声明为 `volatile` 则同 `const` 一样在函数最后声明即可，表示这个函数里访问的变量可能由于其他原因发生改变

`volatile` 成员函数是线程安全的

## 多线程下的 volatile

```cpp
struct SharedDataStructure gSharedStructure; 
int gFlag;

//线程 A 
gSharedStructure.foo = ...; 
gSharedStructure.bar = ...; 
gSharedStructure.baz = ...; 
gFlag = 1; 
...

//线程 B
 if(gFlag) 
        UseSharedStructure(&gSharedStructure;); 
...
/*
结构体和 flag 之间因为不存在依赖关系，执行的顺序可能会被编译器修改(compiler reorder，指令重排)，线程 A 的代码可能是 flag 先被置为 1，然后处理结构体，这样线程 B 的 if 语句内就是不安全的

可以将 gFlag 和 gSharedStructure 都加上 volatile 关键字，保证编译器按照代码顺序产生机器码
*/
```

加了 `volatile` 关键字之后，上述代码在实际执行时仍不能保证安全，因为 CPU 会激进的 reorder 以加快执行速度， 因此内部执行机器码的顺序仍然是未知的，只保证最终结束时的结果与原顺序一致（as-if）

在这种情况下，如果线程 A 和 B 被分配到两个 CPU 执行，B 的 CPU 实际并不知道 A 的 CPU 执行情况，因此仍然存在 gFlag 被先修改而 gSharedStructure 未处理的问题

即使已经熟知 `volatile` 种种特性，实际仍不推荐使用，因为各种编译器的优化器也可能存在 bug

不要预期在多线程中使用 `volatile` 来解决数据的同步问题，该加锁时就加锁


# mutable

`mutable` 是为了突破 `const` 的限制而设置的，被 `mutable` 修饰的变量，将永远处于可变的状态，即使在一个 `const` 函数中