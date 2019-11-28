Irir的 `Context` 有两个方法，返回的是我们已经在前面的章节中提到的 `net/http` 标准的 `http.ResponseWriter` 和 `http.Request`。

- `Context.Request()`
- `Context.ResponseWriter()`

然而，除了 Iris Context 提供的独特的 Iris 特性和帮助，为了更易于开发，我们提供了一些现有 `net/http` 功能的包装器。

这里是完整的方法列表，在你处理 URL 查询字符串时可能帮助你。

```go
// URLParam returns true if the url parameter exists, otherwise false.
URLParamExists(name string) bool

// URLParamDefault returns the get parameter from a request,
// if not found then "def" is returned.
URLParamDefault(name string, def string) string

// URLParam returns the get parameter from a request, if any.
URLParam(name string) string

// URLParamTrim returns the url query parameter with
// trailing white spaces removed from a request.
URLParamTrim(name string) string

// URLParamTrim returns the escaped url query parameter from a request.
URLParamEscape(name string) string

// URLParamInt returns the url query parameter as int value from a request,
// returns -1 and an error if parse failed.
URLParamInt(name string) (int, error)

// URLParamIntDefault returns the url query parameter as int value from a request,
// if not found or parse failed then "def" is returned.
URLParamIntDefault(name string, def int) int

// URLParamInt32Default returns the url query parameter as int32 value from a request,
// if not found or parse failed then "def" is returned.
URLParamInt32Default(name string, def int32) int32

// URLParamInt64 returns the url query parameter as int64 value from a request,
// returns -1 and an error if parse failed.
URLParamInt64(name string) (int64, error)

// URLParamInt64Default returns the url query parameter as int64 value from a request,
// if not found or parse failed then "def" is returned.
URLParamInt64Default(name string, def int64) int64

// URLParamFloat64 returns the url query parameter as float64 value from a request,
// returns -1 and an error if parse failed.
URLParamFloat64(name string) (float64, error)

// URLParamFloat64Default returns the url query parameter as float64 value from a request,
// if not found or parse failed then "def" is returned.
URLParamFloat64Default(name string, def float64) float64

// URLParamBool returns the url query parameter as boolean value from a request,
// returns an error if parse failed or not found.
URLParamBool(name string) (bool, error)

// URLParams returns a map of GET query parameters separated by comma if more than one
// it returns an empty map if nothing found.
URLParams() map[string]string
```

查询字符串参数通过使用已有的底层的 `request` 对象解析。这个请求响应一个匹配 `/welcome?firstname=Jane&lastname=Doe` 的 URL。

- `ctx.URLParam("lastname") == ctx.Request().URL.Query().Get("lastname")`

示例代码：

```go
app.Get("/welcome", func(ctx iris.Context) {
    firstname := ctx.URLParamDefault("firstname", "Guest")
    lastname := ctx.URLParam("lastname") 

    ctx.Writef("Hello %s %s", firstname, lastname)
})
```