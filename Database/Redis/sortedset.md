# SortedSet

## 基本命令

```
// 添加元素
ZADD key score1 member1 score2 member2...
// 如果 member 已存在则更新 score 并重新排序
// score 可以是整数和双精度浮点数

// 获取权重
ZSCORE key member
// 以字符串形式返回权重

// 获取递增排行
ZRANK key member
// 获取递减排行
ZREVRANK key member

// 获取数量
ZCARD key

// 获取区间个数
ZCOUNT key min max
// 闭区间

// 获取字典序区间个数
ZLEXCOUNT key min max

// 获取区间元素
ZRANGE key start stop [WITHSCORES]
// 返回成员按 score 递增排序
// 相同 score 按字典排序

// 获取字典序元素
ZRANGEBYLEX key min max [LIMIT offset count]
// 对元素使用 memcmp() 和 min/max 做比较

// 递减输出
ZREVRANGE key start stop [WITHSCORES]

// 获取区间元素
ZRANGEBYSCORE key min max [WITHSCORES][LIMIT offset count]
// min 可以是 -inf
// max 可以是 +inf
// 默认闭区间，可用 ( 表示开区间
ZREVRANGEBYSCORE key min max [WITHSCORES][LIMIT offset count]

// 权重操作
ZINCRBY key increment member
// member 的 score 加上增量 increment
// increment 可以是负数
// member 不存在时，权重默认为 0

// 移除
ZREM key member1 member2...
// member 不存在则忽略

// 按排行移除
ZREMRANGEBYRANK key start stop
// 按权重移除
ZREMRANGEBYSCORE key start stop
// 字典序移除
ZREMRANGEBYLEX key min max

// 使用游标获取元素
ZSCAN key cursor MATCH pattern count

// 并集
ZUNIONSTORE dst numkeys key1... [WEIGHTS weight...] [AGGREGATE SUM|MIN|MAX]
// 给定 key 的数量必须以 numkeys 参数指定
// 默认结果集中某个成员的 score 值是所有给定集下该成员 score 值之和
// WEIGHTS 为每个集合权重赋值惩罚因子，默认为 1 
// AGGREGATE 指定聚合方式

// 交集
ZINTERSTORE dst numkeys key1... [WEIGHTS weight...] [AGGREGATE SUM|MIN|MAX]
```