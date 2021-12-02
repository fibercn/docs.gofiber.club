# APP

app 通常代表 Fiber 应用程序

------

## 静态文件

使用`Static()`创建静态文件代理

!> 默认情况下，Static 将返回指定目录下的 index.html 文件。

```go
// Signature
func (app *App) Static(prefix, root string, config ...Static) Router
```

以下示例将代理`./public`下的静态文件

```go
// Example
app.Static("/", "./public")

// => http://localhost:3000/hello.html
// => http://localhost:3000/js/jquery.js
// => http://localhost:3000/css/style.css
```

```go
// Example
// Serve files from multiple directories
app.Static("/", "./public")

// Serve files from "./files" directory:
app.Static("/", "./files")
```

你可以为静态文件设置前缀路径

```go
// Example
app.Static("/static", "./public")

// => http://localhost:3000/static/hello.html
// => http://localhost:3000/static/js/jquery.js
// => http://localhost:3000/static/css/style.css
```

如果你想对静态文件代理进行更多的配置，可以使用`fiber.Static`结构体

```go
// fiber.Static{}

// 自定义配置
type Static struct {
    // 当设置为true时，服务器会尝试通过缓存压缩文件来最大限度地减少CPU使用。
    // 这与 github.com/gofiber/compression 中间件的工作方式不同。
    // 默认为 false
    Compress bool `json:"compress"`

    // 当设置为true时，启用字节范围请求。
    // 默认为 false
    ByteRange bool `json:"byte_range"`

    // 当设置为true时，启用目录浏览。
    // 默认为 false
    Browse bool `json:"browse"`

    // 指定的目录索引文件名。
    // 默认为 “index.html”
    Index string `json:"index"`

    // 非活动文件处理程序的到期时间。
    // 使用负时间。禁用它的持续时间。
    //
    // 默认为10秒
    CacheDuration time.Duration `json:"cache_duration"`

    // 请求头中 Cache-Control 的值
    // 这是在文件响应中设置的。MaxAge 以秒为单位定义。
    //
    // 默认为0秒
    MaxAge int `json:"max_age"`

    // 定义一个函数，当返回true时跳过这个中间件。
    //
    // 默认为 nil
    Next func(c *Ctx) bool
}
```

```go
// Example
// Custom config
app.Static("/", "./public", fiber.Static{
  Compress:      true,
  ByteRange:     true,
  Browse:        true,
  Index:         "john.html"
  CacheDuration: 10 * time.Second,
  MaxAge:        3600,
})
```

------

## 注册路由

```go
// Signature
// HTTP methods
func (app *App) Get(path string, handlers ...Handler) Router
func (app *App) Head(path string, handlers ...Handler) Router
func (app *App) Post(path string, handlers ...Handler) Router
func (app *App) Put(path string, handlers ...Handler) Router
func (app *App) Delete(path string, handlers ...Handler) Router
func (app *App) Connect(path string, handlers ...Handler) Router
func (app *App) Options(path string, handlers ...Handler) Router
func (app *App) Trace(path string, handlers ...Handler) Router
func (app *App) Patch(path string, handlers ...Handler) Router

// Add 允许你为某个方法指定回调函数
func (app *App) Add(method, path string, handlers ...Handler) Router

// 将响应所有 http 方法
// 几乎和app一样。使用但不限于前缀
func (app *App) All(path string, handlers ...Handler) Router
```

```go
// Example
// Simple GET handler
app.Get("/api/list", func(c *fiber.Ctx)error{
  return c.SendString("I'm a GET request!")
})

// Simple POST handler
app.Post("/api/register", func(c *fiber.Ctx) error {
  return c.SendString("I'm a POST request!")
})
```

**Use** 可以用于注册中间件，并且可以设置前缀匹配。这些路由将只匹配每条路径的起点，即 /john 将匹配 /john/doe、/johnnn 等

**Signature**

```go
func (app *App) Use(args ...interface{}) Router
```

**Example**

```go
// 匹配所有请求
app.Use(func(c *fiber.Ctx) error {
    return c.Next()
})

// 将匹配 /api 开头的请求
app.Use("/api", func(c *fiber.Ctx) error {
    return c.Next()
})

// 注册多个回调函数
app.Use("/api",func(c *fiber.Ctx) error {
  c.Set("X-Custom-Header", random.String(32))
    return c.Next()
}, func(c *fiber.Ctx) error {
    return c.Next()
})
```

## 挂载

您可以通过创建`*Mount`来挂载 Fiber 实例

**Signature**

```go
func (a *App) Mount(prefix string, app *App) Router
```

**Example**

```go
func main() {
    micro := fiber.New()
    micro.Get("/doe", func(c *fiber.Ctx) error {
        return c.SendStatus(fiber.StatusOK)
    })

    app := fiber.New()
    app.Mount("/john", micro) // GET /john/doe -> 200 OK

    log.Fatal(app.Listen(":3000"))
}
```

------

## 路由分组

你可以通过创建`*Group`结构体来对路由进行分组

**Signature**

```go
func (app *App) Group(prefix string, handlers ...Handler) Router
```

**Example**

```go
func main() {
  app := fiber.New()

  api := app.Group("/api", handler)  // /api

  v1 := api.Group("/v1", handler)   // /api/v1
  v1.Get("/list", handler)          // /api/v1/list
  v1.Get("/user", handler)          // /api/v1/user

  v2 := api.Group("/v2", handler)   // /api/v2
  v2.Get("/list", handler)          // /api/v2/list
  v2.Get("/user", handler)          // /api/v2/user

  log.Fatal(app.Listen(":3000"))
}
```

------

## Server

服务器返回基础的 [fasthttp server](https://godoc.org/github.com/valyala/fasthttp#Server)

**Signature**

```go
func (app *App) Server() *fasthttp.Server
```

**Example**

```go
func main() {
    app := fiber.New()

    app.Server().MaxConnsPerIP = 1

    // ...
}
```

------

## Stack

此方法返回原始路由器堆栈信息

**Signature**

```go
func (app *App) Stack() [][]*Route
```

**Example**

```go
var handler = func(c *fiber.Ctx) error { return nil }

func main() {
    app := fiber.New()

    app.Get("/john/:age", handler)
    app.Post("/register", handler)

    data, _ := json.MarshalIndent(app.Stack(), "", "  ")
    fmt.Println(string(data))

    app.Listen(":3000")
}
```

**Result**

```yaml
[
  [
    {
      "method": "GET",
      "path": "/john/:age",
      "params": [
        "age"
      ]
    }
  ],
  [
    {
      "method": "HEAD",
      "path": "/john/:age",
      "params": [
        "age"
      ]
    }
  ],
  [
    {
      "method": "POST",
      "path": "/register",
      "params": null
    }
  ]
]
```

------

## Config

`Config()`将返回 app 配置（只读）

**Signature**

```go
func (app *App) Config() Config
```

------

## Handler

`Handler()`将返回可用于提供自定义`*fasthttp.RequestCtx`的服务器处理程序。

**Signature**

```go
func (app *App) Handler() fasthttp.RequestHandler
```

------

## 监听

使用`.Listen()`来设置 http 服务监听的端口号及 ip 地址

**Signature**

```go
func (app *App) Listen(addr string) error
```

**Example**

```go
// Listen on port :8080 
app.Listen(":8080")

// Custom host
app.Listen("127.0.0.1:8080")
```

------

## 使用HTTPS

使用`.ListenTLS()`开启 HTTPS 服务

**Signature**

```go
func (app *App) ListenTLS(addr, certFile, keyFile string) error
```

**Example**

```go
app.ListenTLS(":443", "./cert.pem", "./cert.key");
```

使用`ListenTLS`默认为以下配置(使用`Listener`定义您自己的配置)

```go
&tls.Config{
    MinVersion:               tls.VersionTLS12,
    PreferServerCipherSuites: true,
    Certificates: []tls.Certificate{
        cert,
    },
}
```

------

## Listener

你可以通过自己的[`net.Listener`](https://golang.org/pkg/net/#Listener) 来使用Listener方法。此方法可用于通过自定义 tls.Config 启用 **TLS/HTTPS**

**Signature**

```go
func (app *App) Listener(ln net.Listener) error
```

**Example**

```go
ln, _ := net.Listen("tcp", ":3000")

cer, _:= tls.LoadX509KeyPair("server.crt", "server.key")

ln = tls.NewListener(ln, &tls.Config{Certificates: []tls.Certificate{cer}})

app.Listener(ln)
```

------

## Test

使用`.Test()`来测试你的应用程序。使用此方法创建 `_test.go` 文件，或者当您需要调试你的路由逻辑时。默认超时为1秒如果您想完全禁用超时，请将-1作为第二个参数。

**Signature**

```go
func (app *App) Test(req *http.Request, msTimeout ...int) (*http.Response, error)
```

**Example**

```go
// 使用GET方法创建路由进行测试:
app.Get("/", func(c *fiber.Ctx) error {
  fmt.Println(c.BaseURL())              // => http://google.com
  fmt.Println(c.Get("X-Custom-Header")) // => hi

  return c.SendString("hello, World!")
})

// http.Request
req := httptest.NewRequest("GET", "http://google.com", nil)
req.Header.Set("X-Custom-Header", "hi")

// http.Response
resp, _ := app.Test(req)

// Do something with results:
if resp.StatusCode == fiber.StatusOK {
  body, _ := ioutil.ReadAll(resp.Body)
  fmt.Println(string(body)) // => Hello, World!
}
```

















