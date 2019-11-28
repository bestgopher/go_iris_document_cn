这里是 `iris.Context` 提供的完整的方法列表。
	
	type (
	    BodyDecoder interface {
	        Decode(data []byte) error
	    }

	    Unmarshaler interface {
	        Unmarshal(data []byte, outPtr interface{}) error
	    }
	
	    UnmarshalerFunc func(data []byte, outPtr interface{}) error
	)

	func (u UnmarshalerFunc) Unmarshal(data []byte, v interface{}) error {
	    return u(data, v)
	}

- <font color=red>**`BodyDecoder`**</font>：
	- <font color=red>**`BodyDecoder`**</font> 是一个接口，任何结构体都可以实现，以便于实现自定义读取JSON或者XML的 decode 行为。
	- 一个简单的例子：

			type User struct { Username string }
			   
			func (u *User) Decode(data []byte) error {
			        return json.Unmarshal(data, u)
			}
	- `context.ReadJSON/ReadXML(&User{})` 将会调用 `User` 的`Decode`来解码请求体
	- 记住：这是完全可选的，默认的ReadJSON解码器是 `encoding/json` ，ReadXML解码器是 `encoding/xml`

-  <font color=red>**`Unmarshaler`**</font>
	- 这是一个接口，实现了可以反序列化任何类型的原始数据。
	- 提示：任何值的指针实现了 `BodyDecoder` 将会覆写 `unmarshaler`。

-  <font color=red>**`UnmarshalerFunc`**</font>

	- `Unmarahsler` 接口的快捷方式
	- 更多详情看 `Unmarshaler` 和 `BodyDecoder`

-  <font color=red>**`Unmarshal`**</font>
	-  解析 `x-encoded` 的数据，并将结构存储到 `v` 指向的指针中。
	- `Unmarshal ` 使用与 `Marshal` 使用的相反的编码，必须为map，slice和指针。


- <font color=red>**`Context`**</font> 是一个客户端在服务器的 "中间人对象"。一个新的 `Context` 是从每一个连接的一个 `sync.Pool`中获取的。 `Context` 是 Iris 的HTTP流上最重要的东西。开发者通过一个 `Context` 发送客户端请求的响应。开发者也从`Context` 中获取客户端请求的信息。

		type Context interface {

		    BeginRequest(http.ResponseWriter, *http.Request)
	
		    EndRequest()
		
		    ResetResponseWriter(ResponseWriter)
		
		    Request() *http.Request
	
		    ResetRequest(r *http.Request)
	
		    SetCurrentRouteName(currentRouteName string)
	
		    GetCurrentRoute() RouteReadOnly
		
		    Do(Handlers)
		
		    AddHandler(...Handler)
	
		    SetHandlers(Handlers)
	
		    Handlers() Handlers
		
		    HandlerIndex(n int) (currentIndex int)
	
		    Proceed(Handler) bool
	
		    HandlerName() string
	
		    HandlerFileLine() (file string, line int)
	
		    RouteName() string
	
		    Next()
	
		    NextOr(handlers ...Handler) bool
	
		    NextOrNotFound() bool
	
		    NextHandler() Handler
	
		    Skip()
	
		    StopExecution()
	
		    IsStopped() bool
		
		    OnConnectionClose(fnGoroutine func()) bool
	
		    OnClose(cb func())
	
		    Params() *RequestParams
		
		    Values() *memstore.Store
	
		    Translate(format string, args ...interface{}) string
		
		    Method() string
	
		    Path() string
	
		    RequestPath(escape bool) string
	
		    Host() string
	
		    Subdomain() (subdomain string)
	
		    IsWWW() bool
	
		    FullRequestURI() string
		  
		    RemoteAddr() string
	
		    GetHeader(name string) string
		  
		    IsAjax() bool
		   
		    IsMobile() bool
		  
		    GetReferrer() Referrer
		 
		    Header(name string, value string)
		
		    ContentType(cType string)
		
		    GetContentType() string
		
		    GetContentTypeRequested() string
		
		    GetContentLength() int64
	
		    StatusCode(statusCode int)
		
		    GetStatusCode() int
	
		    Redirect(urlToRedirect string, statusHeader ...int)
		
		    URLParamExists(name string) bool
		
		    URLParamDefault(name string, def string) string
		
		    URLParam(name string) string
		
		    URLParamTrim(name string) string
		   
		    URLParamEscape(name string) string
		  
		    URLParamInt(name string) (int, error)
		   
		    URLParamIntDefault(name string, def int) int
		   
		    URLParamInt32Default(name string, def int32) int32
		   
		    URLParamInt64(name string) (int64, error)
		    
		    URLParamInt64Default(name string, def int64) int64
		    
		    URLParamFloat64(name string) (float64, error)
		   
		    URLParamFloat64Default(name string, def float64) float64
		
		    URLParamBool(name string) (bool, error)
		
		    URLParams() map[string]string
	
		    FormValueDefault(name string, def string) string
		 
		    FormValue(name string) string
		  
		    FormValues() map[string][]string
		
		    PostValueDefault(name string, def string) string
		
		    PostValue(name string) string
		
		    PostValueTrim(name string) string
		   
		    PostValueInt(name string) (int, error)
		 
		    PostValueIntDefault(name string, def int) int
		  
		    PostValueInt64(name string) (int64, error)
		
		    PostValueInt64Default(name string, def int64) int64
		 
		    PostValueFloat64(name string) (float64, error)
		    
		    PostValueFloat64Default(name string, def float64) float64
		   
		    PostValueBool(name string) (bool, error)
		    
		    PostValues(name string) []string
		 
		    FormFile(key string) (multipart.File, *multipart.FileHeader, error)
		   
		    UploadFormFiles(destDirectory string, before ...func(Context, *multipart.FileHeader)) (n int64, err error)
		
		    NotFound()
		
		    SetMaxRequestBodySize(limitOverBytes int64)
		
		    GetBody() ([]byte, error)
		   
		    UnmarshalBody(outPtr interface{}, unmarshaler Unmarshaler) error
		   
		    ReadJSON(jsonObjectPtr interface{}) error
		 
		    ReadXML(xmlObjectPtr interface{}) error
		
		    ReadForm(formObject interface{}) error
		   
		    ReadQuery(ptr interface{}) error
		
		    Write(body []byte) (int, error)
		  
		    Writef(format string, args ...interface{}) (int, error)
	
		    WriteString(body string) (int, error)
		
		    SetLastModified(modtime time.Time)
	
		    CheckIfModifiedSince(modtime time.Time) (bool, error)
		  
		    WriteNotModified()
		    
		    WriteWithExpiration(body []byte, modtime time.Time) (int, error)
		   
		    StreamWriter(writer func(w io.Writer) bool)
		
		    ClientSupportsGzip() bool
		    
		    WriteGzip(b []byte) (int, error)
		    
		    TryWriteGzip(b []byte) (int, error)
		   
		    GzipResponseWriter() *GzipResponseWriter
		    
		    Gzip(enable bool)
		
		    ViewLayout(layoutTmplFile string)
		   
		    ViewData(key string, value interface{})
		    
		    GetViewData() map[string]interface{}
		    
		    View(filename string, optionalViewModel ...interface{}) error
		
		    Binary(data []byte) (int, error)
	
		    Text(format string, args ...interface{}) (int, error)
	
		    HTML(format string, args ...interface{}) (int, error)
	
		    JSON(v interface{}, options ...JSON) (int, error)
	
		    JSONP(v interface{}, options ...JSONP) (int, error)
	
		    XML(v interface{}, options ...XML) (int, error)
	
		    Markdown(markdownB []byte, options ...Markdown) (int, error)
	
		    YAML(v interface{}) (int, error)
	
		    ServeContent(content io.ReadSeeker, filename string, modtime time.Time, gzipCompression bool) error
		
		    ServeFile(filename string, gzipCompression bool) error
		
		    SendFile(filename string, destinationName string) error
		
		    SetCookie(cookie *http.Cookie, options ...CookieOption)
		  
		    SetCookieKV(name, value string, options ...CookieOption)
		   
		    GetCookie(name string, options ...CookieOption) string
		  
		    RemoveCookie(name string, options ...CookieOption)
		
		    VisitAllCookies(visitor func(name string, value string))
	
		    MaxAge() int64
		
		    Record()
	
		    Recorder() *ResponseRecorder
	
		    IsRecording() (*ResponseRecorder, bool)
	
		    BeginTransaction(pipe func(t *Transaction))
		
		    SkipTransactions()
	
		    TransactionsSkipped() bool
		
		    Exec(method, path string)
		
		    RouteExists(method, path string) bool
		
		    Application() Application
		
		    String() string
		}

- <font color=red>**`BeginRequest(http.ResponseWriter, *http.Request)`**</font>

	- `BeginRequest` 对每个请求执行一次。
	- 它为新来的请求准备 `context`(新的或者从pool获取) 的字段。
	- 为了遵守 Iris 的流程，开发者应该：

		1. 将处理器重置为 `nil`
		2. 将值重置为空
		3. 将session重置为 `nil`
		4. 将响应writer重置为 `http.ResponseWriter`
		5. 将请求重置为 `*http.Request`
	- 其他可选的步骤，视开发的应用程序类型而定

- <font color=red>**`BeginRequest(http.ResponseWriter, *http.Request)`**</font>

	- 当发送完响应后执行一次，当前的 `context` 无用或者释放。
	- 为了遵守 Iris 的流程，开发者应该：
		1. 刷新响应 writer 的结果
		2. 释放响应 writer
	- 其他可选的步骤，视开发的应用程序类型而定

- <font color=red>**`ResponseWriter() ResponseWriter`**</font>
	- 预期返回与 response writer 兼容的 `http.ResponseWriter`

- <font color=red>**`ResetResponseWriter(ResponseWriter)`**</font>
	-  升级或者修改 `Context` 的 `ResponseWriter`

- <font color=red>**`Request() *http.Request`**</font>
	- 按预期返回原始的 `*http.Request`
- <font color=red>**`ResetRequest(r *http.Request)`**</font>
	 - 设置 `Context` 的 `Request`
	 - 通过标准的 `*http.Request` 的 `WithContext` 方法创建的新请求存储到 `iris.Context` 中很有用。
	 - 当你出于某种愿意要对 `*http.Request` 完全覆写时，使用 `ResetRequest`。
	 -  记住，当你只想改变一些字段的时候，你可以使用`Request()` ，它返回一个 Request 的指针，因此在没有完全覆写的情况下，改变也是用效的。
	 -  用法：你使用原生的 http 处理器，它使用的是标准库 `context` 来替代 `iris.Context.Values`，从而获取值。

			r := ctx.Request()
		    stdCtx := context.WithValue(r.Context(), key, val)
		    ctx.ResetRequest(r.WithContext(stdCtx)).
- <font color=red>**`SetCurrentRouteName(currentRouteName string)`**</font>

	- `SetCurrentRouteName` 在内部设置 route 的名字， 目的是为了开发人员调用 `GetCurrentRoute()` 函数时能找到正确的当前的 `只读` 路由。
	- 它被路由器初始化，如果你手动改变名字，除了通过 `GetCurrentRoute()` 函数你将会获取到别的 route 之外，没有什么其它影响。
	- 此外，要在该 context 执行不同路径的处理器，你应该使用 `Exec` 函数，或者通过 `SetHandlers/AddHandler` 函数改变头部信息。

- <font color=red>**`GetCurrentRoute() RouteReadOnly`**</font>
	- 返回被这个请求路径注册的只读的路由。

- <font color=red>**`ResponseWriter() ResponseWriter`**</font>

- <font color=red>**`ResponseWriter() ResponseWriter`**</font>

- <font color=red>**`ResponseWriter() ResponseWriter`**</font>

- <font color=red>**`ResponseWriter() ResponseWriter`**</font>

- <font color=red>**`ResponseWriter() ResponseWriter`**</font>

- <font color=red>**`ResponseWriter() ResponseWriter`**</font>
