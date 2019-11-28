![](https://i.imgur.com/6bMBwNy.png)

使用 iris MVC 来重用代码。

通过创建彼此独立的组件，开发人员可以在其他应用程序中快速轻松地重用组件。一个程序相同或者相似的视图可以被其他应用使用不同的数据重构，因为视图把数据展示给用户的做法都相似。

Iris对MVC（模型视图控制器）架构模式提供了一流的支持，在Go世界中其他任何地方都找不到这些东西。 您将必须导入 `iris/mvc` 子包。

	import "github.com/kataras/iris/v12/mvc"

Iris web 框架支持请求数据，模型，持续性数据和最快的执行速度绑定。

如果您不熟悉后端Web开发，请先阅读有关MVC架构模式的文章，这是一个不错的开始。


### 特点(Characteristics)

支持所有的 HTTP 状态码，例如，如果想要处理 `GET`方法，控制器需要有一个名为 `Get()` 的函数，你可以在一个控制器中定义多个方法来处理请求。

 通过每个控制器的 `BeforeActivation` 自定义事件回调，将自定义控制器的struct方法用作具有自定义路径（甚至带有regex参数化路径）的处理程序。 例：

	import (
	    "github.com/kataras/iris/v12"
	    "github.com/kataras/iris/v12/mvc"
	)
	
	func main() {
	    app := iris.New()
	    mvc.Configure(app.Party("/root"), myMVC)
	    app.Run(iris.Addr(":8080"))
	}
	
	func myMVC(app *mvc.Application) {
	    // app.Register(...)
	    // app.Router.Use/UseGlobal/Done(...)
	    app.Handle(new(MyController))
	}
	
	type MyController struct {}
	
	func (m *MyController) BeforeActivation(b mvc.BeforeActivation) {
	    // b.Dependencies().Add/Remove
	    // b.Router().Use/UseGlobal/Done
	    // and any standard Router API call you already know
	
	    // 1-> Method
	    // 2-> Path
	    // 3-> The controller's function name to be parsed as handler
	    // 4-> Any handlers that should run before the MyCustomHandler
	    b.Handle("GET", "/something/{id:long}", "MyCustomHandler", anyMiddleware...)
	}
	
	// GET: http://localhost:8080/root
	func (m *MyController) Get() string {
	    return "Hey"
	}
	
	// GET: http://localhost:8080/root/something/{id:long}
	func (m *MyController) MyCustomHandler(id int64) string {
	    return "MyCustomHandler says Hey"
	}

通过为依赖项定义服务或者有一个 `Singleton` 控制器作用域， 在你的控制器结构体中持续性数据(在两个请求间分享数据)。

在控制器间分享依赖或者将它们注册到一个父MVC程序中，有能力在每个控制器的 `BeforeActivate` 可选事件回调函数中修改依赖，例如：

	func(c *MyController) BeforeActivation(b mvc.BeforeActivation) {
			 b.Dependencies().Add/Remove(...) 
	}

作为控制器的字段来访问 `Context`(无需手动绑定)，即 `Ctx iris.Context` 或者通过一个方法的输出参数，即 `func(ctx iris.Context, otherArguments)`

控制器结构体内部的模型(在方法函数中设置，并通过视图渲染)。你可以从一个控制器的方法中返回模型，或者在请求的声明周期中设置一个字段，在同一个请求的生命周期中的另一个方法中返回这个字段。

就像你以前使用的流程一样，MVC 程序有自己的 `Router`，这是 `iris/router.Party` 类型的，标准的 iris api `Controllers` 可以被注册到任何 `Party` 中，包括子域名，`Party` 的开始和完成处理器与预期的一样工作。

可选的 `BeginRequest（ctx）` 函数，用于在方法执行之前执行任何初始化，这对调用中间件或许多方法使用相同的数据收集很有用。


可选的 `EndRequest（ctx）`函数， 可在执行任何方法之后执行任何终结处理。

递归继承，例如 我们的mvc会话控制器示例具有 `Session * sessions.Session` 作为字段，由会话管理器的 `Start` 填充为MVC应用程序的动态依赖项：`mvcApp.Register(sessions.New(sessions.Config{Cookie:"iris_session_id"}).Start`）

通过控制器方法的输入参数访问动态路径参数，不需要绑定。当你使用 Iris 的默认语法从一个控制器中解析处理器，你需要定义方法的后缀为 `By`，大写字母是新的子路径。例如：

如果 `mvc.New(app.Party("/user")).Handle(new(user.Controller))`：

- <font color=red>**`func(*Controller) Get() - GET:/user`**</font>
- <font color=red>**`func(*Controller) Post() - POST:/user`**</font>
- <font color=red>**`func(*Controller) GetLogin() - GET:/user/login`**</font>
- <font color=red>**`func(*Controller) PostLogin() - POST:/user/login`**</font>
- <font color=red>**`func(*Controller) GetProfileFollowers() - GET:/user/profile/followers`**</font>
- <font color=red>**`func(*Controller) PostProfileFollowers() - POST:/user/profile/followers`**</font>
- <font color=red>**`func(*Controller) GetBy(id int64) - GET:/user/{param:long}`**</font>
- <font color=red>**`func(*Controller) PostBy(id int64) - POST:/user/{param:long}`**</font>

如果 `mvc.New(app.Party("/profile")).Handle(new(profile.Controller))`：

- <font color=red>**`func(*Controller) GetBy(username string) - GET:/profile/{param:string}`**</font>

如果 `mvc.New(app.Party("/assets")).Handle(new(file.Controller))`：

- <font color=red>**`func(*Controller) GetByWildard(path string) - GET:/assets/{param:path}`**</font>

方法函数接受的类型可以为：`int`，`int64` ，`bool` 和 `string`

可选的响应输出参数，就像我们前面看到的一样：

	func(c *ExampleController) Get() string |
	                                (string, string) |
	                                (string, int) |
	                                int |
	                                (int, string) |
	                                (string, error) |
	                                error |
	                                (int, error) |
	                                (any, bool) |
	                                (customStruct, error) |
	                                customStruct |
	                                (customStruct, int) |
	                                (customStruct, string) |
	                                mvc.Result or (mvc.Result, error)

这里的 `mvc.Result` 是 `hero.Result` 的别名，就是这个接口：

	type Result interface {
	    // Dispatch should sends the response to the context's response writer.
	    Dispatch(ctx iris.Context)
	}

----------

## 示例

这个例子相当于： `https://github.com/kataras/iris/blob/master/_examples/hello-world/main.go`。

这看起来像多于的代码，不值得书写，但是记住，这个实例没有使用 Iris MVC 的模型，持续化或者视图引擎等特性，没有使用 `Session`，这仅仅是为了学习，或许你在你的程序中不会使用如此简单的控制器。

在这个实例中，在 `/hello` 路径下使用 MVC时，我的个人电脑的消耗是没20MB吞吐大于是2MB，这对于大多数应用程序都可以容忍，但是你可以选择iris中最适合你的，低级处理器的性能或高级控制器：易于维护，大型应用程序上的代码库较小。


#### 仔细阅读注释

	package main
	
	import (
	    "github.com/kataras/iris/v12"
	    "github.com/kataras/iris/v12/mvc"
	
	    "github.com/kataras/iris/v12/middleware/logger"
	    "github.com/kataras/iris/v12/middleware/recover"
	)
	
	func main() {
	    app := iris.New()
	    // Optionally, add two built'n handlers
	    // that can recover from any http-relative panics
	    // and log the requests to the terminal.
	    app.Use(recover.New())
	    app.Use(logger.New())
	
	    // Serve a controller based on the root Router, "/".
	    mvc.New(app).Handle(new(ExampleController))
	
	    // http://localhost:8080
	    // http://localhost:8080/ping
	    // http://localhost:8080/hello
	    // http://localhost:8080/custom_path
	    app.Run(iris.Addr(":8080"))
	}
	
	// ExampleController serves the "/", "/ping" and "/hello".
	type ExampleController struct{}
	
	// Get serves
	// Method:   GET
	// Resource: http://localhost:8080
	func (c *ExampleController) Get() mvc.Result {
	    return mvc.Response{
	        ContentType: "text/html",
	        Text:        "<h1>Welcome</h1>",
	    }
	}
	
	// GetPing serves
	// Method:   GET
	// Resource: http://localhost:8080/ping
	func (c *ExampleController) GetPing() string {
	    return "pong"
	}
	
	// GetHello serves
	// Method:   GET
	// Resource: http://localhost:8080/hello
	func (c *ExampleController) GetHello() interface{} {
	    return map[string]string{"message": "Hello Iris!"}
	}
	
	// BeforeActivation called once, before the controller adapted to the main application
	// and of course before the server ran.
	// After version 9 you can also add custom routes for a specific controller's methods.
	// Here you can register custom method's handlers
	// use the standard router with `ca.Router` to
	// do something that you can do without mvc as well,
	// and add dependencies that will be binded to
	// a controller's fields or method function's input arguments.
	func (c *ExampleController) BeforeActivation(b mvc.BeforeActivation) {
	    anyMiddlewareHere := func(ctx iris.Context) {
	        ctx.Application().Logger().Warnf("Inside /custom_path")
	        ctx.Next()
	    }
	
	    b.Handle(
	        "GET",
	        "/custom_path",
	        "CustomHandlerWithoutFollowingTheNamingGuide",
	        anyMiddlewareHere,
	    )
	
	    // or even add a global middleware based on this controller's router,
	    // which in this example is the root "/":
	    // b.Router().Use(myMiddleware)
	}
	
	// CustomHandlerWithoutFollowingTheNamingGuide serves
	// Method:   GET
	// Resource: http://localhost:8080/custom_path
	func (c *ExampleController) CustomHandlerWithoutFollowingTheNamingGuide() string {
	    return "hello from the custom handler without following the naming guide"
	}
	
	// GetUserBy serves
	// Method:   GET
	// Resource: http://localhost:8080/user/{username:string}
	// By is a reserved "keyword" to tell the framework that you're going to
	// bind path parameters in the function's input arguments, and it also
	// helps to have "Get" and "GetBy" in the same controller.
	//
	// func (c *ExampleController) GetUserBy(username string) mvc.Result {
	//     return mvc.View{
	//         Name: "user/username.html",
	//         Data: username,
	//     }
	// }
	
	/* Can use more than one, the factory will make sure
	that the correct http methods are being registered for each route
	for this controller, uncomment these if you want:
	
	func (c *ExampleController) Post() {}
	func (c *ExampleController) Put() {}
	func (c *ExampleController) Delete() {}
	func (c *ExampleController) Connect() {}
	func (c *ExampleController) Head() {}
	func (c *ExampleController) Patch() {}
	func (c *ExampleController) Options() {}
	func (c *ExampleController) Trace() {}
	*/
	
	/*
	func (c *ExampleController) All() {}
	//        OR
	func (c *ExampleController) Any() {}
	
	
	
	func (c *ExampleController) BeforeActivation(b mvc.BeforeActivation) {
	    // 1 -> the HTTP Method
	    // 2 -> the route's path
	    // 3 -> this controller's method name that should be handler for that route.
	    b.Handle("GET", "/mypath/{param}", "DoIt", optionalMiddlewareHere...)
	}
	
	// After activation, all dependencies are set-ed - so read only access on them
	// but still possible to add custom controller or simple standard handlers.
	func (c *ExampleController) AfterActivation(a mvc.AfterActivation) {}
	*/


在控制器中每个以HTTP方法(`Get`，`Post`，`Put`，`Delete`...) 为前缀的函数，都作为一个 HTTP 端点。在上面的示例中，所有的函数都向响应写了一个字符串。注意每种方法之前的注释。

一个HTTP端点在web程序中是可定位的URL，例如 `http://localhost:8080/helloworld`，结合使用的协议：HTTP，web服务器的网络定位(包括TCP端口)：`localhost:8080` 和 定位的URI：`/helloworld`。

第一个注释指出这是一个HTTP GET方法，该方法通过在基本URL后面附加` /helloworld` 来调用。第三条注释指定HTTP GET方法，该方法通过在URL后面附加 `/helloworld/welcome` 来调用。


控制器知道怎么处理 `GetBy` 上的 "name" 或者 `GetWelcomeBy` 上的 "name" 和 "numTimes"，因为 `By` 关键字，并且建立了没有样板的动态路由；第三个注释指定HTTP GET动态方法，该方法可以由任何以“ / helloworld / welcome”开头的URL调用，然后再加上两个路径部分，第一个可以接受任何值，第二个只能接受数字，例如：`http://localhost:8080/helloworld/welcome/golang/32719`，除此以外，`404 Not Found HTTP Error` 将被发送到客户端。

> `https://github.com/kataras/iris/tree/master/_examples/mvc` 和  `https://github.com/kataras/iris/blob/master/mvc/controller_test.go` 通过简单的范式解释了特性，它们展示了如何利用 Iris MVC 的 Binder、模型等等...

>websocket 控制器请看前面的 `Websocket` 章节。

