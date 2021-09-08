---
title: Beego框架入门
tags:
  - go
  - beego
comments: true
typora-root-url: Beego框架入门
date: 2021-08-29 14:40:20
categories: Beego
index_img: /2021/08/29/Beego框架入门/beego.jpeg
banner_img:
---



# beego 简介

beego 是一个快速开发 Go 应用的 HTTP 框架，他可以用来快速开发 API、Web 及后端服务等各种应用，是一个 RESTful 的框架，主要设计灵感来源于 tornado、sinatra 和 flask 这三个框架，但是结合了 Go 本身的一些特性（interface、struct 嵌入等）而设计的一个框架。

## beego 的架构

beego 的整体设计架构如下所示：

![](architecture.png)

beego 是基于八大独立的模块构建的，是一个高度解耦的框架。当初设计 beego 的时候就是考虑功能模块化，用户即使不使用 beego 的 HTTP 逻辑，也依旧可以使用这些独立模块，例如：你可以使用 cache 模块来做你的缓存逻辑；使用日志模块来记录你的操作信息；使用 config 模块来解析你各种格式的文件。所以 beego 不仅可以用于 HTTP 类的应用开发，在你的 socket 游戏开发中也是很有用的模块，这也是 beego 为什么受欢迎的一个原因。

## beego 的执行逻辑

既然 beego 是基于这些模块构建的，那么它的执行逻辑是怎么样的呢？beego 是一个典型的 MVC 架构，它的执行逻辑如下图所示：

![](flow.png)

## beego 项目结构

一般的 beego 项目的目录如下所示：

```shell
├── conf
│   └── app.conf
├── controllers
│   ├── admin
│   └── default.go
├── main.go
├── models
│   └── models.go
├── static
│   ├── css
│   ├── ico
│   ├── img
│   └── js
└── views
    ├── admin
    └── index.tpl
```

从上面的目录结构我们可以看出来 M（models 目录）、V（views 目录）和 C（controllers 目录）的结构， `main.go` 是入口文件。

# bee 工具简介

bee 工具是一个为了协助快速开发 beego 项目而创建的项目，通过 bee 您可以很容易的进行 beego 项目的创建、热编译、开发、测试、和部署。

## bee 工具的安装

您可以通过如下的方式安装 bee 工具：

```
go get -u github.com/beego/bee/v2
```

安装完之后，`bee` 可执行文件默认存放在 `$GOPATH/bin` 里面，所以您需要把 `$GOPATH/bin` 添加到您的环境变量中，才可以进行下一步。

> > > 如果你本机设置了 `GOBIN`，那么上面的`bee`命令就会安装到 `GOBIN` 目录下，所以我们需要在环境变量中添加相关的配置信息，如何添加可以查看这篇文档: [bee 环境变量配置](https://beego.me/docs/install/env.md)
> > >
> > > ## bee 工具命令详解

我们在命令行输入 `bee`，可以看到如下的信息：

```
Bee is a Fast and Flexible tool for managing your Beego Web Application.

Usage:

    bee command [arguments]

The commands are:

    version     show the bee & beego version
    migrate     run database migrations
    api         create an api application base on beego framework
    bale        packs non-Go files to Go source files    
    new         create an application base on beego framework
    run         run the app which can hot compile
    pack        compress an beego project
    fix         Fixes your application by making it compatible with newer versions of Beego
    dlv         Start a debugging session using Delve
    dockerize   Generates a Dockerfile for your Beego application
    generate    Source code generator
    hprose      Creates an RPC application based on Hprose and Beego frameworks
    pack        Compresses a Beego application into a single file
    rs          Run customized scripts
    run         Run the application by starting a local development server
    server      serving static content over HTTP on port
    
Use bee help [command] for more information about a command.
    
```

### `new` 命令

`new` 命令是新建一个 Web 项目，我们在命令行下执行 `bee new <项目名>` 就可以创建一个新的项目。但是注意该命令必须在 `$GOPATH/src` 下执行。最后会在 `$GOPATH/src` 相应目录下生成如下目录结构的项目：

```
bee new myproject
[INFO] Creating application...
/gopath/src/myproject/
/gopath/src/myproject/conf/
/gopath/src/myproject/controllers/
/gopath/src/myproject/models/
/gopath/src/myproject/static/
/gopath/src/myproject/static/js/
/gopath/src/myproject/static/css/
/gopath/src/myproject/static/img/
/gopath/src/myproject/views/
/gopath/src/myproject/conf/app.conf
/gopath/src/myproject/controllers/default.go
/gopath/src/myproject/views/index.tpl
/gopath/src/myproject/main.go
13-11-25 09:50:39 [SUCC] New application successfully created!
myproject
├── conf
│   └── app.conf
├── controllers
│   └── default.go
├── main.go
├── models
├── routers
│   └── router.go
├── static
│   ├── css
│   ├── img
│   └── js
├── tests
│   └── default_test.go
└── views
    └── index.tpl

8 directories, 4 files
```

### `api` 命令

上面的 `new` 命令是用来新建 Web 项目，不过很多用户使用 beego 来开发 API 应用。所以这个 `api` 命令就是用来创建 API 应用的，执行命令之后如下所示：

```
bee api apiproject
create app folder: /gopath/src/apiproject
create conf: /gopath/src/apiproject/conf
create controllers: /gopath/src/apiproject/controllers
create models: /gopath/src/apiproject/models
create tests: /gopath/src/apiproject/tests
create conf app.conf: /gopath/src/apiproject/conf/app.conf
create controllers default.go: /gopath/src/apiproject/controllers/default.go
create tests default.go: /gopath/src/apiproject/tests/default_test.go
create models object.go: /gopath/src/apiproject/models/object.go
create main.go: /gopath/src/apiproject/main.go
```

这个项目的目录结构如下：

```
apiproject
├── conf
│   └── app.conf
├── controllers
│   └── object.go
│   └── user.go
├── docs
│   └── doc.go
├── main.go
├── models
│   └── object.go
│   └── user.go
├── routers
│   └── router.go
└── tests
    └── default_test.go
```

从上面的目录我们可以看到和 Web 项目相比，少了 static 和 views 目录，多了一个 test 模块，用来做单元测试的。

同时，该命令还支持一些自定义参数自动连接数据库创建相关 model 和 controller:
`bee api [appname] [-tables=""] [-driver=mysql] [-conn="root:<password>@tcp(127.0.0.1:3306)/test"]`
如果 conn 参数为空则创建一个示例项目，否则将基于链接信息链接数据库创建项目。

### `run` 命令

我们在开发 Go 项目的时候最大的问题是经常需要自己手动去编译再运行，`bee run` 命令是监控 beego 的项目，通过 [fsnotify](https://github.com/howeyc/fsnotify)监控文件系统。但是注意该命令必须在 `$GOPATH/src/appname` 下执行。
这样我们在开发过程中就可以实时的看到项目修改之后的效果：

```go
PS D:\go\src\Beego_learn\quickstart> bee run
______
| ___ \
| |_/ /  ___   ___
| ___ \ / _ \ / _ \
| |_/ /|  __/|  __/
\____/  \___| \___| v2.0.2
2021/08/29 15:58:18 INFO     ▶ 0001 Using 'quickstart' as 'appname'
2021/08/29 15:58:18 INFO     ▶ 0002 Initializing watcher...
2021/08/29 15:58:20 SUCCESS  ▶ 0003 Built Successfully!
2021/08/29 15:58:20 INFO     ▶ 0004 Restarting 'quickstart.exe'...
2021/08/29 15:58:20 SUCCESS  ▶ 0005 './quickstart.exe' is running...
2021/08/29 15:58:22.845 [I] [parser.go:413]  generate router from comments

2021/08/29 15:58:22.857 [I] [server.go:241]  http server Running on http://:8080
```

我们打开浏览器就可以看到效果 `http://localhost:8080/`:

![](https://beego.me/docs/images/beerun.png)

如果我们修改了 `Controller` 下面的 `default.go` 文件，我们就可以看到命令行输出：

```go
quickstart/controllers
quickstart/routers
quickstart
2021/08/29 15:59:12 SUCCESS  ▶ 0006 Built Successfully!
2021/08/29 15:59:12 INFO     ▶ 0007 Restarting 'quickstart.exe'...
2021/08/29 15:59:12 SUCCESS  ▶ 0008 './quickstart.exe' is running...
2021/08/29 15:59:15.385 [I] [parser.go:413]  generate router from comments

2021/08/29 15:59:15.397 [I] [server.go:241]  http server Running on http://:8080
```

刷新浏览器我们看到新的修改内容已经输出。

### `pack` 命令

`pack` 目录用来发布应用的时候打包，会把项目打包成 zip 包，这样我们部署的时候直接把打包之后的项目上传，解压就可以部署了：

```
bee pack
app path: /gopath/src/apiproject
GOOS darwin GOARCH amd64
build apiproject
build success
exclude prefix:
exclude suffix: .go:.DS_Store:.tmp
file write to `/gopath/src/apiproject/apiproject.tar.gz`
```

我们可以看到目录下有如下的压缩文件：

```
rwxr-xr-x  1 astaxie  staff  8995376 11 25 22:46 apiproject
-rw-r--r--  1 astaxie  staff  2240288 11 25 22:58 apiproject.tar.gz
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 conf
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 controllers
-rw-r--r--  1 astaxie  staff      509 11 25 22:31 main.go
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 models
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 tests
```

### `bale` 命令

这个命令目前仅限内部使用，具体实现方案未完善，主要用来压缩所有的静态文件变成一个变量申明文件，全部编译到二进制文件里面，用户发布的时候携带静态文件，包括 js、css、img 和 views。最后在启动运行时进行非覆盖式的自解压。

### `version` 命令

这个命令是动态获取 bee、beego 和 Go 的版本，这样一旦用户出现错误，可以通过该命令来查看当前的版本

```
$ bee version
bee   :1.2.2
beego :1.4.2
Go    :go version go1.3.3 darwin/amd64
```

需要注意的是，目前 `bee version` 会试图输出当前`beego`的版本。

但是目前这个实现有点坑，它是通过读取`$GOPATH/src/astaxie/beego`下的文件来进行的。

这意味着，如果你本地并没有下载`beego`源码，或者放置的位置不对，`bee`都无法输出`beego`的版本信息。

### `generate` 命令

这个命令是用来自动化的生成代码的，包含了从数据库一键生成 model，还包含了 scaffold 的，通过这个命令，让大家开发代码不再慢

```
bee generate scaffold [scaffoldname] [-fields=""] [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    The generate scaffold command will do a number of things for you.
    -fields: a list of table fields. Format: field:type, ...
    -driver: [mysql | postgres | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test
    example: bee generate scaffold post -fields="title:string,body:text"

bee generate model [modelname] [-fields=""]
    generate RESTful model based on fields
    -fields: a list of table fields. Format: field:type, ...

bee generate controller [controllerfile]
    generate RESTful controllers

bee generate view [viewpath]
    generate CRUD view in viewpath

bee generate migration [migrationfile] [-fields=""]
    generate migration file for making database schema update
    -fields: a list of table fields. Format: field:type, ...

bee generate docs
    generate swagger doc file

bee generate test [routerfile]
    generate testcase

bee generate appcode [-tables=""] [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"] [-level=3]
    generate appcode based on an existing database
    -tables: a list of table names separated by ',', default is empty, indicating all tables
    -driver: [mysql | postgres | sqlite], the default is mysql
    -conn:   the connection string used by the driver.
             default for mysql:    root:@tcp(127.0.0.1:3306)/test
             default for postgres: postgres://postgres:postgres@127.0.0.1:5432/postgres
    -level:  [1 | 2 | 3], 1 = models; 2 = models,controllers; 3 = models,controllers,router
```

### `migrate` 命令

这个命令是应用的数据库迁移命令，主要是用来每次应用升级，降级的SQL管理。

```
bee migrate [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    run all outstanding migrations
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate rollback [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback the last migration operation
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate reset [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback all migrations
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate refresh [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback all migrations and run them all again
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test
```

### `dockerize` 命令

这个命令可以通过生成Dockerfile文件来实现docker化你的应用。

例子:
生成一个以1.6.4版本Go环境为基础镜像的Dockerfile,并暴露9000端口:

```
$ bee dockerize -image="library/golang:1.6.4" -expose=9000
______
| ___ \
| |_/ /  ___   ___
| ___ \ / _ \ / _ \
| |_/ /|  __/|  __/
\____/  \___| \___| v1.6.2
2016/12/26 22:34:54 INFO     ▶ 0001 Generating Dockerfile...
2016/12/26 22:34:54 SUCCESS  ▶ 0002 Dockerfile generated.
```

更多帮助信息可执行`bee help dockerize`.

## bee 工具配置文件

您可能已经注意到，在 bee 工具的源码目录下有一个 `bee.json` 文件，这个文件是针对 bee 工具的一些行为进行配置。该功能还未完全开发完成，不过其中的一些选项已经可以使用：

- `"version": 0`：配置文件版本，用于对比是否发生不兼容的配置格式版本。
- `"go_install": false`：如果您的包均使用完整的导入路径（例如：`github.com/user/repo/subpkg`）,则可以启用该选项来进行 `go install` 操作，加快构建操作。
- `"watch_ext": []`：用于监控其它类型的文件（默认只监控后缀为 `.go` 的文件）。
- `"dir_structure":{}`：如果您的目录名与默认的 MVC 架构的不同，则可以使用该选项进行修改。
- `"cmd_args": []`：如果您需要在每次启动时加入启动参数，则可以使用该选项。
- `"envs": []`：如果您需要在每次启动时设置临时环境变量参数，则可以使用该选项。

# Beego 的安装

beego 的安装是典型的 Go 安装包的形式：

```shell
go get github.com/beego/beego/v2
```

常见问题：

- git 没有安装，请自行安装不同平台的 git，如何安装请自行搜索。

- git https 无法获取，请配置本地的 git，关闭 https 验证：

  ```shell
  git config --global http.sslVerify false
  ```

# beego 的升级

beego 升级分为 go 方式升级和源码下载升级：

- Go 升级,通过该方式用户可以升级 beego 框架，强烈推荐该方式：

  ```shell
  go get -u github.com/beego/beego/v2
  ```

- 源码下载升级，用户访问 `https://github.com/beego/beego` ,下载源码，然后覆盖到 `$GOPATH/src/github.com/beego/beego` 目录，然后通过本地执行安装就可以升级了：

  ```shell
  go install  github.com/beego/beego/v2
  ```

# 创建项目

beego 的项目基本都是通过 `bee` 命令来创建的，所以在创建项目之前确保你已经安装了 bee 工具和 beego。如果你还没有安装，那么请查阅 [beego 的安装](https://beego.me/docs/install) 和 [bee 工具的安装](https://beego.me/docs/install/bee.md)。

现在一切就绪我们就可以开始创建项目了，打开终端，进入 `$GOPATH/src` 所在的目录：

```
➜  src  bee new quickstart
[INFO] Creating application...
/gopath/src/quickstart/
/gopath/src/quickstart/conf/
/gopath/src/quickstart/controllers/
/gopath/src/quickstart/models/
/gopath/src/quickstart/routers/
/gopath/src/quickstart/tests/
/gopath/src/quickstart/static/
/gopath/src/quickstart/static/js/
/gopath/src/quickstart/static/css/
/gopath/src/quickstart/static/img/
/gopath/src/quickstart/views/
/gopath/src/quickstart/conf/app.conf
/gopath/src/quickstart/controllers/default.go
/gopath/src/quickstart/views/index.tpl
/gopath/src/quickstart/routers/router.go
/gopath/src/quickstart/tests/default_test.go
/gopath/src/quickstart/main.go
2014/11/06 18:17:09 [SUCC] New application successfully created!
```

通过一个简单的命令就创建了一个 beego 项目。他的目录结构如下所示

```
quickstart
|-- conf
|   `-- app.conf
|-- controllers
|   `-- default.go
|-- main.go
|-- models
|-- routers
|   `-- router.go
|-- static
|   |-- css
|   |-- img
|   `-- js
|-- tests
|   `-- default_test.go
`-- views
    `-- index.tpl
```

从目录结构中我们也可以看出来这是一个典型的 MVC 架构的应用，`main.go` 是入口文件。

## 运行项目

beego 项目创建之后，我们还需要初始化`go.mod`文件。进入目录之后，使用`go mod init`初始化模块依赖。

接着我们就开始运行项目，首先进入创建的项目，我们使用 `bee run` 来运行该项目，这样就可以做到热编译的效果：

```
PS D:\go\src\Beego_learn\quickstart> bee run
______
| ___ \
| |_/ /  ___   ___
| ___ \ / _ \ / _ \
| |_/ /|  __/|  __/
\____/  \___| \___| v2.0.2
2021/08/29 15:58:18 INFO     ▶ 0001 Using 'quickstart' as 'appname'
2021/08/29 15:58:18 INFO     ▶ 0002 Initializing watcher...
2021/08/29 15:58:20 SUCCESS  ▶ 0003 Built Successfully!
2021/08/29 15:58:20 INFO     ▶ 0004 Restarting 'quickstart.exe'...
2021/08/29 15:58:20 SUCCESS  ▶ 0005 './quickstart.exe' is running...
2021/08/29 15:58:22.845 [I] [parser.go:413]  generate router from comments

2021/08/29 15:58:22.857 [I] [server.go:241]  http server Running on http://:8080
```

这样我们的应用已经在 `8080` 端口(beego 的默认端口)跑起来了，打开浏览器看看效果：

![](beerun.png)

# 项目路由设置

前面我们已经创建了 beego 项目，而且我们也看到它已经运行起来了，那么是如何运行起来的呢？让我们从入口文件先分析起来吧：

```
package main

import (
    _ "quickstart/routers"
    beego "github.com/beego/beego/v2/server/web"
)

func main() {
    beego.Run()
}
```

我们看到 main 函数是入口函数，但是我们知道 Go 的执行过程是如下图所示的方式：

![](image-20210829153057181.png)

这里我们就看到了我们引入了一个包 `_ "quickstart/routers"`,这个包只引入执行了里面的 init 函数，那么让我们看看这个里面做了什么事情：

```
package routers

import (
    "quickstart/controllers"
    beego "github.com/beego/beego/v2/server/web"
)

func init() {
    beego.Router("/", &controllers.MainController{})
}
```

路由包里面我们看到执行了路由注册 `web.Router`, 这个函数的功能是映射 URL 到 controller，第一个参数是 URL (用户请求的地址)，这里我们注册的是 `/`，也就是我们访问的不带任何参数的 URL，第二个参数是对应的 Controller，也就是我们即将把请求分发到那个控制器来执行相应的逻辑，我们可以执行类似的方式注册如下路由：

```
bee.Router("/user", &controllers.UserController{})
```

这样用户就可以通过访问 `/user` 去执行 `UserController` 的逻辑。这就是我们所谓的路由，更多更复杂的路由规则请查询 [web 的路由设置](https://beego.me/docs/mvc/controller/router.md)

再回来看看 main 函数里面的 `web.Run`， `web.Run` 执行之后，我们看到的效果好像只是监听服务端口这个过程，但是它内部做了很多事情：

- 解析配置文件

  beego 会自动解析在 conf 目录下面的配置文件 `app.conf`，通过修改配置文件相关的属性，我们可以定义：开启的端口，是否开启 session，应用名称等信息。

- 执行用户的 hookfunc

  beego 会执行用户注册的 hookfunc，默认的已经存在了注册 mime，用户可以通过函数 `AddAPPStartHook` 注册自己的启动函数。

- 是否开启 session

  会根据上面配置文件的分析之后判断是否开启 session，如果开启的话就初始化全局的 session。

- 是否编译模板

  beego 会在启动的时候根据配置把 views 目录下的所有模板进行预编译，然后存在 map 里面，这样可以有效的提高模板运行的效率，无需进行多次编译。

- 是否开启文档功能

  根据 EnableDocs 配置判断是否开启内置的文档路由功能

- 是否启动管理模块

  beego 目前做了一个很酷的模块，应用内[监控模块](https://beego.me/docs/advantage/monitor.md)，会在 8088 端口做一个内部监听，我们可以通过这个端口查询到 QPS、CPU、内存、GC、goroutine、thread 等统计信息。

- 监听服务端口

  这是最后一步也就是我们看到的访问 8080 看到的网页端口，内部其实调用了 `ListenAndServe`，充分利用了 goroutine 的优势

一旦 run 起来之后，我们的服务就监听在两个端口了，一个服务端口 8080 作为对外服务，另一个 8088 端口实行对内监控。

通过这个代码的分析我们了解了 beego 运行起来的过程，以及内部的一些机制。接下来让我们去剥离 Controller 如何来处理逻辑的。

# controller 逻辑

前面我们了解了如何把用户的请求分发到控制器，这小节我们就介绍大家如何来写控制器，首先我们还是从源码分析入手：

```go
package controllers

import (
        beego "github.com/beego/beego/v2/server/web"
)

type MainController struct {
        beego.Controller
}

func (this *MainController) Get() {
        this.Data["Website"] = "beego.me"
        this.Data["Email"] = "astaxie@gmail.com"
        this.TplName = "index.tpl"
}
```

上面的代码显示首先我们声明了一个控制器 `MainController`，这个控制器里面组合了 `web.Controller`，这就是 Go 的组合方式，也就是 `MainController` 自动拥有了所有 `web.Controller` 的方法。

而 `web.Controller` 拥有很多方法，其中包括 `Init`、`Prepare`、`Post`、`Get`、`Delete`、`Head` 等方法。我们可以通过重写的方式来实现这些方法，而我们上面的代码就是重写了 `Get` 方法。

我们先前介绍过 beego 是一个 RESTful 的框架，所以我们的请求默认是执行对应 `req.Method` 的方法。例如浏览器的是 `GET` 请求，那么默认就会执行 `MainController` 下的 `Get` 方法。这样我们上面的 Get 方法就会被执行到，这样就进入了我们的逻辑处理。（用户可以改变这个行为，通过注册自定义的函数名，更加详细的请参考[路由设置](https://beego.me/docs/mvc/controller/router.md#自定义方法及-restful-规则)）

里面的代码是需要执行的逻辑，这里只是简单的输出数据，我们可以通过各种方式获取数据，然后赋值到 `this.Data` 中，这是一个用来存储输出数据的 map，可以赋值任意类型的值，这里我们只是简单举例输出两个字符串。

最后一个就是需要去渲染的模板，`this.TplName` 就是需要渲染的模板，这里指定了 `index.tpl`，如果用户不设置该参数，那么默认会去到模板目录的 `Controller/<方法名>.tpl` 查找，例如上面的方法会去 `maincontroller/get.tpl` ***(文件、文件夹必须小写)\***。

用户设置了模板之后系统会自动的调用 `Render` 函数（这个函数是在 `web.Controller` 中实现的），所以无需用户自己来调用渲染。

当然也可以不使用模版，直接用 `this.Ctx.WriteString` 输出字符串，如：

```
func (this *MainController) Get() {
        this.Ctx.WriteString("hello")
}
```

至此我们的控制器分析基本完成了，接下来让我们看看如何来编写 model。

我们知道 Web 应用中我们用的最多的就是数据库操作，而 model 层一般用来做这些操作，我们的 `bee new` 例子不存在 Model 的演示，但是 `bee api` 应用中存在 model 的应用。说的简单一点，如果您的应用足够简单，那么 Controller 可以处理一切的逻辑，如果您的逻辑里面存在着可以复用的东西，那么就抽取出来变成一个模块。因此 Model 就是逐步抽象的过程，一般我们会在 Model 里面处理一些数据读取，如下是一个日志分析应用中的代码片段：

```go
package models

import (
    "path/filepath"
    "strings"
)

var (
    NotPV []string = []string{"css", "js", "class", "gif", "jpg", "jpeg", "png", "bmp", "ico", "rss", "xml", "swf"}
)

const big = 0xFFFFFF

func LogPV(urls string) bool {
    ext := filepath.Ext(urls)
    if ext == "" {
        return true
    }
    for _, v := range NotPV {
        if v == strings.ToLower(ext) {
            return false
        }
    }
    return true
}
```

所以如果您的应用足够简单，那么就不需要 Model 了；如果你的模块开始多了，需要复用，需要逻辑分离了，那么 Model 是必不可少的。接下来我们将分析如何编写 View 层的东西。

# View 编写

在前面编写 Controller 的时候，我们在 Get 里面写过这样的语句 `this.TplName = "index.tpl"`，设置显示的模板文件，默认支持 `tpl` 和 `html` 的后缀名，如果想设置其他后缀你可以调用 `beego.AddTemplateExt` 接口设置，那么模板如何来显示相应的数据呢？beego 采用了 Go 语言默认的模板引擎，所以和 Go 的模板语法一样，Go 模板的详细使用方法请参考[《Go Web 编程》模板使用指南](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/07.4.md)

我们看看快速入门里面的代码（去掉了 css 样式）：

```
<!DOCTYPE html>

<html>
    <head>
        <title>Beego</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    </head>
    <body>
        <header class="hero-unit" style="background-color:#A9F16C">
            <div class="container">
                <div class="row">
                    <div class="hero-text">
                        <h1>Welcome to Beego!</h1>
                        <p class="description">
                            Beego is a simple & powerful Go web framework which is inspired by tornado and sinatra.
                            <br />
                            Official website: <a href="http://{{.Website}}">{{.Website}}</a>
                            <br />
                            Contact me: {{.Email}}
                        </p>
                    </div>
                </div>
            </div>
        </header>
    </body>
</html>
```

我们在 Controller 里面把数据赋值给了 data（map 类型），然后我们在模板中就直接通过 key 访问 `.Website` 和 `.Email` 。这样就做到了数据的输出。接下来我们讲解如何让静态文件输出。

# 静态文件处理

前面我们介绍了如何输出静态页面，但是我们的网页往往包含了很多的静态文件，包括图片、JS、CSS 等，刚才创建的应用里面就创建了如下目录：

```
├── static
    │   ├── css
    │   ├── img
    │   └── js
```

beego 默认注册了 static 目录为静态处理的目录，注册样式：URL 前缀和映射的目录（在`/main.go`文件中`web.Run()`之前加入）：

```
StaticDir["/static"] = "static"
```

用户可以设置多个静态文件处理目录，例如你有多个文件下载目录 download1、download2，你可以这样映射（在 `/main.go` 文件中 `beego.Run()` 之前加入）：

```
beego.SetStaticPath("/down1", "download1")
beego.SetStaticPath("/down2", "download2")
```

这样用户访问 URL `http://localhost:8080/down1/123.txt` 则会请求 download1 目录下的 123.txt 文件。













[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


