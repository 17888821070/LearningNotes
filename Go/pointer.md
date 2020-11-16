# 指针
Go 中指针是引用类型
Go 中的指针不能进行偏移和运算，是安全指针
Go 函数传参都是值拷贝，当想修改某个变量时，需要创建一个指针指向该变量地址的指针变量，传递出具使用指针

## 指针地址和指针类型
`&` 取地址
`*` 取值
值类型都有对应的指针类型，如 `*int; *int64; *string`
```go
ptr := &v
// v:代表被取地址的变量，类型为T
// ptr:用于接收地址的变量，ptr的类型就为*T，称做T的指针类型

func modify1(x int) {
	x = 100
}

func modify2(x *int) {
	*x = 100
}

func main() {
	a := 10
	modify1(a)
	fmt.Println(a) // 10
	modify2(&a)
	fmt.Println(a) // 100
}
```

## new 和 make
Go 中对于应用类型变量，使用时候不仅要声明还要为它分配内存空间
分配内存，可以使用 `new()` 和 `make()` 两个内建函数
```go
func new(Type) *Type
// Type表示类型，new函数只接受一个参数，这个参数是一个类型
// *Type表示类型指针，new函数返回一个指向该类型内存地址的指针

// new()函数得到的是一个类型的指针，并且该指针对应的值为该类型的零值
func main() {
	a := new(int)
	b := new(bool)
	fmt.Printf("%T\n", a) // *int
	fmt.Printf("%T\n", b) // *bool
	fmt.Println(*a)       // 0
	fmt.Println(*b)       // false
}	
func main() {
	var a *int
	a = new(int)
	*a = 10
	fmt.Println(*a)
}

func make(t Type, size ...IntegerType) Type
// make()函数只用于slice、map以及chan的内存创建
// 它返回的类型就是这三个类型本身，而不是他们的指针类型
// 因为这三种类型就是引用类型，所以就没有必要返回他们的指针了
```