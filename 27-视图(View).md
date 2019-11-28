Iris 通过它的视图引擎提供了 6 种开箱即用的模板解析器。当然开发者仍然可以使用各种go模板解析器作为 `Context.ResponseWriter()`， 只要实现了 `http.ResponseWriter` 和 `io.Writer`。

Iris提出了一些默认的解析器不支持的通用规则和功能。例如，我们已经支持 `yield`，`render`，`render_r`，`current`，`urlpath` 模板函数， 并通过中间件和嵌入式模板文件为所有的引擎布局和绑定。

为了使用一个模板引擎独特的功能，你需要通过阅读文档来学习模板引擎的特征和语法(点击下面)。选择最适合你app需要的。

内置的视图引擎：

- 标准 template/html：<font color=red>**`iris.HTML(...)`**</font>
- django：<font color=red>**`iris.Django(...)`**</font>
- handlebars：<font color=red>**`iris.Handlebars(...)`**</font>
- amber：<font color=red>**`iris.Amber(...)`**</font>
- pug(jade)：<font color=red>**`iris.Pug(...)`**</font>
- jet：<font color=red>**`iris.Jet(...)`**</font>

一个或者多个视图引擎可以注册到相同的应用中。使用 `RegisterView(ViewEngine)` 方法注册。

从 `./views` 目录中加载所有后缀为 `.html` 的模板，然后使用标准库 `html/template` 包来解析它们。

	// [app := iris.New...]
	tmpl := iris.HTML("./views", ".html")
	app.RegisterView(tmpl)

在路由的处理器中用 `Context.View` 方法渲染或者执行一个视图。

	ctx.View("hi.html")

在使用 `Context.View` 之前使用 `Context.ViewData` 方法绑定一个Go的键值对。

绑定 `{{.message}}` 为 `hello world`

	ctx.ViewData("message", "Hello world!")

你有两种方法绑定一个go模型：

- 第一种

		ctx.ViewData("user", User{})
		
		// variable binding as {{.user.Name}}

- 第二种

		ctx.View("user-page.html", User{})
		
		// root binding as {{.Name}}

要添加一个模板函数， 请使用首选视图引擎的 `AddFunc` 方法。

	//       func name, input arguments, render value
	tmpl.AddFunc("greet", func(s string) string {
	    return "Greetings " + s + "!"
	})

要重新加载本地文件更改，请调用视图引擎的 `Reload` 方法。

	tmpl.Reload(true)

使用嵌入式的模板并且不依赖本地文件系统，使用 `go-bindata` 外部工具，然后把`Asset` 和 `AssetName` 函数传递到首选视图引擎的 `Binary` 方法。

	tmpl.Binary(Asset, AssetNames)

示例代码，请阅读注释：

	// file: main.go
	package main
	
	import "github.com/kataras/iris/v12"
	
	func main() {
	    app := iris.New()
	
	    // Parse all templates from the "./views" folder
	    // where extension is ".html" and parse them
	    // using the standard `html/template` package.
	    tmpl := iris.HTML("./views", ".html")
	
	    // Enable re-build on local template files changes.
	    tmpl.Reload(true)
	
	    // Default template funcs are:
	    //
	    // - {{ urlpath "myNamedRoute" "pathParameter_ifNeeded" }}
	    // - {{ render "header.html" }}
	    // and partial relative path to current page:
	    // - {{ render_r "header.html" }} 
	    // - {{ yield }}
	    // - {{ current }}
	    // Register a custom template func:
	    tmpl.AddFunc("greet", func(s string) string {
	        return "Greetings " + s + "!"
	    })
	
	    // Register the view engine to the views,
	    // this will load the templates.
	    app.RegisterView(tmpl)
	
	    // Method:    GET
	    // Resource:  http://localhost:8080
	    app.Get("/", func(ctx iris.Context) {
	        // Bind: {{.message}} with "Hello world!"
	        ctx.ViewData("message", "Hello world!")
	        // Render template file: ./views/hi.html
	        ctx.View("hi.html")
	    })
	
	    app.Run(iris.Addr(":8080"))
	}

hi.html：

	<!-- file: ./views/hi.html -->
	<html>
	<head>
	    <title>Hi Page</title>
	</head>
	<body>
	    <h1>{{.message}}</h1>
	    <strong>{{greet "to you"}}</strong>
	</body>
	</html>

浏览器源码：

	<html>
	<head>
	    <title>Hi Page</title>
	</head>
	<body>
	    <h1>Hello world!</h1>
	    <strong>Greetings to you!</strong>
	</body>
	</html>