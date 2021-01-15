# 日志系统

更新流程涉及两个重要的日志模块：redo log（重做日志）和 binlog（归档日志）

redo log 是 InnoDB 引擎特有的日志，Server 层的日志称为 binlog

## redo log

当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log，并更新内存，这个时候更新就算完成了

InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候做

InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB，那么这块“粉板”总共就可以记录 4GB 的操作，从头开始写，写到末尾就又回到开头循环写

[![stScqI.png](https://s3.ax1x.com/2021/01/13/stScqI.png)](https://imgchr.com/i/stScqI)

write pos 是当前记录的位置，一边写一边后移；checkpoint 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件

write pos 和 checkpoint 之间的部分是可以用来记录新的操作；如果 write pos 追上 checkpoint，这时候不能再执行新的更新，需要把把 checkpoint 推进一下

有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失

redo log 的写入拆成了两个步骤：prepare 和 commit，这就是两阶段提交，是为了让 redo log 和 binlog 之间的逻辑一致

由于 redo log 和 binlog 是两个独立的逻辑，如果不用两阶段提交，会出现一定问题

## binlog

与 redo log 区别：

1. redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用

2. redo log 是物理日志，记录的是在某个数据页上做了什么修改；binlog 是逻辑日志，记录的是这个语句的原始逻辑

3. redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的

对于 SQL 语句 `update T set c=c+1 where ID=2`：

1. 执行器先找引擎取 `ID=2` 这一行

2. 如果 `ID=2` 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回

3. 执行器拿到引擎给的行数据，把这个值加上 1，再调用引擎接口写入这行新数据

4. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，然后告知执行器执行完成，此时 redo log 处于 prepare 状态

5. 执行器生成这个操作的 binlog，并把 binlog 写入磁盘

6. 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成

 MySQL 以 binlog 的写入与否作为事务是否成功的标记

## 奔溃恢复规则

redo log 和 binlog 有一个共同的数据字段，叫 XID

崩溃恢复的时候，会按顺序扫描 redo log：

1. 如果碰到既有 prepare、又有 commit 的 redo log，就直接提交

2. 如果碰到只有 parepare、而没有 commit 的 redo log，就拿着 XID 去 binlog 找对应的事务；binlog 无记录，回滚事务；binlog 有记录，提交事务

先写 redo log 后写 binlog：假设在 redo log 写完，binlog 还没有写完的时候，MySQL 进程异常重启；由于 bin log 日志没有修改记录，使用 redo log + bin log 恢复的数据就是数据库旧的数据

先写 bin log 后写redo log：假设在 bin log 写完，redo log 还没有完成的时候，MySQL 进程异常重启；