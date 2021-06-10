# 函数与方法

Scala 方法是类的一部分

Scala 中函数是一个对象可以赋值给一个变量，函数其实就是继承了 `Trait` 的类的对象

Scala 中使用 `val` 语句可以定义函数，`def` 语句定义方法

## 函数

```scala
// 函数声明
val FunctionName = ([参数列表]) => [return type] {}

class Test{
  def m(x: Int) = x + 3
  val f = (x: Int) => x + 3
}
```

## 方法

```scala
// 方法声明
def MethodName ([参数列表]) : [return type] [=] {}
/*
如果不写等于号和方法主体，那么方法会被隐式声明为 abstract，包含它的类型于是也是一个抽象类型

如果方法没有返回值，可以返回为 Unit

方法的返回值类型可以不写，编译器可以自动推断出来

对于递归函数，必须指定返回类型
*/

def addInt(a : Int, b : Int) : Int = {
  val res = a + b
  return  res
}

def addInt(a : Int, b : Int) ：Int = a + b
```

函数可作为一个参数传入到方法中

```scala
object MethodAndFunctionDemo {
  //定义一个方法
  //方法 m1 参数要求是一个函数，`函数的参数必须是两个 Int 类型
  //返回值类型也是 Int 类型
  def m1(f:(Int,Int) => Int) : Int = {
    f(2,6)
  }

  //定义一个函数 f1,参数是两个 Int 类型，返回值是一个 Int 类型
  val f1 = (x:Int,y:Int) => x + y
  //再定义一个函数 f2
  val f2 = (m:Int,n:Int) => m * n

  //main 方法
  def main(args: Array[String]): Unit = {
    //调用 m1 方法，并传入 f1 函数
    val r1 = m1(f1)

    println(r1)

    //调用 m1 方法，并传入 f2 函数
    val r2 = m1(f2)
    println(r2)
  }
}
```

Scala 中无法直接操作方法，如果要操作方法，必须先将其转换成函数

```scala
/* 
方法一
在方法名称后面紧跟一个空格和下划线告诉编译器将方法 m 转换成函数，而不是要调用这个方法
*/
val f1 = m _

/* 
方法二
显示地告诉编译器需要将方法转换成函数
val f1: (Int) => Int = m
*/

object TestMap {

  def ttt(f:Int => Int):Unit = {
    val r = f(10)
    println(r)
  }

  val f0 = (x : Int) => x * x

  //定义了一个方法
  def m0(x:Int) : Int = {
    //传递进来的参数乘以 10
    x * 10
  }

  //将方法转换成函数，利用了神奇的下滑线
  val f1 = m0 _

  def main(args: Array[String]): Unit = {
    ttt(f0)

    //通过 m0 _ 将方法转化成函数
    ttt(m0 _);

    //如果直接传递的是方法名称，scala 相当于是把方法转成了函数
    ttt(m0)

    //通过 x => m0(x) 的方式将方法转化成函数,这个函数是一个匿名函数，等价：(x:Int) => m0(x)
    ttt(x => m0(x))
  }
}
```

## 传值调用和传名调用

- 传值调用（call-by-value）：先计算参数表达式的值，再应用到函数内部

- 传名调用（call-by-name）：将未计算的参数表达式直接应用到函数内部

在进入函数内部前，传值调用方式就已经将参数表达式的值计算完毕，而传名调用是在函数内部进行参数表达式的值计算的

每次使用传名调用时，解释器都会计算一次表达式的值

```scala
object Test {
   def main(args: Array[String]) {
        delayed(time());
   }

   def time() = {
      println("获取时间，单位为纳秒")
      System.nanoTime
   }

   // 传名调用，使用 => 符号来设置
   def delayed( t: => Long ) = {
      println("在 delayed 方法内")
      println("参数： " + t)
      t
   }
}
```

## 指定函数参数名调用

调用函数时通过指定函数参数名，这样就不需要按照顺序向函数传递参数

```scala
object Test {
   def main(args: Array[String]) {
        printInt(b=5, a=7);
   }
   def printInt( a:Int, b:Int ) = {
      println("Value of a : " + a );
      println("Value of b : " + b );
   }
}
```

## 可变参数

允许指明函数的最后一个参数可以是重复的，即不需要指定函数参数的个数，可以向函数传入可变长度参数列表

通过在参数的类型之后放一个星号来设置可变参数

```scala
object Test {
   def main(args: Array[String]) {
        printStrings("Runoob", "Scala", "Python");
   }
   def printStrings( args:String* ) = {
      var i : Int = 0;
      for( arg <- args ){
         println("Arg value[" + i + "] = " + arg );
         i = i + 1;
      }
   }
}
```

## 递归

Scala 支持递归函数

## 默认参数值

Scala 可以为函数参数指定默认参数值，使用了默认参数，你在调用函数的过程中可以不需要传递参数，这时函数就会调用它的默认参数值，如果传递了参数，则传递值会取代默认值

一个函数包含默认参数，同时包含普通参数时，无默认值的参数在前，默认参数在后

```scala
object Test {
   def main(args: Array[String]) {
        println( "返回值 : " + addInt() );
   }
   def addInt( a:Int=5, b:Int=7 ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
```

## 高阶函数

高阶函数就是操作其他函数的函数，可以使用其他函数作为参数，或者使用函数作为输出结果

## 函数嵌套

可以在 Scala 函数内定义函数，定义在函数内的函数称之为局部函数

```scala
object Test {
   def main(args: Array[String]) {
      println( factorial(0) )
      println( factorial(1) )
      println( factorial(2) )
      println( factorial(3) )
   }

   def factorial(i: Int): Int = {
      def fact(i: Int, accumulator: Int): Int = {
         if (i <= 1)
            accumulator
         else
            fact(i - 1, i * accumulator)
      }
      fact(i, 1)
   }
}
```

## 匿名函数

匿名函数中，箭头左边是参数列表，右边是函数体

## 偏应用函数

偏应用函数是一种表达式，不需要提供函数需要的所有参数，只需要提供部分，或不提供所需参数

```scala
object Test {
   def main(args: Array[String]) {
      val date = new Date
      log(date, "message1" )
      Thread.sleep(1000)
      log(date, "message2" )
      Thread.sleep(1000)
      log(date, "message3" )
   }

   def log(date: Date, message: String)  = {
     println(date + "----" + message)
   }
}
/*
log() 方法接收两个参数：date 和 message
在程序执行时调用了三次，参数 date 值都相同，message 不同
可以绑定第一个 date 参数，第二个参数使用下划线 _ 替换缺失的参数列表，并把这个新的函数值的索引的赋给变量
*/
object Test {
   def main(args: Array[String]) {
      val date = new Date
      val logWithDateBound = log(date, _ : String)

      logWithDateBound("message1" )
      Thread.sleep(1000)
      logWithDateBound("message2" )
      Thread.sleep(1000)
      logWithDateBound("message3" )
   }

   def log(date: Date, message: String)  = {
     println(date + "----" + message)
   }
}
```

## 函数柯里化

柯里化指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程

新的函数返回一个以原有第二个参数为参数的函数

```scala
def add(x:Int,y:Int)=x+y

// 柯里化
def add(x:Int)(y:Int) = x + y

/*

*/

object Test {
   def main(args: Array[String]) {
      val str1:String = "Hello, "
      val str2:String = "Scala!"
      println( "str1 + str2 = " +  strcat(str1)(str2) )
   }

   def strcat(s1: String)(s2: String) = {
      s1 + s2
   }
}
```

## 闭包

闭包是一个函数，返回值依赖于声明在函数外部的一个或多个变量

```scala
var factor = 3  
val multiplier = (i:Int) => i * factor
/*
引入一个自由变量 factor，这个变量定义在函数外面
函数变量 multiplier 成为一个闭包
*/
```