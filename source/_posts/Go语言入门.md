---
title: Go语言入门
tags:
  - go
comments: true
typora-root-url: Go语言入门
date: 2021-08-28 17:47:19
categories: Go
index_img:
banner_img: 
---

## 语言特性

- 闭包与匿名函数
- 错误处理
- 函数多返回值
- 自动垃圾回收
- 接口与类型
- 并发编程
- 反射

## 安装环境变量

使用`go env`可以查看所有的环境变量。

### `GOROOT`
`GOROOT` 表示 Go 语言的安装目录。

`GOROOT` 的作用是用来索引 `Go` 语言的安装目录下的相关资源，比如 bin 目录的工具（如 `go` 命令），`src` 目录下的源码等。

### `GOPATH`
`GOPATH` 用于指定我们的开发工作区，可以有多个。

按照 Go 开发规范，`GOPATH` 目录下一般分为三个子目录 `src`，`pkg`，`bin`。

`src` 目录存放我们需要开发的项目源码，`pkg` 存放依赖的包和编译后的静态库文件，`bin` 放源代码编译后台的可执行文件。

注意：
（1）如果 `GOPATH` 未显示设置，则默认为用户主目录中名为 go 的子目录。Unix 下为`$HOME/go，Windows` 下为`%USERPROFILE%\go`（一般为`C:\Users\YourName\go`），Plan 9 下为`$home/go`。

（2）使用模块时，`GOPATH` 不再用于解析导入，但是它仍然用于存储下载的源代码（在`GOPATH/pkg/mod`中）和编译的命令（在`GOPATH/bin`中）。

（3）使用命令`go help gopath`可查看 `GOPATH` 详细说明。

### `GO111MODULE`
在 `go1.11` 的时候推出了这个 `go modules` 来解决依赖管理的问题。

通过变量 `GO111MODULE` 来控制 `Go Module` 的开启和关闭，取值 off、on 或 auto。

从 `Go 1.13` 开始，`Go Module` 作为 `Golang` 中的标准包管理器。

### `GOPROXY`
`go get` 下载依赖时使用的代理地址列表，使用逗号 `,`或竖杠 |`)分隔。

当用 go 命令查找依赖模块时，它会按顺序访问 `GOPROXY` 列表中的每个代理，直到收到成功的响应或出现终端错误。

`GOPROXY` 中可能会存在两个关键字来代替代理URL：

```go
off:不允许从任何源下载依赖的模块
direct:直接从版本控制存储库下载，而不是使用模块代理
```

`GOPROXY` 缺省值为`https://proxy.golang.org,direct`

### `GOPRIVATE`
`go get` 通过代理服务拉取仓库时，因为代理服务不可能访问到私有仓库（一般为企业内部代码管理平台），会出现 404 错误。

`go1.13` 版本提供了一个方便的解决方案：`GOPRIVATE` 环境变量。

如：`set GOPRIVATE=gitlab.com,git.woa.com`

###  `GOBIN`
用于存储我们使用`go install`命令安装的程序。

如果没有设置 `GOBIN`，程序一般会安装到为`GOPATH/bin`目录。

### `GOOS`
当前的操作系统。例如 `linux`、`darwin`、`windows`、`netbsd`、`freebsd`、`openbsd`、`solaris`、`plan9` 等。 `mac os`对应的值是 `darwin`。

### `GOARCH`
表示 CPU 架构。如 `amd64`、`386`、`arm`、`ppc64` 等。







[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


