---
title: 搞定Go语言流程控制
tags:
  - go
comments: true
typora-root-url: 搞定Go语言流程控制
date: 2021-08-29 10:20:45
categories: Go
index_img: /2021/08/29/搞定Go语言流程控制/footer-gopher.jpg
banner_img:
---



# 流程控制

## 流程控制：if-else

### 条件语句模型

Go里的流程控制方法还是挺丰富，整理了下有如下这么多种：

- if - else 条件语句
- switch - case 选择语句
- for - range 循环语句
- goto 无条件跳转语句
- defer 延迟执行

先看if-else语句：

```go
if 条件 1 {
  分支 1
} else if 条件 2 {
  分支 2
} else if 条件 ... {
  分支 ...
} else {
  分支 else
}
```

Go编译器，对于 `{` 和 `}` 的位置有严格的要求，它要求 else if （或 else）和 两边的花括号，必须在同一行。

Go是 强类型，所以要求你条件表达式必须严格返回布尔型的数据（nil 和 0 和 1 都不行。

### 单分支判断

只有一个 if ，没有 else

```go
import "fmt"

func main(){
	age := 20
	if age > 18 {
		fmt.Println("你已经是个大孩子了")
	}
}
```

如果条件里需要满足多个条件，可以使用 `&&` 和 `||`

- `&&`：表示且，左右都需要为true，最终结果才能为 true，否则为 false
- `||`：表示或，左右只要有一个为true，最终结果即为true，否则 为 false

```go
import "fmt"

func main() {
	age := 20
	gender := "male"
	if age > 18 && gender == "male" {
		fmt.Println("你已经是个大男孩子了")
	}
}
```

### 多分支判断

if - else

```go
import "fmt"

func main() {
    age := 20
    if age > 18 {
        fmt.Println("已经成年了")
    } else {
        fmt.Println("还未成年")
    }
}
```

if - else if - else

```go
import "fmt"

func main() {
    age := 20
    if age > 18 {
        fmt.Println("已经成年了")
    } else if age >12 {
        fmt.Println("已经是青少年了")
    } else {
        fmt.Println("还不是青少年")
    }
}
```

### 高级写法

在 if 里可以允许先运行一个表达式，取得变量后，再对其进行判断，比如第一个例子里代码也可以写成这样

```
import "fmt"

func main() {
    if age := 20;age > 18 {
        fmt.Println("已经成年了")
    }
}
```

## 流程控制：switch-case

### 语句模型

Go 里的选择语句模型是这样的

```go
switch 表达式 {
    case 表达式1:
        代码块
    case 表达式2:
        代码块
    case 表达式3:
        代码块
    case 表达式4:
        代码块
    case 表达式5:
        代码块
    default:
        代码块
}
```

拿 switch 后的表达式分别和 case 后的表达式进行对比，只要有一个 case 满足条件，就会执行对应的代码块，然后直接退出 switch - case ，如果 一个都没有满足，才会执行 default 的代码块。

### 最简单的示例

switch 后接一个你要判断变量 `education` （学历），然后 case 会拿这个 变量去和它后面的表达式（可能是常量、变量、表达式等）进行判等。

如果相等，就执行相应的代码块。如果不相等，就接着下一个 case。

```go
mport "fmt"

func main() {
    education := "本科"

    switch education {
    case "博士":
        fmt.Println("我是博士")
    case "研究生":
        fmt.Println("我是研究生")
    case "本科":
        fmt.Println("我是本科生")
    case "大专":
        fmt.Println("我是大专生")
    case "高中":
        fmt.Println("我是高中生")
    default:
        fmt.Println("学历未达标..")
    }
}
```

输出如下

```go
我是本科生
```

### 一个 case 多个条件

case 后可以接多个多个条件，多个条件之间是 `或` 的关系，用逗号相隔。

```go
import "fmt"

func main() {
    month := 2

    switch month {
    case 3, 4, 5:
        fmt.Println("春天")
    case 6, 7, 8:
        fmt.Println("夏天")
    case 9, 10, 11:
        fmt.Println("秋天")
    case 12, 1, 2:
        fmt.Println("冬天")
    default:
        fmt.Println("输入有误...")
    }
}
```

输出如下

```go
冬天
```

### case 条件常量不能重复

当 case 后接的是常量时，该常量只能出现一次。

以下两种情况，在编译时，都会报错： duplicate case “male” in switch

**错误案例一**

```go
gender := "male"

switch gender {
    case "male":
        fmt.Println("男性")
    // 与上面重复
    case "male":
        fmt.Println("男性")
    case "female":
        fmt.Println("女性")
}
```

**错误案例二**

```go
gender := "male"

switch gender {
    case "male", "male":
        fmt.Println("男性")
    case "female":
        fmt.Println("女性")
}
```

### switch 后可接函数

switch 后面可以接一个函数，只要保证 case 后的值类型与函数的返回值 一致即可。

```go
import "fmt"

// 判断一个同学是否有挂科记录的函数
// 返回值是布尔类型
func getResult(args ...int) bool {
    for _, i := range args {
        if i < 60 {
            return false
        }
    }
    return true
}

func main() {
    chinese := 80
    english := 50
    math := 100

    switch getResult(chinese, english, math) {
    // case 后也必须 是布尔类型
    case true:
        fmt.Println("该同学所有成绩都合格")
    case false:
        fmt.Println("该同学有挂科记录")
    }
}
```

### switch 可不接表达式

switch 后可以不接任何变量、表达式、函数。

当不接任何东西时，switch - case 就相当于 if - elseif - else

```go
score := 30

switch {
    case score >= 95 && score <= 100:
        fmt.Println("优秀")
    case score >= 80:
        fmt.Println("良好")
    case score >= 60:
        fmt.Println("合格")
    case score >= 0:
        fmt.Println("不合格")
    default:
        fmt.Println("输入有误...")
```

### switch 的穿透能力

正常情况下 switch - case 的执行顺序是：只要有一个 case 满足条件，就会直接退出 switch - case ，如果 一个都没有满足，才会执行 default 的代码块。

但是有一种情况是例外。

那就是当 case 使用关键字 `fallthrough` 开启穿透能力的时候。

```go
s := "hello"
switch {
case s == "hello":
    fmt.Println("hello")
    fallthrough
case s != "world":
    fmt.Println("world")
}
```

代码输出如下：

```go
hello
world
```

需要注意的是，`fallthrough` 只能穿透一层，意思是它让你直接执行下一个case的语句，而且不需要判断条件。

```go
s := "hello"
switch {
case s == "hello":
    fmt.Println("hello")
    fallthrough
case s == "xxxx":
    fmt.Println("xxxx")
case s != "world":
    fmt.Println("world")
}
```

输出如下，并不会输出 `world`（即使它符合条件）

```go
hello
xxxx
```

## 流程控制：for 循环

### 语句模型

这是 for 循环的基本模型。

```
for [condition |  ( init; condition; increment ) | Range]
{
   statement(s);
}
```

可以看到 for 后面，可以接三种类型的表达式。

1. 接一个条件表达式
2. 接三个表达式
3. 接一个 range 表达式

但其实还有第四种

1. 不接表达式

### 接一个条件表达式

打印 1 到 5 的数值。

```go
a := 1
for a <= 5 {
    fmt.Println(a)
    a ++
}
```

输出如下:

```go
1
2
3
4
5
```

### 接三个表达式

for 后面，紧接着三个表达式，使用 `;` 分隔。

这三个表达式，各有各的用途

- 第一个表达式：初始化控制变量，在整个循环生命周期内，只运行一次；
- 第二个表达式：设置循环控制条件，当返回true，继续循环，返回false，结束循环；
- 第三个表达式：每次循完开始（除第一次）时，给控制变量增量或减量。

这边的例子和上面的例子，是等价的。

```go
import "fmt"

func main() {
    for i := 1; i <= 5; i++ {
        fmt.Println(i)
    }
}
```

输出如下

```go
1
2
3
4
5
```

### 不接表达式：无限循环

在 Go 语言中，没有 while 循环，如果要实现无限循环，也完全可以 for 来实现。

当你不加任何的判断条件时， 就相当于你每次的判断都为 true，程序就会一直处于运行状态，但是一般我们并不会让程序处于死循环，在满足一定的条件下，可以使用关键字 `break` 退出循环体，也可以使用 `continue` 直接跳到下一循环。

下面两种写法都是无限循环的写法。

```go
for {
    代码块
}

// 等价于
for ;; {
    代码块
}
```

举个例子

```go
import "fmt"

func main() {
    var i int = 1
    for {
        if i > 5 {
            break
        }
        fmt.Printf("hello, %d\n", i)
        i++
    }
}
```

输出如下

```go
hello, 1
hello, 2
hello, 3
hello, 4
hello, 5
```

### 接 for-range 语句

遍历一个可迭代对象，是一个很常用的操作。在 Go 可以使用 for-range 的方式来实现。

range 后可接数组、切片，字符串等

由于 range 会返回两个值：索引和数据，若你后面的代码用不到索引，需要使用 `_` 表示 。

```go
import "fmt"

func main() {
    myarr := [...]string{"world", "python", "go"}
    for _, item := range myarr {
        fmt.Printf("hello, %s\n", item)
    }
}
```

输出如下

```
hello, world
hello, python
hello, go
```

如果你用一个变量来接收的话，接收到的是索引

```
import "fmt"

func main() {
    myarr := [...]string{"world", "python", "go"}
    for i := range myarr {
        fmt.Printf("hello, %v\n", i)
    }
}
```

输出如下

```
hello, 0
hello, 1
hello, 2
```

## 流程控制：goto 无条件跳转

### 基本模型

`goto` 顾言思义，是跳转的意思。

goto 后接一个标签，这个标签的意义是告诉 Go程序下一步要执行哪里的代码。

所以这个标签如何放置，放置在哪里，是 goto 里最需要注意的。

```go
goto 标签;
...
...
标签: 表达式;
```

### 最简单的示例

`goto` 可以打破原有代码执行顺序，直接跳转到某一行执行代码。

```go
import "fmt"

func main() {

    goto flag
    fmt.Println("B")
flag:
    fmt.Println("A")

}
```

执行结果，并不会输出 B ，而只会输出 A

```
A
```

### 如何使用？

`goto` 语句通常与条件语句配合使用。可用来实现条件转移， 构成循环，跳出循环体等功能。

这边举一个例子，用 `goto` 的方式来实现一个打印 1到5 的循环。

```
import "fmt"

func main() {
    i := 1
flag:
    if i <= 5 {
        fmt.Println(i)
        i++
        goto flag
    }
}
```

输出如下

```
1
2
3
4
5
```

再举个例子，使用 goto 实现 类型 break 的效果。

```
import "fmt"

func main() {
    i := 1
    for {
        if i > 5 {
            goto flag
        }
        fmt.Println(i)
        i++
    }
flag:
}
```

输出如下

```
1
2
3
4
5
```

最后再举个例子，使用 goto 实现 类型 continue的效果，打印 1到10 的所有偶数。

```
import "fmt"

func main() {
    i := 1
flag:
    for i <= 10 {
        if i%2 == 1 {
            i++
            goto flag
        }
        fmt.Println(i)
        i++
    }
}
```

输出如下

```
2
4
6
8
10
```

### 注意事项

goto语句与标签之间不能有变量声明，否则编译错误。

```
import "fmt"

func main() {
    fmt.Println("start")
    goto flag
    var say = "hello oldboy"
    fmt.Println(say)
flag:
    fmt.Println("end")
}
```

编译错误

```
.\main.go:7:7: goto flag jumps over declaration of say at .\main.go:8:6
```

## 流程控制：defer 延迟语句

### 延迟调用

defer 的用法很简单，只要在后面跟一个函数的调用，就能实现将这个 `xxx` 函数的调用延迟到当前函数执行完后再执行。

```go
defer xxx()
```

这是一个很简单的例子，可以很快帮助你理解 defer 的使用效果。

```go
import "fmt"

func myfunc() {
    fmt.Println("B")
}

func main() {
    defer myfunc()
    fmt.Println("A")
}
```

输出如下

```go
A
B
```

当然了，对于上面这个例子可以简写为成如下，输出结果是一致的

```go
import "fmt"

func main() {
    defer fmt.Println("B")
    fmt.Println("A")
}
```

### 即时求值的变量快照

使用 defer 只是延时调用函数，此时传递给函数里的变量，不应该受到后续程序的影响。

比如这边的例子

```go
import "fmt"

func main() {
    name := "go"
    defer fmt.Println(name) // 输出: go

    name = "python"
    fmt.Println(name)      // 输出: python
}
```

输出如下，可见给 name 重新赋值为 `python`，后续调用 defer 的时候，仍然使用未重新赋值的变量值，就好在 defer 这里，给所有的这是做了一个快照一样。

```go
python
go
```

如果 defer 后面跟的是匿名函数，情况会有所不同， defer 会取到最后的变量值

```go
package main

import "fmt"


func main() {
    name := "go"
    defer func(){
    fmt.Println(name) // 输出: python
}()
    name = "python"
    fmt.Println(name)      // 输出: python
}
```

### 多个defer 反序调用

当我们在一个函数里使用了 多个defer，那么这些defer 的执行函数是如何的呢？

```go
import "fmt"

func main(){

   defer b()
   fmt.Println("function main")
   defer a()
   defer c()
}

func a(){
   fmt.Println("function a")
}

func b(){
   fmt.Println("function b")
}

func c(){
   fmt.Println("function c")
}
```

输出如下，可见 多个defer 是反序调用的，有点类似栈一样，后进先出。

```go
function main
function c
function a
function b
```

### defer 与 return 孰先孰后

defer 和 return 到底是哪个先调用？

使用下面这段代码，可以很容易的观察出来

```go
import "fmt"

var name string = "go"

func myfunc() string {
    defer func() {
        name = "python"
    }()

    fmt.Printf("myfunc 函数里的name：%s\n", name)
    return name
}

func main() {
    myname := myfunc()
    fmt.Printf("main 函数里的name: %s\n", name)
    fmt.Println("main 函数里的myname: ", myname)
}
```

输出如下

```go
myfunc 函数里的name：go
main 函数里的name: python
main 函数里的myname:  go
```

来一起理解一下这段代码，第一行很直观，name 此时还是全局变量，值还是go

第二行也不难理解，在 defer 里改变了这个全局变量，此时name的值已经变成了 python

重点在第三行，为什么输出的是 go ？

解释只有一个，那就是 defer 是return 后才调用的。所以在执行 defer 前，`myname` 已经被赋值成 go 了。















[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


