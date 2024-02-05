# Go依赖管理



## 基础规则

- 建议使用Go Module管理依赖。Go 1.14以后默认采用Go Module管理依赖，因此建议选择新版的go。同时在goland中开启enable go modules integration。
- Go Module类似Maven。
- 我们可以在任何目录创建Go项目，创建时使用``go mod init``即可在根目录生成``go.mod``文件，``go.mod``类似Java的``pom.xml``，用于记录依赖项目。
- 依赖代码一般下载到``$GOPATH/pkg/mod``中，比如我在``go.mod``中引入一个依赖：

```Go
require github.com/cloudwego/hertz v0.7.2
```

则会将依赖的代码放在``$GOPATH/pkg/mod/github.com/cloudwego/hertz@0.7.2``中。

- go install一般安装的是**可执行程序**（而不是依赖代码），安装位置在在``$GOPATH/bin``中。比如我想使用hz工具自动根据thrift文件生成hertz代码，则需要保证安装hz工具，并且将``$GOPATH/bin``加入到环境变量中。

- 同一package但不同go文件的函数和全局变量可以直接调用。

- 不同pachage的函数和全局变量调用时，需要引入依赖路径（通常goland会自动引入），依赖路径的规则是``模块名/包1(/包2...)``。模块名是``go.mod``中的module名字。比如有如下层次的文件：

  - ```Go
    .
    ├── doc1
    │   └── func1.go
    ├── doc2
    │   └── func2.go
    └── go.mod
    ```

go.mod的配置如下：

```Go
module github.com/lihailong/demo

go 1.20
```

那么func1.go需要引入func2.go中的代码时，参考如下方式即可。

```Go
import "github.com/lihailong/demo/doc2"  // github.com/lihailong/demo 是 go.mod中，module名

func func1() {
    doc2.Func2()
}
```

- GOPATH下还有个src目录，在之前是为了存放程序员自己编写的代码，但是引入Go Module后，程序员可以在任何位置编写自己的go代码。但是通常我会将GO代码放在``$GOPATH/src``下面，并且在该目录下创建**三层目录**作为项目的根目录。例如我某个项目的根目录是：``$GOPATH/src/github.com/longcoding/demo``，三层目录是``github.com/longcoding/demo``，命名规范是：代码托管平台名+人命+项目名。不过最新版Go可以在任何目录下编写代码。



## 作业

测验：在没有安装GO的Linux系统下，从0开始运行一个hertz项目。

重要点：环境变量中配置GOPATH和GOROOT。