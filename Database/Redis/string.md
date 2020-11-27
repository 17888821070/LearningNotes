# String 类型

最常用的数据类型

除了记录实际数据，String 类型还需要额外的内存空间记录数据长度、空间使用等信息，这些信息也叫作元数据。当实际保存的数据较小时，元数据的空间开销就显得比较大了

## 二进制安全

String 类型提供的一个键对应一个值的数据的保存形式，而且可以保存二进制字节流

C 语言字符串底层是一个数组，每次创建的时候就创建一个 N+1 长度的字符，多的那个 1 就是为了保存 `\0`，但是有很多数据结构经常会穿插空字符在中间，比如图片，音频，视频，压缩文件的二进制数据

Redis 保存了字符串的长度，不利用空字符做判断

## 命令

```
// 使用 SET 和 GET 来设置和获取字符串值
SET key value

GET key
// 当 key 存在时，SET 命令会覆盖掉你上一次设置的值

// 批量设置
MSET key1 value1 key2 value2

// 批量获取
MGET key1 key2

// set if not exists
SETNX key

// 设置新的 value 并返回旧值
GETSET key newvalue
```

值可以是任何种类的字符串（包括二进制数据），只需要注意不要超过 512 MB 的最大限度

```
// 使用 EXISTS 和 DEL 关键字来查询是否存在和删除键值对
EXISTS key

DEL key
```

如果 value 是一个整数，还可以对它使用 INCR 命令进行原子性的自增操作，这意味着及时多个客户端对同一个 key 进行操作，也决不会导致竞争的情况

```
SET connections 10
INCR connections // 11
DEL connections
INCR connections // 1

INCRBY connections 100 // 101

DECR connections // 100
DECRBY connections // 90
```

设置过期时间

```
// 设置 key 5 秒后过期
EXPIRE key 5 

// 等价于 SET + SETEX
SETEX key 5 value

// 过期时间为毫秒
PSETEX KEY 200 value
```

字符串操作

```
// 获取字符串长度
STRLEN key

// 追加
APPEND key value
// 如果 key 存在则将 value 追加到末尾
// 如果 key 不存在则将 key 设置为 value

// 从偏移量开始， 用 value 覆写键 key 储存的字符串值
// 不存在的键 key 当作空白字符串处理
SETRANGE key offset value
// 确保字符串足够长以便将 value 设置到指定的偏移量上
// 如果键 key 原来储存的字符串长度比偏移量小
// 原字符和偏移量之间的空白将用零字节填充

// 字符串的索引范围由 start 和 end 两个偏移量决定
GETRANGE start end
// -1 表示最后一个字符
// -2 表示倒数第二个字符
```