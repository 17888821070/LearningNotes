# strconv

实现了基本数据类型和其字符串表示的相互转换

## string-int 转换
```go
// Atoi()
// 将字符串类型的整数转换为int类型
func Atoi(s string) (i int, err error)

// Itoa()
// 将int类型数据转换为对应的字符串表示
func Itoa(i int) string
```

## Parse 系列函数
用于转换字符串为给定类型的值
```go
// 1. ParseBool()
// 返回字符串表示的bool值
// 接受1、0、t、f、T、F、true、false、True、False、TRUE、FALSE
func ParseBool(str string) (value bool, err error)

// 2. ParseInt()
// 返回字符串表示的整数值，接受正负号
// base指定进制（2到36）
// bitSize指定结果必须能无溢出赋值的整数类型，0、8、16、32、64 
func ParseInt(s string, base int, bitSize int) (i int64, err error)

// 3. ParseUnit()
// ParseUint类似ParseInt但不接受正负号，用于无符号整型
func ParseUint(s string, base int, bitSize int) (n uint64, err error)

// 4. ParseFloat()
// 解析一个表示浮点数的字符串并返回其值
// bitSize指定了期望的接收类型,32、64
func ParseFloat(s string, bitSize int) (f float64, err error)
```

## Format 系列函数
实现了将给定类型数据格式化为string类型数据的功能
```go
// 1. FormatBool()
// 返回”true”或”false”
func FormatBool(b bool) string

// 2. FormatInt() FormatUint()
// 返回i的base进制的字符串表示
func FormatInt(i int64, base int) string
func FormatUint(i uint64, base int) string

// 3. FormatFloat()
// bitSize表示f的来源类型（32：float32、64：float64）
func FormatFloat(f float64, fmt byte, prec, bitSize int) string
```