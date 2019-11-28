一个响应记录器是 Iris 特定的 `http.ResponseWriter`，它记录了发送的响应体，状态码和响应头，你可以在任何这个路由的请求处理器链上的处理器内部操作它。

- 在发送数据前调用 `Context.Record`
- `Context.Recorder()` 返回一个 `ResponseRecorder`。它的方法可以用来操作和找回响应。

`ResponseRecorder` 类型包含了标准的 Iris ResponseWriter 方法和下列的方法。

`Body` 返回目前为止 Writer 写入的响应体数据。不要使用它来编辑响应体。

	Body() []byte

使用这个来清除响应体

	ResetBody()

使用 `Write/Writef/WriteString` 流式写入，`SetBody/SetBodyString` 设置响应体。

	Write(contents []byte) (int, error)
	
	Writef(format string, a ...interface{}) (n int, err error)
	
	WriteString(s string) (n int, err error)
	
	SetBody(b []byte)
	
	SetBodyString(s string)

在调用 `Context.Record`之前，使用 `ResetHeaders` 重置响应头为原始状态。

	ResetHeaders()

清除所有的头部信息。

	ClearHeaders()

同时重置响应体，响应头和状态码。

	Reset()

### 示例(Example)

在一个全局拦截器中记录操作日志。


	package main
	
	import "github.com/kataras/iris/v12"
	
	func main() {
	    app := iris.New()
	
	    // start record.
	    app.Use(func(ctx iris.Context) {
	        ctx.Record()
	        ctx.Next()
	    })
	
	    // collect and "log".
	    app.Done(func(ctx iris.Context) {
	        body := ctx.Recorder().Body()
	
	        // Should print success.
	        app.Logger().Infof("sent: %s", string(body))
	    })
	}

注册路由：

	app.Get("/save", func(ctx iris.Context) {
	    ctx.WriteString("success")
	    ctx.Next() // calls the Done middleware(s).
	})

或者为了消除你的主处理器中对 `ctx.Next` 的需求，改变 Iris 处理器的执行规则，你可以如下所示：

	// It applies per Party and its children,
	// therefore, you can create a routes := app.Party("/path")
	// and set middlewares, their rules and the routes there as well.
	app.SetExecutionRules(iris.ExecutionRules{ 
	    Done: iris.ExecutionOptions{Force: true},
	})
	
	// [The routes...]
	app.Get("/save", func(ctx iris.Context) {
	    ctx.WriteString("success")
	})