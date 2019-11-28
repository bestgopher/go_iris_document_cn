在前面的章节中我们学习了 `app.Run` 方法传入的第一个参数，接下来我们看一下第二个参数。

我们从基础开始。 `iris.New` 函数返回一个 `iris.Application` 实例。这个实例可以通过它的 `Configure(...iris.Configurator)` 和 `Run` 方法进行配置。

`app.Run` 方法的第二个参数是可选的、可变长的，接受一个或者多个 `iris.Configurator`。一个 `iris.Configurator` 是 `func(app *iris.Application)` 类型的函数。自定义的 `iris.Configurator` 能够修改你的 `*iris.Application`。

每个核心的配置字段都有一个内建的 `iris.Configurator`。例如， `iris.WithoutStartupLog`，`iris.WithCharset("UTF-8")`，`iris.WithOptimizations`，`iris.WithConfiguration(iris.Congiguration{...})` 函数。


每个模块，例如视图引擎，websockets，sessions和每个中间件都有它们各自配置器和选项，它们大多数都与核心的配置分离。


## 使用配置(Using the Configuration)

唯一的配置结构体是 `iris.Configuration`。让我们从通过 `iris.WithConfiguration` 函数来创建 `iris.Configurator` 开始。

所有的 `iris.Configuration` 字段的默认值都是最常用的。 <font color=red>**`Iris`**</font> 在 `app.Run` 运行之前不需要任何的配置。但是你想要在运行服务之前使用自定义的 `iris.Configurator`，你可以把你的配置器传递给`app.Configure` 方法。

	 config := iris.WithConfiguration(iris.Configuration {
	  DisableStartupLog: true,
	  Optimizations: true,
	  Charset: "UTF-8",
	})
	
	app.Run(iris.Addr(":8080"), config)

## 从YAML加载(Load from YAML)

使用 `iris.YAML("path")`

File：iris.yml

	FireMethodNotAllowed: true
	DisableBodyConsumptionOnUnmarshal: true
	TimeFormat: Mon, 01 Jan 2006 15:04:05 GMT
	Charset: UTF-8

File：main.go

	config := iris.WithConfiguration(iris.YAML("./iris.yml"))
	app.Run(iris.Addr(":8080"), config)

## 从TOML加载(Load from TOML)

使用 `iris.TOML("path")`

File：iris.tml

	FireMethodNotAllowed = true
	DisableBodyConsumptionOnUnmarshal = false
	TimeFormat = "Mon, 01 Jan 2006 15:04:05 GMT"
	Charset = "UTF-8"
	
	[Other]
	    ServerName = "my fancy iris server"
	    ServerOwner = "admin@example.com"

File：main.go

	config := iris.WithConfiguration(iris.TOML("./iris.tml"))
	app.Run(iris.Addr(":8080"), config)

## 使用函数式方式(Using the functional way)

我们已经提到，你可以传递任何数量的 `iris.Configurator` 到 `app.Run` 的第二个参数。I<font color=red>**`Iris`**</font> 为每个`iris.Configuration` 的字段提供了一个选项。

	app.Run(iris.Addr(":8080"), iris.WithoutInterruptHandler,
	    iris.WithoutServerError(iris.ErrServerClosed),
	    iris.WithoutBodyConsumptionOnUnmarshal,
	    iris.WithoutAutoFireStatusCode,
	    iris.WithOptimizations,
	    iris.WithTimeFormat("Mon, 01 Jan 2006 15:04:05 GMT"),
	)

当你想要改变一些 `iris.Configuration` 的字段的时候，这是一个很好的做法。通过前缀 :`With` 或者 `Without`，代码编辑器能够帮助你浏览所有的配置选项，甚至你都不需要翻阅文档。

## 自定义值(Custom values)

`iris.Configuration` 包含一个名为 `Other map[string]interface` 的字段，它可以接受任何自定义的 `key:value` 选项，因此你可以依据需求使用这个字段来传递程序需要的指定的值。

	app.Run(iris.Addr(":8080"), 
	    iris.WithOtherValue("ServerName", "my amazing iris server"),
	    iris.WithOtherValue("ServerOwner", "admin@example.com"),
	)

你可以通过 `app.ConfigurationReadOnly` 来访问这些字段。
	
	serverName := app.ConfigurationReadOnly().Other["MyServerName"]
	serverOwner := app.ConfigurationReadOnly().Other["ServerOwner"]

## 从 Context 中访问配置(Access Configuration from Context)

在一个处理器中，通过下面的方式访问这些字段。

	ctx.Application().ConfigurationReadOnly()