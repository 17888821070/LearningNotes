# Array数组

Go 中，数组从声明时就确定了，使用时可以修改数组成员但数组大小不可改变

## 数组定义
```go
// var 数组变量名 [元素数量]T
var a [3]int
// 数组长度必须是常量，并且长度是数组类型的一部分   [5]int 和 [10]int 是不同类型
var b [4]int
// a = b 不可以，因为是不同类型
```
数组可以通过下标进行访问，访问越界会触发访问越界，会 panic

- 数组初始化
```go
// 方法一：初始化列表设置数组元素的值
func main() {
	var testArray [3]int                        //数组会初始化为int类型的零值
	var numArray = [3]int{1, 2}                 //使用指定的初始值完成初始化
	var cityArray = [3]string{"北京", "上海", "深圳"} //使用指定的初始值完成初始化
	fmt.Println(testArray)                      //[0 0 0]
	fmt.Println(numArray)                       //[1 2 0]
	fmt.Println(cityArray)                      //[北京 上海 深圳]
}

// 方法二：让编译器根据初始值个数自行推断数组长度
func main() {
	var testArray [3]int
	var numArray = [...]int{1, 2}
	var cityArray = [...]string{"北京", "上海", "深圳"}
	fmt.Println(testArray)                          //[0 0 0]
	fmt.Println(numArray)                           //[1 2]
	fmt.Printf("type of numArray:%T\n", numArray)   //type of numArray:[2]int
	fmt.Println(cityArray)                          //[北京 上海 深圳]
	fmt.Printf("type of cityArray:%T\n", cityArray) //type of cityArray:[3]string
}

// 方法三：按索引值方式初始化数组
func main() {
	a := [...]int{1: 1, 3: 5}
	fmt.Println(a)                  // [0 1 0 5]
	fmt.Printf("type of a:%T\n", a) //type of a:[4]int
}
```

## 数组遍历
```go
func main() {
	var a = [...]string{"北京", "上海", "深圳"}
	// 方法1：for循环遍历
	for i := 0; i < len(a); i++ {
		fmt.Println(a[i])
	}

	// 方法2：for range遍历
	for index, value := range a {
		fmt.Println(index, value)
	}
}
```

## 多维数组
```go
// 二维数组定义
func main() {
	a := [3][2]string{
		{"北京", "上海"},
		{"广州", "深圳"},
		{"成都", "重庆"},
	}
	fmt.Println(a) //[[北京 上海] [广州 深圳] [成都 重庆]]
	fmt.Println(a[2][1]) //支持索引取值:重庆
	
	for _, v1 := range a {
		for _, v2 := range v1 {
			fmt.Printf("%s\t", v2)
		}
		fmt.Println()
	}
	/*
	北京	上海	
    广州	深圳	
    成都	重庆
	*/
}

// 多维数组只有第一层可以使用 ... 来让编译器推到数组长度
//支持的写法
a := [...][2]string{
	{"北京", "上海"},
	{"广州", "深圳"},
	{"成都", "重庆"},
}
//不支持多维数组的内层使用...
b := [3][...]string{
	{"北京", "上海"},
	{"广州", "深圳"},
	{"成都", "重庆"},
}
```

## 数组是值类型
数组是值类型，赋值和传参会复制整个数组。因此改变副本的值，不会改变本身的值

## 注意
数组支持 “==“、”!=” 操作符，因为内存总是被初始化过的
[n]*T表示指针数组，*[n]T表示数组指针 