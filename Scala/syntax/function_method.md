# 函数与方法

Scala 方法是类的一部分

Scala 中函数是一个对象可以赋值给一个变量，函数其实就是继承了 `Trait` 的类的对象

## 声明

Scala 中使用 `val` 语句可以定义函数，`def` 语句定义方法

```scala
// 方法声明
def functionName ([参数列表]) : [return type]
/*
如果不写等于号和方法主体，那么方法会被隐式声明为 abstract，包含它的类型于是也是一个抽象类型:w

*/
def functionName ([参数列表]) : [return type] = {
   function body
   return [expr]
}

class Test{
  def m(x: Int) = x + 3
  val f = (x: Int) => x + 3
}
```