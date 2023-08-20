# go基础

[toc]



## 结构体

```go
package main

import "fmt"

type user struct {
	name     string
	password string
}

func (u user) printName() {
	fmt.Println(u.name)
	fmt.Println(u.password)
}

// 不建议使用指针
func (u *user) clearValue() {
	u.name = "姓名为空"
	u.password = "密码被清空"
}

func main() {
	u := user{"lhl", "123"}
	u.printName()
	u.clearValue()
	u.printName()
}
```



## 异常

```go
package main

import (
	"errors"
	"fmt"
)

type User struct {
	name string
	age  int
}

func findUser(users []User, name string) (u *User, err error) {
	for _, v := range users {
		if v.name == name {
			return &v, nil
		}
	}
	return nil, errors.New("user not found")
}

func main() {
	users := []User{{"lhl", 8}, {"jiangly", 18}, {"tourist", 28}}
	user, err := findUser(users, "lhl")
	if user == nil {
		fmt.Println(err)
	} else {
		fmt.Println("成功找到此人！姓名为：", user.name, "年龄为：", user.age)
	}
}
```



## 字符串操作

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	a := "hello"
	fmt.Println(strings.Contains(a, "ll"))                // true
	fmt.Println(strings.Count(a, "l"))                    // 2
	fmt.Println(strings.HasPrefix(a, "he"))               // true
	fmt.Println(strings.HasSuffix(a, "llo"))              // true
	fmt.Println(strings.Index(a, "ll"))                   // 2
	fmt.Println(strings.Join([]string{"he", "llo"}, "-")) // he-llo
	fmt.Println(strings.Repeat(a, 2))                     // hellohello
	fmt.Println(strings.Replace(a, "l", "L", -1))         // heLLo
	fmt.Println(strings.Split("a-b-c", "-"))              // [a, b, c]
	fmt.Println(strings.ToLower(a))                       // hello
	fmt.Println(strings.ToUpper(a))                       // HELLO
	b := "你好"
	fmt.Println(len(b)) // 6
}
```



## 字符串格式化

```go
package main

import "fmt"

type point struct {
	x, y int
}

func main() {
	s := "hello"
	n := 123
	p := point{1, 2}
	fmt.Println(s, n) // hello 123
	fmt.Println(p)    // {1 2}

	// %v 可以打印任何类型的变量
	fmt.Printf("s=%v\n", s)  // s=hello
	fmt.Printf("n=%v\n", n)  // n=123
	fmt.Printf("p=%v\n", p)  // p={1 2}
	fmt.Printf("p=%+v\n", p)  // p={x:1 y:2}
	fmt.Printf("p=%#v\n", p)  // p=main.point{x:1, y:2}

	f := 3.141592653
	fmt.Println(f)          // 3.141592653
	fmt.Printf("%.2f\n", f) // 3.14
}
```



## 数字解析

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	f, _ := strconv.ParseFloat("121.234", 64)
	fmt.Println(f) // 121.234

	n, _ := strconv.ParseInt("100001", 2, 32)
	fmt.Println(n) // 33

	n2, _ := strconv.Atoi("123")
	fmt.Println(n2) // 123

	n2, err := strconv.Atoi("AAA")
	fmt.Println(n2, err) // 0 strconv.Atoi: parsing "AAA": invalid syntax
}
```





## 猜字谜

```go
package main

import (
	"bufio"
	"fmt"
	"math/rand"
	"os"
	"strconv"
	"strings"
	"time"
)

func main() {
	// 设定取值范围
	maxNum := 100
	// 初始化随机数函数
	rand.Seed(time.Now().UnixNano())
	ans := rand.Intn(maxNum)
	// 获取输入对象
	reader := bufio.NewReader(os.Stdin)

	for {
		fmt.Printf("请输入您猜测的数：")
		// 读取一行，包含\n
		data, err := reader.ReadString('\n')
		if err != nil {
			fmt.Println("读取错误")
			break
		}
		// 去掉末尾\n
		data = strings.TrimSuffix(data, "\r\n")
		// 将字符串转换成int
		num, err := strconv.Atoi(data)
		if err != nil {
			fmt.Println("data=", data)
			fmt.Println("num=", num)
			fmt.Println("err=", err)
			fmt.Println("请输入整数！")
			continue
		}

		if num < ans {
			fmt.Println("你猜小啦！")
		} else if num > ans {
			fmt.Println("你猜大啦！")
		} else {
			fmt.Println("恭喜你猜测正确！")
			break
		}
	}
}
```





## 电子词典

```go
package main

import (
	"bufio"
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"strings"
)

// 使用marshal函数将结构体导出为json时，只会导出以大写字母开头的变量
type DictRequest struct {
	TransType string `json:"trans_type"`
	Source    string `json:"source"`
	UserId    string `json:"user_id"`
}

type DictResponse struct {
	Rc   int `json:"rc"`
	Wiki struct {
		KnownInLaguages int `json:"known_in_laguages"`
		Description     struct {
			Source string      `json:"source"`
			Target interface{} `json:"target"`
		} `json:"description"`
		ID   string `json:"id"`
		Item struct {
			Source string `json:"source"`
			Target string `json:"target"`
		} `json:"item"`
		ImageURL  string `json:"image_url"`
		IsSubject string `json:"is_subject"`
		Sitelink  string `json:"sitelink"`
	} `json:"wiki"`
	Dictionary struct {
		Prons struct {
			EnUs string `json:"en-us"`
			En   string `json:"en"`
		} `json:"prons"`
		Explanations []string      `json:"explanations"`
		Synonym      []string      `json:"synonym"`
		Antonym      []string      `json:"antonym"`
		WqxExample   [][]string    `json:"wqx_example"`
		Entry        string        `json:"entry"`
		Type         string        `json:"type"`
		Related      []interface{} `json:"related"`
		Source       string        `json:"source"`
	} `json:"dictionary"`
}

func query(word string) {
	client := &http.Client{}
	request := DictRequest{TransType: "en2zh", Source: word, UserId: "63c136bd49885f0014cba93c"}
	buf, err := json.Marshal(request)
	if err != nil {
		log.Fatal(err)
	}
	// data 是 io reader的一种
	data := bytes.NewReader(buf)

	req, err := http.NewRequest("POST", "https://api.interpreter.caiyunai.com/v1/dict", data)
	if err != nil {
		log.Fatal(err)
	}
	req.Header.Set("authority", "api.interpreter.caiyunai.com")
	req.Header.Set("accept", "application/json, text/plain, */*")
	req.Header.Set("accept-language", "zh-CN,zh;q=0.9,en;q=0.8")
	req.Header.Set("app-name", "xy")
	req.Header.Set("content-type", "application/json;charset=UTF-8")
	req.Header.Set("device-id", "")
	req.Header.Set("origin", "https://fanyi.caiyunapp.com")
	req.Header.Set("os-type", "web")
	req.Header.Set("os-version", "")
	req.Header.Set("referer", "https://fanyi.caiyunapp.com/")
	req.Header.Set("sec-ch-ua", `"Not?A_Brand";v="8", "Chromium";v="108", "Google Chrome";v="108"`)
	req.Header.Set("sec-ch-ua-mobile", "?0")
	req.Header.Set("sec-ch-ua-platform", `"Windows"`)
	req.Header.Set("sec-fetch-dest", "empty")
	req.Header.Set("sec-fetch-mode", "cors")
	req.Header.Set("sec-fetch-site", "cross-site")
	req.Header.Set("user-agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36")
	req.Header.Set("x-authorization", "token:qgemv4jr1y38jyq6vhvi")
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	bodyText, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	if resp.StatusCode != 200 {
		log.Fatal("bad StatusCode:", resp.StatusCode, "body", string(bodyText))
	}
	var dictResponse DictResponse
	err = json.Unmarshal(bodyText, &dictResponse)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(word, "UK:", dictResponse.Dictionary.Prons.En, "US:", dictResponse.Dictionary.Prons.EnUs)
	for _, item := range dictResponse.Dictionary.Explanations {
		fmt.Println(item)
	}
}

func main() {
	fmt.Printf("请输入单词：")
	reader := bufio.NewReader(os.Stdin)
	str, err := reader.ReadString('\n')
	if err != nil {
		fmt.Println("读取错误")
		return
	}
	str = strings.TrimSuffix(str, "\r\n")
	query(str)
}
```



## 服务器&客户端通信

```go
package main

import (
	"bufio"
	"log"
	"net"
)

func process(conn net.Conn) {
	defer conn.Close()
	// reader 是带缓冲的流
	reader := bufio.NewReader(conn)
	for {
		b, err := reader.ReadByte()
		if err != nil {
			break
		}
		_, err = conn.Write([]byte{b})
		if err != nil {
			break
		}
	}
}

// 首先运行这个程序，此时开启服务器
// 安装NCcat，并配置环境变量，然后才能使用nc命令
// 然后执行命令"nc 127.0.0.1 1080"让客户端发起TCP或UDP连接
// 客户端中输入一段字符，服务器接收到后会将字符返回给客户端
func main() {
	server, err := net.Listen("tcp", "127.0.0.1:1080")
	if err != nil {
		panic(err)
	}
	for {
		client, err := server.Accept()
		if err != nil {
			log.Printf("Accept fail %v\n", err)
			continue
		}
		// go 开启一个线程
		go process(client)
	}
}
```



## SOCKS5 代理协议

[参考GitHub项目](https://github.com/wangkechun/go-by-example)

 



## 协程

```go
package main

import (
	"fmt"
	"time"
)

func printInfo(x int) {
	fmt.Println("this is:", x)
}

func main() {
	for i := 1; i <= 5; i++ {
		go func(j int) {
			printInfo(j)
		}(i)
	}
	time.Sleep(time.Second)
}
```





## 管道

```go
package main

import "fmt"

func calcSquare() {
	// 无缓冲的channel
	src := make(chan int)
	// 带缓冲的channel，可解决生产者速度大于消费者速度的问题
	dest := make(chan int, 5)

	go func() {
		defer close(src)
		for i := 1; i <= 10; i++ {
			src <- i
		}
	}()

	go func() {
		defer close(dest)
		for i := range src {
			dest <- i * i
		}
	}()

	for i := range dest {
		fmt.Println(i)
	}
}

func main() {
	calcSquare()
}
```





## 并发安全Lock

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	x    int32
	lock sync.Mutex
)

func addWithLock() {
	for i := 0; i < 2000; i++ {
		lock.Lock()
		x++
		lock.Unlock()
	}
}

func addWithoutLock() {
	for i := 0; i < 2000; i++ {
		x++
	}
}

func main() {
	for i := 0; i < 5; i++ {
		go addWithLock()
	}
	
	time.Sleep(time.Second)
	fmt.Println("with lock, x=", x)
	x = 0
	
	for i := 0; i < 5; i++ {
		go addWithoutLock()
	}
	fmt.Println("without lock, x=", x)
}
```





## waitgroup

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	wg := sync.WaitGroup{}
	wg.Add(5)
	for i := 1; i <= 5; i++ {
		go func(j int) {
			defer wg.Done()
			fmt.Printf("j=%v\n", j)
		}(i)
	}
	wg.Wait()
}
```





## 单元测试

> judgement.go

```go
package main

func Judgement(score int8) bool {
	if score >= 60 {
		return true
	}
	return false
}
```



> judgement_test.go

```go
package main

import (
	// goland 会在go.mod文件中引入相关依赖
	"github.com/stretchr/testify/assert"
	"testing"
)

// 测试函数必须以Test开头
func TestJudgement(t *testing.T) {
	output := Judgement(80)
	assert.Equal(t, true, output)
}
```



多种测试方法：

1. 普通测试：`go test judgement_test.go judgement.go`
2. 测试覆盖率：`go test judgement_test.go judgement.go --cover `
   1. 一般覆盖率为50%~60%
   2. 较高覆盖率为80%



## 桩函数

问：为什么要用桩函数？

答：

对于我们平时开发的业务代码，单个函数往往不是独立的，它需要依赖于其他模块、第三方库、数据库、消息交互的结果等等。

对于这种代码做单元测试，就会变得复杂许多，而对于当前要测试的函数来说，这些被依赖的其他函数，无非就是返回不同的数据而已。所以在做单元测试的时候，<u>我们只需要让这些被依赖的其他函数返回我们期望的数据，就可以继续测试我们当前需要测试的函数。</u>

打桩是指**在测试包中创建一个模拟方法，用于替换生成代码中的方法。**



案例：假定我们有以下代码

> process.go

```go
package main

import (
	"bufio"
	"os"
	"strings"
)

func ReadFirstLine() string {
	open, err := os.Open("log")
	defer open.Close()
	if err != nil {
		return ""
	}
	scanner := bufio.NewScanner(open)
	for scanner.Scan() {
		return scanner.Text()
	}
	return ""
}

func ProcessFirstLine() string {
	line := ReadFirstLine()
	destLine := strings.ReplaceAll(line, "11", "00")
	return destLine
}
```

> process_test.go

```go
package main

import (
	"github.com/stretchr/testify/assert"
	"testing"
)

func TestProcessFirstLine(t *testing.T) {
	firstLine := ProcessFirstLine()
	assert.Equal(t, "line00", firstLine)
}
```

> log

```
line00
```

我们的目的是测试函数`ProcessFirstLine`。但是函数`ProcessFirstLine`依赖于函数`ReadFirstLine`，而函数`ReadFirstLine`强依赖于`log`文件，当`log`文件变化时，`ReadFirstLine`函数也会输出不同的值，从而影响`ProcessFirstLine`函数。

因为测试的函数是`ProcessFirstLine`，其余函数都不重要。所以我们它依赖的函数输出一个不变的值，从而方便测试。

> 仅有process_test.go文件发生变化

```go
package main

import (
	"bou.ke/monkey"
	"github.com/stretchr/testify/assert"
	"testing"
)

func TestProcessFirstLine(t *testing.T) {
	monkey.Patch(ReadFirstLine, func() string {
		return "line110"
	})
	defer monkey.Unpatch(ReadFirstLine)
	firstLine := ProcessFirstLine()
	assert.Equal(t, "line000", firstLine)
}
```

通过桩函数拜托了对本地文件的强依赖。





## 基准测试/压力测试

需要提前`go get github.com/bytedance/gopkg `

> select.go

```go
package main

import "github.com/bytedance/gopkg/lang/fastrand"

var servers [10]int

func InitServers() {
	for i := 0; i < 10; i++ {
		servers[i] = 101 + i
	}
}

func Select() int {
	return servers[fastrand.Intn(10)]
}
```



> select_test.go

```go
package main

import (
	"testing"
)

// 串行压力测试
// 压测函数必须以Benchmark开头
func BenchmarkSelect(b *testing.B) {
	InitServers()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		Select()
	}
}

// 并行压力测试
func BenchmarkSelectParallel(b *testing.B) {
	InitServers()
	b.ResetTimer()
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			Select()
		}
	})
}
```





## Gin实战

安装依赖：`go get -u github.com/gin-gonic/gin`

