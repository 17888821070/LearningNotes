# 变量和常量

## 关键字、保留字
关键字和保留字不建议用作变量名

Go 有 25 个关键字，37 个保留字

```go
break   default  func  interface  select
case    defer    go    map        struct
chan    else     goto  package    switch
const   fallthrough  if  range  type
continue  for  import  return  var
```

```go
Constants:    true  false  iota  nil

Types:    int  int8  int16  int32  int64  
          uint  uint8  uint16  uint32 uint64                  uintptr  float32  float64  complex128                complex64  bool  byte  rune  string  error

Functions:   make  len  cap  new  append  
             copy  close  delete
             complex  real  imag
             panic  recover
```

##  变量
Go 的变量声名后必须使用

```go
// var 变量名 变量类型
var name string
var age int 
var isOk bool

// 批声明
var(
    a string
    b int
    c bool
    d float32
)

// 变量初始化
// var 变量名 类型 = 表达式
var name string = "Charon"
var age int = 128
var name, age = "Charon", 128

// 类型推导
// 将变量的类型声明省略，编译器会根据等号
// 右边的值来推导变量的类型完成初始化
var name = "Charon"
var age = 128

// 短变量声明
// 在函数内部可以使用:=方式声明并初始化变量
package main

import (
	"fmt"
)
// 全局变量m
var m = 100

func main() {
	n := 10
	m := 200 // 此处声明局部变量m
	fmt.Println(m, n)
}

// 匿名变量
// 在使用多重赋值时，如果想要忽略某个值，可以使用匿名变量
// 使用下划线 _ 表示
// 匿名变量不占用命名空间，不分配内存，所以匿名变量不存在重复声明
// 下划线 _ 多用于占位，表示忽略值
func foo(){
    return 10, "Charon"
}

func main(){
    x, _ := foo()
    _, y := foo()
}

// 函数外的每个语句都必须以关键字开始(var、const、func等)
// := 不能用在函数外
```

## 常量
```go
// 常量在定义的时候必须赋值
// 把 var 换成 const 
const pi = 3.14
const e = 2.71

// 多个变量一起声明
// 如果省略了值则和上一行的值相同
const(
    n1 = 100
    n2
    n3
)

// iota 是go的常量计数器，只能在常量的表达式中使用
// iota 在const关键字出现时将被重置为0, const 每增加一行变量声明则 iota 计数一次
// iota 可理解成 const 语句块中的行索引
// iota能简化定义，在定义枚举的时候很有用
const(
    n1 = iota  // 0
    n2         // 1     
    n3         // 2
    n4         // 3
)

// 使用 _ 跳过某些值
const(
    n1 = iota  // 0
    n2         // 1     
    _
    n4         // 3
)

// 中间插队
const(
    n1 = iota  // 0
    n2 = 100   // 100     
    n3 = iota  // 2
    n4         // 3
)

// 定义数量级
const (
		_  = iota
		KB = 1 << (10 * iota)
		MB = 1 << (10 * iota)
		GB = 1 << (10 * iota)
		TB = 1 << (10 * iota)
		PB = 1 << (10 * iota)
)

// 将多个 iota 定义在一行
const (
		a, b = iota + 1, iota + 2 //1,2
		c, d                      //2,3
		e, f                      //3,4
)
```