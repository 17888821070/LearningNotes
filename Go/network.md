# 网络编程

## 互联网分层模型

![此处输入图片的描述][1]

越往上的层越靠近用户，越往下的层越靠近硬件

1. 物理层

把电脑连接起来的物理手段，作用是负责传送0和1的电信号

2. 数据链路层（MAC地址）

确定了物理层传输的 0 和 1 的分组方式及代表的意义

以太网规定，一组电信号构成一个数据包，叫做”帧”（Frame），每一帧分成两个部分：标头（Head）和数据（Data）

”标头”包含数据包的一些说明项，比如发送者、接受者、数据类型等等

”数据”则是数据包的具体内容

”标头”的长度，固定为 18 字节；”数据”的长度，最短为 46 字节，最长为 1500 字节;整个”帧”最短为 64 字节，最长为 1518 字节;如果数据很长，就必须分割成多个帧进行发送

连入网络的所有设备都必须具有”网卡”接口，数据包必须是从一块网卡，传送到另一块网卡;网卡的地址，就是数据包的发送地址和接收地址，这叫做 MAC 地址；每块网卡出厂的时候，都有一个全世界独一无二的 MAC 地址，长度是 48 个二进制位，通常用 12 个十六进制数表示
以太网向本网络内所有计算机都发送，让每台计算机读取这个包的”标头”，找到接收方的 MAC 地址，然后与自身的 MAC 地址相比较，如果两者相同，就接受这个包，做进一步处理，否则就丢弃这个包，即广播

3. 网络层（IP信息）

如果两台计算机不在同一个子网络，广播是传不过去的

“网络层”出现以后，每台计算机有了两种地址，一种是 MAC 地址，另一种是网络地址

网络地址帮助我们确定计算机所在的子网络，MAC 地址则将数据包送到该子网络中的目标网卡

规定网络地址的协议，叫做 IP 协议；它所定义的地址，就被称为 IP 地址

IPv4 规定，网络地址由32个二进制位组成，我们通常习惯用分成四段的十进制数表示IP地址，从 0.0.0.0一直到 255.255.255.255

根据 IP 协议发送的数据，就叫做 IP 数据包；分为”标头”和”数据”两个部分：”标头”部分主要包括版本、长度、IP 地址等信息，”数据”部分则是 IP 数据包的具体内容

IP 数据包的”标头”部分的长度为 20 到 60 字节，整个数据包的总长度最大为 65535 字节

4. 传输层（发送端口、接收端口等）

需要一个参数，表示这个数据包到底供哪个程序（进程）使用；参数就叫做”端口”（port），它其实是每一个使用网卡的程序的编号；每个数据包都发到主机的特定端口，所以不同的程序就能取到自己所需要的数据

“端口”是 0 到 65535 之间的一个整数，正好 16 个二进制位；0 到 1023 的端口被系统占用，用户只能选用大于 1023 的端口；有了 IP 和端口我们就能实现唯一确定互联网上一个程序，进而实现网络间的程序通信

在数据包中加入端口信息，最简单的实现叫做 UDP 协议，它的格式几乎就是在数据前面，加上端口号；UDP 数据包，也是由”标头”和”数据”两部分组成：”标头”部分主要定义了发出端口和接收端口，”数据”部分就是具体的内容；UDP 数据包非常简单，”标头”部分一共只有 8 个字节，总长度不超过 65,535 字节，正好放进一个 IP 数据包；UDP 协议的优点是比较简单，容易实现，但是缺点是可靠性较差，一旦数据包发出，无法知道对方是否收到
TCP 协议能够确保数据不会遗失；它的缺点是过程复杂、实现困难、消耗较多的资源；TCP 数据包没有长度限制，理论上可以无限长，但是为了保证网络的效率，通常 TCP 数据包的长度不会超过 IP 数据包的长度，以确保单个 TCP 数据包不必再分割

5. 应用层

应用程序收到”传输层”的数据，接下来就要对数据进行解包；”应用层”的作用就是规定应用程序使用的数据格式

![此处输入图片的描述][2]
  
## socket编程
Socket 是 BSD UNIX 的进程通信机制，通常也称作”套接字”，用于描述 IP 地址和端口，是一个通信链的句柄

Socket 其实就是一个门面模式，它把复杂的 TCP/IP 协议族隐藏在 Socket 后面，对用户来说只需要调用 Socket 规定的相关函数，让 Socket 去组织符合指定的协议数据然后进行通信

![此处输入图片的描述][3]


  [1]: https://www.liwenzhou.com/images/Go/socket/osi.png
  [2]: https://www.liwenzhou.com/images/Go/socket/httptcpip.png
  [3]: https://www.liwenzhou.com/images/Go/socket/socket.png
  
## Go实现服务端
一个 TCP 服务端可以同时连接很多个客户端，Go 中创建多个`goroutine`实现并发非常方便和高效，所以可以每建立一次链接就创建一个`goroutine`去处理
TCP 服务端程序的处理流程：监听端口、接收客户端请求建立链接、创建 `goroutine` 处理链接
```go
// tcp/server/main.go

// TCP server端

// 处理函数
func process(conn net.Conn) {
	defer conn.Close() // 关闭连接
	for {
		reader := bufio.NewReader(conn)
		var buf [128]byte
		n, err := reader.Read(buf[:]) // 读取数据
		if err != nil {
			fmt.Println("read from client failed, err:", err)
			break
		}
		recvStr := string(buf[:n])
		fmt.Println("收到client端发来的数据：", recvStr)
		conn.Write([]byte(recvStr)) // 发送数据
	}
}

func main() {
	listen, err := net.Listen("tcp", "127.0.0.1:20000")
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	for {
		conn, err := listen.Accept() // 建立连接
		if err != nil {
			fmt.Println("accept failed, err:", err)
			continue
		}
		go process(conn) // 启动一个goroutine处理连接
	}
}
```

## Go实现客户端
一个 TCP 客户端进行 TCP 通信的流程如下：建立与服务端链接、进行数据收发、关闭链接
```go
// tcp/client/main.go

// 客户端
func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:20000")
	if err != nil {
		fmt.Println("err :", err)
		return
	}
	defer conn.Close() // 关闭连接
	inputReader := bufio.NewReader(os.Stdin)
	for {
		input, _ := inputReader.ReadString('\n') // 读取用户输入
		inputInfo := strings.Trim(input, "\r\n")
		if strings.ToUpper(inputInfo) == "Q" { // 如果输入q就退出
			return
		}
		_, err = conn.Write([]byte(inputInfo)) // 发送数据
		if err != nil {
			return
		}
		buf := [512]byte{}
		n, err := conn.Read(buf[:])
		if err != nil {
			fmt.Println("recv failed, err:", err)
			return
		}
		fmt.Println(string(buf[:n]))
	}
}
```

## TCP黏包

tcp 数据传递模式是流模式，在保持长连接的时候可以进行多次的收和发

粘包可发生在发送端也可发生在接收端：1.由 Nagle 算法造成的发送端的粘包，当我们提交一段数据给 TCP 发送时，TCP 并不立刻发送此段数据，而是等待一小段时间看看在等待期间是否还有要发送的数据，若有则会一次把这两段数据发送出去；2.接收端接收不及时造成的接收端粘包：TCP 会把接收到的数据存在自己的缓冲区中，然后通知应用层取数据。当应用层由于某些原因不能及时的把 TCP 的数据取出来，就会造成 TCP 缓冲区中存放了几段数据

出现”粘包”的关键在于接收方不确定将要传输的数据包的大小；因此我们可以对数据包进行封包和拆包的操作
封包：封包就是给一段数据加上包头，这样一来数据包就分为包头和包体两部分内容了(过滤非法包时封包会加入”包尾”内容)。包头部分的长度是固定的，并且它存储了包体的长度，根据包头长度固定以及包头中含有包体长度的变量就能正确的拆分出一个完整的数据包


## Go实现UDP服务端
```go
// UDP/server/main.go

// UDP server端
func main() {
	listen, err := net.ListenUDP("udp", &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	defer listen.Close()
	for {
		var data [1024]byte
		n, addr, err := listen.ReadFromUDP(data[:]) // 接收数据
		if err != nil {
			fmt.Println("read udp failed, err:", err)
			continue
		}
		fmt.Printf("data:%v addr:%v count:%v\n", string(data[:n]), addr, n)
		_, err = listen.WriteToUDP(data[:n], addr) // 发送数据
		if err != nil {
			fmt.Println("write to udp failed, err:", err)
			continue
		}
	}
}
```

- Go实现UDP客户端
```go
// UDP 客户端
func main() {
	socket, err := net.DialUDP("udp", nil, &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("连接服务端失败，err:", err)
		return
	}
	defer socket.Close()
	sendData := []byte("Hello server")
	_, err = socket.Write(sendData) // 发送数据
	if err != nil {
		fmt.Println("发送数据失败，err:", err)
		return
	}
	data := make([]byte, 4096)
	n, remoteAddr, err := socket.ReadFromUDP(data) // 接收数据
	if err != nil {
		fmt.Println("接收数据失败，err:", err)
		return
	}
	fmt.Printf("recv:%v addr:%v count:%v\n", string(data[:n]), remoteAddr, n)
}
```