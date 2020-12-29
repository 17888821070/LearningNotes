# 序列式容器

## vector

### 概述

`vector` 是动态空间，随着元素的加入，它的内部机制可自行扩充空间以容纳新元素

`vector` 的实现技术，关键在于其对大小的控制以及重新配置时数据移动效率`

### 迭代器

`vector` 维护的是一个连续线性空间，所以普通指针都可以作为 `vector` 的迭代器而满足所有必要条件

以两个迭代器 `start` 和 `finish` 分别指向配置得来的连续空间中目前已被使用的范围，并以迭代器 `end_of_storage` 指向整块连续空间（含备用空间）的尾端

```cpp
template <class T, class Alloc = alloc>
{
...
protected:
    iterator start;  // 目前使用空间的头
    iterator finish;  // 目前使用空间的尾，即未使用空间的头
    iterator end_of_storage;  // 可用空间的尾，不在 vector 的空间中
}
```

### 数据结构

`vector` 数据结构：线性连续空间

扩容时，如果原大小为 0，则配置 1 个元素大小；如果不是 0，则配置为原来大小的两倍，前半段用来放置原数据，后半段用来放置新数据

### 构造和内存管理

`vector` 缺省使用 `alloc` 作为空间配置器，并据此另外定义了一个 `data_allocator` 来更方便的以元素大小为配置单位

```cpp
template <class T, class Alloc = alloc>
{
...
protected:
    typedef simple_alloc<value_type, Alloc> data_allocator;
}
```

`data::allocator::allocate(n)` 表示配置 n 个元素空间

`vector` 提供多个构造函数，允许我们指定空间大小及所有元素初值

对于 `vector` 的任何操作，一旦引起空间重新配置，指向原 `vector` 的所有迭代器就都失效了

### 主要接口

接口名称|功能描述|时间复杂度
-|-|-
vector<T> v(n, i)|创建长度为 n，所有元素值为 i|
size()|容器现有元素数量|
capacity()|容器大小|
front()|获得第一个元素|
back()|获得最后一个元素|
push_back(T val)|从尾部添加元素|O(1)
pop_back(T val)|从尾部删除元素|O(1)
insert(iterator ite, T val)|在迭代器指定的位置插入元素|O(n)
erase(iterator ite)|删除迭代器指定位置的元素|O(n)
erase(iterator first, iterator second)|删除指定区间 [first, second) 的元素|O(n)
clear()|清空容器|O(n)
reserve(size_type n)|只开辟容器空间，不改变容器已有元素，不去构造元素|
resize(size_type n)|开辟容器空间，并且调用构造函数构造新的元素，改变容器元素数量|
assign(size_type n, T val)|清除所有元素，并从头插入新元素|
shrink_to_fit()|改变容器容量适应容器元素个数|

## list

### 概述

`list` 每次插入或删除一个元素就配置或释放一个元素空间，对空间的运用有绝对的精准

`list` 底层是一个双向链表

### 节点 node

```cpp
template <class T>
struct __list_node
{
    typedef void * void_pointer;
    void_pointer prev;  // 也可以是 __list_node<T> * 
    void_pointer next;
    T data;
}
```

### 迭代器

`list` 的节点不能保证在存储空间中连续，所以 `list` 迭代器必须有能力指向 `list` 的节点并能进行正确的递增、递减、取值、成员存取等操作；由于 `list` 是一个双向链表，所以迭代器必须具备前移、后移的能力

`list` 有一个重要性质：插入操作和结合操作都不会造成原有的 `list` 迭代器失效；删除操作也只有指向被删除的元素的迭代器失效，其他迭代器不受影响

### 数据结构

`list` 不仅是一个双向链表，而且还是一个环状双向链表，所以只需一个指针便可完整表现整个链表

```cpp
template <class T, class Alloc = alloc>
class list
{
protected:
    typedef __list_node<T> list_node;
public:
    typedef list_node * link_type;

protected:
    link_type node;
...
}
```

如果让指针 `node` 指向刻意置于尾端的一个空白节点，`node` 便符合 STL 对于“前闭后开”的区间要求

```cpp
iterator begin()
{
    return (link_type)((*node).next);
}

iterator end()
{
    return node;
}

bool empty()
{
    return node->next == node;
}
```

### 构造函数和内存管理

`list` 缺省使用 `alloc` 作为空间配置器，并据此定义了一个 `list_node_allocator` 从而更方便地以节点大小为配置单位

```cpp
template <class T, class Alloc = alloc>
class list
{
protected:
    typedef __list_node<T> list_node;
    typedef simple_alloc<list_node, Alloc> list_node_allocator;
...
}
```

于是 `list_node_allocator(n)` 表示配置 n 个节点空间

### 主要接口

接口名称|功能描述|时间复杂度
-|-|-
size()|容器现有元素数量|
front()|返回第一个元素的迭代器|
back()|返回最后一个元素的迭代器|
emplace_front(T val)、push_front(T val)|头部插入元素|O(1)
pop_front()|删除头部元素|O(1)
emplace_back(T val)、push_back(T val)|尾部插入元素|O(1)
pop_back()|删除头部元素|O(1)
insert(iterator ite, T val)|在迭代器指定的位置插入元素|O(1)
erase(iterator ite)|删除迭代器指定位置的元素|O(1)
erase(iterator first, iterator second)|删除指定区间 [first, second) 的元素|O(1)
clear()|清空容器|O(n)
resize(size_type n)|改变容器元素数量|
assign(size_type n, T val)|清除所有元素，并从头插入新元素|
splice(iterator ite, list& l)|将链表 l 拼接在 ite 指定位置，l 不再拥有节点|O(1)
splice(iterator ite, list& l, iterator ite1)|将链表 l 的 ite1 指向的元素拼接在 ite 指定位置，l 不再拥有节点|O(1)
splice(iterator ite, list& l, iterator first, iterator last)|将链表 l 的 [first, last) 拼接在指定位置，l 不再拥有节点|O(1)
remove(T val)|删除链表中所有值为 val 的节点|
remove_if(Predicate pred)|删除 pred(*ite) 返回 true 的所有元素|
unique()|删除链表中的值相同节点|
merge(list l)|将两个排序的链表合并，默认升序排列，l 不再拥有节点|
sort()|排序，默认升序|
sort(Compare comp)|按照指定规则排序|
reverse()|反转链表|


## deque

### 概述

`vector` 是单向开口的连续线性空间，`deque` 是双向开口的连续线性空间

- 允许常数时间内对其头端进行元素的插入或移除操作
- 没有容量观念，它是动态地以分段连续空间组合而成，随时可以增加一段新的空间并链接起来

### 主要接口

接口名称|功能描述|时间复杂度
-|-|-
size()|容器现有元素数量|
front()|返回队列顶部元素的引用|
back()|返回队列尾部元素的引用|
push_back(T val)|末尾插入新元素|
emplace_back(T val)|末尾插入新元素|
pop_back()|弹出尾部元素|
push_front(T val)|头部插入新元素|
emplace_front(T val)|头部插入新元素|
pop_front()|弹出顶部元素|

## stack

### 概述

`stack` 先进后出，允许新增元素、移除元素、取得最顶端元素

`stack` 不允许遍历行为，且没有迭代器

由于 `stack` 内部已其他容器完成所有工作，故 `stack` 不被归类为容器，而是配接器

`stack` 默认以 `deque` 作为底部结构，也可以指定以 `list` 为底层结构

### 主要接口

接口名称|功能描述|时间复杂度
-|-|-
size()|容器现有元素数量|
top()|返回栈顶元素的引用|
push(T val)|插入新元素|
emplace(T val)|插入新元素|
pop()|弹出顶部元素|

## queue

### 概述

`queue` 先进先出，有两个出口，允许新增元素、移除元素、从底端加入元素、取得顶端元素

`queue` 不允许遍历行为，且没有迭代器

`queue` 不是容器，是配接器

`queue` 默认以 `deque` 作为底部结构，也可以指定以 `list` 为底层结构


### 主要接口

接口名称|功能描述|时间复杂度
-|-|-
size()|容器现有元素数量|
front()|返回队列顶部元素的引用|
back()|返回队列尾部元素的引用|
push(T val)|插入新元素|
emplace(T val)|插入新元素|
pop()|弹出顶部元素|


## priority_queue

### 概述

`priority_queue` 是一个拥有权值观念的 `queue`，其内的元素自动依照元素的权值排列

由于是 `queue`，所以只允许在底端加入元素，并从顶端取出元素

默认情况下，`priority_queue` 利用 `max-heap` 完成，

`priority_queue` 不允许遍历行为，且没有迭代器

对于 `pair`，默认先比较第一个元素，第一个元素相等时比较第二个元素

### 主要接口

接口名称|功能描述|时间复杂度
-|-|-
template<class T,class Container = std::vector<T>,class Compare = std::less<typename Container::value_type>> class priority_queue|构造函数|
size()|容器现有元素数量|
top()|返回队列顶部元素的引用|
push(T val)|插入新元素|
emplace(T val)|插入新元素|
pop()|弹出顶部元素|

```cpp
// 小顶堆
priority_queue<int, vector<int>, greater<int>> pq;

function<bool(int, int)> cmp = [](int a, int b)->bool {
    // 小顶堆
    return a > b;  
    // 在数组中，a 在 b 的左边，返回 true 代表需要交换
};
priority_queue<int, vector<int>, decltype(cmp)> pq(cmp);
```

## slist 

`slist` 单向链表，提供单向迭代器 Forward Iterator


## heap

### 概述

在 STL 中 heap 以算法的形式提供，包括 `make_heap`，`push_heap`，`pop_heap`， `sort_heap`

```cpp
// make_heap() 创建堆，默认大顶堆
void make_heap( RandomIt first, RandomIt last );
void make_heap( RandomIt first, RandomIt last, Compare comp );

// push_heap() 添加元素
void push_heap( RandomIt first, RandomIt last );
void push_heap( RandomIt first, RandomIt last, Compare comp );

/*
 使用 push_heap(f, l)的话，调用者需要确保 [f, l-1) 已经是一个堆（相同排序规则）. push_heap(f, l) 仅仅会把 *(l-1) 插入到 [f, l-1) 这个区间形成的堆中
*/
vector<int> v1{6, 1, 2, 5, 3, 4};
make_heap(v1.begin(), v1.end());

v1.push_back(200);
// before push_heap: 6 5 4 1 3 2 200
push_heap(v1.begin(), v1.end());
// after push_heap: 200 5 6 1 3 2 4

// pop_heap() 弹出元素
void pop_heap( RandomIt first, RandomIt last );
void pop_heap( RandomIt first, RandomIt last, Compare comp );

/*
交换 *first 和 *(last-1)，然后把 [first, last-1) 建成一个 max heap. 也就是说把堆顶的最大元素交换到区间尾，然后把除了尾部的元素的剩余区间重新调整成堆
在调用 pop_heap 时 [first, last) 已经是一个堆(使用相同的排序准则)
*/
vector<int> v1{6, 1, 2, 5, 3, 4};
make_heap(v1.begin(), v1.end());

pop_heap(v1.begin(), v1.end());  // 5 3 4 1 2 6
auto largest = v1.back();  // 6
v1.pop_back();

// sort_heap() 排序
void sort_heap( RandomIt first, RandomIt last );
void sort_heap( RandomIt first, RandomIt last, Compare comp );

/*
区间需要已经是堆
使用 sort_heap(f, l) 处理过的区间因为已经有序，就不再是一个 heap 了，默认是升序
*/
vector<int> v1{6, 1, 2, 5, 3, 4};    
make_heap(v1.begin(), v1.end());  // 6 1 2 5 3 4
sort_heap(v1.begin(), v1.end());  // 1 2 3 4 5 6
```