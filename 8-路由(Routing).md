## 处理器类 (The Handler type)

处理器，物如其名，就是处理请求的东西。

一个处理器响应一个 HTTP 请求。它写入响应头和数据到 `Context.ResponseWriter()`，然后再返回。返回信号表明请求已经完成；在处理完成调用当时或者之后使用 `Context` 是无效的。

由于 HTTP 客户端软件，HTTP 协议版本，和任何客户端和 iris 服务器中间媒介等因素，在向 `context.ResponseWriter()` 写入后可能无法从 `context.Request().Body` 中读取数据。注意应该先从`context.Request().Body` 中读取数据后，然后再响应它。

除了读取请求体，处理器不应该改变提供的 `context`

如果处理器出现 `panic`，服务器(处理器的调用者)会假定这个 panic 的影响与存活的请求无关。它会 recover 这个 panic，记录栈追踪日志到服务器错误日志中，并中断连接。

```go
type Handler func(iris.Context)
```

一旦处理器被注册，我们可以给返回的 `路由` 实例指定一个名字，以便更加容易地调试和在视图中匹配相对路径。更多信息，查看 `反向查询(Reverse Lookups)` 章节。

## 行为(Behavior)

<font color=red>**`Iris`**</font> 默认接受和注册形如 `/api/user` 这样的路径的路由，且尾部不带斜杠。如果客户端尝试访问 `$your_host/api/user/`，<font color=red>**`Iris`**</font> 路由会自动永久重定向(301)到 `$your_host/api/user`，以便由注册的路由进行处理。这是设计 APIs的现代化的方式。


然而如果你想禁用请求的 `路径更正` 的功能的话，你可以在 `app.Run` 传递 `iris.WithoutPathCorrection` 配置选项。例如：

```go
 // [app := iris.New...]
// [...]

app.Run(iris.Addr(":8080"), iris.WithoutPathCorrection)
```

如果你想 `/api/user` 和 `/api/user/` 在不重定向的情况下(常用场景)拥有相同的处理器，只需要 `iris.WithoutPathCorrectionRedirection` 选项即可：

```go
app.Run(iris.Addr(":8080"), iris.WithoutPathCorrectionRedirection)
```


## API

支持所有的 HTTP 方法，开发者也可以在相同路劲的不同的方法注册处理器(比如 `/user` 的 GET 和 POST)。


第一个参数是 HTTP 方法，第二个参数是请求的路径，第三个可变参数应该包含一个或者多个`iris.Handler`，当客户端请求到特定的资源路径时，这些处理器将会按照注册的顺序依次执行。

示例代码：

```html
app := iris.New()

app.Handle("GET", "/contact", func(ctx iris.Context) {
    ctx.HTML("<h1> Hello from /contact </h1>")
})
```

为了让后端开发者做事更加容易， <font color=red>**`Iris`**</font> 为所有的 HTTP 方法提供了"帮手"。第一个参数是路由的请求路径，第二个可变参数是一个或者多个 `iris.Handler`，也会按照注册的顺序依次执行。

示例代码

```go
app := iris.New()

// Method: "GET"
app.Get("/", handler)

// Method: "POST"
app.Post("/", handler)

// Method: "PUT"
app.Put("/", handler)

// Method: "DELETE"
app.Delete("/", handler)

// Method: "OPTIONS"
app.Options("/", handler)

// Method: "TRACE"
app.Trace("/", handler)

// Method: "CONNECT"
app.Connect("/", handler)

// Method: "HEAD"
app.Head("/", handler)

// Method: "PATCH"
app.Patch("/", handler)

// register the route for all HTTP Methods
app.Any("/", handler)

func handler(ctx iris.Context){
    ctx.Writef("Hello from method: %s and path: %s\n", ctx.Method(), ctx.Path())
}
```

## 离线路由(Offline Routes)

在  <font color=red>**`Iris`**</font> 中有一个特殊的方法你可以使用。它被成为 `None`，你可以使用它向外部隐藏一条路由，但仍然可以从其他路由处理中通过 `Context.Exec` 方法调用它。每个 API 处理方法返回 Route 值。一个 Route 的 `IsOnline` 方法报告那个路由的当前状态。你可以通过它的 `Route.Method` 字段的值来改变路由 `离线` 状态为 `在线` 状态，反之亦然。当然每次状态的改变需要调用 `app.RefreshRouter()` 方法，这个使用是安全的。看看下面一个完整的例子：

```go
// file: main.go
package main

import (
    "github.com/kataras/iris/v12"
)

func main() {
    app := iris.New()

    none := app.None("/invisible/{username}", func(ctx iris.Context) {
        ctx.Writef("Hello %s with method: %s", ctx.Params().Get("username"), ctx.Method())

        if from := ctx.Values().GetString("from"); from != "" {
            ctx.Writef("\nI see that you're coming from %s", from)
        }
    })

    app.Get("/change", func(ctx iris.Context) {

        if none.IsOnline() {
            none.Method = iris.MethodNone
        } else {
            none.Method = iris.MethodGet
        }

        // refresh re-builds the router at serve-time in order to
        // be notified for its new routes.
        app.RefreshRouter()
    })

    app.Get("/execute", func(ctx iris.Context) {
        if !none.IsOnline() {
            ctx.Values().Set("from", "/execute with offline access")
            ctx.Exec("NONE", "/invisible/iris")
            return
        }

        // same as navigating to "http://localhost:8080/invisible/iris"
        // when /change has being invoked and route state changed
        // from "offline" to "online"
        ctx.Values().Set("from", "/execute")
        // values and session can be
        // shared when calling Exec from a "foreign" context.
        //     ctx.Exec("NONE", "/invisible/iris")
        // or after "/change":
        ctx.Exec("GET", "/invisible/iris")
    })

    app.Run(iris.Addr(":8080"))
}
```

#### 怎么运行
1. 运行 `go run main.go`
2. 打开浏览器访问 `http://localhost:8080/invisible/iris`，你将会看到 `404 not found` 的错误。
3. 然而 `http://localhost:8080/execute` 将会执行这个路由。
4. 现在，如果你导航至 `http://localhost:8080/change` ，然后刷新`/invisible/iris` 选项卡，你将会看到它。


## 路由组(Grouping Routes)

一些列路由可以通过路径的前缀(可选的)进行分组，共享相同的中间件处理器和模板布局。一个组也可以有一个内嵌的组。

`.Party` 用来路由分组，开发者可以声明不限数量的组。

示例代码：

```go
app := iris.New()

users := app.Party("/users", myAuthMiddlewareHandler)

// http://localhost:8080/users/42/profile
users.Get("/{id:uint64}/profile", userProfileHandler)
// http://localhost:8080/users/messages/1
users.Get("/messages/{id:uint64}", userMessageHandler)
```

你可以使用 `PartyFunc` 方法编写相同的内容，它接受子路由器或者Party。

```go
app := iris.New()

app.PartyFunc("/users", func(users iris.Party) {
    users.Use(myAuthMiddlewareHandler)

    // http://localhost:8080/users/42/profile
    users.Get("/{id:uint64}/profile", userProfileHandler)
    // http://localhost:8080/users/messages/1
    users.Get("/messages/{id:uint64}", userMessageHandler)
})
```

## 路径参数(Path Parameters)

与你见到的其他路由器不同， Iris 的路由器可以处理各种路由路径而不会发生冲突。

只匹配 `GET "/"`

```go
app.Get("/", indexHandler)
```

下面所示能匹配所有以 `/assets/**/*` 前缀的 GET 请求，它是一个通配符，`ctx.Params().Get("asset")`获取 `/assets/` 后面的所有路径。

```go
app.Get("/assets/{asset:path}", assetsWildcardHandler)
```

下面所示的能匹配所有以 `/profile/` 前缀的 GET 请求，但是获取的只是单个路径的部分。

```go
app.Get("/profile/{username:string}", userHandler)
```

下面所示的只能匹配 `/profile/me` 的 GET 请求，它不与 `/profile/{username:string}` 或者 `/{root:path}` 冲突。

```go
app.Get("/profile/me", userHandler)
```


下面所示的能匹配所有以 `/users/` 前缀的 GET 请求，并且后面的是一个数字，且数字要大于等于1。

```go
app.Get("/user/{userid:int min(1)}", getUserHandler)
```


下面所示的能匹配所有以 `/users/` 前缀的 DELETE 请求，并且后面的是一个数字，且数字要大于等于1。

```go
app.DELETE("/user/{userid:int min(1)}", getUserHandler)
```

下面所示的能匹配所有除了被其他路由器处理的 GET 请求。例如在这种情况下，上面的路由 ：`/`，`/assets/{asset:path}`，`/profile/{username}`，`/profile/me`，`/user/{userid:int ...}`，它将不会与余下的路由冲突。

```go
app.Get("{root:path}", rootWildcardHandler)
```


匹配所有的请求：
1. `/u/adcd` 映射为 `:alphabetical` (如果 `:alphabetical` 注册， 否则 `:string`)

2. `/u/42` 映射为 `:uint ` (如果 `:uint ` 注册， 否则 `:int`)

3. `/u/-1` 映射为 `:int` (如果 `:int ` 注册， 否则 `:string`)

4.  `/u/adcd123` 映射为 `:string` 

	```go
	app.Get("/u/{username:string}", func(ctx iris.Context) {
	    ctx.Writef("username (string): %s", ctx.Params().Get("username"))
	})
	
	app.Get("/u/{id:int}", func(ctx iris.Context) {
	    ctx.Writef("id (int): %d", ctx.Params().GetIntDefault("id", 0))
	})
	
	app.Get("/u/{uid:uint}", func(ctx iris.Context) {
	    ctx.Writef("uid (uint): %d", ctx.Params().GetUintDefault("uid", 0))
	})
	
	app.Get("/u/{firstname:alphabetical}", func(ctx iris.Context) {
	    ctx.Writef("firstname (alphabetical): %s", ctx.Params().Get("firstname"))
	})
	```
	
	

分别匹配 `/abctenchars.xml` 和 `/abcdtenchars` 的所有 GET 请求

```go
app.Get("/{alias:string regexp(^[a-z0-9]{1,10}\\.xml$)}", PanoXML)
app.Get("/{alias:string regexp(^[a-z0-9]{1,10}$)}", Tour)
```

你可能知道 `{id:uint64}`、`:path`、`min(1)` 是什么。它们是在注册时可以键入的动态参数和函数。了解更多请阅读 `路径参数类型(Path Parameter Types)`
