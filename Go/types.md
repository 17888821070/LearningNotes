# 基本数据类型

## 整型
整型分为两大类，按长度分：`int8`, `int16`, `int32`, `int64` 及对应的无符号整型：`uint8`, `uint16`, `uint32`, `uint64`
其中，`uint8` 就是 C 中的 `byte`，`int16` 就是 C 中的 `short`，`int64` 就是 C 中的 `long`

### 特殊整型
`uint`：32 位系统上就是 `uint32`，64 位系统上就是 `uint64`

`int`：32 位系统上就是 `int32`，64 位系统上就是 `int64`

`uintptr`：无符号整型，用于存放一个指针

在使用 `uint`、`int` 时不能假定它的数据类型，而是考虑不同平台上的差异

在涉及到二进制传输、读写文件的结构描述时，为了保持文件的结构不会受到不同编译目标平台字节长度的影响，不要使用 `int` 和 `uint`

获取对象长度的内建 `len()` 函数返回的长度可以根据不同平台的字节长度进行变化

### 八进制、十六进制
Go中无法直接定义二进制数
```go
package main
 
import "fmt"
 
func main(){
	// 十进制
	var a int = 10
	fmt.Printf("%d \n", a)  // 10
	fmt.Printf("%b \n", a)  // 1010  占位符%b表示二进制
 
	// 八进制  以0开头
	var b int = 077
	fmt.Printf("%o \n", b)  // 77
 
	// 十六进制  以0x开头
	var c int = 0xff
	fmt.Printf("%x \n", c)  // ff
	fmt.Printf("%X \n", c)  // FF
}
```

## 浮点型
Go 支持两种浮点型，`float32` 和 `float64`

`float32` 可以用常量 `math.MaxFloat32` 定义支持最大值

`float64`可以用常量 `math.MaxFloat64` 定义支持最大值
```go
package main
import (
        "fmt"
        "math"
)
func main() {
        fmt.Printf("%f\n", math.Pi)
        fmt.Printf("%.2f\n", math.Pi)
}
```

## 复数
`complex64`：实部虚部都是32位

`complex128`：实部虚部都是64位
```go
var c1 complex64
c1 = 1 + 2i
var c2 complex128
c2 = 2 + 3i
fmt.Println(c1)
fmt.Println(c2)
```

## 布尔值
默认值是 `false`

不允许将整型强制转换成布尔型

无法参与数值运算，也不能与其他类型就行转换

## 字符串
Go 的字符串以原生数据类型出现，使用 UTF-8 编码，可以直接添加非 ASCII 字符串
```go
var s1 string = "hello"
var s2 string = "你好"
```
字符串转义符
```go
/*
\r:回车
\n:换行
\t:制表符
\':单引号
\":双引号
\\:反斜杠
*/
```

## 多行字符串
Go 定义一个多行字符串时，必须使用反引号字符
反引号间换行将被作为字符串间的换行，但是所有的转义字符均无效
```go
var s1 = `第一行
第二行
第三行
`
s2 := `第一行
第二行
第三行
`
```

## byte 和 rune
组成每个字符串的元素叫做字符，可以通过遍历或者单个获取字符串元素获得字符，字符用单引号
```go
var a = '中'
var b = 'x'
```
Go 字符有两种：
`uint8` 类型，或者叫 `byte` 型，代表一个 ASCII 码的一个字符

`rune` 类型，代表一个 UTF-8 字符

当需要处理中文、日文或者其他复合字符时，则需要用到 `rune` 类型

`rune` 类型实际是一个 `int32`

Go 使用了特殊的 `rune` 类型来处理 `Unicode`，让基于 `Unicode` 的文本处理更为方便，也可以使用 `byte` 型进行默认字符串处理，性能和扩展性都有照顾
```go
// 遍历字符串
func traversalString() {
	s := "hello沙河"
	for i := 0; i < len(s); i++ { //byte
		fmt.Printf("%v(%c) ", s[i], s[i])
	}
	fmt.Println()
	for _, r := range s { //rune
		fmt.Printf("%v(%c) ", r, r)
	}
	fmt.Println()
}

/*
因为UTF8编码下一个中文汉字由3~4个字节组成，所以我们不能简单的按照字节去遍历一个包含中文的字符串
字符串底层是一个byte数组，所以可以和[]byte类型相互转换
字符串是不能修改的 
字符串是由byte字节组成，所以字符串的长度是byte字节的长度 
rune类型用来表示utf8字符，一个rune字符由一个或多个byte组成
*/
```

### 修改字符串
要修改字符串，需要先将其转换成 `[]rune` 或 `[]byte`，完成后再转换为 `string`。无论哪种转换，都会重新分配内存，并复制字节数组
```go
func changeString() {
	s1 := "big"
	// 强制类型转换
	byteS1 := []byte(s1)
	byteS1[0] = 'p'
	fmt.Println(string(byteS1))

	s2 := "白萝卜"
	runeS2 := []rune(s2)
	runeS2[0] = '红'
	fmt.Println(string(runeS2))
}
```

## 类型转换
Go 中只有强制类型转换，没有隐式类型转换；该语法只能在两个类型之间支持相互转换的时候使用
```go
/*
T(表达式)
T为要转换的类型
表达式包括变量、复杂算子和函数返回值等
*/
```