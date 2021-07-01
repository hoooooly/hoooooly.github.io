---
title: VMwareTools灰显无法安装的问题
tags:
  - Hexo
  - Fluid
comments: true
typora-root-url: VMwareTools灰显无法安装的问题
date: 2021-07-01 16:11:33
categories:
index_img:
banner_img:
---

在`windows`平台下安装虚拟机后，`VMware Tools`灰显无法安装的问题。

![image-20210701161329116](/image-20210701161329116.png)

## 解决办法

首先，关闭虚拟机。

选中当前虚拟机，右击设置。

![image-20210701161703905](/image-20210701161703905.png)

在`CD/DVD2(SATA)`光盘映像文件中选择安安装目录下的`windows.iso`文件。默认的安装路径一般为`C:\Program Files (x86)\VMware\VMware Workstation\windows.iso`。

![image-20210701161828628](/image-20210701161828628.png)

然后，开启虚拟机。打开计算机可以看到`DVD`驱动器加载就是`VMware Tools`安装镜像。

![image-20210701162319168](/image-20210701162319168.png)

点开后双击运行`setip64.exe`安装。

![image-20210701162500194](/image-20210701162500194.png)



[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


