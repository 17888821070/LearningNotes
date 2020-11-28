# 基本命令

```
// 检测 key 是否存在
EXISTS key

// 返回类型
TYPE key

// 改名
RENAME key newkey
// newkey 存在时覆盖旧值
RENAMENX key newkey
// 仅当 newkey 不存在时才改名

// 将 key 移动到 db 数据库
MOVE key db
// 如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key 
// 或者 key 不存在于当前数据库，则没有任何效果

// 删除
DEL key1 key2...

// 随机获取一个 key
RANDOMKEY

// 当前数据库 key 数量
DBSIZE

// 查找符合模式的所有 key
KEYS pattern

// 增量迭代遍历
SCAN cursor [MATCH pattern] [COUNT count]
// 每次执行都只会返回少量元素，不会阻塞服务器很久
// SCAN 用于迭代遍历当前数据库中的键
// SSCAN 用于迭代遍历集合键中的元素
// HSCAN 用于迭代遍历哈希键中的键值对
// ZSCAN 用于迭代遍历有序集合的元素
// 一个元素，它从遍历开始直到遍历结束期间都存在于被遍历的数据集当中
// 那么 SCAN 命令总会在某次迭代中将这个元素返回给用户
// 同一个元素可能会被返回多次
// 一个元素是在迭代过程中被添加到数据集的，又或者是在迭代过程中从数据集中被删除的
// 那么这个元素可能会被返回， 也可能不会

SORT key [BY pattern] [LIMIT offset count] [ASC|DESC]

// 清空
FLUSHDB
// 清空所有数据库数据
FLUSHALL

// 切换数据库
SELECT index

// 交换数据库
SWAPDB db1 db2

// 设置过期时间
EXPIRE key seconds
// 一个命令只是修改一个带生存时间的 key 的值而不是用一个新的 key 值来代替它
// 那么生存时间不会被改变
// 对一个已经带有生存时间的 key 执行 EXPIRE 命令，新指定的生存时间会取代旧的生存时间
PEXPIRE key milliseconds
// 过期时间单位为毫秒

// 移除过期时间
PERSIST

// 返回剩余时间
TTL key
PTTL key
// key 不存在时返回 -2
// key 存在但没有设置过期时间返回 -1
```

## 事务

```
// 标记一个事务块的开始
MULTI

// 执行所有事务块内的命令
EXEC

// 取消事务，放弃执行事务块内的所有命令
DISCARD

// 如果 key 处于 WATCH 命令的监视之下，且事务块中有和这个(或这些) key 相关的命令
// 那么 EXEC 命令只在 key 没有被其他命令所改动的情况下执行并生效，否则该事务被打断
WATCH key1 key2...

// 取消 WATCH 命令对所有 key 的监视
UNWATCH
```

## 持久化

```
// 执行一个同步保存操作
SAVE
// 以 RDB 文件的形式保存到硬盘
// 会阻塞所有客户端

// 后台异步保存当前数据库的数据到磁盘
BGSAVE

// 执行一个 AOF文件 重写操作
BGREWRITEAOF
// 重写操作只会在没有其他持久化工作在后台执行时被触发
```