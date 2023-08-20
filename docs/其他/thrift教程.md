# thrift教程

[toc]

## 下载thrift

去[官网](https://thrift.apache.org/download)下载thrift，Windows可以直接下载`.exe`文件，然后修改文件名为`thrift.exe`，并配置环境变量。



## 编写IDL文件

参考[官方教程](https://github.com/apache/thrift/blob/master/tutorial/tutorial.thrift)，即可编写IDL代码。（真的很简单！）



## 生成指定语言的文件

```bash
# 格式：thrift -gen 语言 文件名.thrift
# 案例如下
thrift -gen java text.thrift
thrift -gen go text.thrift
thrift -gen py text.thrift
```



## 重要文件

Iface：服务端通过实现Iface接口，想客户端提供具体的同步业务逻辑。

AsyncIface：服务端通过实现Iface接口，想客户端提供具体的异步业务逻辑。

Client：客户端通过Client的实例对象，以同步的方式访问服务端提供的服务方法。

AsyncClient：客户端通过Client的实例对象，以异步的方式访问服务端提供的服务方法。



IDL语法

基本类型

| Type   | Desc                      | JAVA             | GO      |
| ------ | ------------------------- | ---------------- | ------- |
| i8     | 有符号的8位整数           | byte             | int8    |
| i16    | 有符号的16位整数          | short            | int16   |
| i32    | 有符号的32位整数          | int              | int32   |
| i64    | 有符号的64位整数          | long             | int64   |
| double | 64位浮点数                | double           | float64 |
| bool   | 布尔值                    | boolean          | bool    |
| string | 文本字符串(UTF-8编码格式) | java.lang.String | string  |

集合容器

| Type      | Desc                             | Java           | Go      |                               |
| --------- | -------------------------------- | -------------- | ------- | ----------------------------- |
| list<T>   | 元素有序列表，允许重复           | java.util.List | []T     |                               |
| set<T>    | 元素无序列表，不允许重复         | java.util.Set  | []T     | Go没有set集合，所以用数组代替 |
| map<K, V> | key-value结构数据，key不允许重复 | java.util.Map  | map[K]V |                               |

struct类型

```idl
struct <结构体名称> {
	<序号>: [字段性质] <字段类型> <字段名称> [= <默认值>] [;|,]
}
```

案例：

```idl
struct Uesr {
	1: required string name,
	2: optional i32 age,
	3: bool gender,  // 默认字段类型是optional
	4: string note = "thrift"
}
```

struct不能继承，但是可以嵌套，不能嵌套自己。

成员分割可以用逗号或分好，可以混用

编号不能重复。

同一个文件可以有多个struct，不同文件可以通过include引入struct。



枚举

thrift不支持枚举嵌套，枚举常量必须是32位正整数

```idl
enum HTTPStatus {
	OK = 200,
	NOTFOUND = 404
}
```



异常

异常继承每种语言的基础异常类。

```idl
exception MyException {
	1: i32 errorCode,
	2: string message
}

service ExampleService {
	string GetName() throws (1: MyException e)
}
```



service

服务相当于接口，里面存放函数。

```idl
service HelloService {
	i32 sayInt(1: i32 param)
	string sayString(1: string param)
	bool sayBoolean(1: bool param)
	void sayVoid()
}
```



命名空间

```idl
namespace java com.example.test  // namespace + 语言 + 包名
```



include

```idl
include "test.thrift"
```





```
thrift -gen 语言 文件名.thrift
```

