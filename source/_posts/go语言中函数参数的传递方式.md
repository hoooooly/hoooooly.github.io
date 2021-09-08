---
title: go语言中函数参数的传递方式
tags:
  - go
comments: true
typora-root-url: go语言中函数参数的传递方式
date: 2021-08-30 21:21:12
categories: Go
index_img: 2021/08/30/go语言中函数参数的传递方式/beego.jpeg
banner_img:
---

# 函数参数的传递方式

## 两种传递方式

1）值传递

2）引用传递

不论是值传递还是引用传递，传给函数的都是变量的副本，不同的是值传递的是值的拷贝，引用传递的是地址的拷贝，一般来说地址拷贝效率高。

## 值类型和引用类型

1）值类型：基本数据类型 int 系列，float系列，bool，string、数组和结构体struct

2）引用类型：指针、slice切片、map、管道chan、interface等都是引用类型

## 值传递和引用传递使用特点

1）值类型默认是值传递：变量直接存储存值，内存通常在栈中分配

2）引用类型默认是引用传递：变量存储的是一个地址，这个地址对应的空间才是真正存储数值，内存通常在堆上分配，当没有任何变量引用这个地址时，改地址对应的数据空间就成为一个垃圾，有`GC`来回收。

3）如果希望函数内的变量能够修改函数外的变量，可以传入变量的地址&，函数内以指针的方式操作变量。从效果上类似引用。

```go
package main

import "fmt"

var a int = 100
var b int = 100

func getVar(x *int) int{
   *x = 1000
   return *x
}

func getVar2(x int) int{
   x = 200
   return x
}

func main()  {
   fmt.Println(a) //100
   fmt.Println(getVar(&a)) //1000
   fmt.Println(a) //1000
   fmt.Println(b) //100
   fmt.Println(getVar2(b)) //200
   fmt.Println(b) //100

}
```

## 变量作用域

1）函数内部声明/定义的变量叫局部变量，作用域仅限于函数内部。

```go
func area()(string, int){
   // name,age定义在函数内部
   name := "holy"
   age:=25
   return name, age
}
```

2）函数外部声明/定义的变量叫全局变量，作用域在整个包都有效，如果其首字母大写，则作用域在整个程序有效。

3）如果变量在一个代码块中，比如for/if中，那么这个变量的作用域就在该代码块





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


