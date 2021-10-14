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

如果需要在处理程序外部持久化这样的值，请使用复制内置来复制它们的底层缓冲区。以下是持久化字符串的示例：