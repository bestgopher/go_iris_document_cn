在一个公共的、远程地址而不是localhost的 "真实的环境" 下测试会不会很棒？

有很多提供此类功能的第三方库，但是以我的观点，[ngrok](https://github.com/inconshreveable/ngrok) 是它们中最棒的一个。像 Iris 一样，它受欢迎并且经过了多年的测试。

Iris 提供了 ngrok 的集成。这个功能简单但是强大。当你想要向你的同事或者项目领导在一个远程会议上快速展示你的开发进度，它将会很有帮助。

请按照以下步骤临时将您的本地Iris Web服务器转换为公共服务器。

1. [下载ngrok](https://ngrok.com/)，然后把它加入到你的 `$PATH` 环境变量中。
2. 简单传递 `WithTunneling` 选项到 `app.Run` 中
3. 启动

![](https://i.imgur.com/h8tgdOq.png)

-  `ctx.Application().ConfigurationReadOnly().GetVHost()` 返回公共域名。很少有用，但是可以为您服务。更多的时候你使用相对 URL 路径来替代绝对路径。
- ngrok是否已在运行都无关紧要，Iris框架足够聪明，可以使用ngrok的Web API创建隧道。

完整的 `Tunneling` 配置：


```go
app.Run(iris.Addr(":8080"), iris.WithConfiguration(
    iris.Configuration{
        Tunneling: iris.TunnelingConfiguration{
            AuthToken:    "my-ngrok-auth-client-token",
            Bin:          "/bin/path/for/ngrok",
            Region:       "eu",
            WebInterface: "127.0.0.1:4040",
            Tunnels: []iris.Tunnel{
                {
                    Name: "MyApp",
                    Addr: ":8080",
                },
            },
        },
}))
```
