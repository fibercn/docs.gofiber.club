# 欢迎

Fibre 是一个以 Express 为启发，建立在 Fasthttp 之上的 Web 框架，Fasthttp 是 Go 生态中最快的 HTTP 引擎。旨在简化快速开发的同时实现零内存分配和性能。

## 安装

首先，[下载](https://studygolang.com/dl)并安装Go。需要==1.14==或更高版本。

使用==go get==安装

```bash
go get github.com/gofiber/fiber/v2
```

## 零内存

!> 注意！默认情况下，`*fiber.Ctx`中的值会有被改变的风险

由于 Fibre 针对高性能进行了优化，因此`fiber.Ctx`中的值在默认情况下不是一成不变的，并且将在请求之间重用。通常，您只能在处理程序中**使用**上下文值，而且不要保留任何引用。一旦您从处理程序返回，您从上下文中获得的任何值都将在以后的请求中重用，并将被更改。下面是一个例子：

```go
func handler(c *fiber.Ctx) error {
    // result 变量只在当前 handler 方法中有效
    result := c.Params("foo") 

    // ...
}
```

如果需要在处理程序外部持久化这样的值，请使用内置`copy()`。以下是持久化字符串的示例：

```go
func handler(c *fiber.Ctx) error {
    // result 变量只在当前 handler 方法中有效
    result := c.Params("foo")

  // 使用 copy() 复制缓冲区
    buffer := make([]byte, len(result))
    copy(buffer, result)
    resultCopy := string(buffer) 
    // resultCopy 现在永久有效

    // ...
}
```

可以使用我们提供的`ImmutableString()`方法来简化上述操作，可以在`gofiber/utils`中找到

```go
app.Get("/:foo", func(c *fiber.Ctx) error {
    // result 现在永久有效
    result := utils.ImmutableString(c.Params("foo")) 

    // ...
})
```

或者，您也可以使用**Immutable**设置。它将使从上下文返回的所有值都不可修改，允许您在任何地方持久化它们。当然，这是以性能为代价的。

了解更多，[#426](https://github.com/gofiber/fiber/issues/426)和[#185](https://github.com/gofiber/fiber/issues/185)

## Hello, World

下面是使用 Fiber 创建一个 http 服务

```go
// server.go
package main

import "github.com/gofiber/fiber/v2"

func main() {
  app := fiber.New()

  app.Get("/", func(c *fiber.Ctx) error {
    return c.SendString("Hello, World!")
  })

  app.Listen(":3000")
}
```

```shell
go run server.go
```

打开浏览器访问`http://localhost:3000`

## 基本路由

*路由*用于确定应用程序如何响应对特定端点的客户机请求，包含一个 URI（或路径）和一个特定的 HTTP 请求方法（GET、POST 等）。

!> 每个路由可以具有一个或多个处理程序函数，这些函数在路由匹配时执行

路由定义采用以下结构：

```go
app.Method(path string, ...func(*fiber.Ctx) error)
```

其中：

- `app` 是 `Fiber` 的实例。
- `Method` 是 [HTTP 请求方法](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)。
- `path` 是服务器上的路径。
- `func(*fiber.Ctx) error` 是在路由匹配时执行的函数。

**简单路由**

```go
// 以主页上的 Hello World! 进行响应
app.Get("/", func(c *fiber.Ctx) error {
  return c.SendString("Hello, World!")
})
```

**参数**

```go
// GET http://localhost:8080/hello%20world

app.Get("/:value", func(c *fiber.Ctx) error {
  return c.SendString("value: " + c.Params("value"))
  // => 获取参数值: hello world
})
```

**可选参数**

```go
// GET http://localhost:3000/john

app.Get("/:name?", func(c *fiber.Ctx) error {
  if c.Params("name") != "" {
    return c.SendString("Hello " + c.Params("name"))
    // => Hello john
  }
  return c.SendString("Where is john?")
})
```

**通配符**

```go
// GET http://localhost:3000/api/user/john

app.Get("/api/*", func(c *fiber.Ctx) error {
  return c.SendString("API path: " + c.Params("*"))
  // => API path: user/john
})
```

## 静态文件

为了提供诸如图像、CSS 文件和 JavaScript 文件之类的静态文件，请使用`app.Static()`方法

函数签名

```go
app.Static(prefix, root string)
```

使用下面的代码在`./public`的目录中提供文件

```go
app := fiber.New()

app.Static("/", "./public") 

app.Listen(":3000")
```

现在就可以加载`./public`下的文件了

```go
http://localhost:8080/hello.html
http://localhost:8080/js/jquery.js
http://localhost:8080/css/style.css
```

## 最后

更多关于如何创建`API`的信息可以阅读这篇文章：[Go Fiber 框架系列教程 02：详解相关 API 的使用](https://polarisxu.studygolang.com/posts/go/fiber/go-fiber-basic-tutorial02/)
