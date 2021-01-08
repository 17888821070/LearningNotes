# 标准化算法

## sort

```cpp
sort 默认使用类或结构体的 < 进行比较，< 返回 true时不交换位置

auto comp = [](int a, int b) {
    return a < b;
};

void sort( RandomIt first, RandomIt last, Compare comp );
/*
returns ​true if the first argument is less than the second

在容器中，a 在 b 的左边，返回 true 代表 a、b 不需要交换位置
*/
```