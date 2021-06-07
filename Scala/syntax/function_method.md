# 函数与方法

Scala 方法是类的一部分

Scala 中函数是一个对象可以赋值给一个变量，函数其实就是继承了 `Trait` 的类的对象

Scala 中使用 `val` 语句可以定义函数，`def` 语句定义方法

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

## 函数

```scala
// 函数声明
val FunctionName = ([参数列表]) => [return type] {}

class Test{
  def m(x: Int) = x + 3
  val f = (x: Int) => x + 3
}
```