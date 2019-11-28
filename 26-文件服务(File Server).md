 通过 `Party.HandleDir` 方法指定特定的目录(系统目录或者嵌入式应用)以获取静态文件。

`HandleDir` 注册一个处理程序，该处理程序使用文件系统的内容（物理或嵌入式）为HTTP请求提供服务。

- 第一个参数：路由路径
- 第二个参数：需要保存文件的系统或者嵌入式目录
- 第三个参数：不必要，目录选项，设置字段是可选的。

返回 `*Route`

```go
HandleDir(requestPath, directory string, opts ...DirOptions) (getRoute *Route)
```


`DirOptions`  结构体是这样的：

```go
type DirOptions struct {
    // 默认为 /index.html, 如果请求路径以 **/*/$IndexName 结尾，它会重定向到 **/*(/)，
    // 然后另一个处理器会处理它，
    // 如果最后开发人员没有设法手动处理它，
    // 框架会自动注册一个名为index handler 的处理器来作为这个处理器
    IndexName string

   // 文件是否需要gzip压缩
    Gzip bool

    // 如果 IndexName没有找到，是否列出当前请求目录中的文件
    ShowList bool

    // 如果 ShowList 为 true，这个函数将会替代默认的列出当前请求目录中文件的函数。
    DirList func(ctx iris.Context, dirName string, dir http.File) error

    // 内嵌时
    Asset      func(name string) ([]byte, error)   
    AssetInfo  func(name string) (os.FileInfo, error)
    AssetNames func() []string

    //  循环遍历每个找到的请求资源的可选验证器。
    AssetValidator func(ctx iris.Context, name string) bool
}
```

让我假设在你的可执行文件目录中有一个 `./assets` 文件夹，你想要处理 `http://localhost:8080/static/**/*` 路由下的文件。

```go
app := iris.New()

app.HandleDir("/static", "./assets")

app.Run(iris.Addr(":8080"))
```

现在，如果您想将静态文件嵌入可执行文件内部以不依赖于系统目录，则可以使用 `go-bindata` 之类的工具将文件转换为程序内的[] byte。 让我们快速学习一下，以及Iris如何帮助服务这些数据。

安装 `go-bindata`

```shell
go get -u github.com/go-bindata/go-bindata/...
```

导航到你程序的目录，并且 `./assets` 子目录存在，然后执行：

```shell
$ go-bindata ./assets/...
```

上面创建了一个go文件，其中包含三个主要的函数：`Asset`，`AssetInfo`，`AssetNames`。在 `iris.DirOptions` 中使用它们：

```go
// [app := iris.New...]

app.HandleDir("/static", "./assets", iris.DirOptions {
    Asset: Asset,
    AssetInfo: AssetInfo,
    AssetNames: AssetNames,
    Gzip: false,
})
```

编译你的程序：

```shell
$ go build
```

`HandleDir` 物理目录和内嵌目录的所有标准，包括 `content-range`。

然而，如果你仅仅想要一个处理器工作，而不注册路由，你可以使用 `iris.FileServer` 包级别函数替代。

`FileServer` 函数返回一个处理器，该处理器为来自特定系统，物理目录，嵌入式目录的文件提供服务。

- 第一个参数：目录
- 第一个参数：可选参数，调用者可选的配置

```shell
	iris.FileServer(directory string, options ...DirOptions)
```

使用：

```go
handler := iris.FileServer("./assets", iris.DirOptions {
    ShowList: true, Gzip: true, IndexName: "index.html",
})
```