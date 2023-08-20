# 基于go语言实现http服务器

[toc]

## 写在前面

该项目我们使用go自带的http库，而不是自己解析http报文，并返回请求。因此程序简化了很多。



## 代码

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	// 定义处理请求的函数
	http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, World!")
	})
	http.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "login page")
	})

	// 启动HTTP服务器，监听8080端口
	fmt.Println("Server is listening on port 8080...")
	http.ListenAndServe(":8080", nil)
}
```

