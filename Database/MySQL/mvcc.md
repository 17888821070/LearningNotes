# MVCC

MVCC 是一种多版本读写并发控制机制，是 InnoDB 存储引擎实现隔离级别的一种具体方式，用于实现 READ COMMIITTED 和 REPEATABLE READ 这两种隔离级别

MVCC 通过保存数据在某个时间点的快照来实现的；每行数据都存在一个版本，对数据库的任何修改的提交都不会直接覆盖之前的数据，而是产生一个新的版本与老版本共存，使得读取时可以不加锁（除非使用 `SELECT ... FOR UPDATE` 强行加锁）

- 通过 MVCC 可以让读写互相不阻塞，读不相互阻塞，写不阻塞读，这样可以提升数据并发处理能力

- 降低了死锁的概率，因为 MVCC 采用了乐观锁的方式，读取数据时，不需要加锁，写操作，只需要锁定必要的行

- 解决了一致性读的问题，MVCC 都是快照读，只能看到这个时间点之前事务提交更新的结果，不能看到时间点之后事务提交的更新结果

## 事务版本号

每次事务开启前都会从数据库获得一个自增长的事务 ID，可以从事务 ID 判断事务的执行先后顺序

## Undo log

Undo log 主要用于记录数据被修改之前的日志，在表信息修改之前先会把数据拷贝到 Undo log 里，当事务进行回滚时可以通过 Undo log 里的日志进行数据还原

- 保证事务进行 rollback 时的原子性和一致性，当事务进行回滚的时候可以用 Undo log 的数据进行恢复

- 通过读取 Undo log 的历史版本数据可以实现不同事务版本号都拥有自己独立的快照数据版本

存储数据表的 B+ 树节点总是只保留最新的数据，而老版本的数据被放在 Undo log 里，并且以指针的形式关联起来，形成一个链表；查询时会在 B+ 树查找后多引入一个链表查询，但是清理废弃数据时会比较简单，只要把 Undo log 找到一个合适的位置一刀切了即可

## 隐藏字段

MySQL 会为每一行真实数据记录添加两三个隐藏的字段，分别为 row_id、transaction_id 和 roll_pointer

- row_id：非必需隐藏字段；如果表中有自定义的主键或者有 Unique 键，就不会添加 row_id 字段，如果两者都没有，MySQL 会自动添加 row_id 字段

- transaction_id：必需隐藏字段；代表这一行数据由哪个事务 id 创建

- roll_pointer：必需隐藏字段；回滚指针，指向这行数据上一个版本在 Undo log 的地址

![wSwOxO.png](https://s1.ax1x.com/2020/09/02/wSwOxO.png)

对于事务 id，只有执行 insert/update/delete 才会产生事务 id，只执行 select 则没有事务 id

![wS0lWV.png](https://s1.ax1x.com/2020/09/02/wS0lWV.png)

## ReadView 

READ COMMITTED 和 REPEATABLE READ 这两个事务隔离级别都要保证读到的数据是其他事务已经提交的，但是还是有一定的区别，最核心的问题就在于到底可以读取这个数据的哪个版本

ReadView 机制只在 Read Committed 和 Repeatable Read 隔离级别下生效，所以只有这两种隔离级别才有 MVCC

### ReadView 组成

ReadView 包含四个比较重要的内容：

- m_ids：表示在生成 ReadView 时，系统中活跃的事务 id 集合

- min_trx_id：表示在生成 ReadView 时，系统中活跃的最小事务 id，也就是 m_ids 中的最小值

- max_trx_id：表示在生成 ReadView 时，系统应该分配给下一个事务的 id

- creator_trx_id：表示生成该 ReadView 的事务 id

### ReadView 用法

有了 ReadView，根据相关信息则可以判断版本信息：

- 如果被访问的版本的 trx_id 和 ReadView 中的 creator_trx_id 相同，就意味着当前版本就是由当前事务创建的，可以读出来

- 如果被访问的版本的 trx_id 小于 ReadView 中的 min_trx_id，表示生成该版本的事务在创建 ReadView 的时候已经提交了，所以该版本可以读出来

- 如果被访问版本的 trx_id 大于或等于 ReadView 中的 max_trx_id 值，说明生成该版本的事务在当前事务生成 ReadView 后才开启，所以该版本不可以被读出来

- 如果生成被访问版本的 trx_id 在 min_trx_id 和 max_trx_id 之间，那就需要判断下 trx_id 在不在 m_ids 中；如果在，说明创建 ReadView 的时候，生成该版本的事务还是活跃的（没有被提交），该版本不可以被读出来；如果不在，说明创建 ReadView 的时候，生成该版本的事务已经被提交了，该版本可以被读出来

- 如果某个数据的最新版本不可以被读出来，就顺着 roll_pointer 找到该数据的上一个版本，继续做如上的判断

### 更新逻辑

更新数据都是先读后写的，且只能读当前的值，称为当前读

除了 `update` 语句外，`select` 语句如果加锁，也是当前读

### READ COMMITTED

每次读取数据都会创建 ReadView

### REPEATABLE READ

首次读取数据会创建 ReadView

## 例子

```sql
CREATE TABLE `t` ( 
    `id` int(11) NOT NULL, 
    `k` int(11) DEFAULT NULL, 
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB;
    
insert into t(id, k) values(1,1),(2,2);
```

[![sRM2fU.png](https://s3.ax1x.com/2021/01/20/sRM2fU.png)](https://imgchr.com/i/sRM2fU)

### 事务启动时机

`begin/start transaction` 命令并不是一个事务的起点，在执行到它们之后的第一个操作 InnoDB 表的语句，事务才真正启动

如果想要马上启动一个事务，可以使用 `start transaction with consistent snapshot` 这个命令

### 结果

事务 C 没有显式地使用 `begin/commit`，表示这个 `update` 语句本身就是一个事务，语句完成的时候会自动提交

事务 B 在更新了行之后查询

事务 A 在一个只读事务中查询，并且时间顺序上是在事务 B 的查询之后

事务 B 查到的 k 的值是 3，而事务 A 查到的 k 的值是 1

### 分析

做如下假设：

- 事务 A 开始前，系统里面只有一个活跃事务 ID 是 99

- 事务 A、B、C 的版本号分别是 100、101、102，且当前系统里只有这四个事务

- 三个事务开始前，(1,1）这一行数据的 transaction_id 是 90

事务 A 的 m_ids 是[99,100]，事务 B 的 m_ids 是[99,100,101], 事务 C 的 m_ids 是[99,100,101,102]

[![sRQCAP.png](https://s3.ax1x.com/2021/01/20/sRQCAP.png)](https://imgchr.com/i/sRQCAP)

第一个有效更新是事务 C，把数据从 (1,1) 改成了 (1,2)。这时候这个数据的最新版本的 transaction_id 是 102，而 90 这个版本已经成为了历史版本

第二个有效更新是事务 B，把数据从 (1,2) 改成了 (1,3)。这时候这个数据的最新版本 transaction_id 是 101，而 102 又成为了历史版本

事务 A 读数据时，它的 m_ids 是 [99,100]，它的读数据流程：

1. 找到 (1,3) 的时候，判断出 transaction_id = 101，比 100 大，不可见

2. 接着，找到上一个历史版本，transaction_id = 102，比 100 大，不可见

3. 接着，找到上一个历史版本，一看 transaction_id = 90，比99小，可见

当事务 B 要去更新数据的时候，就不能再在历史版本上更新了，否则事务 C 的更新就丢失了，因此，事务 B 此时的 `set k=k+1` 是在（1,2）的基础上进行的操作

在更新的时候，当前读拿到的数据是 (1,2)，更新后生成了新版本的数据 (1,3)，这个新版本的 transaction_id 是 101；在执行事务 B 查询语句的时候，一看自己的版本号是 101 和最新数据的版本号相同，可以直接使用，所以查询得到的 k 的值是 3

如果把事务 A 的查询语句 select * from t where id=1 修改一下，加上 lock in share mode 或 for update，也都可以读到版本号是 101 的数据，返回的 k 的值是 3

### 拓展

假设事务 C 不是马上提交的，而是变成了下面的事务 C'

[![sRQw4K.png](https://s3.ax1x.com/2021/01/20/sRQw4K.png)](https://imgchr.com/i/sRQw4K)

事务 C' 的不同是，更新后并没有马上提交，在它提交前，事务 B 的更新语句先发起了

虽然事务 C' 还没提交，但是 (1,2) 这个版本也已经生成了，并且是当前的最新版本

事务 C' 没提交，也就是说 (1,2) 这个版本上的写锁还没释放。而事务 B 是当前读，必须要读最新版本，而且必须加锁，因此就被锁住了，必须等到事务 C' 释放这个锁，才能继续它的当前读

[![sRQD3D.png](https://s3.ax1x.com/2021/01/20/sRQD3D.png)](https://imgchr.com/i/sRQD3D)