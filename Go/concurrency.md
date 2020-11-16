# 并发

## 并发和并行
并发：同一时间段内执行多个任务

并行：同一时刻执行多个任务

Go 并发通过 `goroutine` 实现；`goroutine` 类似于线程，用于用户态的线程，可以创建成千上万个 `goroutine` 并发工作

`goroutine` 由 Go 运行时调度完成，而线程由操作系统调度完成

Go 提供 `channel` 在多个 `goroutine` 间进行通信

## goroutine
go 在语言层面上内置了调度和上下文切换的机制
go使用 `go` 关键字为一个函数创建一个 `goroutine`；一个函数可以被创建多个 `goroutine`，一个 `goroutine` 必定对应一个函数

### `goroutine` 启动
启动 `goroutine` 的方式非常简单，只需要在调用的函数（普通函数和匿名函数）前面加上一个 `go` 关键字
```go
func hello() {
	fmt.Println("Hello Goroutine!")
}

// 串行
func main() {
	hello()
	fmt.Println("main goroutine done!")
}

// 启动一个goroutine
// 在程序启动时，Go程序就会为main()函数创建一个默认的goroutine
// 当main()函数返回的时候该goroutine就结束了
// 所有在main()函数中启动的goroutine会一同结束
func main() {
	go hello() // 启动另外一个goroutine去执行hello函数
	fmt.Println("main goroutine done!")
	time.Sleep(time.Second)
}
```

### `sync.WaitGroup`
使用 `sync.WaitGroup` 来实现并发任务的同步

`sync.WaitGroup` 内部维护着一个计数器，计数器的值可以增加和减少

启动了 N 个并发任务时，就将计数器值增加 N；每个任务完成时通过调用 `Done()`方法将计数器减 1；

通过调用 `Wait()` 来等待并发任务执行完，当计数器值为 0 时，表示所有并发任务已经完成

`sync.WaitGroup` 是一个结构体，传递的时候要传递指针
方法名|功能
-|-
`(wg * WaitGroup)Add(delta int)`|计数器 +delta
`(wg *WaitGroup) Done()`|计数器 -1
`(wg *WaitGroup) Wait()`|阻塞直到计数器变为 0
```go
var wg sync.WaitGroup

func hello() {
	defer wg.Done()
	fmt.Println("Hello Goroutine!")
}
func main() {
	wg.Add(1)
	go hello() // 启动另外一个goroutine去执行hello函数
	fmt.Println("main goroutine done!")
	wg.Wait()
}
```

## `goroutine` 和线程
OS 线程（操作系统线程）一般都有固定的栈内存（通常为 2MB）；一个 `goroutine` 的栈在其生命周期开始时只有很小的栈（典型情况下 2KB），`goroutine` 的栈不是固定的，他可以按需增大和缩小，`goroutine` 的栈大小限制可以达到 1GB

OS 线程是由 OS 内核来调度的；`goroutine` 则是由 Go 运行时（runtime）自己的调度器调度的

`goroutine` 的调度不需要切换内核语境，所以调用一个 `goroutine` 比调度一个线程成本低很多

## `GOMAXPROCS`
调度器使用 `GOMAXPROCS` 参数来确定需要使用多少个 OS 线程来同时执行 Go 代码

通过`runtime.GOMAXPROCS()`函数设置当前程序并发时占用的 CPU 逻辑核心数
```go
func a() {
	for i := 1; i < 10; i++ {
		fmt.Println("A:", i)
	}
}

func b() {
	for i := 1; i < 10; i++ {
		fmt.Println("B:", i)
	}
}

func main() {
	runtime.GOMAXPROCS(1)
	// 两个任务只有一个逻辑核心，此时是做完一个任务再做另一个任务
	go a()
	go b()
	time.Sleep(time.Second)
}

func main() {
	runtime.GOMAXPROCS(2)
	// 将逻辑核心数设为2，此时两个任务并行执行
	go a()
	go b()
	time.Sleep(time.Second)
}
```

## `channel`
虽然可以使用共享内存进行数据交换，但是共享内存在不同的 `goroutine` 中容易发生竞态问题;为了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必造成性能问题

Go 提倡通过通信共享内存而不是通过共享内存而实现通信

`goroutine` 是 Go 程序并发的执行体，`channel` 就是它们之间的连接

`channel` 是可以让一个 `goroutine` 发送特定值到另一个 `goroutine` 的通信机制

通道（channel）是一种特殊的类型；通道像一个传送带或者队列，总是遵循先入先出（First In First Out）的规则，保证收发数据的顺序；每一个通道都是一个具体类型的导管，也就是声明 `channel` 的时候需要为其指定元素类型
```go
// 声明
// var 变量 chan 元素类型
var ch1 chan int   // 声明一个传递整型的通道
var ch2 chan bool  // 声明一个传递布尔型的通道
var ch3 chan []int // 声明一个传递int切片的通道

// 初始化
// channel 是引用类型，空值是nil
// make(chan 元素类型, [缓冲大小])
ch4 := make(chan int)
ch5 := make(chan bool)
ch6 := make(chan []int)

// 操作
// 通道有发送（send）、接收(receive）和关闭（close）三种操作
// 发送和接收都使用<-符号
ch := make(chan int)
ch <- 10    // 把10发送到ch中
x := <- ch  // 从ch中接收值并赋值给变量x
<-ch        // 从ch中接收值，忽略结果
close(ch)   // 关闭

// 只有在通知接收方goroutine所有的数据都发送完毕的时候才需要关闭通道
// 通道是可以被垃圾回收机制回收的
// 对一个关闭的通道再发送值就会导致panic
// 对一个关闭的通道进行接收会一直获取值直到通道为空
// 对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值
// 关闭一个已经关闭的通道会导致panic
```

### 无缓冲的通道
无缓冲的通道又称为阻塞的通道
```go
func recv(c chan int) {
	ret := <-c
	fmt.Println("接收成功", ret)
}

func main() {
	ch := make(chan int)  // 创建的是无缓冲的通道
	// 无缓冲的通道只有在有人接收值的时候才能发送值
	go recv(ch) // 启用goroutine从通道接收值
	ch <- 10
	// 无缓冲通道上的发送操作会阻塞，直到另一个goroutine在该通道上执行接收操作才能发送成功
	// 两个goroutine将继续执行
	// 如果接收操作先执行，接收方的goroutine将阻塞，直到另一个goroutine在该通道上发送一个值
	fmt.Println("发送成功")
	// 使用无缓冲通道进行通信将导致发送和接收的goroutine同步化。因此，无缓冲通道也被称为同步通道
}
```

### 有缓冲的通道
只要通道的容量大于零，那么该通道就是有缓冲的通道，通道的容量表示通道中能存放元素的数量
使用内置的 `len()` 函数获取通道内元素的数量，使用 `cap()` 函数获取通道的容量
```go
func main() {
	ch := make(chan int, 1) // 创建一个容量为1的有缓冲区通道
	ch <- 10
	fmt.Println("发送成功")
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	// 开启goroutine将0~100的数发送到ch1中
	go func() {
		for i := 0; i < 100; i++ {
			ch1 <- i
		}
		close(ch1)
	}()
	// 开启goroutine从ch1中接收值，并将该值的平方发送到ch2中
	go func() {
		for {
			i, ok := <-ch1
			if !ok {
				break
			}
			ch2 <- i * i
		}
		close(ch2)
	}()
	// 在主goroutine中从ch2中接收值打印
	for i := range ch2 {  // 通常使用的是for range的方式
		fmt.Println(i)
	}
}
```

### 单向通道
有的时候我们会将通道作为参数在多个任务函数间传递，很多时候我们在不同的任务函数中使用通道都会对其进行限制，比如只能发送或只能接收
```go
func counter(out chan<- int) {
	for i := 0; i < 100; i++ {
		out <- i
	}
	close(out)
}

// chan<- int是一个只能发送的通道，可以发送但是不能接收
// <-chan int是一个只能接收的通道，可以接收但是不能发送
func squarer(out chan<- int, in <-chan int) {
	for i := range in {
		out <- i * i
	}
	close(out)
}
func printer(in <-chan int) {
	for i := range in {
		fmt.Println(i)
	}
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go counter(ch1)
	go squarer(ch2, ch1)
	printer(ch2)
}
// 在函数传参及任何赋值操作中将双向通道转换为单向通道是可以的，但反过来是不可以的
```

### select多路复用
在某些场景下我们需要同时从多个通道接收数据。通道在接收数据时，如果没有数据可以接收将会发生阻塞
Go 内置了 `select` 关键字，可以同时响应多个通道的操作；`select` 的使用类似于 `switch` 语句，它有一些列 `case` 分支和一个默认的分支。每个 `case` 会对应一个通道的通信（接收或发送）过程。`select` 会一直等待，直到某个 `case` 的通信操作完成时，就会执行 `case` 分支对应的语句
```go
select{
    case <-ch1:
        ...
    case data := <-ch2:
        ...
    case ch3<-data:
        ...
    default:
        // 默认操作
}
// 如果多个case同时满足，select会随机选择一个。对于没有case的select{}会一直等待

func main() {
	ch := make(chan int, 1)
	for i := 0; i < 10; i++ {
		select {
		case x := <-ch:
			fmt.Println(x)
		case ch <- i:
		}
	}
}
```

## 并发安全
可能会存在多个 `goroutine` 同时操作一个资源（临界区），这种情况会发生竞态问题（数据竞态）
```go
var x int64
var wg sync.WaitGroup

func add() {
	for i := 0; i < 5000; i++ {
		x = x + 1
	}
	wg.Done()
}
func main() {
	wg.Add(2)
	go add()
	go add()
	wg.Wait()
	fmt.Println(x)
}
```

### 互斥锁
互斥锁是一种常用的控制共享资源访问的方法，它能够保证同时只有一个 `goroutine` 可以访问共享资源
使用 sync 包的 `Mutex` 类型来实现互斥锁
```go
var x int64
var wg sync.WaitGroup
var lock sync.Mutex

func add() {
	for i := 0; i < 5000; i++ {
		lock.Lock() // 加锁
		x = x + 1
		lock.Unlock() // 解锁
	}
	wg.Done()
}
func main() {
	wg.Add(2)
	go add()
	go add()
	wg.Wait()
	fmt.Println(x)
}
```
使用互斥锁能够保证同一时间有且只有一个 `goroutine` 进入临界区，其他的 `goroutine` 则在等待锁；当互斥锁释放后，等待的 `goroutine` 才可以获取锁进入临界区，多个 `goroutine` 同时等待一个锁时，唤醒的策略是随机的

- 读写互斥锁

互斥锁是完全互斥的，但是有很多实际的场景下是读多写少的，当我们并发的去读取一个资源不涉及资源修改的时候是没有必要加锁的，这种场景下使用读写锁是更好的一种选择

读写锁使用 sync 包中的 `RWMutex` 类型

读写锁分为两种：读锁和写锁

当一个 `goroutine` 获取读锁之后，其他的 `goroutine` 如果是获取读锁会继续获得锁，如果是获取写锁就会等待；当一个 `goroutine` 获取写锁之后，其他的 `goroutine` 无论是获取读锁还是写锁都会等待
```go
var (
	x      int64
	wg     sync.WaitGroup
	lock   sync.Mutex
	rwlock sync.RWMutex
)

func write() {
	// lock.Lock()   // 加互斥锁
	rwlock.Lock() // 加写锁
	x = x + 1
	time.Sleep(10 * time.Millisecond) // 假设读操作耗时10毫秒
	rwlock.Unlock()                   // 解写锁
	// lock.Unlock()                     // 解互斥锁
	wg.Done()
}

func read() {
	// lock.Lock()                  // 加互斥锁
	rwlock.RLock()               // 加读锁
	time.Sleep(time.Millisecond) // 假设读操作耗时1毫秒
	rwlock.RUnlock()             // 解读锁
	// lock.Unlock()                // 解互斥锁
	wg.Done()
}

func main() {
	start := time.Now()
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go write()
	}

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go read()
	}

	wg.Wait()
	end := time.Now()
	fmt.Println(end.Sub(start))
}
// 读写锁非常适合读多写少的场景
```

- `sync.Once`

 延迟一个开销很大的初始化操作到真正用到它的时候再执行是一个很好的实践
 
 对初始化工作添加互斥锁会引起性能问题，`sync` 包中提供了一个针对一次性初始化问题的解决方案

```go
// sync.Once其实内部包含一个互斥锁和一个布尔值，互斥锁保证布尔值和数据的安全，而布尔值用来记录初始化是否完成
// 保证初始化操作的时候是并发安全的并且初始化操作也不会被执行多次

// sync.Once只有一个Do方法
func (o *Once) Do(f func()) {}

// 如果要执行的函数f需要传递参数就需要搭配闭包来使用
var icons map[string]image.Image

var loadIconsOnce sync.Once

func loadIcons() {
	icons = map[string]image.Image{
		"left":  loadIcon("left.png"),
		"up":    loadIcon("up.png"),
		"right": loadIcon("right.png"),
		"down":  loadIcon("down.png"),
	}
}

// Icon 是并发安全的
func Icon(name string) image.Image {
	loadIconsOnce.Do(loadIcons)
	return icons[name]
}
```

- sync.Map
```go
// 内置的map不是并发安全的
var m = make(map[string]int)

func get(key string) int {
	return m[key]
}

func set(key string, value int) {
	m[key] = value
}

func main() {
	wg := sync.WaitGroup{}
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go func(n int) {
			key := strconv.Itoa(n)
			set(key, n)
			fmt.Printf("k=:%v,v:=%v\n", key, get(key))
			wg.Done()
		}(i)  // 当并发多了之后执行上面的代码就会报fatal error: concurrent map writes错误
	}
	wg.Wait()
}

// 需要为map加锁来保证并发的安全性,不用像内置的map一样使用make函数初始化就能直接使用
// sync.Map内置了诸如Store、Load、LoadOrStore、Delete、Range等操作方法

var m = sync.Map{}

func main() {
	wg := sync.WaitGroup{}
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go func(n int) {
			key := strconv.Itoa(n)
			m.Store(key, n)
			value, _ := m.Load(key)
			fmt.Printf("k=:%v,v:=%v\n", key, value)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

## 原子操作
代码中的加锁操作因为涉及内核态的上下文切换会比较耗时、代价比较高；针对基本数据类型我们还可以使用原子操作来保证并发安全，因为原子操作是 Go 语言提供的方法它在用户态就可以完成，因此性能比加锁操作更好
原子操作由内置的标准库`sync/atomic`提供

```go
// 读取操作
func LoadInt32(addr *int32) (val int32)
func LoadInt64(addr *int64) (val int64)
func LoadUint32(addr *uint32) (val uint32)
func LoadUint64(addr *uint64) (val uint64)
func LoadUintptr(addr *uintptr) (val uintptr)
func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)

// 写入操作
func StoreInt32(addr *int32, val int32)
func StoreInt64(addr *int64, val int64)
func StoreUint32(addr *uint32, val uint32)
func StoreUint64(addr *uint64, val uint64)
func StoreUintptr(addr *uintptr, val uintptr)
func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)

// 修改操作
func AddInt32(addr *int32, delta int32) (new int32)
func AddInt64(addr *int64, delta int64) (new int64)
func AddUint32(addr *uint32, delta uint32) (new uint32)
func AddUint64(addr *uint64, delta uint64) (new uint64)
func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)

// 交换操作
func SwapInt32(addr *int32, new int32) (old int32)
func SwapInt64(addr *int64, new int64) (old int64)
func SwapUint32(addr *uint32, new uint32) (old uint32)
func SwapUint64(addr *uint64, new uint64) (old uint64)
func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)
func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)

// 比较并交换操作
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)
func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)
func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)
func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)
func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)

var x int64
var l sync.Mutex
var wg sync.WaitGroup

// 普通版加函数
func add() {
	// x = x + 1
	x++ // 等价于上面的操作
	wg.Done()
}

// 互斥锁版加函数
func mutexAdd() {
	l.Lock()
	x++
	l.Unlock()
	wg.Done()
}

// 原子操作版加函数
func atomicAdd() {
	atomic.AddInt64(&x, 1)
	wg.Done()
}

func main() {
	start := time.Now()
	for i := 0; i < 10000; i++ {
		wg.Add(1)
		// go add()       // 普通版add函数 不是并发安全的
		// go mutexAdd()  // 加锁版add函数 是并发安全的，但是加锁性能开销大 3.9889ms
		go atomicAdd() // 原子操作版add函数 是并发安全，性能优于加锁版 2.9914ms
	}
	wg.Wait()
	end := time.Now()
	fmt.Println(x)
	fmt.Println(end.Sub(start))
}
```