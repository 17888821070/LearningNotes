# 标准化算法

## 严格弱序

对于满足严格弱序的 `f(x, y) -> bool`，有三个条件：

```cpp
// 1. 非自反性
f(x, x) == false
// 2. 非对称性
f(x, y) != f(y, x)
// 3. 传递性
f(x, y) == true, f(y, z) == true  ==> f(x, z) == true
```

C++ 的操作符 `<`、`>` 是严格弱序的，`>=`、`<=` 不是

STL容器和算法需要用到比较器时，是假设了比较器是满足严格弱序的，如果违反则出现 core down

## sort

```cpp
sort 默认使用类或结构体的 < 进行比较，< 返回 true 时不交换位置

auto comp = [](int a, int b) {
    return a < b;
};

void sort( RandomIt first, RandomIt last, Compare comp );
/*
returns ​true if the first argument is less than the second

在容器中，a 在 b 的左边，返回 true 代表 a、b 不需要交换位置
*/
```