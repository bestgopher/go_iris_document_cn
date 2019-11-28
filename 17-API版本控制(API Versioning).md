`versioning` 子包为你的 API 提供 `semver ` 版本控制。它实现了书写在 `api-guidelines` 中所有的建议。

 版本比较是通过`go-version` 包比较的。它支持匹配像 `>= 1.0，< 3` 这种比较模式。

```go
import (
    // [...]

    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/versioning"
)
```

##  特性(Features)

- 每个路由版本匹配，一个寻常的 Iris 处理器通过version => 处理器的映射来作 `switch-case` 匹配。
- 每个组的版本路由和弃用API
- 版本匹配类似 `>=1.0，<2.0` 和 `2.0.1`等等形式
- 版本`not found`处理器(可以通过简单的添加版本来自定义。`NotFound`：在映射上自定义 `NotMatchVersionHandler`)
- 版本从 `Accept` 和 `Accept-Version` 头部取回(可以通过中间件自定义)
- 如果版本找到，响应具有 `X-API-Version` 头
- 通过 `Deprecated` 包装器，弃用了自定义 `X-API-Warn`，`X-API-Deprecation-Data`，`X-API-Deprecation-Info` 头部的选项

## 获取版本(Get version)

当前请求的版本通过 `versioning.GetVersion(ctx)` 获得。
默认情况下 `GetVersion` 将尝试读取以下内容：

-  `Accept header, i.e Accept: "application/json; version=1.0"`
- `Accept-Version header, i.e Accept-Version: "1.0"`

你也可以在中间件中通过使用 context 存储值来设置一个自定义的版本。例如：

```go
func(ctx iris.Context) {
    ctx.Values().Set(versioning.Key, ctx.URLParamDefault("version", "1.0"))
    ctx.Next()
}
```

## 将版本与处理程序匹配(Match version to handler)

`versioning.NewMatcher(versioning.Map) iris.Handler` 创建一个简单的处理器，这个处理器决定基于请求的版本，哪个处理器需要被执行。

```go
app := iris.New()

// middleware for all versions.
myMiddleware := func(ctx iris.Context) {
    // [...]
    ctx.Next()
}

myCustomNotVersionFound := func(ctx iris.Context) {
    ctx.StatusCode(404)
    ctx.Writef("%s version not found", versioning.GetVersion(ctx))
}

userAPI := app.Party("/api/user")
userAPI.Get("/", myMiddleware, versioning.NewMatcher(versioning.Map{
    "1.0":               sendHandler(v10Response),
    ">= 2, < 3":         sendHandler(v2Response),
    versioning.NotFound: myCustomNotVersionFound,
}))
```

## 弃用(Deprecation)
使用 `versioning.Deprecated(handler iris.Handler, options versioning.DeprecationOptions) iris.Handler`
函数，你可以标记特定的处理器版本为被弃用的。

```go
v10Handler := versioning.Deprecated(sendHandler(v10Response), versioning.DeprecationOptions{
    // if empty defaults to: "WARNING! You are using a deprecated version of this API."
    WarnMessage string 
    DeprecationDate time.Time
    DeprecationInfo string
})
userAPI.Get("/", versioning.NewMatcher(versioning.Map{
    "1.0": v10Handler,
    // [...]
}))
```

这将会让处理器发送这些头到客户端：

- `"X-API-Warn": options.WarnMessage`
- `"X-API-Deprecation-Date": context.FormatTime(ctx, options.DeprecationDate))`
- `"X-API-Deprecation-Info": options.DeprecationInfo`

如果你不在意时间和日期，你可以使用`versioning.DefaultDeprecationOptions`替代选项。 

## 通过版本分组路由

也可以按版本对路由进行分组。

使用 `versioning.NewGroup(version string) *versioning.Group` 函数你可以创建一个组来注册你的版本路由。`versioning.RegisterGroups(r iris.Party, versionNotFoundHandler iris.Handler, groups ...*versioning.Group)` 必须在最后被调用，为了使用路由被注册到特定的 `Party` 中。


```go
app := iris.New()

userAPI := app.Party("/api/user")
// [... static serving, middlewares and etc goes here].

userAPIV10 := versioning.NewGroup("1.0")
userAPIV10.Get("/", sendHandler(v10Response))

userAPIV2 := versioning.NewGroup(">= 2, < 3")
userAPIV2.Get("/", sendHandler(v2Response))
userAPIV2.Post("/", sendHandler(v2Response))
userAPIV2.Put("/other", sendHandler(v2Response))

versioning.RegisterGroups(userAPI, versioning.NotFoundHandler, userAPIV10, userAPIV2)
```

> 使用上面我们学习的方法，一个中间件仅仅只能被注册到实际的 `iris.Party`，即使用 `versioning.Match`为了检测当 `x` 或者没有版本要求的时候你想使用什么 `code/handler` 来执行。

### 弃用组(Deprecation for Group)
仅在你想要改变你的API消费者的组上调用 `Deprecated(versioning.DeprecationOptions)`，这个指定的版本就被丢弃了。

```go
userAPIV10 := versioning.NewGroup("1.0").Deprecated(versioning.DefaultDeprecationOptions)
```

## 在内部的处理器中手动比较版本

```go
// reports if the "version" is matching to the "is".
// the "is" can be a constraint like ">= 1, < 3".
If(version string, is string) bool
```


```go
// same as `If` but expects a Context to read the requested version.
Match(ctx iris.Context, expectedVersion string) bool

app.Get("/api/user", func(ctx iris.Context) {
    if versioning.Match(ctx, ">= 2.2.3") {
        // [logic for >= 2.2.3 version of your handler goes here]
        return
    }
})
```