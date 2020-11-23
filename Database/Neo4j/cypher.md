# Cypher 查询语句

## 字符画语法

Cypher 使用字符画来表示模式

- 节点由圆括号表示，看起来像是圆圈

- 关系用箭头来表示

- 关系相关的信息可以插入到方括号中

## 定义数据

- 节点通常有标签（一个或多个）

- 节点通常有属性，属性提供有关节点的额外信息

- 关系也可以有属性

- 关系通常有一个类型（类似于节点的标签）

## CREATE

```
// 创建单一节点
CREATE (a:Artist { name : "Jay" })
// 创建一个带有 Artist 标签的节点，节点有一个 name 属性，该属性的值是 Jay

// 创建多个节点
CREATE (a:Album { name: "aaaa"}), (b:Album { name: "bbbb"}) 
RETURN a,b
// 通过用逗号分隔来一次性创建多个节点

// 创建关系
MATCH (a:Artist),(b:Album)
WHERE a.name = "Jay" AND b.name = "aaaa"
CREATE (a)-[r:RELEASED]->(b)
RETURN r
// 通过用逗号分隔来一次性创建多个关系

// 创建索引
CREATE INDEX ON :Album(name)
// 为标签的某个属性创建索引

// 创建唯一约束
CREATE CONSTRAINT ON (a:Artist) ASSERT a.name IS UNIQUE
// 当创建一个唯一约束时，Neo4j 将同时创建一个索引
```

## MATCH

查询符合条件的数据，但实际上它并不返回数据，为了从 `MATCH` 语句返回数据，需要使用 `RETURN` 子句

```
// 检索节点
MATCH (p:Person)
WHERE p.name = "Jay"
RETURN p

MATCH (p:Person {name: "王太利"})
RETURN p

// 检索所有节点
MATCH (n) return n

// 检索关系
MATCH (a:Artist)-[:RELEASED]->(b:Album)
WHERE b.name = "aaaa" 
RETURN a
```

## LIMIT

使用 `LIMIT` 来限制输出记录的数量

```
MATCH (n) RETURN n 
LIMIT 5
```

## DELETE

在 `MATCH` 语句中使用 `DELETE` 子句来删除任何匹配的数据

在删除节点本身前，必须先删除和它相关的关系

```
// 删除单个节点
MATCH (a:Album {name: "Panmax"}) DELETE a;

// 删除多个节点
MATCH (a:Artist {name: "jiapan"}), (b:Album {name: "Panmax"}) 
DELETE a, b

// 删除所有节点
MATCH (n) DELETE n;
// 只能删除没有关系的节点

// 删除关系
MATCH ()-[r:RELEASED]-() 
DELETE r

MATCH (:Artist {name: "Jay"})-[r:RELEASED]-(:Album {name: "aaaa"}) 
DELETE r

// 删除带有关系的节点
MATCH (a:Artist {name: "Jay"}) DETACH DELETE a
// 允许删除一个节点的同时删除与其相连的所有关系

// 删除所有节点
MATCH (n) DETACH DELETE n
```

## DROP

```
// 删除索引
DROP INDEX ON :Album(name)

// 删除约束
DROP CONSTRAINT ON (a:Artist) ASSERT a.name IS UNIQUE
```