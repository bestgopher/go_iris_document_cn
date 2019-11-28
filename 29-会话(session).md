当你使用一个应用程序，你打开它，做了一些改变，然后关闭它。这就像一个会话。计算机知道你是谁。它知道你什么时候开始这个程序，什么什么时候关闭。但是在互联网上，这是一个问题：web服务器不知道你是谁，也不知道你做什么，因为HTTP不能保持状态。

`Session` 变量通过存储要在多个页面上使用的用户信息（例如用户名，喜欢的颜色等）来解决此问题。默认情况下，`Session` 变量会一直存在，直到浏览器关闭。

因此，`Session` 变量保存有关一个用户的信息，并且可用于一个应用程序中的所有页面。

> Tip：如果你想要持久化存储，你可以存储数据到数据库中。

Iris 在 `iris/sessions` 子包中有自己的会话实现和会话管理。你只需导入这个包就能使用了。

一个会话通过 `Session` 对象的 `Start` 函数开始，`Session` 对象是通过 `New` 函数创建的，这个函数会返回一个 `Sessoin`。

`Session` 变量通过 `Session.Set` 方法设置，通过 `Session.Get` 方法取回。使用 `Session.Delete` 删除一个变量。要删除整个会话并使之无效，请使用 `Session.Destroy` 方法。


session 管理器通过 `New` 方法创建的。


	import "github.com/kataras/iris/v12/sessions
	sess := sessions.New(sessions.Config{Cookie: "cookieName", ...})

`Config`：

	Config struct {
			// session 的 cookie名字，例如："mysessionid"
			// 默认为 "irissessionid"
			Cookie string
	
			// 如果服务器通过 TLS 运行，CookieSecureTLS 设置为true，
			// 你需要把 session的cookie的 "Secure" 字段设置为 true。
			//
			// 记住：为了正常工作用户应该指定"Decode" 配置字段。
			// 建议：您不需要将此字段设置为true，只需在_examples文件夹中提供
			// example的第三方库（例如secure cookie）填充Encode和Decode字段即可。
			// 
			// 默认为 false
			CookieSecureTLS bool
	
			// AllowReclaim 将允许在想用的请求处理器中清除然后重新开始一个session
			// 它所做的只是在“Destroy”时删除“Request”和“ ResponseWriter”的cookie，
			// 或者在“Start”时将新cookie添加到“请求”。
			//
			// 默认为 false
			AllowReclaim bool

			// 当cookie值不为nil时编码此cookie值(config.Cookie字段指定的值)。
			// 接受cookie的名字为第一个参数，第二个参数为服务器产生的session id。
			// 返回新的session id，如果发生错误，session id将置为空，这是无效的。
			// 
			// 提示：错误将不会打印，因此你应该清楚你所做的。
			// 记住：如果你使用 AES，它仅支持key的大小为16,24，或者32 bytes。
			// 你要么提供准确的值或者从你键入的内容中得到key
			// 
			// 默认为nil
			Encode func(cookieName string, value interface{}) (string, error)
			
			// 如果cookie值不为nil则对其解码
			// 第一个参数为cookie的名字(config.Cookie字段指定的值)，
			// 第二个参数为客户端的cookie值(也就是被编码后的 session id)，
			// 当操作失败时返回错误
			//
			// 提示：错误将不会打印，因此你应该清楚你所做的。
			// 记住：如果你使用 AES，它仅支持key的大小为16,24，或者32 bytes。=
			// 你要么提供准确的值或者从你键入的内容中得到key
			// 
			// 默认为nil
			Decode func(cookieName string, cookieValue string, v interface{}) error
	
			// Defaults to nil.
			// Encoding 功能与 Encode和Decode类似，但是接受一个实例，
			// 这个实例实现了 "CookieEncoder" 接口(Encode和Decode方法)。
			//
			// 默认为nil
			Encoding Encoding
	
			// 指定cookie必须的生存时间(created_time.Add(Expires))，
			// 如果你想要在浏览器关闭时删除cookie，就设置为 -1。
			//  0 意味着没有过期时间(24年)，
			// -1 意味着浏览器关闭时删除
			// >0 就是指定session 的cookie生存期(time.Duration类型)。 
			Expires time.Duration
	
			// SessionIDGenerator 可以设置一个函数，用于返回一个唯一的session id。
			// 默认将使用uuid包来生成session id，但是开发者可以通过指定这个字段来改变这个行为。
			SessionIDGenerator func(ctx context.Context) string

			// DisableSubdomainPersistence 设置为true时，
			// 将不允许你的子域名拥有访问session 的cookie的权利
			//
			// 默认为false
			DisableSubdomainPersistence bool
		}

`New` 返回一个 `Sessions` 的指针，并拥有这些方法：
	
	// 为特定的请求创建或者取回一个已经存在的session。
	Start(ctx iris.Context, cookieOptions ...iris.CookieOption)
	
	// Handler 返回一个session中间件，用以注册到应用程序路由中。
	Handler(cookieOptions ...iris.CookieOption) iris.Handler	

	// 移除session数据和相关cookie。
	Destroy(ctx context.Context)

	// 移除服务器端内存(和已注册的数据库)中的所有session。
	// 客户端的session cookie将依然存在，但是它将会被下个请求重新设置
	DestroyAll()
	
	// DestroyByID 移除服务器端内存(和已注册的数据库)中的session 条目。
	// 客户端的session cookie将依然存在，但是它将会被下个请求重新设置
	// 使用这个很安全，即使你不确定这个id对应的session是否存在。
	// 提示：sid应该是原始的数据(即从存储中获取)，不是已经解码的。
	DestroyByID(sessID string)
	
	// OnDestroy 注册一个或者多个移除监听者。
	// 当一个服务端或者客户端(cookie)的session被移除，将会触发监听者。
	// 记住如果一个监听者被阻塞，会话管理器将都会延迟，
	// 在侦听器中使用goroutine避免这种情况。
	OnDestroy(callback func(sid string))
	
	// ShiftExpiration 通过session默认的超时配置，将会话的过期日期更改到新的日期
	// 如果使用数据库保存session将会抛出 "ErrNotImplemented" 错误，因为数据库不支持这个特性。
	ShiftExpiration(ctx iris.Context,
		 	cookieOptions ...iris.CookieOption) error
	
	// UpdateExpiration 通过 "expires" 这个超时值将session的超时日期改为新的日期。
	// 如果更新一个不存在或者无效的session条目，将会返回 "ErrNotFound" 异常。
	//  如果使用数据库保存session将会抛出 "ErrNotImplemented" 错误，因为数据库不支持这个特性。
	UpdateExpiration(ctx iris.Context, expires time.Duration，
			 cookieOptions ...iris.CookieOption) error
	
	// UseDatabase 添加一个session数据库到session管理器中，
	// 会话数据库没有写访问权
	UseDatabase(db Database)

这里的 `CookieOption` 仅仅是一个 `func(*http.Cookie)` 函数，允许自定义配置cookie的属性。

`Start` 方法返回一个 `Session` 指针值，这个指针有自己的方法来为每个session工作。

	func (ctx iris.Context) {
		session := sess.Start(ctx)
			
			// 返回session的id
			.ID() string
			
			// 如果这个session是在当前处理流程中创建，将返回true
			.IsNew() bool
			
			//  根据会话的键，填充相应的值
	 		.Set(key string, value interface{})

			// 根据会话的键，填充相应的值。
			// 与 Set 不同，当使用 Get 时，无法改变输出值
			//  一个不可变的条目只能通过 SetImmutable 改变，Set将不能工作
			// 如果条目是不可变的，这将是安全的。         
			// 谨慎使用，它比 Set 慢
			.SetImmutable(key string, value interface{})

			// 获取所有值的一个备份
			.GetAll() map[string]interface{}
			
			// 返回这个session保存的值的总数
			.Len() int
	
			// 移除key对应的条目，如果有条目移除则返回true
			.Delete(key string) bool
			
			// 移除所有的session条目
			.Clear()
			
			// 返回key对应的值
			.Get(key string) interface{}
		
			// 与Get类似，但是返回的是值的字符串表示形式。
			// 如果不存在key，返回空字符串。
			.GetString(key string) string
			
			// 与Get类似，但是返回的是值的字符串表示形式。
			// 如果不存在key，返回defaultValue定义的默认值。
			.GetStringDefault(key string, defaultValue string) string

			// 与Get类似，但是返回的是值的int表示形式。
			// 如果不存在key，返回-1和一个非nil的错误
			.GetInt(key string) (int, error)

			// 与Get类似，但是返回的是值的int表示形式。
			// 如果不存在key，返回defaultValue定义的默认值。
			.GetIntDefault(key string, defaultValue int) int

			// 将保存的key的值+n，
			// 如果key不存在，则设置key的值为n
			// 返回增加后的值
			.Increment(key string, n int) (newValue int)
			
			// 将保存的key的值-n，
			// 如果key不存在，则设置key的值为n
			// 返回减少后的值
			.Decrement(key string, n int) (newValue int)
			
			// 与Get类似，但是返回的是值的int64表示形式。
			// 如果不存在key，返回-1和一个非nil的错误
			.GetInt64(key string) (int64, error)

			// 与Get类似，但是返回的是值的int64表示形式。
			// 如果不存在key，返回defaultValue定义的默认值。
			.GetInt64Default(key string, defaultValue int64) int64

			// 与Get类似，但是返回的是值的float32表示形式。
			// 如果不存在key，返回-1和一个非nil的错误
			.GetFloat32(key string) (float32, error)

			// 与Get类似，但是返回的是值的float32表示形式。
			// 如果不存在key，返回defaultValue定义的默认值。
			.GetFloat32Default(key string, defaultValue float32) float32

			// 与Get类似，但是返回的是值的float64表示形式。
			// 如果不存在key，返回-1和一个非nil的错误
			.GetFloat64(key string) (float64, error)

			// 与Get类似，但是返回的是值的float64表示形式。
			// 如果不存在key，返回defaultValue定义的默认值。
			.GetFloat64Default(key string, defaultValue float64) float64

			// 与Get类似，但是返回的是值的boolean表示形式。
			// 如果不存在key，返回-1和一个非nil的错误
			.GetBoolean(key string) (bool, error)

			// 与Get类似，但是返回的是值的boolean表示形式。
			// 如果不存在key，返回defaultValue定义的默认值。
			.GetBooleanDefault(key string, defaultValue bool) bool
			
			// 通过key设置一个即时信息。
			// 即时信息是为了保存一个信息到session中，这样同一个用户的多个请求都能获取到这个信息。
			// 当这个信息展示给用户后就会被移除。
			// 即时信息通常用于与HTTP重定向组合，
			// 因为这种情况下是没有视图的，信息只能在重定向后展示给用户。
			// 
			// 一条即时信息拥有它的key和内容。
			// 这是一个有关联的数组。
			// 名字是一个字符串：通常为"notice","sucess","error"，但是可以为任何string。
			// 内容通常是stirng。如果你想直接显示它，你可以放一些HTML标签在信息中。
			// 也可以放置一个数组或者数组：将会被序列化，然后以字符串类型保存。
			// 
			// 即时信息可以是用哪个 SetFlash 方法设置。
			// 例如，你想要通知用户他的改变成功保存了，
			// 你可以在你的处理中添加下面一行：SetFlash("success", "data saved")
			// 在这个例子中我们使用key"success"，如果你想要定义更多即时信息，你可以使用不同的key。
			.SetFlash(key string, value interface{})

			// 如果这个session有可用的即时信息将返回true
			.HasFlash() bool

			// GetFlashes 返回所有的即时信息的值，使用的是 map[string]interface{} 格式。
			// 记住：这将导致在同一用户的下一个请求上删除所有当前的即时消息。
 			.GetFlashes() map[string]interface{}
			
			// PeekFlash 返回key对应的暂存的即时信息。
			// 与GetFlash不同，这个信息在下个请求时可用，除非使用GetFlashes或者GetFlash
			.PeekFlash(key string) interface{}

			// GetFlash返回key对应的存储的即时信息，并且在下个请求中移除这个即时信息。
			// 检查即时信息我们使用 HashFlash() 方法，获得即时信息我们使用GetFlash()方法。
			// GetFlashes() 获取所有的信息。
			// 获取信息并从session删除，这意味着一条消息只能在提供给用户的第一个页面上显示。
			.GetFlash(key string) interface{}
			
			// 与GetFlash类似，但是返回的是string表示形式，
			// 如果key不存在，则返回空string
			.GetFlashString(key string) string
			
			// 与GetFlash类似，但是返回的是string表示形式，
			// 如果key不存在，则返回defaultValue指定的默认值
			.GetFlashStringDefault(key string, defaultValue string) string
			
			// 删除key这个即时信息
			.DeleteFlash(key string)
			
			// 移除所有的即时信息
			.ClearFlashes()
			
			// "摧毁"这个session，它移除session的值和所有即时信息。
			// session条目将会从服务器中移除，注册的session数据库也会被通知删除。
			// 记住这个方法不会移除客户端的cookie，如果附上新的session客户端的cookie将会被重置。
			// 使用会话管理器的 "Destroy(ctx)" 来移除cookie。
			.Destroy()
	}

### 示例：

在这里例子中我们将允许通过验证的用户访问 `/secret` 的隐秘信息。为了有访问的权限，我们首先需要访问 `/login` 来获取一个可用的session 的 cookie，使它登录成功。另外也可以访问 `/logout` 撤销访问权限。

	// sessions.go
	package main
	
	import (
	    "github.com/kataras/iris/v12"
	
	    "github.com/kataras/iris/v12/sessions"
	)
	
	var (
	    cookieNameForSessionID = "mycookiesessionnameid"
	    sess                   = sessions.New(sessions.Config{Cookie: cookieNameForSessionID})
	)
	
	func secret(ctx iris.Context) {
	    // Check if user is authenticated
	    if auth, _ := sess.Start(ctx).GetBoolean("authenticated"); !auth {
	        ctx.StatusCode(iris.StatusForbidden)
	        return
	    }
	
	    // Print secret message
	    ctx.WriteString("The cake is a lie!")
	}
	
	func login(ctx iris.Context) {
	    session := sess.Start(ctx)
	
	    // Authentication goes here
	    // ...
	
	    // Set user as authenticated
	    session.Set("authenticated", true)
	}
	
	func logout(ctx iris.Context) {
	    session := sess.Start(ctx)
	
	    // Revoke users authentication
	    session.Set("authenticated", false)
	    // Or to remove the variable:
	    session.Delete("authenticated")
	    // Or destroy the whole session:
	    session.Destroy()
	}
	
	func main() {
	    app := iris.New()
	
	    app.Get("/secret", secret)
	    app.Get("/login", login)
	    app.Get("/logout", logout)
	
	    app.Run(iris.Addr(":8080"))
	}

访问：

	$ go run sessions.go
	
	$ curl -s http://localhost:8080/secret
	Forbidden
	
	$ curl -s -I http://localhost:8080/login
	Set-Cookie: mysessionid=MTQ4NzE5Mz...
	
	$ curl -s --cookie "mysessionid=MTQ4NzE5Mz..." http://localhost:8080/secret
	The cake is a lie!