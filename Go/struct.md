# 结构体

Go 中没有类的概念，也不支持继承等面向对象的概念
通过结构体的内嵌再配合接口，比面向对象具有更高的扩展性和灵活性

## 自定义类型和类型别名
```go
//  自定义类型
type MyInt int

// 类型别名
type byte = uint8
type rune = int32

// 区别
type NewInt int
type MyInt = int

func main() {
	var a NewInt
	var b MyInt
	
	fmt.Printf("type of a:%T\n", a) //type of a:main.NewInt
	fmt.Printf("type of b:%T\n", b) //type of b:int
}
// MyInt类型只会在代码中存在，编译完成时并不会有MyInt类型
```

## 结构体
```go
/*
type 类型名 struct {
    字段名 字段类型
    字段名 字段类型
    …
}
类型名：标识自定义结构体的名称，在同一个包内不能重复
字段名：表示结构体字段名。结构体中的字段名必须唯一
*/

type person struct {
	name string
	city string
	age  int8
}

type person1 struct {
	name, city string
	age        int8
}

func main() {
	var p1 person
	p1.name = "沙河娜扎"
	p1.city = "北京"
	p1.age = 18
	fmt.Printf("p1=%v\n", p1)  //p1={沙河娜扎 北京 18}
	fmt.Printf("p1=%#v\n", p1) //p1=main.person{name:"沙河娜扎", city:"北京", age:18}
}
```

## 匿名结构体
在定义一些临时数据结构等场景下还可以使用匿名结构体
```go
func main() {
    var user struct{Name string; Age int}
    user.Name = "小王子"
    user.Age = 18
    fmt.Printf("%#v\n", user)
}
```

## 创建结构体指针
```go
var p2 = new(person)
fmt.Printf("%T\n", p2)     //*main.person
fmt.Printf("p2=%#v\n", p2) //p2=&main.person{name:"", city:"", age:0}
// 支持对结构体指针直接使用 . 来访问结构体的成员

// 去结构体地址实例化
p3 := &person{}
```

## 使用键值对初始化结构体
```go
p5 := person{
	name: "小王子",
	city: "北京",
	age:  18,
}
fmt.Printf("p5=%#v\n", p5) //p5=main.person{name:"小王子", city:"北京", age:18}

p6 := &person{
	name: "小王子",
	city: "北京",
	age:  18,
}
fmt.Printf("p6=%#v\n", p6) //p6=&main.person{name:"小王子", city:"北京", age:18}

// 当某些字段没有初始值的时候，该字段可以不写
p7 := &person{
	city: "北京",
}
fmt.Printf("p7=%#v\n", p7) //p7=&main.person{name:"", city:"北京", age:0}

// 使用值的列表初始化
// 1. 必须初始化结构体的所有字段
// 2. 初始值的填充顺序必须与字段在结构体中的声明顺序一致
// 3. 该方式不能和键值初始化方式混用
p8 := &person{
	"沙河娜扎",
	"北京",
	28,
}
fmt.Printf("p8=%#v\n", p8) //p8=&main.person{name:"沙河娜扎", city:"北京", age:28}

```

## 结构体内存布局
结构体占用一块连续内存

[Go内存对齐][1]

## 构造函数
Go 结构体没有构造函数，但可以自己实现
```go
func newPerson(name, city string, age int8) *person {
	return &person{
		name: name,
		city: city,
		age:  age,
	}
}
p9 := newPerson("张三", "沙河", 90)
fmt.Printf("%#v\n", p9) //&main.person{name:"张三", city:"沙河", age:90}
```

## 方法和接受者
Go 中方法 Method 是一种作用于特定类型变量的函数
特定类型变量叫做接受者 Receiver，类似于 `this`

```go
/*
func (接收者变量 接收者类型) 方法名(参数列表) (返回参数) {
    函数体
}
1. 接收者变量：接收者中的参数变量名在命名时，官方建议使用接收者类型名的第一个小写字母，而不是self、this之类的命名;例如，Person类型的接收者变量应该命名为p，Connector类型的接收者变量应该命名为c等
2. 接收者类型：接收者类型和参数类似，可以是指针类型和非指针类型
3. 方法名、参数列表、返回参数：具体格式与函数定义相同
*/

//Person 结构体
type Person struct {
	name string
	age  int8
}

//NewPerson 构造函数
func NewPerson(name string, age int8) *Person {
	return &Person{
		name: name,
		age:  age,
	}
}

//Dream Person做梦的方法
func (p Person) Dream() {
	fmt.Printf("%s的梦想是学好Go语言！\n", p.name)
}

func main() {
	p1 := NewPerson("小王子", 25)
	p1.Dream()
}
//方法与函数的区别是，函数不属于任何类型，方法属于特定的类型

// 值类型的接受者
// 当方法作用于值类型接受者时，Go会将接受者的值复制一份
// 在值类型接收者的方法中可以获取接收者的成员值，但修改操作只是针对副本，无法修改接收者变量本身
// SetAge2 设置p的年龄
// 使用值接收者
func (p Person) SetAge2(newAge int8) {
	p.age = newAge
}

func main() {
	p1 := NewPerson("小王子", 25)
	p1.Dream()
	fmt.Println(p1.age) // 25
	p1.SetAge2(30) // (*p1).SetAge2(30)
	fmt.Println(p1.age) // 25
}

// 指针类型的接受者
// 由于指针的特性，调用方法时修改接收者指针的任意成员变量，在方法结束后，修改都是有效的
// SetAge 设置p的年龄
// 使用指针接收者
func (p *Person) SetAge(newAge int8) {
	p.age = newAge
}

func main() {
	p1 := NewPerson("小王子", 25)
	fmt.Println(p1.age) // 25
	p1.SetAge(30)
	fmt.Println(p1.age) // 30
}
```

## 任意类型添加方法
接收者的类型可以是任何类型，不仅仅是结构体，任何类型都可以拥有方法
```go
//MyInt 将int定义为自定义MyInt类型
type MyInt int

//SayHello 为MyInt添加一个SayHello的方法
func (m MyInt) SayHello() {
	fmt.Println("Hello, 我是一个int。")
}
func main() {
	var m1 MyInt
	m1.SayHello() //Hello, 我是一个int。
	m1 = 100
	fmt.Printf("%#v  %T\n", m1, m1) //100  main.MyInt
}
//  非本地类型不能定义方法，也就是说我们不能给别的包的类型定义方法
```

## 结构体的匿名字段
结构体允许其成员字段在声明时没有字段名而只有类型，这种没有名字的字段就称为匿名字段
```go
//Person 结构体Person类型
type Person struct {
	string
	int
}

func main() {
	p1 := Person{
		"小王子",
		18,
	}
	fmt.Printf("%#v\n", p1)        //main.Person{string:"北京", int:18}
	fmt.Println(p1.string, p1.int) //北京 18
}
//匿名字段默认采用类型名作为字段名，结构体要求字段名称必须唯一，因此一个结构体中同种类型的匿名字段只能有一个
```

## 嵌套结构体
一个结构体中可以嵌套包含另一个结构体或结构体指针

```go
//Address 地址结构体
type Address struct {
	Province string
	City     string
}

//User 用户结构体
type User struct {
	Name    string
	Gender  string
	Address Address
}

func main() {
	user1 := User{
		Name:   "小王子",
		Gender: "男",
		Address: Address{
			Province: "山东",
			City:     "威海",
		},
	}
	fmt.Printf("user1=%#v\n", user1)//user1=main.User{Name:"小王子", Gender:"男", Address:main.Address{Province:"山东", City:"威海"}}
}

// 嵌套匿名结构体
//User 用户结构体
type User struct {
	Name    string
	Gender  string
	Address //匿名结构体
}

func main() {
	var user2 User
	user2.Name = "小王子"
	user2.Gender = "男"
	user2.Address.Province = "山东"    //通过匿名结构体.字段名访问
	user2.City = "威海"                //直接访问匿名结构体的字段名
	fmt.Printf("user2=%#v\n", user2) //user2=main.User{Name:"小王子", Gender:"男", Address:main.Address{Province:"山东", City:"威海"}}
}

// 嵌套结构体的字段名冲突
// 嵌套结构体内部可能存在相同的字段名。这个时候为了避免歧义需要指定具体的内嵌结构体的字段
//Address 地址结构体
type Address struct {
	Province   string
	City       string
	CreateTime string
}

//Email 邮箱结构体
type Email struct {
	Account    string
	CreateTime string
}

//User 用户结构体
type User struct {
	Name   string
	Gender string
	Address
	Email
}

func main() {
	var user3 User
	user3.Name = "沙河娜扎"
	user3.Gender = "男"
	// user3.CreateTime = "2019" //ambiguous selector user3.CreateTime
	user3.Address.CreateTime = "2000" //指定Address结构体中的CreateTime
	user3.Email.CreateTime = "2000"   //指定Email结构体中的CreateTime
}
```

## 结构体的“继承”
```go
//Animal 动物
type Animal struct {
	name string
}

func (a *Animal) move() {
	fmt.Printf("%s会动！\n", a.name)
}

//Dog 狗
type Dog struct {
	Feet    int8
	*Animal //通过嵌套匿名结构体实现继承
}

func (d *Dog) wang() {
	fmt.Printf("%s会汪汪汪~\n", d.name)
}

func main() {
	d1 := &Dog{
		Feet: 4,
		Animal: &Animal{ //注意嵌套的是结构体指针
			name: "乐乐",
		},
	}
	d1.wang() //乐乐会汪汪汪~
	d1.move() //乐乐会动！
}
```

## 结构体字段可见性
结构体中字段大写开头表示可公开访问，小写表示私有（仅在定义当前结构体的包中可访问

## 结构体标签Tag
Tag是结构体的元信息，可以在运行的时候通过反射的机制读取出来
```go
// Tag在结构体字段的后方定义，由一对反引号包裹起来
// 结构体标签由一个或多个键值对组成
// 键与值使用冒号分隔，值用双引号括起来
// 键值对之间使用一个空格分隔
`key1:"value1" key2:"value2"`

//Student 学生
type Student struct {
	ID     int    `json:"id"` //通过指定tag实现json序列化该字段时的key
	Gender string //json序列化是默认使用字段名作为key
	name   string //私有不能被json包访问
}

func main() {
	s1 := Student{
		ID:     1,
		Gender: "男",
		name:   "沙河娜扎",
	}
	data, err := json.Marshal(s1)
	if err != nil {
		fmt.Println("json marshal failed!")
		return
	}
	fmt.Printf("json str:%s\n", data) //json str:{"id":1,"Gender":"男"}
}
```


  [1]: https://segmentfault.com/a/1190000017527311?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com