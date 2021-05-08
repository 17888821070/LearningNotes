# 关联式容器

标准 STL 关联式容器分为 set（集合）和 map（映射表）两大类，以及其衍生体 multiset（多键集合）和 multimap（多键映射表），这些均以 RB-tree 完成

SGI STL 还提供了以 hash-table 为底层结构的 hash_set、hash_map、hash_multiset、hash_multimap，C++11 后被替换成 unordered_set、unordered_multiset、unordered_map、unordered_multimap

RB-tree 里的数据是经过排序的，它的增删查时间复杂度为 O(logn)，查找时使用的是二分查找
hash-table 比较占空间，但增删查时间复杂度为 O(1)，里面的数据是无序的 

## set

### 概述

`set` 的所有元素都会根据元素的键值自动被排序，`set` 的键值就是实值，`set` 不允许两个元素有相同键值

不能通过 `set` 的迭代器改变其元素值，因为 `set` 的元素值就是键值，关系到 `set` 的排列规则

默认采用递增排序，接受自定义的可调用对象

### 主要接口

接口名称|功能描述|时间复杂度
-|-|-
size()|容器现有元素数量|
insert(T val)|插入元素，新元素若已存在则插入失败，返回 pair<iterator, bool>|
erase(iterator ite)、erase(T val)、erase(iterator first, iterator last)|删除指定的元素|
clear()|清空容器|
emplace(T val)|插入新元素|
find(T val)|返回查找元素的指针|
count(T val)|返回查找元素的个数|
lower_bound(T val)|返回第一个大于等于指定元素的指针|
upper_bound(T val)|返回第一个大于指定元素的指针|
equal_range(T val)|返回等于指定元素的指针范围 pair<iterator first, iterator last>，[first, last)|


## map

### 概述

`map` 的所有元素会根据元素的键值自动排序，默认是递增排序，`map` 不允许两个元素拥有相同的键值

`map` 的所有元素都是 `pair`，同时拥有实值和键值，`pair` 的第一元素被视为键值，第二元素被视为实值

`map` 的迭代器不能改变元素键值，因为键值涉及到排序规则，但可以修改元素的实值

默认采用递增排序，接受自定义的可调用对象

### 主要接口

map 可以想数组一样使用 `[]` 和 `at()`，根据键值访问实值

接口名称|功能描述|时间复杂度
-|-|-
size()|容器现有元素数量|
insert(pair<T1, T2> val)|插入元素，新元素若已存在则插入失败，返回 pair<iterator, bool>|
erase(iterator ite)、erase(T1 key)、erase(iterator first, iterator last)|删除指定的元素|
clear()|清空容器|
emplace(pair<T1, T2> val)|插入新元素|
find(T1 val)|返回查找元素的指针|
count(T1 val)|返回查找元素的个数|
lower_bound(T1 val)|返回第一个大于等于指定元素的指针|
upper_bound(T1 val)|返回第一个大于指定元素的指针|
equal_range(T1 val)|返回等于指定元素的指针范围 pair<iterator first, iterator last>，[first, last)|

## multiset

### 概述

`multiset` 与 `set` 特性及用法相同，唯一差别在于它允许键值重复


## multimap

### 概述

`multimap` 与 `map` 特性及用法相同，唯一差别在于它允许键值重复


## unordered_set

### 概述

`unordered_set` 底层结构是 hash-table，`unordered_set`不会自动排序


### 主要接口

接口名称|功能描述|时间复杂度
-|-|-
size()|容器现有元素数量|
insert(T val)|插入元素，新元素若已存在则插入失败，返回 pair<iterator, bool>|
erase(iterator ite)、erase(T val)、erase(iterator first, iterator last)|删除指定的元素|
clear()|清空容器|
emplace(T val)|插入新元素|
find(T val)|返回查找元素的指针|
count(T val)|返回查找元素的个数|
equal_range(T val)|返回等于指定元素的指针范围 pair<iterator first, iterator last>，[first, last)|


## unordered_map

### 概述

`unordered_map` 底层结构是 hash-table，`unordered_map`不会自动排序


unordered_map 可以想数组一样使用 `[]` 和 `at()`，根据键值访问实值

接口名称|功能描述|时间复杂度
-|-|-
size()|容器现有元素数量|
insert(pair<T1, T2> val)|插入元素，新元素若已存在则插入失败，返回 pair<iterator, bool>|
erase(iterator ite)、erase(T1 key)、erase(iterator first, iterator last)|删除指定的元素|
clear()|清空容器|
emplace(pair<T1, T2> val)|插入新元素|
find(T1 val)|返回查找元素的指针，没有该元素返回 end() 指针|
count(T1 val)|返回查找元素的个数|
equal_range(pair<T1, T2> val)|返回等于指定元素的指针范围 pair<iterator first, iterator last>，[first, last)|
[T1 val]|如果查询不到 val，会增加容器元素数量一个|

## unordered_multiset

### 概述

`unordered_multiset` 底层结构是 hash-table，`unordered_multiset`不会自动排序，允许重复键值


## unordered_multimap

### 概述

`unordered_multimap` 底层结构是 hash-table，`unordered_multimap`不会自动排序，允许重复键值









