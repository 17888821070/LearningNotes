# class

## 命名

对于所有的类名的第一个字母要大写，如果需要使用几个单词来构成一个类的名称，每个单词的第一个字母要大写

所有的方法名称的第一个字母用小写，其他单词的第一个字母应大写

程序文件的名称应该与对象名称完全匹配，如果文件名和对象名称不匹配，程序将无法编译

## 类定义

Scala 的类定义可以有参数，称为类参数，类参数在整个类中都可以访问

```scala
class Point(xc: Int, yc: Int) {
   var x: Int = xc
   var y: Int = yc

   def move(dx: Int, dy: Int) {
      x = x + dx
      y = y + dy
      println ("x 的坐标点: " + x);
      println ("y 的坐标点: " + y);
   }

   override def toString: String = s"($x, $y)"
}
/* 
xc 和 yc 为类参数
Point 有变量 x、y、xc、yc，方法 move 和 toString
因为 toString 覆盖了 AnyRef 中的 toString 方法，所以用了 override 关键字标记
*/
```

## 构造器

主构造方法在类的签名中

构造器可以通过提供一个默认值来拥有可选参数

```scala
class Point(var x: Int = 0, var y: Int = 0)

val origin = new Point  // x and y are both set to 0
val point1 = new Point(1)

// 构造器支持带名传参
val point2 = new Point(y=2)
```

## 私有成员

成员默认是公有 `public` 的，使用 `private` 访问修饰符可以在类外部隐藏它们

```scala
class Point {
  private var _x = 0
  private var _y = 0
  private val bound = 100

  def x = _x
  def x_= (newValue: Int): Unit = {
    if (newValue < bound) _x = newValue else printWarning
  }

  def y = _y
  def y_= (newValue: Int): Unit = {
    if (newValue < bound) _y = newValue else printWarning
  }

  private def printWarning = println("WARNING: Out of bounds")
}

val point1 = new Point
point1.x = 99
point1.y = 101 // prints the warning

/*
私有变量 _x 和 _y
def x 和def y 方法用于访问私有数据
*/
```

主构造方法中带有 `val` 和 `var` 的参数是公有的，然而由于 `val` 是不可变的，所以不能通过访问修改

主构造方法中不带 `val` 或 `var` 的参数是私有的，仅在类中可见

## 继承

Scala 使用 `extends` 关键字来继承一个类

继承会继承父类的所有属性和方法，Scala 只允许继承一个父类

- 重写一个非抽象方法必须使用 `override` 修饰符

- 只有主构造函数才可以往基类的构造函数里写参数

- 在子类中重写超类的抽象方法时，不需要使用 `override` 关键字

## 单例对象

单例模式使用关键字 `object`

单例模式时，除了定义的类之外，还要定义一个同名的 `object` 对象，它和类的区别是，`object` 对象不能带参数

## 伴生对象


## 样例类

使用了 `case` 关键字的类定义就是样例类

样例类是种特殊的类，经过优化以用于模式匹配

```scala
object Test {
   def main(args: Array[String]) {
        val alice = new Person("Alice", 25)
        val bob = new Person("Bob", 32)
        val charlie = new Person("Charlie", 32)
   
    for (person <- List(alice, bob, charlie)) {
        person match {
            case Person("Alice", 25) => println("Hi Alice!")
            case Person("Bob", 32) => println("Hi Bob!")
            case Person(name, age) =>
               println("Age: " + age + " year, name: " + name + "?")
         }
      }
   }
   // 样例类
   case class Person(name: String, age: Int)
}
```

在声明样例类时，下面的过程自动发生了：

- 构造器的每个参数都成为 `val`，除非显式被声明为 `var`，但是并不推荐这么做
- 在伴生对象中提供了 `apply` 方法，所以可以不使用 `new` 关键字就可构建对象
- 提供 `unapply` 方法使模式匹配可以工作；
- 生成 `toString`、`equals`、`hashCode` 和 `copy` 方法，除非显示给出这些方法的定义