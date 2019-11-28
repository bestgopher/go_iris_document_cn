<font color=red>**`Iris`**</font> 拥有你见过的的最简单和强大路由处理。

<font color=red>**`Iris`**</font> 自己拥有用于路由路径语法解析和判定的解释器(就像一门编程语言)。

它是快速的。它计算它的需求，如果没有特殊的正则需要，它仅仅会使用低级的路径语法来注册路由，除此之外，它预编译正则，然后加入到必需的中间件中。这就意味着与其他的路由器或者 web 框架相比，你的性能成本为零。



## 参数(Parameters)

一个路径参数的名字应该仅仅包含 `字母`。数字和类似 "_" 这样的符号是 `不允许的`。

不要迷惑于 `ctx.Params()` 和 `ctx.Values()`

-   路径的参数值可以通过 `ctx.Params()` 取出。
-   `ctx` 中用于处理器与中间件之间通信的本地存储可以存储在 `ctx.Values()` 中。

下表是内建可用的参数类型：

| 参数类型 | golang类型 | 取值范围 | 取值方式|
| :------ | :----: | :----: | ------ |
| **`:string`** | **`string`** | 任何值(单个字段路径) |**`Params().Get`**|
| **`:int`** | **`int`** | -9223372036854775808 - 9223372036854775807 (x64) </br>-2147483648 - 2147483647 (x32) |**`Params().GetInt`**|
| **`:int8`** | **`int8`** | -128 - 127 |**`Params().GetInt8`**|
| **`:int16`** | **`int16`** | -32768 - 32767 |**`Params().GetInt16`**|
| **`:int32`** | **`int32`** | -2147483648 - 2147483647 |**`Params().GetInt32`**|
| **`:int64`** | **`int64`** | -9223372036854775808 - 9223372036854775807 |**`Params().GetInt64`**|
| **`:uint8`** | **`uint8`** | 0 - 255 |**`Params().GetUint8`**|
| **`:uint16`** | **`uint16`** | 0 - 65535 |**`Params().GetUint16`**|
| **`:uint32`** | **`uint32`** | 0 - 4294967295 |**`Params().GetUint32`**|
| **`:uint64`** | **`uint64`** | 0 - 18446744073709551615 |**`Params().GetUint64`**|
| **`:bool`** | **`bool`** | "1"，"t"，"T"，"TRUE"，"true"，"True"，"0"，"f"， "F"， "FALSE",，"false"，"False" |**`Params().GetBool`**|
| `:alphabetical` | **`string`** | 小写或大写字母 |**`Params().Get`**|
| **`:file`** | **`string`** | 大小写字母，数字，下划线(_)，横线(-)，点(.)，以及没有空格或者其他对文件名无效的特殊字符 |**`Params().Get`**|
| **`:path`** | **`string`** | 任何可以被斜线(/)分隔的路径段，但是应该为路由的最后一部分 |**`Params().Get`**|

##### 示例：

```go
app.Get("/users/{id:uint64}", func(ctx iris.Context){
    id := ctx.Params().GetUint64Default("id", 0)
    // [...]
})
```

### 内建函数

| 内建函数 | 参数类型 |
| :-----: | :----: |
| **`regexp(expr string)`** | **`:string`** |
| **`prefix(prefix string)`** | **`:string`** |
| **`suffix(suffix string)`** | **`:string`** |
| **`contains(s string)`** | **`:string`** |
| **`min(最小值)，接收：int，int8，int16，int32，int64，uint8`**，<br>**`uint16，uint32，uint64，float32，float64)`** | **`:string(字符长度)，:int，:int16，:int32，:int64`**，</br>**`:uint，:uint16，:uint32，:uint64`** |
| **`max(最大值)，接收：int，int8，int16，int32，int64，uint8`**，<br>**`uint16，uint32，uint64，float32，float64)`** | **`:string(字符长度)，:int，:int16，:int32，:int64`**，</br>**`:uint，:uint16，:uint32，:uint64`** |
| **`range(最小值，最大值)，接收：int，int8，int16，int32，int64，uint8`**，<br>**`uint16，uint32，uint64，float32，float64)`** | **`:int，:int16，:int32，:int64`**，</br>**`:uint，:uint16，:uint32，:uint64`** |

##### 示例：

```go
app.Get("/profile/{name:alphabetical max(255)}", func(ctx iris.Context){
    name := ctx.Params().Get("name")
    // len(name) <=255 otherwise this route will fire 404 Not Found
    // and this handler will not be executed at all.
})
```

### 自己做

`RegisterFunc` 可以接受任何返回 `func(paramValue string) bool` 的函数。如果验证失败将会触发 `404` 或者任意 `else`关键字拥有的状态码。

```go
latLonExpr := "^-?[0-9]{1,3}(?:\\.[0-9]{1,10})?$"
latLonRegex, _ := regexp.Compile(latLonExpr)

// Register your custom argument-less macro function to the :string param type.
// MatchString is a type of func(string) bool, so we use it as it is.
app.Macros().Get("string").RegisterFunc("coordinate", latLonRegex.MatchString)

app.Get("/coordinates/{lat:string coordinate()}/{lon:string coordinate()}",
func(ctx iris.Context) {
    ctx.Writef("Lat: %s | Lon: %s", ctx.Params().Get("lat"), ctx.Params().Get("lon"))
})
```

注册接受两个 `int` 参数的自定义的宏函数。

```go
app.Macros().Get("string").RegisterFunc("range",
func(minLength, maxLength int) func(string) bool {
    return func(paramValue string) bool {
        return len(paramValue) >= minLength && len(paramValue) <= maxLength
    }
})

app.Get("/limitchar/{name:string range(1,200) else 400}", func(ctx iris.Context) {
    name := ctx.Params().Get("name")
    ctx.Writef(`Hello %s | the name should be between 1 and 200 characters length
    otherwise this handler will not be executed`, name)
})
```

注册接受一个 `[]string` 参数的自定义的宏函数。

```go
app.Macros().Get("string").RegisterFunc("has",
func(validNames []string) func(string) bool {
    return func(paramValue string) bool {
        for _, validName := range validNames {
            if validName == paramValue {
                return true
            }
        }

        return false
    }
})

app.Get("/static_validation/{name:string has([kataras,maropoulos])}",
func(ctx iris.Context) {
    name := ctx.Params().Get("name")
    ctx.Writef(`Hello %s | the name should be "kataras" or "maropoulos"
    otherwise this handler will not be executed`, name)
})
```

示例代码：

```go
func main() {
    app := iris.Default()

    // This handler will match /user/john but will not match neither /user/ or /user.
    app.Get("/user/{name}", func(ctx iris.Context) {
        name := ctx.Params().Get("name")
        ctx.Writef("Hello %s", name)
    })

    // This handler will match /users/42
    // but will not match /users/-1 because uint should be bigger than zero
    // neither /users or /users/.
    app.Get("/users/{id:uint64}", func(ctx iris.Context) {
        id := ctx.Params().GetUint64Default("id", 0)
        ctx.Writef("User with ID: %d", id)
    })

    // However, this one will match /user/john/send and also /user/john/everything/else/here
    // but will not match /user/john neither /user/john/.
    app.Post("/user/{name:string}/{action:path}", func(ctx iris.Context) {
        name := ctx.Params().Get("name")
        action := ctx.Params().Get("action")
        message := name + " is " + action
        ctx.WriteString(message)
    })

    app.Run(iris.Addr(":8080"))
}
```

当没有指定参数类型时，默认为 `string`，因此 `{name:string}` 和 `{name}` 是完全相同的。