# Go 白盒测试

Sep 16 2020

---

## 从待测函数开始

假设要写一些数学相关的代码，我们有一个 mymath 包，当前只有一个 sum.go，内容如下：

```go
package mymath

// 计算从 1 到 n 的和
func Sum(n uint) uint {
    var res uint
    for i := uint(1); i <= n; i++ {
        res += i
    }
    return res
}
```

---

## 单元测试

Unit Test， 简称 UT

关注实现逻辑是否正确，是否能按照预期工作

UT 的覆盖率指标比较重要，最理想的情况是覆盖了待测试代码所有可能情况

---

## 第一版 UT

- 创建 sum_test.go 文件

  > 文件名有规范，需要是待测 Go 文件原名 .go 前加 \_test 后缀；测试文件和待测文件放在同一目录下

- 测试文件内容

```go
package mymath

import "testing"

func TestSum0(t *testing.T) {
    res := Sum(10)
    if res != 55 {
        t.Error("failed")
    }
}

func TestSum1(t *testing.T) {
    res := Sum(100)
    if res != 5050 {
        t.Error("failed")
    }
}
```

---

## 运行第一版 UT

进入 sum.go 和 sum_test.go 所在目录，执行如下命令

```shell
go test .
```

会有类似如下回显

```text
ok      xxxxxx/mymath   0.114s
```

这表示 TestSum0 和 TestSum1 两个用例都通过了测试

> 可以故意修改 Sum 函数实现，比如最后返回时加个 1，再执行测试会有类似如下回显：

```text
--- FAIL: TestSum0 (0.00s)
    sum_test.go:8: failed
--- FAIL: TestSum1 (0.00s)
    sum_test.go:15: failed
FAIL
FAIL    xxxxxx/mymath   0.086s
FAIL
```

---

## 第二版 UT

TestSum0 和 TestSum1 的重复代码太多，可以抽取相同的部分，统一起来

```go
func TestSum2(t *testing.T) {
    type info struct {
        input    uint
        expected uint
    }
    cases := []info{
        {input: 10, expected: 55},
        {input: 100, expected: 5050},
    }
    for _, c := range cases {
        res := Sum(c.input)
        if res != c.expected {
            t.Errorf("failed. want: %d, got: %d\n", c.expected, res)
        }
    }
}
```

这么写的好处是什么？

---

## 第三版 UT

info 结构体可以匿名化

```go
func TestSum(t *testing.T) {
    cases := []struct {
        input    uint
        expected uint
    }{
        {input: 10, expected: 55},
        {input: 100, expected: 5050},
    }
    for _, c := range cases {
        res := Sum(c.input)
        if res != c.expected {
            t.Errorf("failed. want: %d, got: %d\n", c.expected, res)
        }
    }
}
```

业界把这种写法称作表格驱动

    可能是会以文件形式写个表格，再解析成 cases 切片来测试，每次只需向表格里加用例
    实际开发，我们只需要向 cases 切片加用例即可，可以叫这种写法为数组驱动或切片驱动~

---

## 第四版 UT

第三版用例写法已经比较完美了，但有个小问题，如果测试失败了，是哪个用例失败了呢？  

> 解决的方法也很简单，可以给匿名结构体加个 string 类型的字段 name，
>
> for 循环时发现结果不符合预期打印当前用例的 name 即可区分

实际上 testing 包有更到位的封装，t.Run 函数就可以携带用例名称参数

```go
func TestSum(t *testing.T) {
    type args struct {
        n uint
    }
    tests := []struct {
        name string
        args args
        want uint
    }{}
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Sum(tt.args.n); got != tt.want {
                t.Errorf("Sum() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

---

注意到标准模板的代码对函数入参也做了抽象，这在参数较多的时候比较好

对于我们的例子，不必定义 args 结构体

修改后的 UT 代码如下：

```go
func TestSum(t *testing.T) {
    tests := []struct {
        name string
        n uint
        want uint
    }{
        {name: "n=10", n: 10, want: 55},
        {name: "n=100", n: 100, want: 5050},
        {name: "n=3", n: 3, want: 7},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Sum(tt.n); got != tt.want {
                t.Errorf("Sum() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

---

## 运行第四版 UT

上边的三个用例，故意写错了第三个，运行看看会不会精确报错：

```shell
go test .
```

```text
--- FAIL: TestSum (0.00s)
--- FAIL: TestSum/n=3 (0.00s)
sum_test.go:18: Sum() = 6, want 7
FAIL
FAIL xxxxxx/mymath 0.087s
FAIL
PS xxxxxx\mymath>

```

非常明确地打印出了出错的待测试函数，待测试用例以及实际运行结果和预期值

---

## UT 覆盖率

要保证所有用例都通过，才能统计 UT 代码的覆盖率

修复上边错误的第三条用例，即把 want 值改成 6 后，执行：

```shell
go test . -cover
```

预期会有类似如下回显：

```text
ok xxxxxx/mymath (cached) coverage: 100.0% of statements
```

对于我们的 Sum 函数，因为只有一个主分支，即使只有一个测试用例，覆盖率都会是 100%

---

## 基准测试

就是压力测试，着重考察代码的性能

如当前的 Sum 函数，时间复杂度 O(n)，并不是一个太好的实现，我们写个基准测试测一下

```go
func BenchmarkSum(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = Sum(10000)
    }
}
```

可以看到基准测试以“Benchmark”开头，后加待测试函数名，参数为 \*testing.B 类型

```shell
go test  -bench="BenchmarkSum"
```

```text
...
BenchmarkSum-12           443883              2667 ns/op
...
```

意为做了 443883 次调用，每次耗时 2667 ns

---

## 重构 Sum

- 用等差数列前 n 项和公式优化 Sum 函数实现，将时间复杂度降低到 O(1)：

  ```go
  func Sum(n uint) uint {
    return (1+n)*n/2
  }
  ```

- 重构后先做单元测试，保证重构后 Sum 按照预期工作

  ```shell
  go test -run="TestSum"
  PASS
  ok      xxxxxx/mymath   0.080s
  ```

- 再看看现在的性能

  ```shell
  go test  -bench="BenchmarkSum"
  ...
  BenchmarkSum-12         1000000000               0.476 ns/op
  ...
  ```

---

## 示例测试

严格来说，这并不算测试，可以作为 Api 使用文档的一部分

> 举个例子，假如对标准库优先队列"container/heap" 的 Api 不太了解,就可以看标准库官方源码
> container/heap 下的 example_intheap_test.go 和 example_pq_test.go,
> 如 example_intheap_test.go 后半部分代码：

```go
...
// This example inserts several ints into an IntHeap, checks the minimum,
// and removes them in order of priority.
func Example_intHeap() {
    h := &IntHeap{2, 1, 5}
    heap.Init(h)
    heap.Push(h, 3)
    fmt.Printf("minimum: %d\n", (*h)[0])
    for h.Len() > 0 {
        fmt.Printf("%d ", heap.Pop(h))
    }
    // Output:
    // minimum: 1
    // 1 2 3 5
}
```

---

注意到 代码后边的 Output 注释，并不是单纯的注释，而是预期输出，可以跑一下 Example_intHeap

```text
xxx\src\container\heap> go test -run="Example_intHeap"
PASS
ok      container/heap  0.080s
```

如果实际运行输出结果与 output 注释里的不一致，则测试不通过

---

## 更进一步

- go test 更多参数及用法，可用以下命令查看：

  ```shell
  go help test
  go help testflag
  ```

- testing.T 及 testing.B 更多 Api

  查看标准库 testing 源码及测试代码

- 基准测试里的 b.N 到底是多少？怎么确定的？

  查看标准库 testing 源码

- 代码有网络、数据库等依赖，怎么打桩测试

  关于 rest 请求的，可以参考下一个问题；另外 github 上有不少 mock 库可参考学习

- Go 长于 web 编程，标准库有没有好用的 rest 请求相关测试模块

  `net/http/httptest` 正是所求
