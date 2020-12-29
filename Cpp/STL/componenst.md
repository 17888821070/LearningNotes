# STL 六大组件

## 容器 containers

各种数据结构，如 `vector`、`list`、`deque`、`set`、`map` 等，都是一种 class template


## 算法 algorithms

各种常用算法，如 `sort`、`search`、`copy`、`erase` 等，都是一种 function template


## 迭代器 iterators

泛型指针，扮演容器与算法之间的胶合剂，共有 5 中类型及其衍生变化

从实现角度看，迭代器将 `operator*`、`operator->`、`operator++`、`operator--` 等指针相关操作予以重载的 class template

所有容器都附带自己专属的迭代器


## 仿函数 functors

行为类似函数，一般函数指针可视为狭义的仿函数

从实现角度看，仿函数重载了 `operator()` 的 class 或 class template


## 配接器 adapters

用来修饰容器、仿函数、迭代器接口的东西


## 配置器 allocators

负责空间配置与管理

从实现角度看，配置器是一个实现了动态空间配置、空间管理、空间释放的 class template


## 组件间交互关系

容器通过配置器取得数据存储空间，算法通过迭代器存取容器内容，仿函数协助算法完成不同的策略变化，配接器可以修饰或套接仿函数





