## 监听和开启服务(Listen and Serve)

你可以开启服务监听任何 `net.Listener` 或者 `http.Server` 类型的实例。初始化服务器的方法应该在最后传递给 `Run` 函数。

Go 开发者最常用的方法是通过传递一个形如 `hostname:ip` 形式的网络地址来开启一个服务。<font color=red>**`Iris`**</font> 中我们使用`iris.Addr`，它是一个 `iris.Runner` 类型。

```go
// Listening on tcp with network address 0.0.0.0:8080
app.Run(iris.Addr(":8080"))
```

有时候你在你的应用程序的其他地方创建一个标准库 `het/http` 服务器，并且你想使用它作为你的 Iris web 程序提供服务。

```go
// Same as before but using a custom http.Server which may being used somewhere else too
app.Run(iris.Server(&http.Server{Addr:":8080"}))
```

最高级的用法是创建一个自定义的或者标准的 `net/Listener`，然后传递给 `app.Run`。

```go
// Using a custom net.Listener
l, err := net.Listen("tcp4", ":8080")
if err != nil {
    panic(err)
}
app.Run(iris.Listener(l))
```

一个更加完整的示例，使用的是仅Unix套接字文件特性

```go
package main

import (
    "os"
    "net"

    "github.com/kataras/iris/v12"
)

func main() {
    app := iris.New()

    // UNIX socket
    if errOs := os.Remove(socketFile); errOs != nil && !os.IsNotExist(errOs) {
        app.Logger().Fatal(errOs)
    }

    l, err := net.Listen("unix", socketFile)

    if err != nil {
        app.Logger().Fatal(err)
    }

    if err = os.Chmod(socketFile, mode); err != nil {
        app.Logger().Fatal(err)
    }

    app.Run(iris.Listener(l))
}
```

UNIX 和 BSD 主机可以使用重用端口的功能。

```go
package main

import (
    // Package tcplisten provides customizable TCP net.Listener with various
    // performance-related options:
    //
    //   - SO_REUSEPORT. This option allows linear scaling server performance
    //     on multi-CPU servers.
    //     See https://www.nginx.com/blog/socket-sharding-nginx-release-1-9-1/ for details.
    //
    //   - TCP_DEFER_ACCEPT. This option expects the server reads from the accepted
    //     connection before writing to them.
    //
    //   - TCP_FASTOPEN. See https://lwn.net/Articles/508865/ for details.
    "github.com/valyala/tcplisten"

    "github.com/kataras/iris/v12"
)

// go get github.com/valyala/tcplisten
// go run main.go

func main() {
    app := iris.New()

    app.Get("/", func(ctx iris.Context) {
        ctx.HTML("<h1>Hello World!</h1>")
    })

    listenerCfg := tcplisten.Config{
        ReusePort:   true,
        DeferAccept: true,
        FastOpen:    true,
    }

    l, err := listenerCfg.NewListener("tcp", ":8080")
    if err != nil {
        app.Logger().Fatal(err)
    }

    app.Run(iris.Listener(l))
}
```

## HTTP/2和安全(HTTP/2 and Secure)
如果你有签名文件密钥，你可以使用 `iris.TLS` 基于这些验证密钥开启 `https` 服务。

```go
// TLS using files
app.Run(iris.TLS("127.0.0.1:443", "mycert.cert", "mykey.key"))
```

当你的应用准备部署**生产**时，你可以使用 `iris.AutoTLS` 方法，它通过[ https://letsencrypt.org]( https://letsencrypt.org " https://letsencrypt.org") 免费提供的证书来开启一个安全的服务。
	
```go
// Automatic TLS
app.Run(iris.AutoTLS(":443", "example.com", "admin@example.com"))
```

## 任意iris.Runner(Any iris.Runner)
有时你想要监听一些特定的东西，并且这些东西不是 `net.Listener` 类型的。你能够通过 `iris.Raw`
方法做到，但是你得对此方法负责。

```go
// Using any func() error,
// the responsibility of starting up a listener is up to you with this way,
// for the sake of simplicity we will use the
// ListenAndServe function of the `net/http` package.
app.Run(iris.Raw(&http.Server{Addr:":8080"}).ListenAndServe)
```


## host配置(Host configurators)

形如上面所示的监听方式都可以在最后接受一个 <font color=red>**`func(*iris.Supervisor)`**</font> 的可变参数。通过函数的传递用来为特定 host 添加配置器。

例如，我们想要当服务器关闭的时候触发的回调函数：

```go
app.Run(iris.Addr(":8080", func(h *iris.Supervisor) {
    h.RegisterOnShutdown(func() {
        println("server terminated")
    })
}))
```

你甚至可以在再 `app.Run` 之前配置，但是不同的是，这个 host 配置器将会在所有的主机上执行(我们将在稍后看到 `app.NewHost` )


```go
app := iris.New()
app.ConfigureHost(func(h *iris.Supervisor) {
    h.RegisterOnShutdown(func() {
        println("server terminated")
    })
})
app.Run(iris.Addr(":8080"))
```

当 `Run` 方法运行之后，通过 `Application#Hosts` 字段提供的所有 hosts 你的应用服务都可以访问。

但是最常用的场景是你可能需要在运行 `app.Run` 之前访问 hosts，这里有2中方法来获得访问 hosts 的监管，阅读下面。

我们已经看到通过 `app.Run` 的第二个参数或者 `app.ConfigureHost` 方法来配置所有的应用程序 hosts 。还有一种更加适合简单场景的方法，那就是使用 `app.NewHost` 来创建一个新的 host，然后使用它的 `Serve` 或者 `Listen` 函数， 通过 `iris#Raw` 来启动服务。

记住这个方法需要额外导入 `net/http` 包。

示例代码：

```go
h := app.NewHost(&http.Server{Addr:":8080"})
h.RegisterOnShutdown(func(){
    println("server terminated")
})

app.Run(iris.Raw(h.ListenAndServe))
```


## 多个主机(Multi hosts)
你可以使用多个 hosts 来启动你的 iris 程序，`iris.Router` 兼容 `net/http/Handler` 函数，因此我们可以理解为，它可以被适用于任何 `net/http` 服务器，然而，通过使用 `app.NewHost` 是一个更加简单的方法，它也会复制所有的 host 配置器，并在 `app.Shutdown` 时关闭所有依附在特定web 服务的主机 host 。
	app := iris.New()
	app.Get("/", indexHandler)
	
```go
// run in different goroutine in order to not block the main "goroutine".
go app.Run(iris.Addr(":8080"))
// start a second server which is listening on tcp 0.0.0.0:9090,
// without "go" keyword because we want to block at the last server-run.
app.NewHost(&http.Server{Addr:":9090"}).ListenAndServe()
```

## 优雅地关闭(Shutdown gracefully)

让我们继续学习怎么接受 `CONTROL + C` / `COMMAND + C` 或者 unix `kill` 命令，优雅地关闭服务器。(默认是启用的)

为了手动地管理app被中断时需要做的事情，我们需要通过使用 `WithoutInterruptHandler` 选项禁用默认的行为，然后注册一个新的中断处理器(在所有可能的hosts上)。

示例代码：



```go
package main

import (
    "context"
    "time"

    "github.com/kataras/iris/v12"
)
```

```go
func main() {
    app := iris.New()

    iris.RegisterOnInterrupt(func() {
        timeout := 5 * time.Second
        ctx, cancel := context.WithTimeout(context.Background(), timeout)
        defer cancel()
        // close all hosts
        app.Shutdown(ctx)
    })

    app.Get("/", func(ctx iris.Context) {
        ctx.HTML(" <h1>hi, I just exist in order to see if the server is closed</h1>")
    })

    app.Run(iris.Addr(":8080"), iris.WithoutInterruptHandler)
```
