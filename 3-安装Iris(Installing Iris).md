<font color=red>**`Iris`**</font> 是一个跨平台的软件。

唯一的需求就是 Go 语言，版本为1.13。

```shell
$ cd $YOUR_PROJECT_PATH 
$ export GO111MODULE=on
```

## 安装(Install)

```shell
go get github.com/kataras/iris/v12@latest
```

或者编辑你项目的 <font color=red>**`go.mod`**</font> 文件。

	module your_project_name
	
	go 1.13
	
	require (
	    github.com/kataras/iris/v12 v12.0.0
	)

然后：

```shell
go build
```


## 怎么更新(How to update)
这个 `go-get` 命令来获取最新和最好的  <font color=red>**`Iris`**</font> 版本。 `Master` 分支通常足够稳定了。

```shell
$ go get -u github.com/kataras/iris/v12@latest
```

## 故障排除(Troubleshooting)

如果你在安装过程中出现了网络错误，你确保你设置了一个可用的 `GOPROXY` 环境变量，例如 `GOPROXY=https://goproxy.io` 或者 `GOPROXY=https://goproxy.cn`。