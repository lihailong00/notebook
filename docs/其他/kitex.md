# kitex教程

[toc]

去官网，安装。（注意配置go proxy）

编写IDL代码：

文件名：echo.thrift

内容：

```idl
namespace go api

struct Request {
	1: string message
}

struct Response {
	1: string message
}

service MyService {
	Response echo(1: Request req)
}
```



用kitex生成代码：

```
语法：
kitex -module 随便写一个module名 -service 服务名 thrift文件
案例：
kitex -module mykitex -service MyService hello.thrift
```



生成如下文件

```
.
|-- build.sh
|-- go.mod
|-- handler.go
|-- kitex.yaml
|-- kitex_gen
|   `-- api
|       |-- hello.go
|       |-- k-consts.go
|       |-- k-hello.go
|       `-- myservice
|           |-- client.go
|           |-- invoker.go
|           |-- myservice.go
|           `-- server.go
|-- main.go
`-- script
    `-- bootstrap.sh
```



kitex基本使用：

实现handler.go文件中的代码

```go
```

