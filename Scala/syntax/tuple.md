# Tuple

`Tuple`  一个可以容纳不同类型元素的类

`Tuple` 是不可变的

```scala
val ingredient = ("Sugar" , 25):Tuple2[String, Int]

// 解析
val (name, quantity) = ingredient
```

Scala 中的包含一系列：`Tuple2`，`Tuple3`等，直到 `Tuple22`

使用下划线语法来访问元组中的元素，`Tuple._n` 取出了第 n 个元素