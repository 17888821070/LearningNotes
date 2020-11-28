# List

## 基本命令

```
// 将一个或多个值插入到列表的表头
LPUSH key value1 value2...
// 多个 value 则按从左到右的顺序依次插入
// key 不存在则创建一个空列表
// key 存在但不是列表类型则返回错误

// 插入表尾
RPUSH key value1 value2...

// 只有当 key 存在且为 list 时执行 push
// 不存在时什么都不做
LPUSHX key value
RPUSHX key value

// 弹出
LPOP key
RPOP key

// 弹出 src 最后一个元素，并返回给客户端
// 弹出的元素插入到 dst 列表，作为头元素
RPOPLPUSH src dst
// src 不存在则报错
// src = dst 则将表尾元素移动到表头

// 插入
LINSERT key BEFORE|AFTER pivot value
// 没有找到 pivot 返回 -1

// 设置值
LSET key index value
// 将列表 key 下标为 index 的元素的值设置为 value
// 当 key 不存在或 index 超出范围时返回错误

// 移除
LREM key count value
// 根据参数 count 的值，移除列表中与参数 value 相等的元素
// count > 0: 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count
// count < 0: 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值
// count = 0: 移除表中所有与 value 相等的值

// 修剪
LTRIM key start stop
// 只保留区间内的元素

// 获取链表长度
LLEN key
// 不存在返回 0

// 返回下标为 index 的元素
LINDEX key index
// index 以 0 为底
//-1 为最后一个元素

// 范围查询
LRANGE key start stop
```