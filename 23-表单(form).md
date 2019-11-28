表单，post的数据和上传的文件可以使用下面的 Context 的方法获取。

```go
// FormValueDefault 返回一个根据名字获取的form的值，
// 其中可能是URL的查询参数和POST或者PUT的数据，
// 如果没有找到返回 def 指定的值。
FormValueDefault(name string, def string) string

// FormValue 返回一个根据名字获取的form的值，
// 其中可能是URL的查询参数和POST或者PUT的数据，
FormValue(name string) string

//  FormValues 返回一个根据名字获取的form的值，
// 其中可能是URL的查询参数和POST或者PUT的数据，
// 默认的form的内存最大尺寸是32MB，
// 这个可以通过在 app.Run() 的第二个参数传入 iris.WithPostMaxMemory 配置器来改变这个大小。
// 记住：检查返回值是否为 nil 是有必要的！
FormValues() map[string][]string

// PostValueDefault 返回通过解析POST，PATCH或者PUT请求体参数，指定名字对应的值。
// 如果没有找到这个名字则返回 def 指定的默认值。
PostValueDefault(name string, def string) string

// PostValue 返回通过解析POST，PATCH或者PUT请求体参数，指定名字对应的值。
PostValue(name string) string

// PostValueTrim 返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的没有前后空格的值。
PostValueTrim(name string) string

// PostValueInt 返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的int的值。
// 如果没有找到name对应的值，则返回-1和一个非nil的错误。
PostValueInt(name string) (int, error)

// PostValueIntDefault  返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的int的值。
// 如果没有找到name对应的值，则 def 指定的默认值。
PostValueIntDefault(name string, def int) int

// PostValueInt64 返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的int64的值。
// 如果没有找到name对应的值，则返回-1和一个非nil的错误。
PostValueInt64(name string) (int64, error)

// PostValueInt64Default  返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的int64的值。
// 如果没有找到name对应的值，则 def 指定的默认值。
PostValueInt64Default(name string, def int64) int64

// PostValueFloat64 返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的float6464的值。
// 如果没有找到name对应的值，则返回-1和一个非nil的错误。
PostValueFloat64(name string) (float64, error)

// PostValueFloat64Default  返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的float64的值。
// 如果没有找到name对应的值，则 def 指定的默认值。
PostValueFloat64Default(name string, def float64) float64

// PostValueBool  返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的bool的值。
// 如果没有找到name对应的值，则返回false和一个非nil的错误。
PostValueBool(name string) (bool, error)

// PostValues  返回通过解析POST，PATCH或者PUT请求体参数，
// 指定名字对应的[]string的值。
// 默认的form的内存最大尺寸是32MB，
// 这个可以通过在 app.Run() 的第二个参数传入 iris.WithPostMaxMemory 配置器来改变这个大小。
// 记住：检查返回值是否为 nil 是有必要的！
PostValues(name string) []string

// FormFile 返回第一个从客户端上传的文件。
// 默认的form的内存最大尺寸是32MB，
// 这个可以通过在 app.Run() 的第二个参数传入 iris.WithPostMaxMemory 配置器来改变这个大小。
// 记住：检查返回值是否为 nil 是有必要的！
FormFile(key string) (multipart.File, *multipart.FileHeader, error)
```


### Multipart/Urlencoded Form

```go
func main() {
    app := iris.Default()

    app.Post("/form_post", func(ctx iris.Context) {
        message := ctx.FormValue("message")
        nick := ctx.FormValueDefault("nick", "anonymous")

        ctx.JSON(iris.Map{
            "status":  "posted",
            "message": message,
            "nick":    nick,
        })
    })

    app.Run(iris.Addr(":8080"))
}
```

### 另一个例子：query + post form

```shell
POST /post?id=1234&page=1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=manu&message=this_is_great
```

```go
func main() {
    app := iris.Default()

    app.Post("/post", func(ctx iris.Context) {
        id := ctx.URLParam("id")
        page := ctx.URLParamDefault("page", "0")
        name := ctx.FormValue("name")
        message := ctx.FormValue("message")
        // or `ctx.PostValue` for POST, PUT & PATCH-only HTTP Methods.

        app.Logger().Infof("id: %s; page: %s; name: %s; message: %s",
            id, page, name, message)
    })

    app.Run(iris.Addr(":8080"))
}
```
	id: 1234; page: 1; name: manu; message: this_is_great

### 上传文件

Context 提供了上传一个用于上传文件的助手(从请求的文件数据中保存文件到主机系统的硬盘上)。阅读下面的 `Context.UploadFormFiles` 方法。


```go
UploadFormFiles(destDirectory string, 
		before ...func(Context, *multipart.FileHeader)) (n int64, err error)
```

`UploadFromFile`  上载任何从客户端获取的文件到系统物理 `destDirctory` 位置。

第二个参数 `before` 给定可调用的函数，这些函数可以在保存到磁盘之前改变 `*miltipart.FileHeader`，它可以用来基于当前请求改变文件的名字，并且所有的 `FileHeader` 的选项都可以改变。如果你不需要在保存文件到硬盘之前使用这个特性，你可以忽略这个参数。

 请注意，它不会检查请求正文是否流式传输。

返回复制的长度的int64值， 和由于操作系统权限导致一个新文件无法创建的非 `nil`的错误, 或者由于没有文件获取而返回 `net/http.ErrMissingFile` 错误。

 如果你想接收并接受文件，并且手动管理它们，你可以使用 `Context.FormFile`，创建一个复制的函数，满足你的需求，下面是通用用法。

默认的form的内存限制是32MB，你通过在主配置时传递 `iris.WithPostMaxMemory` 配置器到 `app.Run` 的第二个参数来改变这个限制。

示例代码：

```go
func main() {
    app := iris.Default()
    app.Post("/upload", iris.LimitRequestBodySize(maxSize), func(ctx iris.Context) {
        //
        // UploadFormFiles
        // uploads any number of incoming files ("multiple" property on the form input).
        //

        // The second, optional, argument
        // can be used to change a file's name based on the request,
        // at this example we will showcase how to use it
        // by prefixing the uploaded file with the current user's ip.
        ctx.UploadFormFiles("./uploads", beforeSave)
    })

    app.Run(iris.Addr(":8080"))
}

func beforeSave(ctx iris.Context, file *multipart.FileHeader) {
    ip := ctx.RemoteAddr()
    // make sure you format the ip in a way
    // that can be used for a file name (simple case):
    ip = strings.Replace(ip, ".", "_", -1)
    ip = strings.Replace(ip, ":", "_", -1)

    // you can use the time.Now, to prefix or suffix the files
    // based on the current time as well, as an exercise.
    // i.e unixTime :=    time.Now().Unix()
    // prefix the Filename with the $IP-
    // no need for more actions, internal uploader will use this
    // name to save the file into the "./uploads" folder.
    file.Filename = ip + "-" + file.Filename
}
```