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

| 参数          | 类型     | 描述                                                         | 默认值  |
| ------------- | -------- | :----------------------------------------------------------- | ------- |
| Prefork       | `bool`   | 启用[`SO_REUSEPORT`](https://lwn.net/Articles/542629/)，使多个应用程序能够监听同一个端口。**注意：如果启用，则应用程序需要使用 shell 运行，因为 prefork 模式设置环境变量。如果你是用的是 Docker 请确保应用程序运行的是`CMD ./app`或`CMD ["sh", "-c", "/app"]`。更多信息，请查看**[**此评论**](https://github.com/gofiber/fiber/issues/1021#issuecomment-730537971) | `false` |
| ServerHeader  | `string` | 启用具有给定值的服务器 HTTP 标头。                           | `""`    |
| StrictRouting | `bool`   | 启用后，路由器会将`/foo`和`/foo/`视为不同。否则，路由器将`/foo`和/`foo/`视为相同。 | `false` |
| CaseSensitive | `bool`   | 启用后，路由器会将`/Foo`和`/foo`视为不同。禁用时，路由器将`/Foo`和`/foo`视为相同。 | `false` |
| Immutable     | `bool`   | 启用后，`ctx`中返回的所有值都是不可变的。默认情况下，它们在您从处理程序返回之前一直有效；请参阅[问题](https://github.com/gofiber/fiber/issues/185) | `false` |
| UnescapePath  |          | 在为上下文设置路径之前，将路由中的所有编码字符转换回来，以便路由也可以使用URL编码的特殊字符 | `false` |
|               |          |                                                              |         |
|               |          |                                                              |         |
|               |          |                                                              |         |





















