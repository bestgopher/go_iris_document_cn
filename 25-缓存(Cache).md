有时缓存路由的静态内容是很重要的，因为这样可以使你的web程序性能更快，不会花时间在重构响应上。

这里有两个方式实现 HTTP 缓存。一个是在服务器端存储每个处理器的内容，另一个是检查请求头，然后发送 `304 not modified`，以便让浏览器或者任何兼容的客户端自己来处理缓存。

服务器和客户端缓存的方式，Iris 都提供了，通过 `iris/cache` 子包来实现的，这个子包提供了 `iris.Cache` 和 `iris.Cache304` 中间件。

----------

### Cache
`Cache` 是一个中间件，为接下来的处理器提供了服务器端缓存功能，可以这样来使用：`app.Get("/", iris.Cache(time.Hour), aboutHandler)`。

`Cache` 仅仅只需接收一个参数：缓存的生存时间。如果生存时间无效或者 <= 2秒，会从 `cache-control's maxage` 头部获取生存时间。

```go
func Cache(expiration time.Duration) Handler
```

所有类型的响应都可以被缓存，模板，json，文本，任何类型。

使用它来达到服务器端缓存，查看 `Cache304` 获取另外的方法可能会更加适合你的需求。

有关更多选项和自定义，请使用kataras / iris / cache.Cache，它返回一个结构，您可以从中添加或删除“规则”。

----------


### NoCache

`NoCache` 是一个中间件，它重写了 `Cache-Control`，`Pragma` 和 `Expires` 头部以便在浏览器使用前进和后退功能期间禁用缓存。

在 HTML 路由上使用这个中间件；即使在浏览器的“后退”和“前进”箭头按钮上也可以刷新页面。

```go
func NoCache(Context)
```

查看 `StaticCache` 获取相反的行为。

----------

### StaticCache

 `StaticCache` 返回一个中间件，通过向客户端发送 `Cache-Control` 和 `Expires` 头来缓存静态文件。它仅接受一个参数，是一个 `time.Duration` 类型的，用于计算有效期。

如果 `cacheDur` <= 0，则返回 `Nocache` 中间件来禁用浏览器在前进和后退行为时的缓存。

使用：`app.Use(iris.StaticCache(24 * time.Hour))` 或者 `app.Use(iris.StaticCache(-1))`


```go
func StaticCache(cacheDur time.Duration)Handler
```

一个中间件，是一个简单的处理器，可以在其他处理器内部调用，例如：

```go
 cacheMiddleware := iris.StaticCache(...)

 func(ctx iris.Context){
  cacheMiddleware(ctx)
  [...]
}
```

### Cache304

`Cache304` 返回一个中间件，没当 `If-Modify-Since` 请求头(时间值) 在 time.Now() + 生存时间之前，就会发送 `StatusNotModified(304)`。

所有兼容HTTP RCF的客户端(所有的浏览器和类似postman的工具)将会正确地处理缓存。

这个方法的唯一缺点就是将会发送一个 304 的状态码而不是 200，因此，如果您将其与其他微服务一起使用，则必须检查该状态码以及有效的响应。

开发人员可以通过手动查看系统目录的更改并根据文件修改日期使用 `ctx.WriteWithExpiration`（带有“ modtime”）来自由扩展此方法的行为，类似于 `HandleDir`（发送状态为OK（200）和浏览器磁盘缓存的方式， 304）。

```go
func Cache304(expiresEvery time.Duration) Handler
```