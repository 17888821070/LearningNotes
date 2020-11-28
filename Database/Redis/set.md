# Set

## 基本命令

```
// 将元素添加进集合
SADD key member1 member2...

// 检测元素是否在集合中
SISMEMBER key member

// 返回随机元素
SRANDMEMBER key count
// count 表示返回元素个数

// 移除元素
SREM key member1 member2...

// 移除一个随机元素
SPOP key

// 将 src 元素 member 移动到 dst
SMOVE src dst member
// 如果 src 不存在或 member 不存在则不执行任何操作

// 获取集合元素数量
SCARD key

// 获取集合所有元素
SMEMBERS key

// 获取元素
SSCAN key cursor MATCH pattern count
// cursor 游标，0 代表从头开始遍历
// SSCAN 会返回一个游标，之后可以使用它延续之前的遍历
// pattern 模式匹配
// count 返回元素个数，默认 10

// 交集
SINTER key1 key2...
SINTERSTORE dst key1 key2...
// 将交集保存到 dst
// dst 存在则覆盖

// 并集
SUNION key1 key2...
SUNIONSTORE dst key1 key2

// 差集
SDIFF key1 key2...
SDIFFSTORE dst key1 key2...
```