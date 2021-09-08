---
title: 搞定go语言数据类型
tags:
  - go
comments: true
typora-root-url: go语言数据类型
date: 2021-08-28 18:44:20
categories: Go
index_img: /2021/08/28/go语言数据类型/footer-gopher.jpg
banner_img:
---



# 数据类型

先上图：

![](image-20210828185246317.png)

# 变量

## 格式化输出

通用：

```go
%v	值的默认格式表示
%+v	类似%v，但输出结构体是会增加字段名
%#v	值的Go语法表示
%T	值的类型的Go语法表示
%%	百分号
```

布尔值：

```go
%t	单词true或false
```

进制数：

```go
%b	表示为二进制	
%c	该值对应的unicode码值
%d	表示为十进制
%o	表示为八进制
%q	该值对应单引号括起来的go语法字符字面值，必要时会采用安全的转义表示
%x	表示为十六进制，使用a-f
%X 	表示为十六进制，使用A-F
%U	表示为Unicode格式：U+1234，等价于"U+%04X"
```

浮点数与复数的两个组分：

```go
%b	无小数部分、二进制指数的科学计数法
%e	科学计数法，如-1234.456e+78
%E	科学计数法，如-1234.456E+78
%f	有小数部分但无指数部分，如123.456
%F	等价于%f
```

## 变量定义和使用

**第一种** ：一行声明一个变量

```go
var <name> <type>
```

其中 var 是关键字（固定不变），name 是变量名，type 是类型。

使用 var ，虽然只指定了类型，但是 Go 会对其进行隐式初始化，比如 string 类型就初始化为空字符串，int 类型就初始化为0，float 就初始化为 0.0，bool类型就初始化为false，指针类型就初始化为 nil。

也可以在声明时顺便赋值。

```go
var name string = "I lovw the world"
```

**第二种**：多个变量一起声明

声明多个变量，除了可以按照上面写成多行之外，还可以写成下面这样

```go
var (
    name string
    age int
    gender string
)
```

**第三种**：声明和初始化一个变量

使用 `:=` （推导声明写法或者短类型声明法：编译器会自动根据右值类型推断出左值的对应类型。），可以声明一个变量，并对其进行（显式）初始化。

```
name := "Go编程时光"

// 等价于

var name string = "hello"

// 等价于

var name = "hello"
```

但这种方法有个限制就是，只能用于**函数**内部

**第四种**：声明和初始化多个变量

```go
name, age := "wangbm", 28
```

这种方法，也经常用于变量的交换

```go
var a int = 100
var b int = 200
b, a = a, b
```

**第五种**：new 函数声明一个指针变量

在这里要先讲一下，指针的相关内容。

变量分为两种 `普通变量` 和 `指针变量`

普通变量，存放的是数据本身，而指针变量存放的是数据的地址。

如下代码，age 是一个普通变量，存放的内容是 28，而 ptr 是 存放变量age值的内存地址：`0xc000010098`

```
package main

import "fmt"

func main()  {
    var age int = 28
    var ptr = &age  // &后面接变量名，表示取出该变量的内存地址
    fmt.Println("age: ", age)
    fmt.Println("ptr: ", ptr)
}
```

输出

```
age:  28
ptr:  0xc000010098
```

而这里要说的 new 函数，是 Go 里的一个内建函数。

使用表达式 new(Type) 将创建一个Type类型的匿名变量，初始化为Type类型的零值，然后返回变量地址，返回的指针类型为`*Type`。

```
package main

import "fmt"

func main()  {
    ptr := new(int)
    fmt.Println("ptr address: ", ptr)
    fmt.Println("ptr value: ", *ptr)  // * 后面接指针变量，表示从内存地址中取出值
}
```

输出

```
ptr address:  0xc000010098
ptr value:  0
```

用new创建变量和普通变量声明语句方式创建变量没有什么区别，除了不需要声明一个临时变量的名字外，我们还可以在表达式中使用new(Type)。换言之，new函数类似是一种语法糖，而不是一个新的基础概念。

如下两种写法，可以说是等价的

```go
// 使用 new
func newInt() *int {
    return new(int)
}

// 使用传统的方式
func newInt() *int {
    var dummy int
    return &dummy
}
```

以上不管哪种方法，变量/常量都只能声明一次，声明多次，编译就会报错。

但也有例外，这就要说到一个特殊变量：**匿名变量**，也称作占位符，或者空白标识符，用下划线表示。

匿名变量，优点有三：

- 不分配内存，不占用内存空间
- 不需要你为命名无用的变量名而纠结
- 多次声明不会有任何问题

通常我们用匿名接收必须接收，但是又不会用到的值。

```go
func GetData() (int, int) {
    return 100, 200
}
func main(){
    a, _ := GetData()
    _, b := GetData()
    fmt.Println(a, b)
}
```

原文连接：[1.2 五种变量创建的方法 — Go编程时光 1.0.0 documentation (iswbm.com)](https://golang.iswbm.com/c01/c01_02.html)]()

# 常量

定义常量：

```go
const constantName [type] = value
```

`const`：定义常量的关键字

`constantName`：常量名称

`type`：常量类型

`value`：常量的值

# 整数类型

Go语言中，整数类型可以分成10个类型。

![](20200120204329.png)

`int` 和 `uint` 的区别就在于一个 `u`，有 `u` 说明是无符号，没有 `u` 代表有符号。

**解释这个符号的区别**

以 `int8` 和 `uint8` 举例，8 代表 8个bit，能表示的数值个数有 2^8 = 256。

`uint8` 是无符号，能表示的都是正数，0-255，刚好256个数。

`int8` 是有符号，既可以正数，也可以负数，那怎么办？对半分呗，-128-127，也刚好 256个数。

`int8` `int16` `int32` `int64` 这几个类型的最后都有一个数值，这表明了它们能表示的数值个数是固定的。

而 int 并没有指定它的位数，说明它的大小，是可以变化的，那根据什么变化呢？

- 当你在32位的系统下，int 和 `uint` 都占用 4个字节，也就是32位。
- 若你在64位的系统下，int 和 `uint` 都占用 8个字节，也就是64位。

出于这个原因，在某些场景下，你应当避免使用 int 和 `uint` ，而使用更加精确的 `int32` 和 `int64`，比如在二进制传输、读写文件的结构描述（为了保持文件的结构不会受到不同编译目标平台字节长度的影响）

**不同进制的表示方法**

出于习惯，在初始化数据类型为整型的变量时，我们会使用10进制的表示法，因为它最直观，比如这样，表示整数10.

```
var num int = 10
```

不过，你要清楚，你一样可以使用其他进制来表示一个整数，这里以比较常用的2进制、8进制和16进制举例。

2进制：以`0b`或`0B`为前缀

```
var num01 int = 0b1100
```

8进制：以`0o`或者 `0O`为前缀

```
var num02 int = 0o14
```

16进制：以`0x` 为前缀

```
var num03 int = 0xC
```

下面用一段代码分别使用二进制、8进制、16进制来表示 10 进制的数值：12

```go
package main

import (
    "fmt"
)

func main() {
    var num01 int = 0b1100
    var num02 int = 0o14
    var num03 int = 0xC

    fmt.Printf("2进制数 %b 表示的是: %d \n", num01, num01)
    fmt.Printf("8进制数 %o 表示的是: %d \n", num02, num02)
    fmt.Printf("16进制数 %X 表示的是: %d \n", num03, num03)
}
```

输出如下

```go
2进制数 1100 表示的是: 12
8进制数 14 表示的是: 12
16进制数 C 表示的是: 12
```

以上代码用过了 `fmt` 包的格式化功能，你可以参考这里去看上面的代码

```go
%b    表示为二进制
%c    该值对应的unicode码值
%d    表示为十进制
%o    表示为八进制
%q    该值对应的单引号括起来的go语法字符字面值，必要时会采用安全的转义表示
%x    表示为十六进制，使用a-f
%X    表示为十六进制，使用A-F
%U    表示为Unicode格式：U+1234，等价于"U+%04X"
%E    用科学计数法表示
%f    用浮点数表示
```

# 浮点数

浮点数类型的值一般由整数部分、小数点“`.`”和小数部分组成。

其中，整数部分和小数部分均由10进制表示法表示。不过还有另一种表示方法。那就是在其中加入指数部分。指数部分由“E”或“e”以及一个带正负号的10进制数组成。比如，`3.7E-2`表示浮点数`0.037`。又比如，`3.7E+1`表示浮点数`37`。

有时候，浮点数类型值的表示也可以被简化。比如，`37.0`可以被简化为`37`。又比如，`0.037`可以被简化为`.037`。

有一点需要注意，在Go语言里，浮点数的相关部分只能由10进制表示法表示，而不能由8进制表示法或16进制表示法表示。比如，`03.7`表示的一定是浮点数`3.7`。

### `float32` 和 `float64`

Go语言中提供了两种精度的浮点数 `float32` 和 `float64`。

**`float32`**，也即我们常说的单精度，存储占用4个字节，也即4*8=32位，其中1位用来符号，8位用来指数，剩下的23位表示尾数

![](v2-749cc641eb4d5dafd085e8c23f8826aa_hd.jpg)

**`float64`**，也即我们熟悉的双精度，存储占用8个字节，也即8*8=64位，其中1位用来符号，11位用来指数，剩下的52位表示尾数

![](v2-48240f0e1e0dd33ec89100cbe2d30707_hd.jpg)

**那么精度是什么意思？有效位有多少位？**

精度主要取决于尾数部分的位数。

对于 `float32`（单精度）来说，表示尾数的为23位，除去全部为0的情况以外，最小为2^-23，约等于1.19*10^-7，所以float小数部分只能精确到后面6位，加上小数点前的一位，即有效数字为7位。

同理 `float64`（单精度）的尾数部分为 52位，最小为2^-52，约为2.22*10^-16，所以精确到小数点后15位，加上小数点前的一位，有效位数为16位。

通过以上，可以总结出以下几点：

**一、float32 和 float64 可以表示的数值很多**

浮点数类型的取值范围可以从很微小到很巨大。浮点数取值范围的极限值可以在 math 包中找到：

- 常量 `math.MaxFloat32` 表示 `float32` 能取到的最大数值，大约是 `3.4e38`；
- 常量 `math.MaxFloat64` 表示 `float64` 能取到的最大数值，大约是 `1.8e308`；
- `float32` 和 `float64` 能表示的最小值分别为 `1.4e-45` 和 `4.9e-324`。

**二、数值很大但精度有限**

人家虽然能表示的数值很大，但精度位却没有那么大。

- `float32`的精度只能提供大约6个十进制数（表示后科学计数法后，小数点后6位）的精度
- `float64`的精度能提供大约15个十进制数（表示后科学计数法后，小数点后15位）的精度

这里的精度是什么意思呢？

比如 10000018这个数，用 `float32` 的类型来表示的话，由于其有效位是7位，将10000018 表示成科学计数法，就是 1.0000018 * 10^7，能精确到小数点后面6位。

此时用科学计数法表示后，小数点后有7位，刚刚满足我们的精度要求，意思是什么呢？此时你对这个数进行+1或者-1等数学运算，都能保证计算结果是精确的

```go
import "fmt"
var myfloat float32 = 10000018
func main()  {
    fmt.Println("myfloat: ", myfloat)
    fmt.Println("myfloat: ", myfloat+1)
}
```

输出如下

```go
myfloat:  1.0000018e+07
myfloat:  1.0000019e+07
```

# byte、rune与字符串

## byte与rune

**byte**，占用1个节字，就 8 个比特位（2^8 = 256，因此 byte 的表示范围 0->255），所以它和 `uint8` 类型本质上没有区别，它表示的是 `ACSII` 表中的一个字符。

如下这段代码，分别定义了 byte 类型和 uint8 类型的变量 a 和 b

```go
import "fmt"

func main() {
    var a byte = 65
    // 8进制写法: var a byte = '\101'     其中 \ 是固定前缀
    // 16进制写法: var a byte = '\x41'    其中 \x 是固定前缀

    var b uint8 = 66
    fmt.Printf("a 的值: %c \nb 的值: %c", a, b)

    // 或者使用 string 函数
    // fmt.Println("a 的值: ", string(a)," \nb 的值: ", string(b))
}
```

在 ASCII 表中，由于字母 A 的ASCII 的编号为 65 ，字母 B 的ASCII 编号为 66，所以上面的代码也可以写成这样：

```go
import "fmt"

func main() {
    var a byte = 'A'
    var b uint8 = 'B'
    fmt.Printf("a 的值: %c \nb 的值: %c", a, b)
}
```

他们的输出结果都是一样的。

```go
a 的值: A
b 的值: B
```

**rune**，占用4个字节，共32位比特位，所以它和 `int32` 本质上也没有区别。它表示的是一个 Unicode字符（Unicode是一个可以表示世界范围内的绝大部分字符的编码规范）。

```go
import (
    "fmt"
    "unsafe"
)

func main() {
    var a byte = 'A'
    var b rune = 'B'
    fmt.Printf("a 占用 %d 个字节数\nb 占用 %d 个字节数", unsafe.Sizeof(a), unsafe.Sizeof(b))
}
```

输出如下

```
a 占用 1 个字节数
b 占用 4 个字节数
```

由于 byte 类型能表示的值是有限，只有 2^8=256 个。所以如果你想表示中文的话，你只能使用 rune 类型。

```go
var name rune = '中'
```

或许你已经发现，上面我们在定义字符时，不管是 byte 还是 rune ，我都是使用单引号，而没使用双引号。

对于从 Python 转过来的人，这里一定要注意了，在 Go 中单引号与 双引号并不是等价的。

单引号用来表示字符，在上面的例子里，如果你使用双引号，就意味着你要定义一个字符串，赋值时与前面声明的会不一致，这样在编译的时候就会出错。

```
cannot use "A" (type string) as type byte in assignment
```

上面我说了，`byte` 和 `uint8` 没有区别，`rune` 和 `uint32` 没有区别，那为什么还要多出一个 byte 和 rune 类型呢？

理由很简单，因为`uint8` 和 `uint32` ，直观上让人以为这是一个数值，但是实际上，它也可以表示一个字符，所以为了消除这种直观错觉，就诞生了 byte 和 rune 这两个别名类型。

## 字符串

字符串，可以说是大家很熟悉的数据类型之一。定义方法很简单

```go
var mystr string = "hello"
```

上面说的byte 和 rune 都是字符类型，若多个字符放在一起，就组成了字符串，也就是这里要说的 string 类型。

比如 `hello` ，对照 `ascii` 编码表，每个字母对应的编号是：104,101,108,108,111

```go
var str string = "hello"
var mystr [5]byte = [5]byte{104, 101, 108, 108, 111}
fmt.Printf("str:%s\n", str)
fmt.Printf("mystr:%s\n", mystr)
```

输出如下，`str`和 `mystr`输出一样，说明了 string 的本质，其实是一个 byte数组


```go
str:hello
mystr:hello
```

表示字符串除了双引号，也可以用反引号。

大多情况下，二者并没有区别，但如果你的字符串中有转义字符`\` ，这里就要注意了，它们是有区别的。

使用反引号包裹的字符串，相当于 Python 中的 raw 字符串，会忽略里面的转义。

比如我想表示 `\r\n` 这个 字符串，使用双引号是这样写的，这种叫解释型表示法

```
var mystr01 string = "\\r\\n"
```

而使用反引号，就方便多了，所见即所得，这种叫原生型表示法

```
var mystr02 string = `\r\n`
```

他们的打印结果 都是一样的

```go
import (
    "fmt"
)

func main() {
    var mystr01 string = "\\r\\n"
    var mystr02 string = `\r\n`
    fmt.Println(mystr01)
    fmt.Println(mystr02)
}

// output
\r\n
\r\n
```

同时反引号可以不写换行符（因为没法写）来表示一个多行的字符串。

```
import (
    "fmt"
)

func main() {
	var hello string = `你好呀！
	你一定可以学好golang的`
	fmt.Println(hello)
}
```

输出如下

```go
你好呀！
	你一定可以学好golang的
```

# 数组和切片

## 数组

数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因为数组的长度是固定的，所以在Go语言中很少直接使用数组。

声明数组，并给该数组里的每个元素赋值（索引值的最小有效值和其他大多数语言一样是 0，不是1）

```
// [3] 里的3 表示该数组的元素个数及容量
var arr [3]int
arr[0] = 1
arr[1] = 2
arr[2] = 3
```

声明并直接初始化数组

```
// 第一种方法
var arr [3]int = [3]int{1,2,3}

// 第二种方法
arr := [3]int{1,2,3}
```

上面的 3 表示数组的元素个数 ，万一你哪天想往该数组中增加元素，你得对应修改这个数字，为了避免这种硬编码，你可以这样写，使用 `...` 让Go语言自己根据实际情况来分配空间。

```
arr := [...]int{1,2,3}
```

`[3]int` 和 `[4]int` 虽然都是数组，但他们却是不同的类型，使用 fmt 的 `%T` 可以查得。

```go
import (
    "fmt"
)

func main() {
    arr01 := [...]int{1, 2, 3}
    arr02 := [...]int{1, 2, 3, 4}
    fmt.Printf("%d 的类型是: %T\n", arr01, arr01)
    fmt.Printf("%d 的类型是: %T", arr02, arr02)
}
```

输出 如下

```go
[1 2 3] 的类型是: [3]int
[1 2 3 4] 的类型是: [4]int
```

如果你觉得每次写 `[3]int` 有点麻烦，你可以为 `[3]int` 定义一个类型字面量，也就是别名类型。

使用 `type` 关键字可以定义一个类型字面量，后面只要你想定义一个容器大小为3，元素类型为int的数组 ，都可以使用这个别名类型。

```go
import (
    "fmt"
)

func main() {
    type arr3 [3]int

    myarr := arr3{1,2,3}
    fmt.Printf("%d 的类型是: %T", myarr, myarr)
}
```

输出 如下

```go
[1 2 3] 的类型是: main.arr3
```

其实定义数组还有一种偷懒的方法，比如下面这行代码

```go
arr:=[4]int{2:3}
```

打印 arr，会是

```go
[0 0 3 0]
```

可以看出`[4]int{2:3}`，4表示数组有4个元素，2 和 3 分别表示该数组索引为2（初始索引为0）的值为3，而其他没有指定值的，就是 int 类型的零值，即0。

## 切片

切片（Slice）与数组一样，也是可以容纳若干类型相同的元素的容器。与数组不同的是，无法通过切片类型来确定其值的长度。每个切片值都会将数组作为其底层数据结构。我们也把这样的数组称为切片的底层数组。

切片是对数组的一个连续片段的引用，所以切片是一个**引用类型**，这个片段可以是整个数组，也可以是由起始和终止索引标识的一些项的子集，需要注意的是，终止索引标识的项不包括在切片内（意思是这是个**左闭右开的区间**）

```go
import (
    "fmt"
)

func main() {
    myarr := [...]int{1, 2, 3}
    fmt.Printf("%d 的类型是: %T", myarr[0:2], myarr[0:2])
}
```

输出 如下

```go
[1 2] 的类型是: []int
```

切片的构造，有四种方式

1. 对数组进行片段截取，主要有如下两种写法

   ```go
   // 定义一个数组
   myarr := [5]int{1,2,3,4,5}
   
   // 【第一种】
   // 1 表示从索引1开始，直到到索引为 2 (3-1)的元素
   mysli1 := myarr[1:3]
   
   // 【第二种】
   // 1 表示从索引1开始，直到到索引为 2 (3-1)的元素
   mysli2 := myarr[1:3:4]
   ```

   如果你把上面的 `mysli1` 和 `mysli2` 打印出来，会发现他们居然是一样的。那第二种的 `myarr[1:3:4]` 的 4有什么用呢？

   在切片时，若不指定第三个数，那么切片终止索引会一直到原数组的最后一个数。而如果指定了第三个数，那么切片终止索引只会到原数组的该索引值。

   用下面这段代码来验证一下

   ```go
   package main
   
   import "fmt"
   
   func main(){
       myarr := [5]int{1,2,3,4,5}
       fmt.Printf("myarr 的长度为：%d，容量为：%d\n", len(myarr), cap(myarr))
   
       mysli1 := myarr[1:3]
       fmt.Printf("mysli1 的长度为：%d，容量为：%d\n", len(mysli1), cap(mysli1))
       fmt.Println(mysli1)
   
       mysli2 := myarr[1:3:4]
       fmt.Printf("mysli2 的长度为：%d，容量为：%d\n", len(mysli2), cap(mysli2))
       fmt.Println(mysli2)
   }
   ```

   输出如下，说明切片的第三个数，影响的只是切片的容量，而不会影响长度

   ```
   myarr 的长度为：5，容量为：5
   mysli1 的长度为：2，容量为：4
   [2 3]
   mysli2 的长度为：2，容量为：3
   [2 3]
   ```

2. 从头声明赋值（例子如下）

```go
// 声明字符串切片
var strList []string

// 声明整型切片
var numList []int

// 声明一个空切片
var numListEmpty = []int{}
```

3. 使用 make 函数构造，make 函数的格式：`make( []Type, size, cap )`

   这个函数刚好指出了，一个切片具备的三个要素：类型（Type），长度（size），容量（cap）

```
import (
 "fmt"
)

func main() {
 a := make([]int, 2)
 b := make([]int, 2, 10)
 fmt.Println(a, b)
 fmt.Println(len(a), len(b))
 fmt.Println(cap(a), cap(b))
}
```

​		输出 如下

```go
[0 0] [0 0]
2 2
2 10
```

1. 使用和数组一样，偷懒的方法

   ```
   import (
    "fmt"
   )
   
   func main() {
       a := []int{4:2}
       fmt.Println(a)
       fmt.Println(len(a), cap(a))
   }
   ```

   输出如下

   ```
   [0 0 0 0 2]
   5 5
   ```

关于 len 和 cap 的概念，可能不好理解 ，这里举个例子：

- 公司名，相当于字面量，也就是变量名。
- 公司里的所有工位，相当于已分配到的内存空间
- 公司里的员工，相当于元素。
- cap 代表你这个公司最多可以容纳多少员工
- len 代表你这个公司当前有多少个员工

由于 切片是引用类型，所以你不对它进行赋值的话，它的零值（默认值）是 nil

```
var myarr []int
fmt.Println(myarr == nil)
// true
```

数组 与 切片 有相同点，它们都是可以容纳若干类型相同的元素的容器

也有不同点，数组的容器大小固定，而切片本身是引用类型，它更像是 Python 中的 list ，我们可以对它 append 进行元素的添加。

```go
import (
    "fmt"
)

func main() {
    myarr := []int{1}
    // 追加一个元素
    myarr = append(myarr, 2)
    // 追加多个元素
    myarr = append(myarr, 3, 4)
    // 追加一个切片, ... 表示解包，不能省略
    myarr = append(myarr, []int{7, 8}...)
    // 在第一个位置插入元素
    myarr = append([]int{0}, myarr...)
    // 在中间插入一个切片(两个元素)
    myarr = append(myarr[:5], append([]int{5,6}, myarr[5:]...)...)
    fmt.Println(myarr)
}
```

输出 如下

```go
[0 1 2 3 4 5 6 7 8]
```

# 字典与布尔类型

   ## 字典

字典（Map 类型），是由若干个 `key:value` 这样的键值对映射组合在一起的数据结构。

它是哈希表的一个实现，这就要求它的每个映射里的key，都是唯一的，可以使用 `==` 和 `!=` 来进行判等操作，换句话说就是key必须是可哈希的。

什么叫可哈希的？简单来说，一个不可变对象，都可以用一个哈希值来唯一表示，这样的不可变对象，比如字符串类型的对象（可以说除了切片、 字典，函数之外的其他内建类型都算）。

意思就是，你的 key 不能是切片，不能是字典，不能是函数。。

字典由key和value组成，它们各自有各自的类型。

在声明字典时，必须指定好你的key和value是什么类型的，然后使用 map 关键字来告诉Go这是一个字典。

```go
map[KEY_TYPE]VALUE_TYPE
```

### 声明初始化字典

三种声明并初始化字典的方法

```
// 第一种方法
var scores map[string]int = map[string]int{"english": 80, "chinese": 85}

// 第二种方法
scores := map[string]int{"english": 80, "chinese": 85}

// 第三种方法
scores := make(map[string]int)
scores["english"] = 80
scores["chinese"] = 85
```

要注意的是，第一种方法如果拆分成多步（声明、初始化、再赋值），和其他两种有很大的不一样了，相对会比较麻烦。

```go
import "fmt"

func main() {
    // 声明一个名为 score 的字典
    var scores map[string]int

    // 未初始化的 score 的零值为nil，无法直接进行赋值
    if scores == nil {
        // 需要使用 make 函数先对其初始化
        scores = make(map[string]int)
    }

    // 经过初始化后，就可以直接赋值
    scores["chinese"] = 90
    fmt.Println(scores)
}
```

### **字典的相关操作**

添加元素

```go
scores["math"] = 95
```

更新元素，若key已存在，则直接更新value

```
scores["math"] = 100
```

读取元素，直接使用 `[key]` 即可 ，如果 key 不存在，也不报错，会返回其value-type 的零值。

```
fmt.Println(scores["math"])
```

删除元素，使用 delete 函数，如果 key 不存在，delete 函数会静默处理，不会报错。

```go
delete(scores, "math")
```

当访问一个不存在的key时，并不会直接报错，而是会返回这个 value 的零值，如果 value的类型是int，就返回0。

```go
package main

import "fmt"

func main() {
    scores := make(map[string]int)
    fmt.Println(scores["english"]) // 输出 0
}
```

### 判断 key 是否存在

当key不存在，会返回value-type的零值 ，所以你不能通过返回的结果是否是零值来判断对应的 key 是否存在，因为 key 对应的 value 值可能恰好就是零值。

其实字典的下标读取可以返回两个值，使用第二个返回值都表示对应的 key 是否存在，若存在ok为true，若不存在，则ok为false

```go
import "fmt"

func main() {
    scores := map[string]int{"english": 80, "chinese": 85}
    math, ok := scores["math"]
    if ok {
        fmt.Printf("math 的值是: %d", math)
    } else {
        fmt.Println("math 不存在")
    }
}
```

我们将上面的代码再优化一下

```go
import "fmt"

func main() {
    scores := map[string]int{"english": 80, "chinese": 85}
    if math, ok := scores["math"]; ok {
        fmt.Printf("math 的值是: %d", math)
    } else {
        fmt.Println("math 不存在")
    }
}
```

### **如何对字典进行循环**

Go 语言中没有提供类似 Python 的 keys() 和 values() 这样方便的函数，想要获取，你得自己循环。

循环还分三种

1. 获取 key 和 value

```
import "fmt"

func main() {
    scores := map[string]int{"english": 80, "chinese": 85}

    for subject, score := range scores {
        fmt.Printf("key: %s, value: %d\n", subject, score)
    }
}
```

1. 只获取key，这里注意不用占用符。

```
import "fmt"

func main() {
    scores := map[string]int{"english": 80, "chinese": 85}

    for subject := range scores {
        fmt.Printf("key: %s\n", subject)
    }
}
```

1. 只获取 value，用一个占位符替代。

```go
import "fmt"

func main() {
    scores := map[string]int{"english": 80, "chinese": 85}

    for _, score := range scores {
        fmt.Printf("value: %d\n", score)
    }
}
```

## 布尔类型

关于布尔值，无非就两个值：true 和 false。只是这两个值，在不同的语言里可能不同。

在 Python 中，真值用 True 表示，与 1 相等，假值用 False 表示，与 0 相等

```go
>>> True == 1
True
>>> False == 0
True
>>>
```

而在 Go 中，真值用 true 表示，不但不与 1 相等，并且更加严格，不同类型无法进行比较，而假值用 false 表示，同样与 0 无法比较。

Go 中确实不如 Python 那样灵活，`bool` 与 `int` 不能直接转换，如果要转换，需要你自己实现函数。

**bool 转 int**

```go
func bool2int(b bool) int {
    if b {
        return 1
    }
    return 0
}
```

**int 转 bool**

```go
func int2bool(i int) bool {
    return i != 0
}
```

在 Python 中使用 not 对逻辑值取反，而 Go 中使用 `!` 符号

```go
import "fmt"

var male bool = true
func main()  {
    fmt.Println( !male == false)
    // 或者
    fmt.Println( male != false)
}

// output: true
```

一个 if 判断语句，有可能不只一个判断条件，在 Python 中是使用 `and` 和 `or` 来执行逻辑运算

```go
>>> age = 15
>>> gender = "male"
>>>
>>> gender == "male" and age >18
False
```

而在 Go 语言中，则使用 `&&` 表示`且`，用 `||` 表示`或`，并且有短路行为（即左边表达式已经可以确认整个表达式的值，那么右边将不会再被求值。

```go
import "fmt"

var age int = 15
var gender string = "male"
func main()  {
    //  && 两边的表达式都会执行
    fmt.Println( age > 18 && gender == "male")
    // gender == "male" 并不会执行
    fmt.Println( age < 18 || gender == "male")
}

// output: false
// output: true
```

# 指针

## 什么是指针

当我们定义一个变量 name

```
var name string = "hello"
```

此时，name 是变量名，它只是编程语言中方便程序员编写和理解代码的一个标签。

当我们访问这个标签时，机算机会返回给我们它指向的内存地址里存储的值：`hello`。

出于某些需要，我们会将这个内存地址赋值给另一个变量名，通常叫做 `ptr`（pointer的简写），而这个变量，我们称之为指针变量。

换句话说，指针变量（一个标签）的值是指针，也就是内存地址。

根据变量指向的值，是否是内存地址，我把变量分为两种：

- 普通变量：存数据值本身
- 指针变量：存值的内存地址

### 指针的创建

指针创建有三种方法

**第一种方法**

先定义对应的变量，再通过变量取得内存地址，创建指针

```go
// 定义普通变量
aint := 1
// 定义指针变量
ptr := &aint
```

**第二种方法**

先创建指针，分配好内存后，再给指针指向的内存地址写入对应的值。

```go
// 创建指针
astr := new(string)
// 给指针赋值
*astr = "hello"
```

**第三种方法**

先声明一个指针变量，再从其他变量取得内存地址赋值给它

```go
aint := 1
var bint *int  // 声明一个指针
bint = &aint   // 初始化
```

上面的三段代码中，指针的操作都离不开这两个符号：

- `&` ：从一个普通变量中取得内存地址
- `*`：当`*`在赋值操作符（=）的右边，是从一个指针变量中取得变量值，当`*`在赋值操作符（=）的左边，是指该指针指向的变量

```go
package main

import "fmt"

func main() {
    aint := 1     // 定义普通变量
    ptr := &aint  // 定义指针变量
    fmt.Println("普通变量存储的是：", aint)
    fmt.Println("普通变量存储的是：", *ptr)
    fmt.Println("指针变量存储的是：", &aint)
    fmt.Println("指针变量存储的是：", ptr)
}
```

输出如下

```
普通变量存储的是： 1
普通变量存储的是： 1
指针变量存储的是： 0xc0000100a0
指针变量存储的是： 0xc0000100a0
```

要想打印指针指向的内存地址，方法有两种

```go
// 第一种
fmt.Printf("%p", ptr)

// 第二种
fmt.Println(ptr)
```

## 指针的类型

我们知道字符串的类型是 string，整型是int，那么指针如何表示呢？

写段代码试验一下就知道了

```go
package main

import "fmt"

func main() {
    astr := "hello"
    aint := 1
    abool := false
    arune := 'a'
    afloat := 1.2

    fmt.Printf("astr 指针类型是：%T\n", &astr)
    fmt.Printf("aint 指针类型是：%T\n", &aint)
    fmt.Printf("abool 指针类型是：%T\n", &abool)
    fmt.Printf("arune 指针类型是：%T\n", &arune)
    fmt.Printf("afloat 指针类型是：%T\n", &afloat)
}
```

输出如下，可以发现用 `*`+所指向变量值的数据类型，就是对应的指针类型。

```go
astr 指针类型是：*string
aint 指针类型是：*int
abool 指针类型是：*bool
arune 指针类型是：*int32
afloat 指针类型是：*float64
```

所以若我们定义一个只接收指针类型的参数的函数，可以这么写

```go
func test(ptr *int)  {
    fmt.Println(*ptr)
}
```

## 指针的零值

当指针声明后，没有进行初始化，其零值是 nil。

```go
func main() {
    a := 25
    var b *int  // 声明一个指针

    if b == nil {
        fmt.Println(b)
        b = &a  // 初始化：将a的内存地址给b
        fmt.Println(b)
    }
}
```

输出如下

```go
<nil>
0xc0000100a0
```

## 指针与切片

切片与指针一样，都是引用类型。

如果我们想通过一个函数改变一个数组的值，有两种方法

1. 将这个数组的切片做为参数传给函数
2. 将这个数组的指针做为参数传给函数

尽管二者都可以实现我们的目的，但是按照 Go 语言的使用习惯，建议使用第一种方法，因为第一种方法，写出来的代码会更加简洁，易读。具体你可以参数下面两种方法的代码实现

**使用切片**

```go
func modify(sls []int) {
    sls[0] = 90
}

func main() {
    a := [3]int{89, 90, 91}
    modify(a[:])
    fmt.Println(a)
}
```

**使用指针**

```go
func modify(arr *[3]int) {
    (*arr)[0] = 90
}

func main() {
    a := [3]int{89, 90, 91}
    modify(&a)
    fmt.Println(a)
}
```





[//]:#(设置表格整体居中显示)

<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


