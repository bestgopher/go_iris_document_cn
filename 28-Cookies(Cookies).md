通过 Context 的请求实例可以地访问Cookies。`ctx.Request()` 返回一个 `net/http.Request` 实例。

Iris 的 `Context` 提供了一个工具使得你可以更加容易地访问最常见的cookies用例，并且不需要任何你的自定义额外的代码，只需要使用 Request 的 cookie 方法就能满足。

### 设置cookie

`SetCookie` 方法添加一个cookie

```go
SetCookie(cookie *http.Cookie, options ...CookieOption)
```

`options` 不是必需的，它们可以用来改变 cookie。稍后您将看到可用的选项，可以根据您的Web应用程序要求添加自定义选项，这也有助于避免在代码库中重复您的内容。

如果你也使用 `SetCookieKV` 方法，这个方法不需要导入 `net/http` 包。

```go
SetCookieKV(name, value string, options ...CookieOption)
```

记住：通过 `SetCookieKV` 方法设置的cookie的默认有效期为365天。你可以使用 `CookieExpires` 这个cookie选项设置，或者使用 `kataras/iris/Context.SetCookieKVExpiration` 来全局设置。


`CookieOption` 这是一个 `func(*http.Cookie)` 类型的函数。

##### 设置路径

```go
CookiePath(path string) CookieOption
```

##### 设置有效期

```go
iris.CookieExpires(durFromNow time.Duration) CookieOption
```

##### HttpOnly

```go
iris.CookieHTTPOnly(httpOnly bool) CookieOption
```
- 对于 `RemoveCookie` 和 `SetCookieKV` 来说 `HttpOnly` 字段默认为 `true`。

##### 编码
当添加cookie时提供了编码功能。

接受一个 `CookieEncoder` 并把cookie的值设置为编码后的值。

`SetCookie` 和 `SetCookieKV` 会使用它。

```go
iris.CookieEncode(encode CookieEncoder) CookieOption
```

##### 解码

当获取cookie是提供了解码的功能。

接受一个 `CookieDecoder` ，在通过 `GetCookie` 返回cookie之前把cookie值解码。

`GetCookie` 时使用。

```go
iris.CookieDecode(decode CookieDecoder) CookieOption
```


这里的 `CookieEncoder` 可以描述为：一个 CookieEncoder 编码cookie值。

- 接受cookie 的名字作为第一个参数
- 第二个参数为cookie的值的指针
- 返回的第一个值为编码后的值，当编码操作失败是返回空
- 第二个返回值为编码失败时的错误。

```go
	CookieEncoder func(cookieName string, value interface{}) (string, error)
```

`CookieDecoder` 应该解码cookie值：

- 第一个参数为Cookie的名字
- 第二个参数是编码值，第三个参数是解码值的指针
- 返回的第一个值为解码值，当发生错误返回空
- 返回的第二个值为错误，当解码发生错误时返回不为空的错误

```go
	CookieDecoder func(cookieName string, cookieValue string, v interface{}) error
```

异常不会被打印，因此你必须知道你所做的，记住：如果你使用 AES，它只支持键的大小为 16，24或者32bytes。

### 获取cookie

`GetCookie` 通过cookie的名字返回cookie值，没有找到则返回空。

```go
GetCookie(name string, options ...CookieOption) string
```

如果你想要获得除了值之外更多的信息，使用下面的方法：

```go
cookie, err := ctx.Request().Cookie("name")
```

### 获取所有的cookie

`ctx.Request().Cookies()` 方法返回所有可用的请求cookie的切片。有时你想要改变他们，或者为它们每个都执行一个操作，最简单的方法就是通过 `VisitAllCookies` 方法。

```go
VisitAllCookies(visitor func(name string, value string))
```

### 移除一个Cookie

`RemoveCookie` 方法删除对应名字和路径为"/"的 cookie。

Tip：通过 `iris.CookieCleanPath` 选项改变cookie的路径，例如：`RemoveCookie("nname", iris.CookieCleanPath)`

另外，请注意，默认行为是将其HttpOnly设置为true。 它根据网络标准执行cookie的删除。

```go
RemoveCookie(name string, options ...CookieOption)
```