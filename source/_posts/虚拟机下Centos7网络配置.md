---
title: 虚拟机下Centos7网络配置
tags:
  - centos
  - vmware
  - linux
comments: true
typora-root-url: 虚拟机下Centos7网络配置
date: 2021-07-02 15:05:09
categories: 运维
index_img: /2021/07/02/虚拟机下Centos7网络配置/1.webp
banner_img:
---

本次实验环境：`VWware`版本：16 pro，`Centos`版本：`Centos 7`。

## 虚拟机下`Centos`配置联网

首先，确保主机可以联网。查看主机的`DNS`地址。

`cmd`下输入`ipconfig`/all：

![](image-20210702151438174.png)

记住此地址。

然后，修改`Centos`网络配置文件。

在`Centos`终端中，输入`vi /etc/sysconfig/network-scripts/ifcfg-ens33`

配置文件内容如下：

```shell
TYPE=Ethernet                 # 网卡类型：为以太网
PROXY_METHOD=none               # 代理方式：关闭状态
BROWSER_ONLY=no                # 只是浏览器：否
BOOTPROTO=dhcp                # 网卡的引导协议：DHCP
DEFROUTE=yes                 # 默认路由：是 
IPV4_FAILURE_FATAL=no             # 是不开启IPV4致命错误检测：否
IPV6INIT=yes                 # IPV6是否自动初始化: 是
IPV6_AUTOCONF=yes               # IPV6是否自动配置：是
IPV6_DEFROUTE=yes               # IPV6是否可以为默认路由：是
IPV6_FAILURE_FATAL=no             # 是不开启IPV6致命错误检测：否
IPV6_ADDR_GEN_MODE=stable-privacy       # IPV6地址生成模型：stable-privacy 
NAME=ens33                  # 网卡物理设备名称
UUID=42773503-99ed-443f-a957-66dbc1258347   # 通用唯一识别码
DEVICE=ens33                 # 网卡设备名称
ONBOOT=no                   # 是否开机启动， 可用systemctl restart network重启网络
```

修改`ONBOOT=yes`并添加`DNS1=127.0.0.1`，此`DNS`地址设置为本机的`DNS`地址。 

![](image-20210702152401623.png)

输入`ESC`，`:wq!`退出。

最后，输入 `systemctl restart network` 重启网络，没有提示任何信息，则表示网络重启成功。

![](image-20210702152300781.png)

测试`ping`百度[`www.baidu.com`](https://www.baidu.com)可以访问。

![](image-20210702152616087.png)

## 主机名配置

为了更改或设置`CentOS 7`计算机主机名称，请使用`hostnamectl`命令，如下面的命令摘录所示。

```shell
# hostnamectl set-hostname your-new-hostname
```



除`hostname`命令外，还可以使用`hostnamectl`命令显示`Linux`机器主机名。

```shell
# hostnamectl
```

为了应用新的主机名，需要重启系统，请执行以下命令之一来重新启动`CentOS 7`机器。

```shell
# init 6
# systemctl reboot
# shutdown -r
```

第二种设置`CentOS 7`机器主机名的方法是手动编辑`/ etc/hostname`文件并输入新的主机名。另外，为了应用新的机器名称，系统重新启动是必要的。

```shell
# vi /etc/hostname
```

## 修改`IP`地址

在配置文件里修改为如下：

![](image-20210702155511437.png)

`IP`地址设置为`192.168.1.3`。





[//]:#(设置表格整体居中显示)

<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


