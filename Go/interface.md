# 接口

接口定义了一个对象的行为规范，只定义规范不实现，由具体的对象来实现规范的细节

## 接口类型
Go 中接口是一种抽象的类型

`interface` 是一组 method 的集合

接口就像定义一个协议，不关心属性，只关心行为

## 定义
Go 提倡面向接口编程
```go
/*
每个接口由数个方法组成
type 接口名 interface{
    方法名1( 参数列表1 ) 返回值列表1
    方法名2( 参数列表2 ) 返回值列表2
    …
}
1. 接口名：使用type将接口定义为自定义的类型名;接口在命名时，一般会在单词后面添加er，如有写操作的接口叫Writer
2. 方法名：当方法名首字母是大写且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包（package）之外的代码访问
3. 参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以省略
*/
type writer interface{
    Write([]byte) error
}
```

## 实现
```go
// 只要实现了接口中的所有方法，就实现了这个接口
// Sayer 接口
type Sayer interface {
	say()
}

type dog struct {}
type cat struct {}

// dog实现了Sayer接口
func (d dog) say() {
	fmt.Println("汪汪汪")
}

// cat实现了Sayer接口
func (c cat) say() {
	fmt.Println("喵喵喵")
}
```

## 接口类型变量
接口类型变量能够存储所有实现了该接口的实例
```go
func main() {
	var x Sayer // 声明一个Sayer类型的变量x
	a := cat{}  // 实例化一个cat
	b := dog{}  // 实例化一个dog
	x = a       // 可以把cat实例直接赋值给x
	x.say()     // 喵喵喵
	x = b       // 可以把dog实例直接赋值给x
	x.say()     // 汪汪汪
}
```

## 值接受者和指针接受者实现接口区别
```go
type Mover interface {
	move()
}

type dog struct {}

// 值接受者
func (d dog) move() {
	fmt.Println("狗会动")
}

func main() {
	var x Mover
	var wangcai = dog{} //旺财是dog类型
	x = wangcai         //x可以接收dog类型
	var fugui = &dog{}  //富贵是*dog类型
	x = fugui           //x可以接收*dog类型
	x.move()
}
// 使用值接收者实现接口之后，不管是dog结构体还是结构体指针*dog类型的变量都可以赋值给该接口变量

// 指针接受者
func (d *dog) move() {
	fmt.Println("狗会动")
}
func main() {
	var x Mover
	var wangcai = dog{} //旺财是dog类型
	x = wangcai         //x不可以接收dog类型
	var fugui = &dog{}  //富贵是*dog类型
	x = fugui           //x可以接收*dog类型
}
// 不能给指针接受者传入值类型
```

## 类型与接口的关系
```go
// 1. 一个类型实现多个接口，接口间彼此独立
// Sayer 接口
type Sayer interface {
	say()
}

// Mover 接口
type Mover interface {
	move()
}

type dog struct {
	name string
}

// 实现Sayer接口
func (d dog) say() {
	fmt.Printf("%s会叫汪汪汪\n", d.name)
}

// 实现Mover接口
func (d dog) move() {
	fmt.Printf("%s会动\n", d.name)
}

func main() {
	var x Sayer
	var y Mover

	var a = dog{name: "旺财"}
	x = a
	y = a
	x.say()
	y.move()
}

// 2. 多个类型实现同一接口
// Mover 接口
type Mover interface {
	move()
}

type dog struct {
	name string
}

type car struct {
	brand string
}

// dog类型实现Mover接口
func (d dog) move() {
	fmt.Printf("%s会跑\n", d.name)
}

// car类型实现Mover接口
func (c car) move() {
	fmt.Printf("%s速度70迈\n", c.brand)
}

func main() {
	var x Mover
	var a = dog{name: "旺财"}
	var b = car{brand: "保时捷"}
	x = a
	x.move()
	x = b
	x.move()
}

// 一个接口的方法，不一定需要由一个类型完全实现，接口的方法可以通过在类型中嵌入其他类型或者结构体来实现
// WashingMachine 洗衣机
type WashingMachine interface {
	wash()
	dry()
}

// 甩干器
type dryer struct{}

// 实现WashingMachine接口的dry()方法
func (d dryer) dry() {
	fmt.Println("甩一甩")
}

// 海尔洗衣机
type haier struct {
	dryer //嵌入甩干器
}

// 实现WashingMachine接口的wash()方法
func (h haier) wash() {
	fmt.Println("洗刷刷")
}
```

## 接口嵌套
```go
// Sayer 接口
type Sayer interface {
	say()
}

// Mover 接口
type Mover interface {
	move()
}

// 接口嵌套
type animal interface {
	Sayer
	Mover
}

// 使用与普通接口一样
type cat struct {
	name string
}

func (c cat) say() {
	fmt.Println("喵喵喵")
}

func (c cat) move() {
	fmt.Println("猫会动")
}

func main() {
	var x animal
	x = cat{name: "花花"}
	x.move()
	x.say()
}
```

## 空接口
```go
// 1. 定义：没有定义任何方法的接口，因此任何类型都实现了空接口
// 空接口类型的变量可以存储任意类型的变量

func main() {
	// 定义一个空接口x
	var x interface{}
	s := "Hello 沙河"
	x = s
	fmt.Printf("type:%T value:%v\n", x, x)
	i := 100
	x = i
	fmt.Printf("type:%T value:%v\n", x, x)
	b := true
	x = b
	fmt.Printf("type:%T value:%v\n", x, x)
}

// 2. 空接口作为函数的参数
// 空接口实现可以接收任意类型的函数参数
func show(a interface{}) {
	fmt.Printf("type:%T value:%v\n", a, a)
}
// 3. 空接口作为map的值
// 空接口实现可以保存任意值的字典
	var studentInfo = make(map[string]interface{})
	studentInfo["name"] = "沙河娜扎"
	studentInfo["age"] = 18
	studentInfo["married"] = false
	fmt.Println(studentInfo)
	
// 4. 类型断言
// 接口值 = 一个具体类型(动态类型) + 具体类型的值(动态值)
// 判断空接口的值，使用类型断言 x.(T)
// x：表示类型为interface{}的变量
// T：表示断言x可能是的类型
// 返回两个参数，第一个参数是x转化为T类型后的变量
// 第二个值是一个布尔值，若为true则表示断言成功，为false则表示断言失败
func main() {
	var x interface{}
	x = "Hello 沙河"
	v, ok := x.(string)
	if ok {
		fmt.Println(v)
	} else {
		fmt.Println("类型断言失败")
	}
}
// 断言多次可以使用switch语句
func justifyType(x interface{}) {
	switch v := x.(type) {
	case string:
		fmt.Printf("x is a string，value is %v\n", v)
	case int:
		fmt.Printf("x is a int is %v\n", v)
	case bool:
		fmt.Printf("x is a bool is %v\n", v)
	default:
		fmt.Println("unsupport type！")
	}
}

// 空接口可以存储任意类型值的特点，所以空接口在Go语言中的使用十分广泛
```

