Iris 为 `httpexpect`  (一个web测试框架) 提供了丰富的支持。`iris/httptest` 子包为 iris + httpexpect 提供了帮助。

如果你更喜欢 golang 的 `net/http/httptest` 标准库，你也可以使用它。Iris 尽可能与其它外部测试工具兼容。

---

## 基本认证(Basic Authentication)

在第一个示例中，我们使用 `iris/httptest` 来测试权限验证。

1. `main.go` 文件像这样：

```go
package main

import (
    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/middleware/basicauth"
)

func newApp() *iris.Application {
    app := iris.New()

    authConfig := basicauth.Config{
        Users: map[string]string{"myusername": "mypassword"},
    }

    authentication := basicauth.New(authConfig)

    app.Get("/", func(ctx iris.Context) { ctx.Redirect("/admin") })

    needAuth := app.Party("/admin", authentication)
    {
        //http://localhost:8080/admin
        needAuth.Get("/", h)
        // http://localhost:8080/admin/profile
        needAuth.Get("/profile", h)

        // http://localhost:8080/admin/settings
        needAuth.Get("/settings", h)
    }

    return app
}

func h(ctx iris.Context) {
    username, password, _ := ctx.Request().BasicAuth()
    // third parameter  ^ will be always true because the middleware
    // makes sure for that, otherwise this handler will not be executed.

    ctx.Writef("%s %s:%s", ctx.Path(), username, password)
}

func main() {
    app := newApp()
    app.Run(iris.Addr(":8080"))
}
```



2. 现在，创建一个 `main_test.go` 文件，然后复制下面的代码：

```go
package main

import (
    "testing"

    "github.com/kataras/iris/v12/httptest"
)

func TestNewApp(t *testing.T) {
    app := newApp()
    e := httptest.New(t, app)

    // redirects to /admin without basic auth
    e.GET("/").Expect().Status(httptest.StatusUnauthorized)
    // without basic auth
    e.GET("/admin").Expect().Status(httptest.StatusUnauthorized)

    // with valid basic auth
    e.GET("/admin").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin myusername:mypassword")
    e.GET("/admin/profile").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin/profile myusername:mypassword")
    e.GET("/admin/settings").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin/settings myusername:mypassword")

    // with invalid basic auth
    e.GET("/admin/settings").WithBasicAuth("invalidusername", "invalidpassword").
        Expect().Status(httptest.StatusUnauthorized)

}
```

3. 打开你的命令行然后执行：

```shell
$ go test -v
```



### 其他示例

```go
package main

import (
    "fmt"
    "testing"

    "github.com/kataras/iris/v12/httptest"
)

func TestCookiesBasic(t *testing.T) {
    app := newApp()
    e := httptest.New(t, app, httptest.URL("http://example.com"))

    cookieName, cookieValue := "my_cookie_name", "my_cookie_value"

    // Test Set A Cookie.
    t1 := e.GET(fmt.Sprintf("/cookies/%s/%s", cookieName, cookieValue)).
        Expect().Status(httptest.StatusOK)
    // Validate cookie's existence, it should be available now.
    t1.Cookie(cookieName).Value().Equal(cookieValue)
    t1.Body().Contains(cookieValue)

    path := fmt.Sprintf("/cookies/%s", cookieName)

    // Test Retrieve A Cookie.
    t2 := e.GET(path).Expect().Status(httptest.StatusOK)
    t2.Body().Equal(cookieValue)

    // Test Remove A Cookie.
    t3 := e.DELETE(path).Expect().Status(httptest.StatusOK)
    t3.Body().Contains(cookieName)

    t4 := e.GET(path).Expect().Status(httptest.StatusOK)
    t4.Cookies().Empty()
    t4.Body().Empty()
}
```

```she
$ go test -v -run=TestCookiesBasic$
```

