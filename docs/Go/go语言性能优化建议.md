# go语言性能优化建议

[toc]

## 尽可能在使用make()初始化切片时提供容量信息

```go
package pro2

import "testing"

func NoPreAlloc(size int) {
	data := make([]int, 0)
	for k := 0; k < size; k++ {
		data = append(data, k)
	}
}

func PreAlloc(size int) {
	data := make([]int, 0, size)
	for k := 0; k < size; k++ {
		data = append(data, k)
	}
}

func BenchmarkNoPreAlloc(b *testing.B) {
	for i := 0; i < b.N; i++ {
		NoPreAlloc(1000)
	}
}

func BenchmarkPreAlloc(b *testing.B) {
	for i := 0; i < b.N; i++ {
		PreAlloc(1000)
	}
}
```

测试指令：

```bash
# cmd中
go test -bench=. -benchmem
# powershell中
go test -bench="." -benchmem
```



测试结果：

```
goos: windows
goarch: amd64
pkg: awesomeProject/pro2
cpu: AMD Ryzen 5 3400G with Radeon Vega Graphics
BenchmarkNoPreAlloc-8             174656              6075 ns/op           25208 B/op         12 allocs/op
BenchmarkPreAlloc-8               629514              2087 ns/op            8192 B/op          1 allocs/op
PASS
ok      awesomeProject/pro2     2.718s
```





## 用copy代替re-slice

在已有切片上创建新的切片，不会创建新的底层数组，导致旧的大切片迟迟不能释放。

```go
import (
	"math/rand"
	"testing"
	"time"
)

func generateWithCap(n int) []int {
	rand.Seed(time.Now().UnixNano())
	nums := make([]int, 0, n)
	for i := 0; i < n; i++ {
		nums = append(nums, rand.Int())
	}
	return nums
}

func GetLastBySlice(origin []int) []int {
	return origin[len(origin)-2:]
}

func GetLastByCopy(origin []int) []int {
	result := make([]int, 2)
	copy(result, origin[len(origin)-2:])
	return result
}

func testGetLast(t *testing.T, f func([]int) []int) {
	result := make([][]int, 0)
	for k := 0; k < 100; k++ {
		origin := generateWithCap(10 * 1024 * 1024)
		result = append(result, f(origin))
	}
}

func TestGetLastBySlice(t *testing.T) {
	testGetLast(t, GetLastBySlice)
}

func TestGetLastByCopy(t *testing.T) {
	testGetLast(t, GetLastByCopy)
}
```



测试指令：

```powershell
go test -run="." -v
```



测试结果：

```powershell
=== RUN   TestGetLastBySlice
--- PASS: TestGetLastBySlice (20.87s)
=== RUN   TestGetLastByCopy
--- PASS: TestGetLastByCopy (18.71s)
PASS
ok      awesomeProject/pro3     40.552s
```



## 使用map时预分配内存

```go
package pro4

import "testing"

func NoPreAlloc(size int) {
	data := make(map[int]int)
	for i := 0; i < size; i++ {
		data[i] = i
	}
}

func PreAlloc(size int) {
	data := make(map[int]int, size)
	for i := 0; i < size; i++ {
		data[i] = i
	}
}

func BenchmarkNoPreAlloc(b *testing.B) {
	for i := 0; i < b.N; i++ {
		NoPreAlloc(1000)
	}
}

func BenchmarkPreAlloc(b *testing.B) {
	for i := 0; i < b.N; i++ {
		PreAlloc(1000)
	}
}
```



测试指令：

```powershell
go test -bench="." -benchmem
```



测试结果：

```
goos: windows
goarch: amd64                                       
pkg: awesomeProject/pro4                            
cpu: AMD Ryzen 5 3400G with Radeon Vega Graphics    
BenchmarkNoPreAlloc-8              15423             76923 ns/op           86543 B/op         64 allocs/op
BenchmarkPreAlloc-8                39940             31000 ns/op           41097 B/op          6 allocs/op
PASS
ok      awesomeProject/pro4     3.748s
```

