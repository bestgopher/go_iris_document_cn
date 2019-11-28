Iris 通过它的 `jwt` 中间件，提供请求权限验证。这个章节你将会学习怎么在 Iris 中使用 JWT 的基础。

1. 使用下面的命令安装 
	
		$ go get github.com/iris-contrib/middleware/jwt

2. 使用 `jwt,New` 函数创建一个新的 jwt中间件。这个示例中通过 `token` url参数提取 token。经过身份验证的客户端应设计为使用签名令牌进行设置。默认的 jwt 中间件的行为是通过 `Authentication: Bearer $TOKEN` 头部来提取 token 的值。

	jwt 中间件有3个方法来验证 token：
	
	- 第一个方法是 `Serve` 方法，这是一个 `iris.Handler`
	- 第二个方法是 `CheckJWT(iris.Context) bool` 
	- 第三个方法是 `Get(iris.Context) *jwt.Token`，这是一个用于取回已验证的token。

3.   要注册它，您只需在jwt j.Serve中间件之前添加特定的路由组，单个路由或全局路由即可。

		app.Get("/secured", j.Serve, myAuthenticatedHandler)

4. 在一个处理器中生成一个token，接受一个用户的有效负载和响应已签名的token，然后可以通过客户端的请求头或者url参数发送。`jwt.NewToken`，`jwt.NewTokenWithClaims`


### 示例(Example)

	import (
	    "github.com/kataras/iris/v12"
	    "github.com/iris-contrib/middleware/jwt"
	)
	
	func getTokenHandler(ctx iris.Context) {
	    token := jwt.NewTokenWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
	        "foo": "bar",
	    })
	
	    // Sign and get the complete encoded token as a string using the secret
	    tokenString, _ := token.SignedString([]byte("My Secret"))
	
	    ctx.HTML(`Token: ` + tokenString + `<br/><br/>
	    <a href="/secured?token=` + tokenString + `">/secured?token=` + tokenString + `</a>`)
	}
	
	func myAuthenticatedHandler(ctx iris.Context) {
	    user := ctx.Values().Get("jwt").(*jwt.Token)
	
	    ctx.Writef("This is an authenticated request\n")
	    ctx.Writef("Claim content:\n")
	
	    foobar := user.Claims.(jwt.MapClaims)
	    for key, value := range foobar {
	        ctx.Writef("%s = %s", key, value)
	    }
	}
	
	func main(){
	    app := iris.New()
	
	    j := jwt.New(jwt.Config{
	        // Extract by "token" url parameter.
	        Extractor: jwt.FromParameter("token"),
	
	        ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
	            return []byte("My Secret"), nil
	        },
	        SigningMethod: jwt.SigningMethodHS256,
	    })
	
	    app.Get("/", getTokenHandler)
	    app.Get("/secured", j.Serve, myAuthenticatedHandler)
	    app.Run(iris.Addr(":8080"))
	}