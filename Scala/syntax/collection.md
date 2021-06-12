# 集合

Scala 集合分为可变的和不可变的集合

可变集合可以在适当的地方被更新或扩展

不可变集合类永远不会改变，但仍然可以模拟添加，移除或更新操作，这些操作将在每一种情况下都返回一个新的集合，同时使原来的集合不发生改变

## List

列表是不可变的，值一旦被定义了就不能改变

`Nil` 可以表示为一个空列表

`::` 可以构造列表

```scala
// 字符串列表
val site: List[String] = List("Runoob", "Google", "Baidu")
val site = "Runoob" :: ("Google" :: ("Baidu" :: Nil))

// 整型列表
val nums: List[Int] = List(1, 2, 3, 4)
val nums = 1 :: (2 :: (3 :: (4 :: Nil)))

// 空列表
val empty: List[Nothing] = List()
val empty = Nil

// 二维列表
val dim: List[List[Int]] =
   List(
      List(1, 0, 0),
      List(0, 1, 0),
      List(0, 0, 1)
   )
```

- `head()` 返回列表第一个元素

- `tail()` 返回一个列表，包含除了第一元素之外的其他元素

- `isEmpty()` 在列表为空时返回 `true`

可以使用 `:::` 运算符或 `List.:::()` 方法或 `List.concat()` 方法来连接两个或多个列表

```scala
object Test {
   def main(args: Array[String]) {
      val site1 = "Runoob" :: ("Google" :: ("Baidu" :: Nil))
      val site2 = "Facebook" :: ("Taobao" :: Nil)

      // 使用 ::: 运算符
      var fruit = site1 ::: site2
      println( "site1 ::: site2 : " + fruit )
      // Runoob, Google, Baidu, Facebook, Taobao
     
      // 使用 List.:::() 方法
      fruit = site1.:::(site2)
      println( "site1.:::(site2) : " + fruit )
      // Facebook, Taobao, Runoob, Google, Baidu

      // 使用 concat 方法
      fruit = List.concat(site1, site2)
      println( "List.concat(site1, site2) : " + fruit  )
      // Runoob, Google, Baidu, Facebook, Taobao
   }
}
```

`List.tabulate()` 方法通过给定的函数来创建列表

```scala
def tabulate[A](n: Int)(f: (Int) => A): LazyList[A]

def tabulate[A](n1: Int, n2: Int, n3: Int, n4: Int, n5: Int)(f: (Int, Int, Int, Int, Int) => A): LazyList[LazyList[LazyList[LazyList[LazyList[A]]]]]

object Test {
   def main(args: Array[String]) {
      // 通过给定的函数创建 5 个元素
      val squares = List.tabulate(6)(n => n * n)
      println( "一维 : " + squares  )

      // 创建二维列表
      val mul = List.tabulate( 4,5 )( _ * _ )      
      println( "多维 : " + mul  )
   }
}
```

```scala
def +:(elem: A): List[A]
// 为列表预添加元素
val x = List(1)
val y = 2 +: x

def ::(x: A): List[A]
// 在列表开头添加元素
val z = 2 :: x

def :+(elem: A): List[A]
// 复制添加元素后列表
val a = List(1)
val b = a :+ 2

def take(n: Int): List[A]
// 提取列表的前 n 个元素

def takeRight(n: Int): List[A]
// 提取列表的后 n 个元素

def toArray: Array[A]
// 列表转换为数组

def toMap[T, U]: Map[T, U]
// List 转换为 Map

def toBuffer[B >: A]: Buffer[B]
// 返回缓冲区，包含了列表的所有元素

def toMap[T, U]: Map[T, U]
// List 转换为 Map

def toSeq: Seq[A]
// List 转换为 Seq

def toSet[B >: A]: Set[B]
// List 转换为 Set

def toString(): String
// 列表转换为字符串
```

由于 `List` 是 immutable 集合，可以使用 `ListBuffer` 来生成链表

```scala
var l = ListBuffer[Int]
for(i <- 0 to 10) {
  l += i
}
val ll = l.toList
```

## Set

`Set` 是没有重复的对象集合，所有的元素都是唯一的

`Set` 分为可变的和不可变的集合

默认情况下，Scala 使用的是不可变集合，如果想使用可变集合，需要引用 `scala.collection.mutable.Set` 包，默认引用 `scala.collection.immutable.Set`

## Map

`Map` 有两种类型，可变与不可变，默认情况下 Scala 使用不可变 `Map`，如果需要使用可变集合，需要显式的引入 `import scala.collection.mutable.Map` 类

- `keys()` 返回 `Map` 所有的键

- `values()` 返回 `Map` 所有的值

- `isEmpty()` 在 `Map` 为空时返回 `true`

可以使用 `++` 运算符或 `Map.++()` 方法来连接两个 `Map`，`Map` 合并时会移除重复的 key

可以使用 `Map.contains` 方法来查看 `Map` 中是否存在指定的 Key

## Iterator

Scala `Iterator` 不是一个集合，它是一种用于访问集合的方法

迭代器的两个基本操作是 `next` 和 `hasNext`

`it.next()` 会返回迭代器的下一个元素，并且更新迭代器的状态

`it.hasNext()` 用于检测集合中是否还有元素

```scala
object Test {
   def main(args: Array[String]) {
      val it = Iterator("Baidu", "Google", "Runoob", "Taobao")
     
      while (it.hasNext){
         println(it.next())
      }
   }
}
```

可以使用 `++` 运算符或 `Set.++()` 方法来连接两个集合，如果元素有重复的就会移除重复的元素

## Option

`Option` 类型用来表示一个值是可选的，即有值或无值

`Option[T]` 是一个类型为 `T` 的可选值的容器：如果值存在，`Option[T]` 就是一个 `Some[T]`，如果不存在， `Option[T]` 就是对象 `None`

```scala
val myMap: Map[String, String] = Map("key1" -> "value")
val value1: Option[String] = myMap.get("key1")
val value2: Option[String] = myMap.get("key2")
 
println(value1) // Some("value1")
println(value2) // None
```

`Option` 有两个子类别，一个是 `Some`，一个是 `None`，当回传 `Some` 的时候，代表这个函式成功地返回一个返回值，如果返回的是 `None`，则代表没有返回值返回

```scala
object Test {
   def main(args: Array[String]) {
      val sites = Map("runoob" -> "www.runoob.com", "google" -> "www.google.com")
     
      println("show(sites.get( \"runoob\")) : " +  
                                          show(sites.get( "runoob")) )
      println("show(sites.get( \"baidu\")) : " +  
                                          show(sites.get( "baidu")) )
   }
   
   def show(x: Option[String]) = x match {
      case Some(s) => s
      case None => "?"
   }
}
```

可以使用 `getOrElse(val : T) : T` 方法来获取元组中存在的元素或者使用其默认的值

可以使用 `isEmpty() : Boolean` 方法来检测元组中的元素是否为 None

```scala
object Test {
   def main(args: Array[String]) {
      val a:Option[Int] = Some(5)
      val b:Option[Int] = None
     
      println("a.getOrElse(0): " + a.getOrElse(0) )
      println("b.getOrElse(10): " + b.getOrElse(10) )

      println("a.isEmpty: " + a.isEmpty )
      println("b.isEmpty: " + b.isEmpty )
   }
}
```

