<font color=red>**`Iris`**</font> 具有用于路由的表达语法，感觉就像在家一样。路由算法是强大的，通过
[muxie](https://github.com/kataras/muxie "https://github.com/kataras/muxie") 来处理请求和比其替代品(httprouter、gin、echo) 更快地匹配路由。

让我们没有任何麻烦地开始吧！

新建一个空文件，命名为 `example.go`，然后打开它复制粘贴下面的代码。

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.Default()
    app.Use(myMiddleware)

    app.Handle("GET", "/ping", func(ctx iris.Context) {
        ctx.JSON(iris.Map{"message": "pong"})
    })

    // Listens and serves incoming http requests
    // on http://localhost:8080.
    app.Run(iris.Addr(":8080"))
}

func myMiddleware(ctx iris.Context) {
    ctx.Application().Logger().Infof("Runs before %s", ctx.Path())
    ctx.Next()
}
```

打开终端会话，执行下面的指令，然后再浏览器上访问 `http://localhost:8000/ping`：

```shell
$ go run example.go
```

## 展示更多(Show me more)
我们简单地启动和运行程序来获取一个页面。

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()
    // Load all templates from the "./views" folder
    // where extension is ".html" and parse them
    // using the standard `html/template` package.
    app.RegisterView(iris.HTML("./views", ".html"))

    // Method:    GET
    // Resource:  http://localhost:8080
    app.Get("/", func(ctx iris.Context) {
        // Bind: {{.message}} with "Hello world!"
        ctx.ViewData("message", "Hello world!")
        // Render template file: ./views/hello.html
        ctx.View("hello.html")
    })

    // Method:    GET
    // Resource:  http://localhost:8080/user/42
    //
    // Need to use a custom regexp instead?
    // Easy;
    // Just mark the parameter's type to 'string'
    // which accepts anything and make use of
    // its `regexp` macro function, i.e:
    // app.Get("/user/{id:string regexp(^[0-9]+$)}")
    app.Get("/user/{id:uint64}", func(ctx iris.Context) {
        userID, _ := ctx.Params().GetUint64("id")
        ctx.Writef("User ID: %d", userID)
    })

    // Start the server using a network address.
    app.Run(iris.Addr(":8080"))
}
```

html 文件：`./views/hello.html`

```html
<html>
<head>
    <title>Hello Page</title>
</head>
<body>
    <h1>{{ .message }}</h1>
</body>
</html>
```

想要当你的代码发生改变时自动重启你的程序？安装 [rizla](https://github.com/kataras/rizla) 工具，执行 `rizla main.go` 来替代 `go run main.go`。