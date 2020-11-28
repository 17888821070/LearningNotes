# Bitmap

Bitmap 本身是用 String 类型作为底层数据结构实现的一种统计二值状态的数据类型

String 类型是会保存为二进制的字节数组，所以 Redis 就把字节数组的每个 bit 位利用起来，用来表示一个元素的二值状态，可以把 Bitmap 看作是一个 bit 数组

## 基本命令

```
// 对 key 所储存的字符串值，设置或清除指定偏移量上的位
SETBTI key offset value
// 当 key 不存在时，自动生成一个新的字符串值

GETBIT key offset
// 当 offset 比字符串值的长度大，或者 key 不存在时，返回 0

// 返回 1 的数量
GETCOUNT key [start] [end]
// key 不存在则为 0

// 返回第一个值为 bit 的位置
BITPOS key bit [start] [end]

// 对多个 key 做并、或、异或、非计算
BITOP AND|OR|XOR|NOT dst key1 key2...


```