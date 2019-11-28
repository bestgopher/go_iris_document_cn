你可能永远不需要这个，但是以防万一。

有时候你可能需要覆写或者决定一个请求进入时这个路由器是否执行。如果你在以前有许多 `net/http` 和其他    web 框架的经验，这个函数将会使你感到熟悉(它有 `net/http` 中间件的格式，但是不接收下一个处理器，它接收的是一个处理器，来作为是否执行的函数)。

	// WrapperFunc is used as an expected input parameter signature
	// for the WrapRouter. It's a "low-level" signature which is compatible
	// with the net/http.
	// It's being used to run or no run the router based on a custom logic.
	type WrapperFunc func(w http.ResponseWriter, r *http.Request, router http.HandlerFunc)
	
	// WrapRouter adds a wrapper on the top of the main router.
	// Usually it's useful for third-party middleware
	// when need to wrap the entire application with a middleware like CORS.
	//
	// Developers can add more than one wrappers,
	// those wrappers' execution comes from last to first.
	// That means that the second wrapper will wrap the first, and so on.
	//
	// Before build.
	func WrapRouter(wrapperFunc WrapperFunc)

路由器基于 `Subdomain`，`HTTP 方法` 和他的动态路径来寻找它的路由。路由包装器可以重写这种行为，执行自定义的代码。

在这个示例中，你将会看到 `.WrapRouter` 的一个用例。你可以使用 `.WrapRouter` 添加自定义逻辑，来决定路由器什么时候执行或者不执行，来达到控制是否执行注册的路由处理器的目的。这仅仅是为了证明概念，你可以跳过本篇教程。

示例代码：

	package main
	
	import (
	    "net/http"
	    "strings"
	
	    "github.com/kataras/iris/v12"
	)
	
	func newApp() *iris.Application {
	    app := iris.New()
	
	    app.OnErrorCode(iris.StatusNotFound, func(ctx iris.Context) {
	        ctx.HTML("<b>Resource Not found</b>")
	    })
	
	    app.Get("/profile/{username}", func(ctx iris.Context) {
	        ctx.Writef("Hello %s", ctx.Params().Get("username"))
	    })
	
	    app.HandleDir("/", "./public")
	
	    myOtherHandler := func(ctx iris.Context) {
	        ctx.Writef("inside a handler which is fired manually by our custom router wrapper")
	    }
	
	    // wrap the router with a native net/http handler.
	    // if url does not contain any "." (i.e: .css, .js...)
	    // (depends on the app , you may need to add more file-server exceptions),
	    // then the handler will execute the router that is responsible for the
	    // registered routes (look "/" and "/profile/{username}")
	    // if not then it will serve the files based on the root "/" path.
	    app.WrapRouter(func(w http.ResponseWriter, r *http.Request, router http.HandlerFunc) {
	        path := r.URL.Path
	
	        if strings.HasPrefix(path, "/other") {
	                // acquire and release a context in order to use it to execute
	                // our custom handler
	                // remember: we use net/http.Handler because here
	                // we are in the "low-level", before the router itself.
	                ctx := app.ContextPool.Acquire(w, r)
	                myOtherHandler(ctx)
	                app.ContextPool.Release(ctx)
	                return
	            }
	
	            // else continue serving routes as usual.
	            router.ServeHTTP(w, r) 
	    })
	
	    return app
	}
	
	func main() {
	    app := newApp()
	
	    // http://localhost:8080
	    // http://localhost:8080/index.html
	    // http://localhost:8080/app.js
	    // http://localhost:8080/css/main.css
	    // http://localhost:8080/profile/anyusername
	    // http://localhost:8080/other/random
	    app.Run(iris.Addr(":8080"))
	
	    // Note: In this example we just saw one use case,
	    // you may want to .WrapRouter or .Downgrade in order to
	    // bypass the Iris' default router, i.e:
	    // you can use that method to setup custom proxies too.
	}

这里不需要多说，它仅是一个接受原生`ResponseWriter` 、`Request` 以及路由器下一个处理器的函数包装器，**它是全部路由器的一个中间件**。