如“路由”一章所述， <font color=red>**`Iris`**</font> 提供了一些处理器注册方法，每个方法都返回一个 `Route` 实例。


## 路由命名(Route naming)

路由命名非常简单，我们只需要调用返回的 `*Route` 的 `Name` 字段来定义名字。

	package main
	
	import "github.com/kataras/iris/v12"
	
	func main() {
	    app := iris.New()
	    // define a function
	    h := func(ctx iris.Context) {
	        ctx.HTML("<b>Hi</b1>")
	    }
	
	    // handler registration and naming
	    home := app.Get("/", h)
	    home.Name = "home"
	    // or
	    app.Get("/about", h).Name = "about"
	    app.Get("/page/{id}", h).Name = "page"
	
	    app.Run(iris.Addr(":8080"))
	}

## 从路由名字生成URL(Route reversing AKA generating URLs from the route name)

当我们为特定的路径注册处理器时，我们可以根据传递给 Iris 的结构化数据创建 URLs。如上面的例子所示，我们命名了三个路由，其中之一甚至带有参数。如果我们使用默认的 `html/template` 视图引擎，我们可以使用一个简单的操作来反转路由(生成示例的 URLs)：

	Home: {{ urlpath "home" }}
	About: {{ urlpath "about" }}
	Page 17: {{ urlpath "page" "17" }}

上面的代理可以生成下面的输出：

	Home: http://localhost:8080/ 
	About: http://localhost:8080/about
	Page 17: http://localhost:8080/page/17

## 在代码中使用路由名字

我们可以使用以下方法/函数来处理命名路由（及其参数）：

- `GetRoutes` 函数获取所有注册的路由
- `GetRoute(routeName string)` 方法通过名字获得路由
- `URL(routeName string, paramValues ...interface{})` 方法通过提供的值来生成 URL 字符串
- `Path(routeName string, paramValues ...interface{})` 方法通过提供的值生成URL的路径部分(没有主机地址和协议)。