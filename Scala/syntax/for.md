# for 循环

## 基本语法

```scala
for( var x <- Range ){
   statement(s);
}

// i to j (包含 j)
object Test {
   def main(args: Array[String]) {
      var a = 0;
      // for 循环
      for( a <- 1 to 10){
         println( "Value of a: " + a );
      }
   }
}

//  i until j (不包含 j)
object Test {
   def main(args: Array[String]) {
      var a = 0;
      // for 循环
      for( a <- 1 until 10){
         println( "Value of a: " + a );
      }
   }
}

// 在 for 循环 中可以使用分号 (;) 来设置多个区间
// 它将迭代给定区间所有的可能值
object Test {
   def main(args: Array[String]) {
      var a = 0;
      var b = 0;
      // for 循环
      for( a <- 1 to 3; b <- 1 to 3){
         println( "Value of a: " + a );
         println( "Value of b: " + b );
      }
   }
}
/*
Value of a: 1
Value of b: 1
Value of a: 1
Value of b: 2
Value of a: 1
Value of b: 3
Value of a: 2
Value of b: 1
Value of a: 2
Value of b: 2
Value of a: 2
Value of b: 3
Value of a: 3
Value of b: 1
Value of a: 3
Value of b: 2
Value of a: 3
Value of b: 3
*/

// 循环集合
object Test {
   def main(args: Array[String]) {
      var a = 0;
      val numList = List(1,2,3,4,5,6);

      // for 循环
      for( a <- numList ){
         println( "Value of a: " + a );
      }
   }
}

// 循环过滤
// 使用一个或多个 if 语句来过滤一些元素
// 可以使用分号(;)来为表达式添加一个或多个的过滤条件
for(var x <- List
    if condition1; if condition2...){
        statement(s);
}

object Test {
   def main(args: Array[String]) {
      var a = 0;
      val numList = List(1,2,3,4,5,6,7,8,9,10);

      // for 循环
      for( a <- numList
           if a != 3; if a < 8 ){
         println( "Value of a: " + a );
      }
   }
}
```

## yield

将 `for` 循环的返回值作为一个变量存储

```scala
var retVal = for( var x <- List
    if condition1; if condition2...) yield x;
// 大括号中用于保存变量和条件，retVal 是变量
// 循环中的 yield 会把当前的元素记下来，保存在集合中
// 循环结束后将返回该集合

object Test {
    def main(args: Array[String]) {
       var a = 0;
       val numList = List(1,2,3,4,5,6,7,8,9,10);

       // for 循环
       var retVal = for{ a <- numList
                         if a != 3; if a < 8
                    }yield a;

      // 输出返回值
       for( a <- retVal){
            println( "Value of a: " + a );
        }
    }
}

/*
1. {} 和 () 对于 for 表达式来说都可以

2. 当 for 推导式仅包含单一表达式时使用圆括号，当其包含多个表达式时使用大括号

3. 当使用 {} 来换行写表达式时，分号就不用写了
*/
```