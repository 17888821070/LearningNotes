# 流程控制

## `if-else`

```go
if 表达式1 {
    分支1
} else if 表达式2 {
    分支2
} else{
    分支3
}
// 与 if 匹配的左括号 { 必须与 if 和表达式放在同一行，否则会编译错误
// 同理，else 匹配的 { 也必须在同一行

// 可以在 if 表达式之前添加一个执行语句
func ifDemo2() {
	if score := 65; score >= 90 {
		fmt.Println("A")
	} else if score > 75 {
		fmt.Println("B")
	} else {
		fmt.Println("C")
	}
}
```

## `for`
Go 中所有循环类型均可使用 `for` 来完成
```go
for 初始语句;条件表达式;结束语句{
    循环体语句
}

func forDemo() {
	for i := 0; i < 10; i++ {
		fmt.Println(i)
	}
}

// 初始语句可以忽略
func forDemo2() {
	i := 0
	for ; i < 10; i++ {
		fmt.Println(i)
	}
}

// 结束语句可以忽略
func forDemo3() {
	i := 0
	for i < 10 {
		fmt.Println(i)
		i++
	}
}

// 无限循环
// 可以通过 break goto return panic 强制退出循环
for {
    循环体语句
}
```

# `for range` 键值循环
Go 中使用 `for range` 遍历数组、切片、字符串、`map`、`channel`
遍历数组、切片、字符串时返回索引和值
遍历 `map` 返回键和值
遍历 `channel` 返回 `channel` 内的值


## `switch case`

```go
func switchDemo1() {
	finger := 3
	switch finger {
	case 1:
		fmt.Println("大拇指")
	case 2:
		fmt.Println("食指")
	case 3:
		fmt.Println("中指")
	case 4:
		fmt.Println("无名指")
	case 5:
		fmt.Println("小拇指")
	default:
		fmt.Println("无效的输入！")
	}
}

// 一个分支可以用多个值，多个case值中间使用英文逗号分隔
func testSwitch3() {
	switch n := 7; n {
	case 1, 3, 5, 7, 9:
		fmt.Println("奇数")
	case 2, 4, 6, 8:
		fmt.Println("偶数")
	default:
		fmt.Println(n)
	}
}

// 分支可以使用表达式，此时 switch 后面不需要再跟判断变量
func switchDemo4() {
	age := 30
	switch {
	case age < 25:
		fmt.Println("好好学习吧")
	case age > 25 && age < 35:
		fmt.Println("好好工作吧")
	case age > 60:
		fmt.Println("好好享受吧")
	default:
		fmt.Println("活着真好")
	}
}

// fallthrough语法可以执行满足条件的case的下一个case，是为了兼容C语言中的case设计的
func switchDemo5() {
	s := "a"
	switch {
	case s == "a":
		fmt.Println("a")  // 打印
		fallthrough
	case s == "b":
		fmt.Println("b")  // 打印
	case s == "c":
		fmt.Println("c")
	default:
		fmt.Println("...")
	}
}
```

## `goto`
`goto` 语句通过标签进行代码间的无条件跳转，在快速跳出循环、避免重复退出上有一定的帮助
```go
func gotoDemo2() {
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			if j == 2 {
				// 设置退出标签
				goto breakTag
			}
			fmt.Printf("%v-%v\n", i, j)
		}
	}
	return
	// 标签
breakTag:
	fmt.Println("结束for循环")
}
```

## `break`
`break` 语句可以结束 `for`、`switch`和 `select` 的代码块
`break` 语句还可以在语句后面添加标签，表示退出某个标签对应的代码块，标签要求必须定义在对应的 `for`、`switch`和 `select` 的代码块上
```go
func breakDemo1() {
BREAKDEMO1:
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			if j == 2 {
				break BREAKDEMO1
			}
			fmt.Printf("%v-%v\n", i, j)
		}
	}
	fmt.Println("...")
}
```

## `continue`
`continue` 语句可以结束当前循环，开始下一次的循环迭代过程，仅限在 `for` 循环内使用
在 `continue` 语句后添加标签时，表示开始标签对应的循环
```go
func continueDemo() {
forloop1:
	for i := 0; i < 5; i++ {
		// forloop2:
		for j := 0; j < 5; j++ {
			if i == 2 && j == 2 {
				continue forloop1
			}
			fmt.Printf("%v-%v\n", i, j)
		}
	}
}
```