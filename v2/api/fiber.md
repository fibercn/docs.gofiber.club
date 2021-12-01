# Fiber

使用 Fiber 包来创建 Fiber 实例

------

## New

使用new来创建一个新的App实例，你可以在创建时传递可选配置

```go
func New(config ...Config) *App
```

```go
// 默认
app := fiber.New()

// ...
```

------

## 配置

你可以通过配置项来创建新的实例

```go
// 自定义配置
app := fiber.New(fiber.Config{
    Prefork:       true,
    CaseSensitive: true,
    StrictRouting: true,
    ServerHeader:  "Fiber",
    AppName: "Test App v1.0.1"
})

// ...
```

### 配置参数

| 参数                         | 类型            | 描述                                                         | 默认值                |
| ---------------------------- | --------------- | :----------------------------------------------------------- | --------------------- |
| Prefork                      | `bool`          | 启用[`SO_REUSEPORT`](https://lwn.net/Articles/542629/)，使多个应用程序能够监听同一个端口。**注意：如果启用，则应用程序需要使用 shell 运行，因为 prefork 模式设置环境变量。如果你是用的是 Docker 请确保应用程序运行的是`CMD ./app`或`CMD ["sh", "-c", "/app"]`。更多信息，请查看**[**此评论**](https://github.com/gofiber/fiber/issues/1021#issuecomment-730537971) | `false`               |
| ServerHeader                 | `string`        | 启用具有给定值的服务器 HTTP 标头。                           | `""`                  |
| StrictRouting                | `bool`          | 启用后，路由器会将`/foo`和`/foo/`视为不同。否则，路由器将`/foo`和/`foo/`视为相同。 | `false`               |
| CaseSensitive                | `bool`          | 启用后，路由器会将`/Foo`和`/foo`视为不同。禁用时，路由器将`/Foo`和`/foo`视为相同。 | `false`               |
| Immutable                    | `bool`          | 启用后，`ctx`中返回的所有值都是不可变的。默认情况下，它们在您从处理程序返回之前一直有效；请参阅[问题](https://github.com/gofiber/fiber/issues/185) | `false`               |
| UnescapePath                 | `bool`          | 在为上下文设置路径之前，将路由中的所有编码字符转换回来，以便路由也可以使用URL编码的特殊字符 | `false`               |
| ETag                         | `bool`          | 启用或禁用ETag报头生成。因为弱eTag和强eTag都是使用相同的散列方法(CRC-32)生成的，所以启用时，默认设置为弱eTag。 | `false`               |
| BodyLimit                    | `int`           | 设置请求的`body`体的最大允许大小，如果大小超过限制，则返回`413 - Request Entity Too Large`。 | `4 * 1024 * 1024`     |
| Concurrency                  | `int`           | 并发连接的最大数量。                                         | `256 * 1024`          |
| Views                        | `Views`         | 视图。查看我们的**模板中间件**了解支持的模板引擎。           | `nil`                 |
| ReadTimeout                  | `time.Duration` | 允许读取完整请求(包括`body`)的时间量。默认超时为无限制。     | `nil`                 |
| WriteTimeout                 | `time.Duration` | 响应写入超时之前的最长持续时间。默认超时为无限制。           | `nil`                 |
| IdleTimeout                  | `time.Duration` | 启用长连接时等待下一个请求的最长时间。如果IdleTimeout为零，则使用ReadTimeout的值。 | `nil`                 |
| ReadBufferSize               | `int`           | 用于请求读取时的每个连接的缓冲区大小。这也限制了`header`大小。如果您的客户端发送多字节 RequestURI 或多字节`header`(例如，大cookies)，则增加此缓冲区。 | 4096                  |
| WriteBufferSize              | `int`           | 用于响应写入时的每个连接的缓冲区大小                         | 4096                  |
| CompressedFileSuffix         | `string`        | 为原始文件名添加后缀，并尝试用新文件名保存生成的压缩文件。   | `".fiber.gz"`         |
| ProxyHeader                  | `string`        | 这将使`c.IP()`返回`header`中指定的key的值。默认情况下，`c.IP()`将从 TCP 连接返回远程 IP，如果你使用了负载平衡，例如`X-Forwarded-*`，此属性会很有用。 | `""`                  |
| GETOnly                      | `bool`          | 如果设置为`true`，则拒绝所有非GET请求。这个选项对于只接受GET请求的服务器来说是非常有用的反DoS保护。如果启用了GETOnly，请求大小将受到ReadBufferSize的限制。 | `false`               |
| ErrorHandler                 | `ErrorHandler`  | 当从fiber.Handler返回错误时，将执行ErrorHandler。已安装的fiber错误处理程序由顶级应用程序保留，并应用于前缀关联的请求。 | `DefaultErrorHandler` |
| DisableKeepalive             | `bool`          | 禁用长连接，服务器将在响应客户端后关闭连接                   | `false`               |
| DisableDefaultDate           | `bool`          | 设置为true时，会将默认`date`从响应的`header`中移除           | `false`               |
| DisableDefaultContentType    | `bool`          | 设置为true时，会导致默认`Content-Type`从响应的`header`中移除。 | `false`               |
| DisableHeaderNormalizing     | `bool`          | 默认情况下，`header`中所有参数名都是规范化的: conteNT-tYPE -> Content-Type | `false`               |
| DisableStartupMessage        | `bool`          | 设置为true时，不会打印出调试信息                             | `false`               |
| AppName                      | `string`        | 为应用程序设置应用程序名称                                   | `""`                  |
| EnableTrustedProxyCheck      | `bool`          | 当设置为true时，fiber 将根据 TrustedProxies 列表检查代理是否可信。<br />默认情况下，`c.Protocol()`将从`header`中的`X-Forwarded-Proto`、`X-Forwarded-Protocol`、`X-Forwarded-Ssl`或`X-Url-Scheme`中获取值；`c.IP()`将从`ProxyHeader`中获取值；`c.Hostname()`将从`X-Forwarded-Host`头中获取值。<br />如果`Enabletrustedproyccheck`为true，并且`RemoteIP`在 TrustedProxies 列表中，`c.Protocol()`、`c.IP()`和`c.Hostname()`在Enabletrustedproyccheck被禁用时将具有相同的行为；如果`RemoteIP`不在列表中，`c.Protocol()`将在tls连接由应用程序处理时返回https，否则返回http；`c.IP()`将从fasthttp context返回`RemoteIP()`，`c.Hostname()`将返回`fasthttp.Request.URI().Host()` | `false`               |
| TrustedProxies               | `[]string`      | 包含可信代理IP的列表。查看`EnableTrustedProxyCheck`文档。 <br />它可以采用IP或IP范围地址。如果它得到IP范围，它迭代所有可能的地址。 | `[]string*__*`        |
| DisablePreParseMultipartForm | `bool`          | 如果设置为真，将不会预分析多部分表单数据。对于希望将多部分表单数据视为二进制blob或选择何时解析数据的服务器，此选项非常有用。 | `false`               |
| StreamRequestBody            | `bool`          | StreamRequestBody启用请求body流处理，当给定的body大于当前限制时，会更快地调用处理程序。 | `false`               |

------

## NewError

使用`.NewError()`创建一个新的HTTPError实例。

```go
func NewError(code int, message ...string) *Error
```

```go
app.Get("/", func(c *fiber.Ctx) error {
    return fiber.NewError(782, "Custom error message")
})
```

------

## IsChild

使用`.IsChild()`确定当前进程是否是子进程。

```go
func IsChild() bool
```

```go
// Prefork will spawn child processes
app := fiber.New(fiber.Config{
    Prefork: true,
})

if !fiber.IsChild() {
    fmt.Println("I'm the parent process")
} else {
    fmt.Println("I'm a child process")
}

// ...
```

