# RedisObject

Redis 键值对中的每一个值都是用 RedisObject 保存的

## 基本对象结构

内部组成包括了 type,、encoding,、lru 和 refcount 4 个元数据，以及 1 个*ptr 指针

- type：表示值的类型，涵盖了五大基本类型

- encoding：值的编码方式，用来表示 Redis 中实现各个基本类型的底层数据结构，如 SDS、压缩列表、哈希表、跳表等

- lru：记录了这个对象最后一次被访问的时间，用于淘汰过期的键值对

- refcount：记录了对象的引用计数

- *ptr：是指向数据的指针

[![DshPk4.png](https://s3.ax1x.com/2020/11/28/DshPk4.png)](https://imgchr.com/i/DshPk4)

## 定义新数据

定义了新的数据类型，只要在 RedisObject 中设置好新类型的 type 和 encoding，再用 *ptr 指向新类型的实现

1. newtype.h 文件中定义好新类型

2. server.h 文件中增加一个宏定义，用来在代码中指代新类型

3. object.c 文件中增加创建函数和释放函数

4. 开发新类型的命令操作

