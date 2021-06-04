# 访问修饰符

Scala 访问修饰符有：`private`、`protected`，`public`

如果没有指定访问修饰符，默认情况下，Scala 对象的访问级别都是 `public`

## private

用 `private` 关键字修饰，带有此标记的成员仅在包含了成员定义的类或对象内部可见，同样的规则还适用内部类

在嵌套类情况下，外层类甚至不能访问被嵌套类的私有成员

```scala
class Outer{
    class Inner{
        private def f(){println("f")}
 
        class InnerMost{
            f() // 正确
        }
    }
 
    (new Inner).f() //错误
}
```

### protected

在 scala 中，`protected` 成员只允许在定义了该成员的的类的子类中被访问

```scala
package p{
    class Super{
        protected def f() {println("f")}
    }
    
    class Sub extends Super{
        f()
    }
    
    class Other{
        (new Super).f() //错误
    }
}
```

## public

Scala 中，如果没有指定任何的修饰符，则默认为 public，这样的成员在任何地方都可以被访问

```scala
class Outer {
    class Inner {
        def f() { println("f") }
      
        class InnerMost {
            f() // 正确
        }
   }

   (new Inner).f() // 正确因为 f() 是 public
}
```

## 作用域保护

Scala中，访问修饰符可以通过使用限定词强调，允许你定义一些在你项目的若干子包中可见但对于项目外部的客户却始终不可见的东西

```scala
private[x]

protected[x]
```

这里的 x 指代某个所属的包、类或单例对象

如果写成 `private[x]`，读作这个成员除了对 x 中的类或包中的类及它们的伴生对像可见外，对其它所有类都是 `private`

```scala
package bobsrockets{
    package navigation{
        private[bobsrockets] class Navigator{
            protected[navigation] def useStarChart(){}
            class LegOfJourney{
                private[Navigator] val distance = 100
            }
            
            private[this] var speed = 200
        }
    }

    package launch{
        import navigation._
        object Vehicle{
            private[launch] val guide = new Navigator
        }
    }
}
/*
类 Navigator 被标记为 private[bobsrockets] 就是说这个类对包含在 bobsrockets 包里的所有的类和对象可见
*/
```