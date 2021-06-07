# 变量

在 Scala 中，使用关键词 `var` 声明变量，使用关键词 `val` 声明常量

```scala
var myVar : String = "Foo"
var myVar : String = "Too"

val myVal : String = "Foo"
```

如果程序尝试修改常量的值，程序将会在编译时报错

## 变量类型声明

```scala
var VariableName : DataType [=  Initial Value]

val VariableName : DataType [=  Initial Value]
```

### 变量类型推导

在 Scala 中声明变量和常量不一定要指明数据类型，在没有指明数据类型的情况下，其数据类型是通过变量或常量的初始值推断出来的

如果在没有指明数据类型的情况下声明变量或常量必须要给出其初始值，否则将会报错

### 多个变量声明

```scala
val xmax, ymax = 100  // xmax, ymax都声明为100
```