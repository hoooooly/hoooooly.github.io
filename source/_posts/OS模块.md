---
title: OS模块
tags:
  - os
  - python
comments: true
date: 2021-06-28 15:31:47
categories: Python模块
index_img:
banner_img:
---

`os` 模块提供了很多允许你的程序与操作系统直接交互的功能。

```python
得到当前工作目录，即当前Python脚本工作的目录路径: os.getcwd()
>>> os.getcwd()
'D:\\PythonProjs\\python模块\\os模块'

返回指定目录下的所有文件和目录名:os.listdir()，返回列表
>>> os.listdir() 
['os中常用的值和路径介绍.py']

函数用来删除一个文件:os.remove()
>>> os.remove("a.txt") # 没有提示

删除多个目录：os.removedirs（r“c：\python”）

检验给出的路径是否是一个文件：os.path.isfile()

检验给出的路径是否是一个目录：os.path.isdir()

判断是否是绝对路径：os.path.isabs()
>>> os.path.isabs("/") 
True

检验给出的路径是否真地存:os.path.exists()
>>> os.path.exists("/") 
True

返回一个路径的目录名和文件名:os.path.split()    
>>> os.path.split("D:/tfsServer.rar") 
('D:/', 'tfsServer.rar')

分离扩展名：os.path.splitext()
>>> os.path.splitext('D:/tfsServer.rar') 
('D:/tfsServer', '.rar')

获取路径名：os.path.dirname()
>>> os.path.dirname('D:/tfsServer.rar')  
'D:/'

获得绝对路径: os.path.abspath()
>>> os.path.abspath('D:/tfsServer.rar') 
'D:\\tfsServer.rar'

获取文件名：os.path.basename()
>>> os.path.basename('D:/tfsServer.rar') 
'tfsServer.rar'

运行shell命令: os.system()
>>> os.system("dir") 

读取操作系统环境变量HOME的值:os.getenv("HOME") 
>>> os.getenv("PATH") 

返回操作系统所有的环境变量： os.environ 
>>> os.environ

设置系统环境变量，仅程序运行时有效：os.environ.setdefault('HOME','/home/alex')
给出当前平台使用的行终止符:os.linesep    Windows使用'\r\n'，Linux and MAC使用'\n'
指示你正在使用的平台：os.name       对于Windows，它是'nt'，而对于Linux/Unix用户，它是'posix'
重命名：os.rename（old， new）
创建多级目录：os.makedirs（r“c：\python\test”）
创建单个目录：os.mkdir（“test”）
获取文件属性：os.stat（file）
修改文件权限与时间戳：os.chmod（file）
获取文件大小：os.path.getsize（filename）
结合目录名与文件名：os.path.join(dir,filename)
改变工作目录到dirname: os.chdir(dirname)
获取当前终端的大小: os.get_terminal_size()
杀死进程: os.kill(10884,signal.SIGKILL)
```











[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


