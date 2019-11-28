WebSocket是一种协议，可通过TCP连接启用双向持久通信通道。它用于聊天，股票报价，游戏等应用程序，以及您希望在Web应用程序中具有实时功能的任何位置。

当你需要直接使用套接字连接直接工作时，使用 WebSockets。例如，你可能需要在实时游戏中的最好的性能。

首先，阅读 `kataras/neffos(https://github.com/kataras/neffos/wiki)` 的介绍，掌握这个为`net/http` 和 `iris` 构建的 websocket 库。

它在安装 iris 时就已经预装了，但是你也可以使用下面的命令单独安装：

	$ go get github.com/kataras/neffos@latest

继续阅读怎么注册neffos websockets 服务器到你的 iris 程序中。

这里可以查看一系列全面的使用websockets的示例：
	
	https://github.com/kataras/iris/tree/master/_examples/websocket

`iris/websocket` 子包仅包含iris 特定的为 `neffos websockets` 准备的迁移器(migrations，不知咋翻译)和助手。

例如，为了获得请求的 `context` 权限，你可以在处理器或者回调函数中调用 `websocket.GetContext(conn)`来获取。

	// GetContext从一个websocket连接中返回一个iris.Context
	func GetContext(c *neffos.Conn) Context

使用 `websocket.Handler` 注册 websocket `neffos.Server` 到一个路由中。

	 // IDGenerator is an iris-specific IDGenerator for new connections.
	type IDGenerator func(Context) string
	
	// Handler returns an Iris handler to be served in a route of an Iris application.
	// Accepts the neffos websocket server as its first input argument
	// and optionally an Iris-specific `IDGenerator` as its second one.
	func Handler(s *neffos.Server, IDGenerator ...IDGenerator) Handler

使用：

	import (
	    "github.com/kataras/neffos"
	    "github.com/kataras/iris/v12/websocket"
	)
	
	// [...]
	
	onChat := func(ns *neffos.NSConn, msg neffos.Message) error {
	    ctx := websocket.GetContext(ns.Conn)
	    // [...]
	    return nil
	}
	
	app := iris.New()
	ws := neffos.New(websocket.DefaultGorillaUpgrader, neffos.Namespaces{
	    "default": neffos.Events {
	        "chat": onChat,
	    },
	})
	app.Get("/websocket_endpoint", websocket.Handler(ws))


----------

### websocket 控制器

Iris具有一种通过Go结构注册websocket事件的简单方法。 Websocket控制器是MVC功能的一部分。

Iris 提供了 `iris/mvc/Application.HandleWebsocket(v interface{}) *neffos.Struct` 来注册控制器到一个存在的 iris MVC程序中(提供功能齐全的依赖项注入容器，用于请求值和静态服务)。

	// HandleWebsocket handles a websocket specific controller.
	// Its exported methods are the events.
	// If a "Namespace" field or method exists then namespace is set,
	// otherwise empty namespace will be used for this controller.
	//
	// Note that a websocket controller is registered and ran under
	// a connection connected to a namespace
	// and it cannot send HTTP responses on that state.
	// However all static and dynamic dependencies behave as expected.
	func (*mvc.Application) HandleWebsocket(controller interface{}) *neffos.Struct


我们来看看使用例子，我们想要通过我们的控制器方法绑定 `OnNamespaceConnected`，`OnNamespaceDisconnect` 内置的实现和一个自定义的 `OnChat` 事件。

1. 我们创建一个控制器，声明 `NSConn` 类型字段为 `stateless`，然后写我们需要的方法。

		type websocketController struct {
		    *neffos.NSConn `stateless:"true"`
		    Namespace string
		
		    Logger MyLoggerInterface
		}
		
		func (c *websocketController) OnNamespaceConnected(msg neffos.Message) error {
		    return nil
		}
		
		func (c *websocketController) OnNamespaceDisconnect(msg neffos.Message) error {
		    return nil
		}
		
		func (c *websocketController) OnChat(msg neffos.Message) error {
		    return nil
		}

	Iris 足够聪明，通过 `Namespace string` 结构体字段规定的命名空间来注册控制器方法作为事件函数， 或者你可以创建一个控制器方法 `Namespace() string {return "default"}`，或者使用 `HandleWebsocket` 的返回值的 `SetNamespace("default")`，这取决于你。

2. 我们将MVC应用程序目标初始化为websocket端点，就像我们以前使用常规HTTP控制器进行HTTP路由一样。

		import (
		    // [...]
		    "github.com/kataras/iris/v12/mvc"
		)
		// [app := iris.New...]
		
		mvcApp := mvc.New(app.Party("/websocket_endpoint"))

3. 注册我们的依赖项(如果有的话)

		mvcApp.Register(
		    &prefixedLogger{prefix: "DEV"},
		)

4. 我们注册一个或者多个websocket 控制器，每个控制器匹配一个namespace(只需一个就足够了，因为在大多数情况下，您不需要更多，但这取决于您的应用程序的需求和要求)。

		mvcApp.HandleWebsocket(&websocketController{Namespace: "default"})

5. 接下来，我们通过将mvc应用程序作为连接处理程序映射到websocket服务器来继续处理(一个websocket服务器可以在多个mvc应用上使用，通过 `neffos.JoinConnHandlers(mvcApp1, mvcApp2)`)。

		websocketServer := neffos.New(websocket.DefaultGorillaUpgrader, mvcApp)

6. 最后一步是通过普通的 `.Get` 方法将该服务器注册到我们的端点。

		mvcApp.Router.Get("/", websocket.Handler(websocketServer))

