# fmt标准包

## 向外输出
```go
// Print系列函数：
// Print函数直接输出内容
// Printf函数支持格式化输出字符串
// Println函数会在输出内容的结尾添加一个换行符
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)

fmt.Printf("%v\n", 100)
fmt.Printf("%v\n", false)
o := struct{ name string }{"小王子"}
fmt.Printf("%v\n", o)
fmt.Printf("%#v\n", o)
fmt.Printf("%T\n", o)
fmt.Printf("100%%\n")
/* 输出
100
false
{小王子}
struct { name string }{name:"小王子"}
struct { name string }
100%
*/

// Fprint系列函数：
// 将内容输出到一个io.Writer接口类型的变量w中，通常用这个函数往文件中写入内容
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)

// Sprint系列函数：
// 把传入的数据生成并返回一个字符串
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string

// Errorf函数根据format参数生成格式化字符串并返回一个包含该字符串的错误
func Errorf(format string, a ...interface{}) error
// 通常使用这种方式来自定义错误类型
err := fmt.Errorf("这是一个错误")
```

占位符|说明
:-:|:-:
%v|值的默认格式表示
%+v|类似%v，但输出结构体时会添加字段名
%#v|值的Go语法表示
%T|打印值的类型
%%|百分号
%t|true或false
%b|表示为二进制
%c|该值对应的unicode码值
%d|表示为十进制
%o|表示为八进制
%x|表示为十六进制，使用a-f
%X|	表示为十六进制，使用A-F
%U|表示为Unicode格式：U+1234，等价于”U+%04X”
%q|该值对应的单引号括起来的go语法字符字面值，必要时会采用安全的转义表示
%b|无小数部分、二进制指数的科学计数法，如-123456p-78
%e|科学计数法，如-1234.456e+78
%E|科学计数法，如-1234.456E+78
%f|有小数部分但无指数部分，如123.456
%F|等价于%f
%g|根据实际情况采用%e或%f格式
%G|根据实际情况采用%E或%F格式
%s|直接输出字符串或者[]byte
%q|该值对应的双引号括起来的go语法字符串字面值，必要时会采用安全的转义表示
%x|每个字节用两字符十六进制数表示（使用a-f
%X|每个字节用两字符十六进制数表示（使用A-F）
%p|表示为十六进制，并加上前导的0x
%f|默认宽度，默认精度
%9f|宽度9，默认精度
%.2f|默认宽度，精度2
%9.2f|宽度9，精度2
%9.f|宽度9，精度0
+|总是输出数值的正负号
-|在输出右边填充空白而不是默认的左边
\#|八进制数前加0（%#o），十六进制数前加0x（%#x）或0X（%#X），指针去掉前面的0x（%#p）对%q（%#q），对%U（%#U）会输出空格和单引号括起来的go字面值
0|使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面



## 获取输入
```go
// 1. Scan
// 从标准输入扫描文本，读取由空白符分隔的值保存到传递给本函数的参数中，换行符视为空白符
// 本函数返回成功扫描的数据个数和遇到的任何错误
func Scan(a ...interface{}) (n int, err error)

// 2. Scanf
// 从标准输入扫描文本，根据format参数指定的格式去读取由空白符分隔的值保存到传递给本函数的参数中
// Scanf为输入数据指定了具体的输入内容格式，只有按照格式输入数据才会被扫描并存入对应变量
func main() {
	var (
		name    string
		age     int
		married bool
	)
	fmt.Scanf("1:%s 2:%d 3:%t", &name, &age, &married)
	fmt.Printf("扫描结果 name:%s age:%d married:%t \n", name, age, married)
}

// 3. Scanln
// 遇到换行时才停止扫描;多个输入间空格隔开
func Scanln(a ...interface{}) (n int, err error)

// 4. Fscan
// 从io.Reader中读取数据
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```