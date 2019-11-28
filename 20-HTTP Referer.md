HTTP referer (本来是 `referrer` 的拼写错误) 是一个可选的 HTTP 头部字段， 用于标记链接到请求资源的网页的地址(即 URI 或者 IRI)。通过检查 referrer，新的网页可以知道请求的来源。

Iris 使用 ` Shopify's goreferrer` 包来实现 `Context.GetReferrer()` 方法。

`GetReferrer` 方法提取和返回 `Referer` 头的信息，`Referer` 通过 **https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy** 或者 URL 的 `referer` 查询参数(`query parameter`)指定。

	GetReferrer() Referrer

`Referrer` 是这样的：

	type (
	    Referrer struct {
	        Type       ReferrerType
	        Label      string
	        URL        string
	        Subdomain  string
	        Domain     string
	        Tld        string         
	        Path       string              
	        Query      string                 
	        GoogleType ReferrerGoogleSearchType
	    }

`ReferrerType` 是 `Referrer.Type` 值( `indirect`，`direct`，`email`，`search`，`social`)的枚举。可以的类型有：

	ReferrerInvalid
	ReferrerIndirect
	ReferrerDirect
	ReferrerEmail
	ReferrerSearch
	ReferrerSocial

`GoogleType` 可以是下列之一：

	ReferrerNotGoogleSearch
	ReferrerGoogleOrganicSearch
	ReferrerGoogleAdwords


### 示例(Example)

	package main
	
	import "github.com/kataras/iris/v12"
	
	func main() {
	    app := iris.New()
	
	    app.Get("/", func(ctx iris.Context) {
	        r := ctx.GetReferrer()
	        switch r.Type {
	        case iris.ReferrerSearch:
	            ctx.Writef("Search %s: %s\n", r.Label, r.Query)
	            ctx.Writef("Google: %s\n", r.GoogleType)
	        case iris.ReferrerSocial:
	            ctx.Writef("Social %s\n", r.Label)
	        case iris.ReferrerIndirect:
	            ctx.Writef("Indirect: %s\n", r.URL)
	        }
	    })
	
	    app.Run(iris.Addr(":8080"))
	}

`curl`：

	curl http://localhost:8080?\
	referer=https://twitter.com/Xinterio/status/1023566830974251008
	
	curl http://localhost:8080?\
	referer=https://www.google.com/search?q=Top+6+golang+web+frameworks\
	&oq=Top+6+golang+web+frameworks