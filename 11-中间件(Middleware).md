当我们谈论 Iris 中的中间件时，我们谈论的是一个 HTTP 请求的生命周期中主处理器代码运行前/后运行的代码。例如，日志中间件可能记录一个传入请求的详情到日志中，然后调用处理器代码，然后再编写有关响应的详细信息到日志中。关于中间件的一件很酷的事情是，这些单元非常灵活且可重复使用。

中间件仅是一个 `Handler` 格式的函数 `func(ctx iris.Context)`，当前一个中间件调用 `ctx.Next()` 方法时，此中间件被执行，这可以用作身份验证，即如果请求验证通过，就调用 `ctx.Next()` 来执行该请求剩下链上的处理器，否则触发一个错误响应。

## 编写一个中间件(Writing a middleware)

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()
    // or app.Use(before) and app.Done(after).
    app.Get("/", before, mainHandler, after)
    app.Run(iris.Addr(":8080"))
}

func before(ctx iris.Context) {
    shareInformation := "this is a sharable information between handlers"

    requestPath := ctx.Path()
    println("Before the mainHandler: " + requestPath)

    ctx.Values().Set("info", shareInformation)
    ctx.Next() // execute the next handler, in this case the main one.
}

func after(ctx iris.Context) {
    println("After the mainHandler")
}

func mainHandler(ctx iris.Context) {
    println("Inside mainHandler")

    // take the info from the "before" handler.
    info := ctx.Values().GetString("info")

    // write something to the client as a response.
    ctx.HTML("<h1>Response</h1>")
    ctx.HTML("<br/> Info: " + info)

    ctx.Next() // execute the "after".
}
```

```shell
$ go run main.go # and navigate to the http://localhost:8080
Now listening on: http://localhost:8080
Application started. Press CTRL+C to shut down.
Before the mainHandler: /
Inside mainHandler
After the mainHandler
```

## 全局范围(Globally)

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()

    // register our routes.
    app.Get("/", indexHandler)
    app.Get("/contact", contactHandler)

    // Order of those calls does not matter,
    // `UseGlobal` and `DoneGlobal` are applied to existing routes
    // and future routes also.
    //
    // Remember: the `Use` and `Done` are applied to the current party's and its children,
    // so if we used the `app.Use/Done before the routes registration
    // it would work like UseGlobal/DoneGlobal in this case,
    // because the `app` is the root "Party".
    app.UseGlobal(before)
    app.DoneGlobal(after)

    app.Run(iris.Addr(":8080"))
}

func before(ctx iris.Context) {
     // [...]
}

func after(ctx iris.Context) {
    // [...]
}

func indexHandler(ctx iris.Context) {
    // write something to the client as a response.
    ctx.HTML("<h1>Index</h1>")

    ctx.Next() // execute the "after" handler registered via `Done`.
}

func contactHandler(ctx iris.Context) {
    // write something to the client as a response.
    ctx.HTML("<h1>Contact</h1>")

    ctx.Next() // execute the "after" handler registered via `Done`.
}
```

你也可以使用 `ExecutionRules` 强制处理器在没有 `ctx.Next()` 的情况下完成执行。你可以这样做：

```go
app.SetExecutionRules(iris.ExecutionRules{
    // Begin: ...
    // Main:  ...
    Done: iris.ExecutionOptions{Force: true},
})
```

## 转化 `http.Handler/HandlerFunc`(Convert http.Handler/HandlerFunc)

使用它们是没有限制的 - 你很自由地使用任何与 `net/http` 包兼容的第三方的中间件。

Iris与其他框架不同，它是 100% 与标准库兼容，这就是为什么多数大型公司信任 Iris，使用 Go 来完成它们的工作流程，如著名的 US Television Network；它是最新的，并且始终与标准库 `net/http` 包保持一致，标准库 `net/http` 在每次 Go 版本更新的时候都会进行优化。


任意用 `net/http` 编写的第三方中间件通过 `iris.FromStd(AThirdPartyMiddleware)` 与 Iris 兼容。记住， `ctx.ResponseWriter()` 和 `ctx.Request()` 返回与 `net/http` 的 `http.Handler` 相同的输入参数。

这里是一系列为 Iris 特定功能创建的 handlers：

#### 内建的

- basic authentication
- Google reCAPTCHA
- localization and internationalization
- request logger
- profiling (pprof)
- recovery


#### 社区创建的

请查看文档。