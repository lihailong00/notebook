# socket编程之基于tcp通信

> TCP客户端

```go
package main

import (
	"fmt"
	"net"
	"os"
)

func main() {
	// 建立TCP连接
	conn, err := net.Dial("tcp", "127.0.0.1:8888")
	if err != nil {
		fmt.Println("连接失败：", err)
		os.Exit(1)
	}
	defer conn.Close()

	// 发送消息
	msg := "Hello, server!"
	_, err = conn.Write([]byte(msg))
	if err != nil {
		fmt.Println("发送消息失败：", err)
		os.Exit(1)
	}

	// 接收响应
	buf := make([]byte, 1024)
	n, err := conn.Read(buf)
	if err != nil {
		fmt.Println("接收响应失败：", err)
		os.Exit(1)
	}
	response := string(buf[:n])
	fmt.Println("收到响应：", response)
}
```



> TCP服务端

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	// 监听本地8888端口
	listener, err := net.Listen("tcp", ":8888")
	if err != nil {
		fmt.Println("Error listening:", err.Error())
		return
	}
	defer listener.Close()

	fmt.Println("Server started, waiting for clients...")

	// 循环等待客户端连接
	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("Error accepting:", err.Error())
			continue
		}

		fmt.Println("Client connected:", conn.RemoteAddr().String())

		// 创建一个goroutine处理客户端请求
		go handleRequest(conn)
	}
}

func handleRequest(conn net.Conn) {
	defer conn.Close()

	// 读取客户端发送的数据
	buf := make([]byte, 1024)
	// n为读取的数据长度
	n, err := conn.Read(buf)
	if err != nil {
		fmt.Println("Error reading:", err.Error())
		return
	}

	// 将收到的数据转为字符串并打印
	data := string(buf[:n])
	fmt.Println("Received data:\n", data)

	// 给客户端发送响应
	response := "Hello, client!"
	_, err = conn.Write([]byte(response))
	if err != nil {
		fmt.Println("Error writing:", err.Error())
		return
	}
}
```

