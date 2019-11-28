你可以定义自己的处理器来处理特定 http 错误。

>错误码是 大于等于400的Http 状态码，例如 `404 not found`，`500 internal server`。

示例代码：

```go
package main

import "github.com/kataras/iris/v12"

func main(){
    app := iris.New()
    app.OnErrorCode(iris.StatusNotFound, notFound)
    app.OnErrorCode(iris.StatusInternalServerError, internalServerError)
    // to register a handler for all "error"
    // status codes(kataras/iris/context.StatusCodeNotSuccessful)
    // defaults to < 200 || >= 400:
    // app.OnAnyErrorCode(handler)
    app.Get("/", index)
    app.Run(iris.Addr(":8080"))
}

func notFound(ctx iris.Context) {
    // when 404 then render the template
    // $views_dir/errors/404.html
    ctx.View("errors/404.html")
}

func internalServerError(ctx iris.Context) {
    ctx.WriteString("Oups something went wrong, try again")
}

func index(ctx iris.Context) {
    ctx.View("index.html")
}
```

更多内容查看 视图(View) 章节。

## 问题类型(The Problem type)

Iris 内建支持 HTTP APIs 的错误详情。

`Context.Problem` 编写一个 JSON 或者 XML 问题响应，行为完全类似 `Context.JSON`，但是默认 `ProblemOptions.JSON` 的缩进是  `" "`，响应的 `Content-type` 为 `application/problem+json`。

使用 `options.RenderXML` 和 `XML` 字段来改变他的行为，用 `application/problem+xml` 的文本类型替代。

```go
func newProductProblem(productName, detail string) iris.Problem {
    return iris.NewProblem().
        // The type URI, if relative it automatically convert to absolute.
        Type("/product-error"). 
        // The title, if empty then it gets it from the status code.
        Title("Product validation problem").
        // Any optional details.
        Detail(detail).
        // The status error code, required.
        Status(iris.StatusBadRequest).
        // Any custom key-value pair.
        Key("productName", productName)
        // Optional cause of the problem, chain of Problems.
        // .Cause(other iris.Problem)
}

func fireProblem(ctx iris.Context) {
    // Response like JSON but with indent of "  " and
    // content type of "application/problem+json"
    ctx.Problem(newProductProblem("product name", "problem details"),
        iris.ProblemOptions{
            // Optional JSON renderer settings.
            JSON: iris.JSON{
                Indent: "  ",
            },
            // OR
            // Render as XML:
            // RenderXML: true,
            // XML:       iris.XML{Indent: "  "},
            // Sets the "Retry-After" response header.
            //
            // Can accept:
            // time.Time for HTTP-Date,
            // time.Duration, int64, float64, int for seconds
            // or string for date or duration.
            // Examples:
            // time.Now().Add(5 * time.Minute),
            // 300 * time.Second,
            // "5m",
            //
            RetryAfter: 300,
            // A function that, if specified, can dynamically set
            // retry-after based on the request.
            // Useful for ProblemOptions reusability.
            // Overrides the RetryAfter field.
            //
            // RetryAfterFunc: func(iris.Context) interface{} { [...] }
    })
}
```

输出 `application/problem+json`

```json
{
  "type": "https://host.domain/product-error",
  "status": 400,
  "title": "Product validation problem",
  "detail": "problem error details",
  "productName": "product name"
}
```

当 `RenderXML` 设置为 `true` 的时候，响应将被渲染为 `xml`。

输出 `application/problem+xml`

```xml
<Problem>
    <Type>https://host.domain/product-error</Type>
    <Status>400</Status>
    <Title>Product validation problem</Title>
    <Detail>problem error details</Detail>
    <ProductName>product name</ProductName>
</Problem>
```