---
title: go语言字符串常用的系统函数
tags:
  - go
comments: true
typora-root-url: go语言字符串常用的系统函数
date: 2021-08-30 21:55:27
categories: Go
index_img: 2021/08/30/go语言字符串常用的系统函数/beego.jpeg
banner_img:
---

# 字符串常用的系统函数

1）统计字符串的长度，按字节`len(str)`

```go
str := "hello world"
fmt.Println("str len =", len(str)) // str len = 11
```

2）字符串遍历，同时处理有中文的问题`r:=[]rune(str)`

```go
str2 := "hello世界"
r := []rune(str2)
for i := 0; i < len(r); i++ {
   fmt.Printf("字符=%c\n", r[i])
}
//字符=h
//字符=e
//字符=l
//字符=l
//字符=o
//字符=世
//字符=界
```

3）字符串转整数`n,err := strconv.Atoi("12")`

```go
str3 := "121242354"
n, err := strconv.Atoi(str3)
if err != nil{
   fmt.Println("转换错误")
}else{
   fmt.Println(n) //121242354
}
```

4）整数转字符串`str=strconv.Itoa(123)`)

```go
str4 := strconv.Itoa(1234)
fmt.Printf("str=%v, str=%T", str4, str4) //str=1234, str=string
```

5）字符串转[]byte:`var bytes = []byte("hello go")`

```go
var bytes = []byte("hello go")
fmt.Printf("bytes=%v\n", bytes) //str=1234, str=stringbytes=[104 101 108 108 111 32 103 111]
```

6）[]byte转字符串：`str = string([]byte{97,98,99})`

```go
str5 := string([]byte{97,98,99})
fmt.Printf("str5=%v\n", str5) //str5=abc
```

7）十进制转2，8，16进制：`str = strconv.FormatInt(123, 2) //2->8,16`

```go
fmt.Println(strconv.FormatInt(25, 2)) //11001
fmt.Println(strconv.FormatInt(25, 8)) //31
fmt.Println(strconv.FormatInt(25, 16)) //19
```

8）查找子串是否在指定的字符串中：`strings.Contains("seafood", "foo") //true`

```go
b := strings.Contains("seafood", "food")
fmt.Printf("b=%v\n", b) //b=true
```

9）统计一个字符串有几个指定的子串：`strings.Count("ceheese", "e") //4`

```go
num := strings.Count("chinese", "e")
fmt.Println(num) //2
```

10）不区分大小写的字符串比较（=+是区分字母大小写的）∶ `fmt.Println（strings.EqualFold（"abc"，"Ab")//true`

```go
b := strings.EqualFold("abc", "aBc")
fmt.Printf("%v\n", b) //true
fmt.Println("结果", "abc" == "ABc") //false
```

11）返回子串在字符串第一次出现的 index值，如果没有返回-1∶

```go
index := strings.Index("hello abc", "abc")
fmt.Printf("index=%v\n", index) //index=6
```

12）返回子串在字符串最后一次出现的 index，如没有返回-1∶

```go
index = strings.LastIndex("go golang", "go")
fmt.Printf("index=%v\n", index) //index=3
```

13）将指定的子串替换成 另外一个子串，第三个参数可选择替换几个，如果为-1全部替换：

```go
str := "go go hello"
str2 := strings.Replace(str, "go", "holy", -1)
fmt.Printf("str=%v, str2=%v\n", str, str2) //str=go go hello, str2=holy holy hello
```

14）按 照指定 的某个字 符，为分割标识，将 一个 字符串拆分成字符串 数组∶ `strings.Split("hello,wold,ok",",")`

```go
strArr := strings.Split("hello, world, holy", ",")
for i := 0; i < len(strArr); i++ {
   fmt.Printf("str[%v]=%v\n", i, strArr[i])
}
fmt.Printf("strArr=%v\n", strArr)
```

15）将字符串的字母进行大小写的转换∶

```go
str := "golang hello"
str = strings.ToLower(str)
fmt.Printf("str=%v\n", str) //str=golang hello
str = strings.ToUpper(str)
fmt.Printf("str=%v\n", str) //str=GOLANG HELLO
```

16）将字符串左右两边的空格去掉∶ 

```go
str = strings.TrimSpace(" d you  f   hello     ")
fmt.Printf("str=%v\n", str) //str=d you  f   hello
```

17）将字符串左右两边指定的字符去掉 ∶

```go
str = strings.Trim("!hello!", "!")
fmt.Printf("str=%v\n", str) //str=hello
```

18）将字符串左边指定的字符去掉 ∶

```go
str = strings.TrimLeft("!hello!", "!")
	fmt.Printf("str=%v\n", str) //str=hello!
```

19）将字符串右边指定的字符去掉∶

```go
str = strings.TrimRight("!hello!", "!")
fmt.Printf("str=%v\n", str) //str=hello!
```

20）判断字符串是否以指定的字符串开头 :

```go
b := strings.HasPrefix("ftp://192.168.10.1", "ftp://")
fmt.Printf("b=%v\n", b) //b=true
```

21）判断字符串是否以指定的字符串结束 :

```go
b = strings.HasSuffix("ftp://192.168.10.1", "192.168.10.1")
fmt.Printf("b=%v\n", b) //b=true
```





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


