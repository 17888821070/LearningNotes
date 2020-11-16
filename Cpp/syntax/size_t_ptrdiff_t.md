# size_t / ptrdiff_t

## size_t

`size_t` 是 `unsigned` 类型，用于指明数组长度或下标，它必须是一个正数

设计 `std::size_t` 就是为了适应多个平台，增强了不同平台上的可移植性

## ptrdiff_t

`ptrdiff_t` 是 `signed` 类型，用于存放同一数组中两个指针之间的差距，可以是负值

`std::ptrdiff_t` 可以得到独立于平台的地址差值

## size_type

`size_type` 是 `unsigned` 类型，表示容器中元素长度或者下标

## difference_type

`difference_type` 是 `signed` 类型,表示迭代器差距