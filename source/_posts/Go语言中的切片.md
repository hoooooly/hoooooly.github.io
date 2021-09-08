---
title: Go语言中的切片
tags:
  - go
  - 切片
comments: true
typora-root-url: Go语言中的切片
date: 2021-08-31 20:42:51
categories: Go
index_img: 2021/08/31/Go语言中的切片/beego.jpeg
banner_img:
---

# Go语言中的切片

## 切片的基本介绍

切片，slice

是数组的一个引用，因此切片是引用类型，在进行传值时，遵守引用传递的机制。

切片的使用和数组类似，遍历切片、访问切片的元素和求切片长度len(slice)都一样。

切片的长度是可以变化的，因此切片是一个可以动态变化的数组。

定义的基本语法：

```go
var 切片名 []类型
比如：var a []int
```

## 快速入门

演示一个切片的基本使用：

```go
package main

import "fmt"

func main(){
	var intArr [5]int = [...]int{1, 22, 33, 66, 99}
	// 声明/定义一个切片
	slice := intArr[1:3]
	fmt.Println("intArr", intArr) //intArr [1 22 33 66 99]
	fmt.Println("slice 的元素是 =", slice) //slice 的元素是 = [22 33]
	fmt.Println("slice 的元素个数 =", len(slice)) //slice 的元素个数 = 2
	fmt.Println("slice 的容量 =", cap(slice)) //slice 的容量 = 4
}
```

## 切片在内存中形式（重要）

### 基本的介绍：

![](image-20210831205851193.png)

slice的确是一个引用类型

slice从底层来说，其实是一个数据结构（结构体）

```go
type slice struct{
    ptr *[2]int
    len int
    cap int
}
```

## 切片的使用

- 方式一：

第一种方式：定义一个切片，然后让切片去引用一个已经创建好的数组。

- 方式二：

第二种方式：通过make来创建切片

基本语法：`var 切片名 []type = make([]type, len, [cap])`

参数说明：`type`数据类型，`len`大小，`cap`指定切片容量，可选，如果你分配了`cap`，则要求`cap>=len`

```go
var slice1 []float64 = make([]float64, 5, 10)
slice1[1] = 10
slice1[3] = 10
fmt.Println(slice1) //[0 10 0 10 0]
fmt.Println("slice1的size=", len(slice1)) //slice1的size= 5
fmt.Println("slice的cap=", cap(slice1)) //slice的cap= 10
```

![](image-20210831211122280.png)

1）通过make方式创建切片可以指定切片的大小和容量

2）如果没有给切片的各个元素赋值，那么就会使用默认值

3）通过make方式创建的切片对应的数组是由make底层维护，对外不可见，即只能通过slice访问各个元素

- 方式三

第3种方式：定义一个切片，直接就指定具体数组，使用原理类似make的方式。

```go
var strSlice []string = []string{"tom", "jack", "mary"}
fmt.Println("strSlice=", strSlice) //strSlice= [tom jack mary]
fmt.Println("strSlice size=", len(strSlice)) //strSlice size= 3
fmt.Println("strSlice cap=", cap(strSlice)) //strSlice cap= 3
```

方式1是直接引用数组，这个数组是事先存在的，程序员是可见的。

方式2是通过make来创建切片，make也会创建一个数组，是由切片在底层进行维护，程序员是看不见的。make创建切片的示意图∶

![](image-20210831212029532.png)

## 切片的遍历

切片的偏历和数组一样，也有两种方式

- for循环常规方式遍历
- for-range结构遍历切片

```go
var arr [5]int = [...]int{10, 20, 30, 40, 50}
	slice := arr[1:4]
	// 传统方式
	for i := 0; i < len(slice); i++ {
		fmt.Printf("slice[%v]=%v", i, slice[i])
	}
	fmt.Println()
	// 使用for-range方式遍历切片
	for i, v := range slice {
		fmt.Printf("i=%v v=%v\n", i, v)
	}
```

## 切片的使用的注意事项

1）切片初始化时`var slice = arr[startIndex:endIndex]`

说明：从`arr`数组下标为`startIndex`，取到下标为`endIndex`的元素

2）切片初始化时，仍然不能越界。范围在`[0-len(arr)]`之间，但是可以动态增长。

`var slice = arr[0:end]` 可以简写 `var slice = arr[:end]`

3）cap是一个内置函数，用于统计切片的容量，即最大可以存放多少元素。

4）切片定义完后，还不能使用，因为本身是一个空的，需要让其引用到一个数组，或者make一个空间供切片来使用。

5）切片可以继续切片。

6）用append内置函数，可以对切片进行动态追加。

```go
slice = append(slice, 400, 500, 600)
fmt.Println("slice", slice) //slice [20 30 40 400 500 600]
slice = append(slice, slice...)
fmt.Println(slice) // [20 30 40 400 500 600 20 30 40 400 500 600]
```

7）切片的拷贝操作

切片使用copy内置函数完成拷贝，举例说明

```go
var newslice = make([]int, 10)
	copy(newslice, slice)
	fmt.Println("newslice=", newslice) //newslice= [20 30 40 400 500 600 20 30 40 400]
```

## string和slice

1）string底层是一个byte数组，因此string也可以进行切片处理

```go
str := "hello@outlook.com"
slice := str[6:]
fmt.Println("slice=", slice) //slice= outlook.com
```

2）string是不可以改变的，不能通过str[0]='z'方式来修改字符串

3）如果要修改字符串，可以先将string->[]byte/或者[]rune->修改->重写转成string

```go
arr1 := []byte(str)
arr1[0] = 'H'
str = string(arr1)
fmt.Println(str) //Hello@outlook.com

// 如果要处理中文，就必须转成[]rune
arr2 := []rune(str)
arr2[0] = '陈'
str = string(arr2)
fmt.Println(str) //陈ello@outlook.com
```





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


