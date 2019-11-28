`hero` 子包包含一些功能，用于绑定可被handlers作为参数接受的任意对象或者函数，这些被称为依赖。

依赖即可以是想服务这种 `静态的`，也可以是 `动态`的，例如请求的值。

使用 Iris，你将得到真正安全的绑定。这非常快，几乎可以达到原始处理程序的性能，因为Iris试图在服务器甚至没有联机之前就计算所有内容！

Iris提供了内置的依赖关系，可以将路由的参数与您可以立即使用的函数输入参数进行匹配。

使用这个特性你要导入 `hero` 包：

```go
import (
    // [...]
    "github.com/kataras/iris/v12/hero"
)
```

使用 `hero.Handler` 函数将一个函数创建为处理器，它可以接受依赖，然后从它的输出中发送响应，像这样：


```go
func printFromTo(from, to string) string { /* [...]*/ }

// [...]
app.Get("/{from}/{to}", hero.Handler(printFromTo))
```

正如您在上方看到的那样，Context输入参数是完全可选的。

当软你仍然可以声明它作为第一个输入参数 - Iris足够智能， 可以轻松绑定它。

在下面，您将看到一些旨在帮助您理解的屏幕截图：

1. **路径参数-内建的依赖**
	
		```go
import "github.com/kataras/iris/hero"
	
	helloHandler  := hero.Handler(hello)
	app.Get("/{to:string}", helloHandler)
	// 路径参数基于 hero 函数的输入类型绑定
	func hello(to string) string {
		return "Hello" + to
}
	```
	
2. **服务-静态依赖**

    ```go
    type Service interface {
    		SayHello(to string) string
    }
    
    type myTestService struct {
    	prefix string
    }
    
    func (s *myTestService) SayHello(to string) string {
    	return s.prefix + "  " + to
    }
    
    hero.Register(&myTestService{
    	prefix : "Service: Hello"
    })
    
    helloServiceHandler := hero.Handler(helloService)
    
    app.Get("/{to: string}", helloServiceHandler)
    // 服务使用 Register 方法绑定。
    // &myTestService 实现了 Service 接口，因此 hero函数可以接受一个Service。
    func helloService(to string, service Service) string {
    	return service.SayHello(to)
    }
    ```

3. **每个请求-动态请求**

		```go
	type LoginForm struct {
			Username string `form:"username"`
			Password  string `form:"password"`
	}
		
	hero.Register(func(ctx iris.Context) form LoginForm {
		// 
		ctx.ReadForm(&form)
		return
	})
	
	loginHandler := hero.Handler(login)
	
	app.Post("/login", loginHandler)
	
	func login(form LoginForm) string {
		return "Hello" + form.Username
	}
	```

另外，`hero` 子包增加通过函数的输出值发送响应的支持，例如：

- 如果返回的值是 `string`，将会把返回值作为响应体。

- 如果返回值是 `int`，将会返回值作为响应状态码。

- 如果返回值是 `error`，将会返回(bad request 400)，并且返回的错误作为错误原因。

- 如果返回值是 `error` 和 `int`，那错误码将会是输出的值，而不是400(bad request)。

- 如果是一个自定义的 `struct`，当 `Content-Type` 头还没有设置时，将会作为 `JSON` 发送。

- 如果是一个自定义的 `struct` 和 `string`，则第二个返回值 `string` 将会是 `Content-Type`，以此类推。

```go
	func myHandler(...dependencies) string |
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
	                                hero.Result |
	                                (hero.Result, error) {
	    return a_response
	}
```

`hero.Result` 是一个接口，可以使用 `Dispatch(ctx iris.Context)` 通过自定义的逻辑将自定义的结构体呈现出来。

```go
type Result interface {
    Dispatch(ctx iris.Context)
}
```

老实说，`hero` 函数理解非常简单，当你使用时，你再也不想回头了。
