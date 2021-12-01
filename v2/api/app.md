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
    // Optional. Default value false
    ByteRange bool `json:"byte_range"`

    // When set to true, enables directory browsing.
    // Optional. Default value false.
    Browse bool `json:"browse"`

    // The name of the index file for serving a directory.
    // Optional. Default value "index.html".
    Index string `json:"index"`

    // Expiration duration for inactive file handlers.
    // Use a negative time.Duration to disable it.
    //
    // Optional. Default value 10 * time.Second.
    CacheDuration time.Duration `json:"cache_duration"`

    // The value for the Cache-Control HTTP-header
    // that is set on the file response. MaxAge is defined in seconds.
    //
    // Optional. Default value 0.
    MaxAge int `json:"max_age"`

    // Next defines a function to skip this middleware when returned true.
    //
    // Optional. Default: nil
    Next func(c *Ctx) bool
}
```

```
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































