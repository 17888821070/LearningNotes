# trait

`traits` 用于在 `class` 之间共享程序接口和字段，但是 `trait` 不能被实例化，因此没有参数

## 定义

`trait` 作为泛型类型和抽象方法非常有用

```scala
trait Iterator[A] {
  def hasNext: Boolean
  def next(): A
}
```

## 使用

使用 `extends` 关键字来扩展 `trait`

使用 `override` 关键字来实现 `trait` 里面的任何抽象成员

```scala
trait Iterator[A] {
  def hasNext: Boolean
  def next(): A
}

class IntIterator(to: Int) extends Iterator[Int] {
  private var current = 0
  override def hasNext: Boolean = current < to
  override def next(): Int =  {
    if (hasNext) {
      val t = current
      current += 1
      t
    } else 0
  }
}

val iterator = new IntIterator(10)
iterator.next()  // returns 0
iterator.next()  // returns 1
```

一般情况下 Scala 的类只能够继承单一父类，但是如果是 `trait` 的话就可以继承多个，从结果来看就是实现了多重继承