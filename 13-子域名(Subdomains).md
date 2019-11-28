<font color=red>**`Iris`**</font> 具有已知最简单的注册子域名到单个应用程序的方式。当然你也可以在生成环境中使用 `nginx` 或者 `caddy`。

子域名被分为两类：**静态** 和 **动态/通配**。

- `静态`：你所知的子域名，例如：`analytics.mydomain.com`。
- `通配`：翻译不通(when you don't know the subdomain but you know that it's before a particular subdomain or root domain, i.e : )，即：`user_created.mydomain.com`，`otheruser.mydomain.com`，就像 `username.github.io` 这样。

我们使用 `iris.Party` 或者 `iris.Application` 的 `Subdomain` 和 `WildcardSubdomain` 方法注册子域名。

`Subdomain` 方法返回的是一个新的 `Party` 对象，它负责为特定的子域名注册路由。

与常规 `Party` 不同的是，如果子 party 调用它，域名将会被添加到路径的前面，而不是追加到路径后面。因此如果 `app.Sundomain("admin").Subdomain("panel")`，结果是：`panel.admin`。

	Subdomain(subdomain string, middleware ...Handler) Party

`WildcardSubdomain` 方法返回一个新的 `Party`，它负责注册路由到一个动态的，通配的子域名中。一个动态的子域名能处理多个子域名请求。服务器将会接受多个子域名(如果没有找到对应的静态子域名)，它也会搜索和和执行这个 `Party` 的处理器。

```go
WildcardSubdomain(middleware ...Handler) Party
```

示例代码：

```go
// [app := iris.New...]
admin := app.Subdomain("admin")

// admin.mydomain.com
admin.Get("/", func(ctx iris.Context) {
    ctx.Writef("INDEX FROM admin.mydomain.com")
})

// admin.mydomain.com/hey
admin.Get("/hey", func(ctx iris.Context) {
    ctx.Writef("HEY FROM admin.mydomain.com/hey")
})

// [other routes here...]

app.Run(iris.Addr("mydomain.com:80"))
```

对于本地开发系统，你要修改你的 `hosts` 文件，例如在windows操作系统中，打开 `C:\Windows\System32\Drivers\etc\hosts` 文件，然后追加：
	
```tex
127.0.0.1 mydomain.com
127.0.0.1 admin.mydomain.com
```

为了证明子域名像其它正则 `Party` 一样工作，你也可以用下面这种另类的方法注册一个子域名：
	
```go
adminSubdomain:= app.Party("admin.")
// or
adminAnalayticsSubdomain := app.Party("admin.analytics.")
// or for a dynamic one:
anySubdomain := app.Party("*.")
```

还有一个 `iris.Application` 方法，允许为子域名创建全局重定向规则。

`SubdomainRedirect` 设置(当使用超过1次时添加)一个路由包装器，它可以使一个(子)域名在执行路由处理器之前尽可能快地重定向(永久重定向)到另一个子域名或根域名。

它接收2个参数，它们是 `from` 和 `to/target` 的位置， `from` 也可以是一个通配的子域名(`app.WildcardSubdomain()`)，`to` 不允许是通配的。当 `to` 不是根域名时 `from`  可以是跟域名，反正亦然。

```go
SubdomainRedirect(from, to Party) Party
```

使用：

```go
www := app.Subdomain("www")
app.SubdomainRedirect(app, www)
```

上面的所有 `htt(s)://mydomain.com/%anypath%` 将会重定向到 `https(s)://www.mydomain.com/%anypath%`。

当你使用子域名时，`Context`提供了四个对你很有帮助的主要方法。

```go
 // Host returns the host part of the current url.
Host() string
// Subdomain returns the subdomain of this request, if any.
// Note that this is a fast method which does not cover all cases.
Subdomain() (subdomain string)
// IsWWW returns true if the current subdomain (if any) is www.
IsWWW() bool
// FullRqeuestURI returns the full URI,
// including the scheme, the host and the relative requested path/resource.
FullRequestURI() string
```

使用：

```go
func info(ctx iris.Context) {
    method := ctx.Method()
    subdomain := ctx.Subdomain()
    path := ctx.Path()

    ctx.Writef("\nInfo\n\n")
    ctx.Writef("Method: %s\nSubdomain: %s\nPath: %s", method, subdomain, path)
}
```