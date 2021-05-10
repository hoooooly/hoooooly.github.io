---
title: Hexo在腾讯云的部署
date: 2021-05-10 12:51:56
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

## 简介

`Hexo`在`GitHub pages`上的访问太慢了，迁移到腾讯云服务器上。

## 部署环境

腾讯云服务器（Centos 64位）。

## 服务器配置

### 安装git

```shell
yum install git
```

创建git用户并修改权限

```shell
adduser git
passwd git
chmod 740 /etc/sudoers
vim /etc/sudoers
```

找到一下内容

```shell
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
```

在该语句下添加

```shell
git ALL=(ALL) ALL
```

退出（esc + :wq）并修改权限

```shell
chmod 400 /etc/sudoers
```

本地使用gitbash创建密钥

```shell
ssh-keygen -t rsa //因为我在GitHub上部署博客时已经创建过密钥，这里可以直接跳过生成，用以前的密钥
```

在腾讯云中创建`ssh`，并将本地的`id_rsa.pub`中的文件内容全部复制到`authorized_keys`中。

```shell
su git
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
```

修改权限

```shell
cd ~
chmod 600 .ssh/authorized_keys
chmod 700 .ssh
```

本地测试

```shell
ssh -v git@SERVER //@后是你自己的服务器公网IP，如果不出现failed字样，说明成功
```

云服务器中创建网站目录并设置权限

```shell
su root
mkdir /home/hexo
chown git:git -R /home/hexo
```

### 安装nginx

```shell
yum install -y nginx    // 安装
systemctl start nginx.service     // 启动服务
```

以上执行完之后，在浏览器中输入你的公网IP如果可以进入CentOs界面，说明Nginx安装成功。

### 配置nginx

```shell
nginx -t  // 命令查看位置，一般为 /etc/nginx/nginx.conf。
vim /etc/nginx/nginx.conf //修改配置文件，在server_name后添加自己的域名（要备案），root后添加/home/hexo
```

重启服务

```shell
systemctl restart nginx.service
```

## 建立git仓库并修改权限

```shell
su root
cd /home/git
git init --bare blog.git
chown git:git -R blog.git
```

同步网站根目录

```shell
vim blog.git/hooks/post-receive
```

填入如下内容

```shell
#!/bin/sh
git --work-tree=/home/hexo --git-dir=/home/git/blog.git checkout -f
```

修改权限

```shell
chmod +x /home/git/blog.git/hooks/post-receive
```

在本地Hexo目录下修改_config.yml文件中的deploy后的repo改为：

```shell
git@SERVER:/home/git/blog.git   //@后为你的服务器公网IP
```

以上全部完成后，执行hexo的部署命令即可完成在腾讯云服务器上的博客部署。