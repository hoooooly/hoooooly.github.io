---
title: Hexo搭建个人博客
date: 2021-05-08 15:39:22
categories: 博客教程
tags:
- Hexo
# sticky: 100
pic:
comments: true
toc: true
only:
- home
- category
- tag
---


# Hexo

[Hexo](https://hexo.io/zh-cn/)是一个快速、简洁且高效的博客框架。

## 安装

### git安装

```shell
    Windows：下载并安装 git.
    Mac：使用 Homebrew, MacPorts 或者下载 安装程序。
    Linux (Ubuntu, Debian)：sudo apt-get install git-core
    Linux (Fedora, Red Hat, CentOS)：sudo yum install git-core
```

### 安装 Node.js

Node.js 为大多数平台提供了官方的 安装程序。对于中国大陆地区用户，可以前往 淘宝 Node.js 镜像 下载。

其它的安装方法：

```shell
    Windows：通过 nvs（推荐）或者nvm 安装。
    Mac：使用 Homebrew 或 MacPorts 安装。
    Linux（DEB/RPM-based）：从 NodeSource 安装。
    其它：使用相应的软件包管理器进行安装，可以参考由 Node.js 提供的 指导
```

### 安装 Hexo

所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。

```shell
    npm install -g hexo-cli
```

安装完成后，`win`+`R`输入`cmd`调出命令行，输入`hexo`提示如下，说明安装正确。

```shell
    C:\Users\espho>hexo
    Usage: hexo <command>

    Commands:
    help     Get help on a command.
    init     Create a new Hexo folder.
    version  Display version information.

    Global Options:
    --config  Specify config file instead of using _config.yml
    --cwd     Specify the CWD
    --debug   Display all verbose messages in the terminal
    --draft   Display draft posts
    --safe    Disable all plugins and scripts
    --silent  Hide output on console

    For more help, you can use 'hexo help [command]' for the detailed information
    or you can check the docs: http://hexo.io/docs/
```

## 建站

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```shell
    hexo init <folder>
    cd <folder>
    npm install
```

新建完成后，指定文件夹的目录如下：

```shell
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

### _config.yml

网站的 配置 信息，您可以在此配置大部分的参数。

### package.json

应用程序的信息。EJS, Stylus 和 Markdown renderer 已默认安装，您可以自由移除。

```shell
package.json
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": ""
  },
  "dependencies": {
    "hexo": "^3.8.0",
    "hexo-generator-archive": "^0.1.5",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-index": "^0.2.1",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.3.1",
    "hexo-renderer-stylus": "^0.3.3",
    "hexo-renderer-marked": "^0.3.2",
    "hexo-server": "^0.3.3"
  }
}
```

### scaffolds

`模版`文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。

Hexo的模板是指在新建的文章文件中默认填充的内容。例如，如果您修改scaffold/post.md中的Front-matter内容，那么每次新建一篇文章时都会包含这个修改。

### source

资源文件夹是存放用户资源的地方。除 `_posts`文件夹之外，开头命名为 `_ (下划线)`的文件`/文件夹`和`隐藏的文件`将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public`文件夹，而其他文件会被拷贝过去。

### themes

`主题`文件夹。Hexo 会根据主题来生成静态页面。

## 配置

相关配置可直接访问[官方文档](https://hexo.io/zh-cn/docs/configuration)查看，我们先从使用别人的主题开始，[官方](https://hexo.io/themes/)提供了335个主题下载使用，你也可以根据规范制定自己的主题。

## 主题

创建`Hexo`主题非常容易，您只要在`themes`文件夹内，新增一个任意名称的文件夹，并修改`_config.yml`内的`theme`设定，即可切换主题。一个主题可能会有以下的结构：

```shell
.
├── _config.yml
├── languages
├── layout
├── scripts
└── source
```

### _config.yml

主题的配置文件。和 Hexo 配置文件不同，主题配置文件修改时会自动更新，无需重启`Hexo Server`。

## 获取主题

选择相应的主题，从`github`上获取到`themes`目录下。

修改主目录下`_config.yml`中的配置文件，将`theme`修改为获取主题的`文件夹名`。

```shell
theme: Kratos-Rebirth
```

## 运行

在主目录下调用`cmd`命令`hexo server`运行服务，访问`http://localhost:4000`进入博客。

```shell
espho@Holy-Surface MINGW64 /e/blog
$ hexo server
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
INFO  感谢使用 Kratos-Rebirth 主题，您的版本是 v1.6.2
INFO  如有任何疑问，您可以查阅文档，或是去 https://github.com/Candinya/Kratos-Rebirth/issues 提出对应的 issue 。
INFO  预祝您使用愉快。
```

## 添加文章

