# 链式编程

httpclient，一个轮子

11 May 2019

---

## 为什么写这个轮子

- 包装标准库
- 不能忍受组内服务现有包装

[httpclient](https://github.com/zrcoder/httpclient)

---

## 常规调用

```go
package main

import (
    hc "github.com/zrcoder/httpclient"
)

func main() {
    p := struct {
        Age  int
        Name string
    }{Age: 27, Name: "Tom"}
    client := hc.New()
    client.Post("http://127.0.0.1:8888/test")
    client.Header("someKey", "someValue")
    client.ContentType(hc.ContentTypeJson)
    client.Body(p)
    resp, body, err := client.Go()
    // do something with resonse
}
```

---

## 链式调用

```go
package main

import (
    hc "github.com/zrcoder/httpclient"
    "net/http"
)

func main() {
    p := struct {
        Age  int
        Name string
    }{Age: 27, Name: "Tom"}

    hc.New().
        Post("http://127.0.0.1:8888/test").
        Header("some key", "some value").
        ContentType(hc.ContentTypeJson).
        Body(p).
        Do(func(response *http.Response, body []byte, err error) {
        // do something with response
    })
}
```

---

## 更多例子

- 以标准库 httptest 包起一个服务，实现对 Person 对象的增删改查
- 用我们的链式库做两次 Put 调用，增加 Tom 和 Joe
- Get 调用查询当前的 Person 列表
- Post 调用修改 Joe 的年龄
- 删除 Tom
- 再次 Get 调用查询当前的 Person 列表

> 详见单元测试文件 [client_test.go](https://github.com/zrcoder/httpclient) 的 `Example` 函数

---

## Body()函数支持的参数

- string
- []byte
- 自定义的结构对象，无需 json 转换

---

## httpclient 如何处理错误

- 用一个数组记录所有错误？
- 只保持第一个产生的错误？

我们用后者
