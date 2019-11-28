**有时候一个服务器应用程序需要在相同的 URI 上提供资源不同表示形式。**当然这也可以手动实现，检查 `Accept` 请求头，设置推送的请求表单的文本。然而，当你的程序管理更多的资源和不同形式的时候，这将变得非常痛苦，你需要检查 `Accept-Charset`，`Accept-Encoding`，设置一些服务器端优先级，正确处理错误等等。

有一些 Go 的 web 框架已经艰难地实现了这个特性，但是它们不能正确地做以下的事情：

- 不处理 `accept-charset`
- 不处理 `accept-encoding`
- 不会发送 `RFC` 提出的一些错误状态码( `406 not acceptable`)。

但是我对我来说幸运的是，Iris 始终遵循最佳实践和Web标准。

基于：

- `https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation`
- `https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept`
- `https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Charset`
- `https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding`

实现于：

- `https://github.com/kataras/iris/pull/1316/commits/8ee0de51c593fe0483fbea38117c3c88e065f2ef`

## 示例(Example)

```go
type testdata struct {
    Name string `json:"name" xml:"Name"`
    Age  int    `json:"age" xml:"Age"`
}
```

通过 `gzip` 编码算法将资源渲染为 `application/json` 或者 `text/html` 或者 `application/xml`。

- 当客户端的 `Accept` 头部包含上述之一时，
- 如果接收的为空就声明为 `JSON`(首先声明)
- 当客户端的 `Accpet-Encoding` 头包含 `gzip` 或者为空时


```go
	app.Get("/resource", func(ctx iris.Context) {
	    data := testdata{
	        Name: "test name",
	        Age:  26,
	    }
	
	        ctx.Negotiation().JSON().XML().EncodingGzip()
	
	    _, err := ctx.Negotiate(data)
	    if err != nil {
	        ctx.Writef("%v", err)
	    }
	})
```

或者在一个中间件中定义他们，然后在最后的处理器中以 `nil` 参数调用`Negotiate`。
	
```go
ctx.Negotiation().JSON(data).XML(data).Any("content for */*")
ctx.Negotiate(nil) 

app.Get("/resource2", func(ctx iris.Context) {
    jsonAndXML := testdata{
        Name: "test name",
        Age:  26,
    }

    ctx.Negotiation().
        JSON(jsonAndXML).
        XML(jsonAndXML).
        HTML("<h1>Test Name</h1><h2>Age 26</h2>")

    ctx.Negotiate(nil)
})
```

## 文献资料(Documentation)

`Context.Negotiotion` 方法创建一次，返回 "协商构造器"，以便构建适用于特定文本类型，字符和编码算法的服务器端的可用内容。

	Context.Negotiation() *context.NegotiationBuilder

`Context.Negotiotion`  方法用于服务相同 URI 的不同表现形式的资源。当无法匹配 MIME 类型它返回 `context.ErrContentNotSupported` 。

	Context.Negotiate(v interface{}) (int, error)

- `v` 接口可以是一个 `iris.N` 结构体类型值。
- `v` 接口可以是任何实现了 `context.ContentSelector` 接口的值。
- `v` 接口可以是任何实现了 `context.ContentNegotiator` 接口的值。
- `v` 接口可以是结构体 `(JSON、JSONP、XML、YAML)`， 或者 字符串 `(TEXT，HTML)`，或者字节切片`(Markdown，Binary)`，或者匹配的 MIME 类型的字节切片。
- 如果 `v` 接口是 `nil`，将会使用 `Context.Negotitation()` 构造器的内容来替代，除此之外，`v` 接口覆写了构造器的内容(服务器的 MIME 类型通过其注册的、支持的 MIME 列表进行检索)
- 通过 `Negotiation()` 返回的构造器 的 `MIME`，`Text`，`JSON`，`XML`，`HTML` 等等方法设置MIME类型优先级。
- 通过 `Negotiation()` 返回的构造器 的 `Charset` 方法设置字符串类型优先级。
- 通过 `Negotiation()` 返回的构造器 的 `Encoding` 方法设置编码算法优先级。
- 通过 `Negotiation()` 返回的构造器 的 `Accept`，`Override`，`XML` 等等方法修改 `Accept`。