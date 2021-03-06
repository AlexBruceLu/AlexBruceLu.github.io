title: git 新建仓库，添加远程分之到本地

tags:

- go
- go mod

categories:

- 编程语言
- GO

date: 2019/12/09 18:56:13

------

@[TOC]

## 准备

### 开启go mod

1. 把 golang 升级到 1.11（现在1.13 已经发布了，建议使用1.13）
2. 设置 `GO111MODULE`

GO111MODULE

`GO111MODULE` 有三个值：`off`, `on`和`auto（默认值）`。

- `GO111MODULE=off`，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
- `GO111MODULE=on`，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
- `GO111MODULE=auto`，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：
  - 当前目录在GOPATH/src之外且该目录包含go.mod文件
  - 当前文件在包含go.mod文件的目录下面。

> 当modules 功能启用时，依赖包的存放位置变更为`$GOPATH/pkg`，允许同一个package多个版本并存，且多个项目可以共享缓存的 module。

```bash
export GO111MODULE=on
```

### go mod常用命令

golang 提供了 `go mod`命令来管理包。

go mod 有以下命令：

| 命令     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| download | download modules to local cache(下载依赖包)                  |
| edit     | edit go.mod from tools or scripts（编辑go.mod                |
| graph    | print module requirement graph (打印模块依赖图)              |
| init     | initialize new module in current directory（在当前目录初始化mod） |
| tidy     | add missing and remove unused modules(拉取缺少的模块，移除不用的模块) |
| vendor   | make vendored copy of dependencies(将依赖复制到vendor下)     |
| verify   | verify dependencies have expected content (验证依赖是否正确） |
| why      | explain why packages or modules are needed(解释为什么需要依赖) |

## 在项目中使用

### 示例一：创建一个新项目

1. 在`GOPATH 目录之外`新建一个目录，并使用`go mod init` 初始化生成`go.mod` 文件

```
➜  ~ mkdir hello
➜  ~ cd hello
➜  hello go mod init hello
go: creating new go.mod: module hello
➜  hello ls
go.mod
➜  hello cat go.mod
module hello

go 1.12
```

> go.mod文件一旦创建后，它的内容将会被go toolchain全面掌控。go toolchain会在各类命令执行时，比如go get、go build、go mod等修改和维护go.mod文件。

go.mod 提供了`module`, `require`、`replace`和`exclude` 四个命令

- `module` 语句指定包的名字（路径）
- `require` 语句指定的依赖项模块
- `replace` 语句可以替换依赖项模块
- `exclude` 语句可以忽略依赖项模块

1. 添加依赖

新建一个 server.go 文件，写入以下代码：

```
package main

import (
    "net/http"
    
    "github.com/labstack/echo"
)

func main() {
    e := echo.New()
    e.GET("/", func(c echo.Context) error {
        return c.String(http.StatusOK, "Hello, World!")
    })
    e.Logger.Fatal(e.Start(":1323"))
}
```

执行 `go run server.go` 运行代码会发现 go mod 会自动查找依赖自动下载：

```
$ go run server.go
go: finding github.com/labstack/echo v3.3.10+incompatible
go: downloading github.com/labstack/echo v3.3.10+incompatible
go: extracting github.com/labstack/echo v3.3.10+incompatible
go: finding github.com/labstack/gommon/color latest
go: finding github.com/labstack/gommon/log latest
go: finding github.com/labstack/gommon v0.2.8
# 此处省略很多行
...

   ____    __
  / __/___/ /  ___
 / _// __/ _ \/ _ \
/___/\__/_//_/\___/ v3.3.10-dev
High performance, minimalist Go web framework
https://echo.labstack.com
____________________________________O/_______
                                    O\
⇨ http server started on [::]:1323
```

现在查看go.mod 内容：

```
$ cat go.mod

module hello

go 1.12

require (
    github.com/labstack/echo v3.3.10+incompatible // indirect
    github.com/labstack/gommon v0.2.8 // indirect
    github.com/mattn/go-colorable v0.1.1 // indirect
    github.com/mattn/go-isatty v0.0.7 // indirect
    github.com/valyala/fasttemplate v1.0.0 // indirect
    golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a // indirect
)
```

go module 安装 package 的原則是先拉最新的 release tag，若无tag则拉最新的commit，详见 [Modules官方介绍](https://github.com/golang/go/wiki/Modules)。 go 会自动生成一个 go.sum 文件来记录 dependency tree：

```
$ cat go.sum
github.com/labstack/echo v3.3.10+incompatible h1:pGRcYk231ExFAyoAjAfD85kQzRJCRI8bbnE7CX5OEgg=
github.com/labstack/echo v3.3.10+incompatible/go.mod h1:0INS7j/VjnFxD4E2wkz67b8cVwCLbBmJyDaka6Cmk1s=
github.com/labstack/gommon v0.2.8 h1:JvRqmeZcfrHC5u6uVleB4NxxNbzx6gpbJiQknDbKQu0=
github.com/labstack/gommon v0.2.8/go.mod h1:/tj9csK2iPSBvn+3NLM9e52usepMtrd5ilFYA+wQNJ4=
github.com/mattn/go-colorable v0.1.1 h1:G1f5SKeVxmagw/IyvzvtZE4Gybcc4Tr1tf7I8z0XgOg=
github.com/mattn/go-colorable v0.1.1/go.mod h1:FuOcm+DKB9mbwrcAfNl7/TZVBZ6rcnceauSikq3lYCQ=
... 省略很多行
```

1. 再次执行脚本 `go run server.go` 发现跳过了检查并安装依赖的步骤。
2. 可以使用命令 `go list -m -u all` 来检查可以升级的package，使用`go get -u need-upgrade-package` 升级后会将新的依赖版本更新到go.mod
   - 也可以使用 `go get -u` 升级所有依赖

#### go get 升级

- 运行 go get -u 将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号)
- 运行 go get -u=patch 将会升级到最新的修订版本
- 运行 go get package@version 将会升级到指定的版本号version
- 运行go get如果有版本的更改，那么go.mod文件也会更改

### 示例二：改造现有项目(helloword)

项目目录为：

```
$ tree
.
├── api
│   └── apis.go
└── server.go

1 directory, 2 files
```

server.go 源码为：

```
package main

import (
    api "./api"  // 这里使用的是相对路径
    "github.com/labstack/echo"
)

func main() {
    e := echo.New()
    e.GET("/", api.HelloWorld)
    e.Logger.Fatal(e.Start(":1323"))
}
```

api/apis.go 源码为：

```
package api

import (
    "net/http"

    "github.com/labstack/echo"
)

func HelloWorld(c echo.Context) error {
    return c.JSON(http.StatusOK, "hello world")
}
```

1. 使用 `go mod init ***` 初始化go.mod

```
$ go mod init helloworld
go: creating new go.mod: module helloworld
```

1. 运行 `go run server.go`

```
go: finding github.com/labstack/gommon/color latest
go: finding github.com/labstack/gommon/log latest
go: finding golang.org/x/crypto/acme/autocert latest
go: finding golang.org/x/crypto/acme latest
go: finding golang.org/x/crypto latest
build command-line-arguments: cannot find module for path _/home/gs/helloworld/api
```

首先还是会查找并下载安装依赖，然后运行脚本 `server.go`，这里会抛出一个错误：

```
build command-line-arguments: cannot find module for path _/home/gs/helloworld/api
```

但是`go.mod` 已经更新：

```
$ cat go.mod
module helloworld

go 1.12

require (
        github.com/labstack/echo v3.3.10+incompatible // indirect
        github.com/labstack/gommon v0.2.8 // indirect
        github.com/mattn/go-colorable v0.1.1 // indirect
        github.com/mattn/go-isatty v0.0.7 // indirect
        github.com/valyala/fasttemplate v1.0.0 // indirect
        golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a // indirect
)
```

#### 那为什么会抛出这个错误呢？

这是因为 server.go 中使用 internal package 的方法跟以前已经不同了，由于 go.mod会扫描同工作目录下所有 package 并且`变更引入方法`，必须将 helloworld当成路径的前缀，也就是需要写成 import helloworld/api，以往 GOPATH/dep 模式允许的 import ./api 已经失效，详情可以查看这个 [issue](https://github.com/golang/go/issues/26645)。

1. 更新旧的package import 方式

所以server.go 需要改写成：

```
package main

import (
    api "helloworld/api"  // 这是更新后的引入方法
    "github.com/labstack/echo"
)

func main() {
    e := echo.New()
    e.GET("/", api.HelloWorld)
    e.Logger.Fatal(e.Start(":1323"))
}
```

> `一个小坑`：开始在golang1.11 下使用go mod 遇到过 `go build github.com/valyala/fasttemplate: module requires go 1.12` [这种错误](https://github.com/golang/go/issues/27565)，遇到类似这种需要升级到1.12 的问题，直接升级golang1.12 就好了。幸亏是在1.12 发布后才尝试的`go mod` 🤷‍♂️

1. 到这里就和新创建一个项目没什么区别了

#### 使用replace替换无法直接获取的package

由于某些已知的原因，并不是所有的package都能成功下载，比如：`golang.org`下的包。

modules 可以通过在 go.mod 文件中使用 replace 指令替换成github上对应的库，比如：

```
replace (
    golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
)
```

或者

```
replace golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
```

## 参考链接

- [Modules官方介绍](https://github.com/golang/go/wiki/Modules)
- [Golang 1.11 新功能介紹 – Modules](https://www.lightblue.asia/golang-1-11-new-festures-modules/?doing_wp_cron=1552464864.6369309425354003906250)
- [What are Go modules and how do I use them?](https://talks.godoc.org/github.com/myitcv/talks/2018-08-15-glug-modules/main.slide#1)
- [go mod doesn't work for github.com/gomarkdown/markdown/html](https://github.com/golang/go/issues/27565)
- [再探go modules：使用与细节](https://www.cnblogs.com/apocelipes/p/10295096.html)
- [初窥Go module](https://tonybai.com/2018/07/15/hello-go-module/)

#### References

[1] Modules官方介绍: [https://github.com/golang/go/...](https://github.com/golang/go/wiki/Modules)
[2] issue: [https://github.com/golang/go/...](https://github.com/golang/go/issues/26645)
[3] 这种错误: [https://github.com/golang/go/...](https://github.com/golang/go/issues/27565)
[4] Modules官方介绍: [https://github.com/golang/go/...](https://github.com/golang/go/wiki/Modules)
[5] Golang 1.11 新功能介紹 – Modules: [https://www.lightblue.asia/go...](https://www.lightblue.asia/golang-1-11-new-festures-modules/?doing_wp_cron=1552464864.6369309425354003906250)
[6] What are Go modules and how do I use them?: [https://talks.godoc.org/githu...](https://talks.godoc.org/github.com/myitcv/talks/2018-08-15-glug-modules/main.slide#1)
[7] go mod doesn't work for github.com/gomarkdown/markdown/html : [https://github.com/golang/go/...](https://github.com/golang/go/issues/27565)
[8] 再探go modules：使用与细节: [https://www.cnblogs.com/apoce...](https://www.cnblogs.com/apocelipes/p/10295096.html)
[9] 初窥Go module: [https://tonybai.com/2018/07/1...](https://tonybai.com/2018/07/15/hello-go-module/)