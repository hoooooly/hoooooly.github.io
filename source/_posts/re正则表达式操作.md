---
title: re正则表达式操作
tags:
  - re
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-04 22:02:30
categories: Python库
pic:
---


`Regular Expression`，正则表达式，使用表达式的方式对字符串进行匹配的语法规则。

## 正则表达式语法

一个*正则表达式*指定了一集与之匹配的字符串；模块内的函数可以让你检查某个字符串是否跟给定的正则表达式匹配。

要看正则写得对不对，可以去[测试网站](https://tool.oschina.net/regex/)测试匹配结果。

首先了解`元字符`的概念：*具有固定含义的特殊符号*。

特殊字符有：

```
 .    
 (点) 在默认模式，匹配**除了换行的任意字符**。

 ^    
 (插入符号) 匹配字符串的**开头**。

 $    
 匹配**字符串尾或者在字符串尾的换行符的前一个字符**。在 *MULTILINE* 模式下也会匹配换行符之前的文本。

 *    
 对它前面的正则式匹配**0到任意次重复**，尽量多的匹配字符串。 ab* 会匹配 a，ab，或者 a 后面跟随任意个 b。

 +    
 对它前面的正则式匹配**1到任意次重复**。ab+ 会匹配 a 后面跟随1个以上到任意个 b，它不会匹配 a。

 ?    
 对它前面的正则式匹配**0到1次重复**。 ab? 会匹配 a 或者 ab。

 *?, +?, ??    
 \*, +，和 ? 修饰符都是 **贪婪的**；它们在字符串进行尽可能多的匹配。如果正则式 <.*> 希望找到 <a> b <c>，它将会匹配整个字符串，而不仅是 <a>。在修饰符之后添加 ? 将使样式以 非贪婪 方式进行匹配； 尽量 **少** 的字符将会被匹配。使用正则式 <.*?> 将会仅仅匹配 <a>。

 {m}    
 对其之前的正则式指定匹配 m 个重复；少于 m 的话就会导致匹配失败。比如， a{6} 将匹配6个 a , 但是不能是5个。

 {m,n}    
 对正则式进行 m 到 n 次匹配，在 m 和 n 之间取尽量多。 比如，a{3,5} 将匹配 3 到5个 a。忽略 m 意为指定下界为0，忽略 n 指定上界为无限次。比如 a{4,}b 将匹配 aaaab 或者1000个 a 尾随一个 b，但不能匹配 aaab。逗号不能省略，否则无法辨别修饰符应该忽略哪个边界。

 {m,n}?    
 前一个修饰符的非贪婪模式，只匹配尽量少的字符次数。比如，对于 aaaaaa， a{3,5} 匹配5个 a ，而 a{3,5}? 只匹配3个 a。

 \    
 转义特殊字符（允许你匹配 '*', '?', 或者此类其他），或者表示一个特殊序列；

 []
 用于表示一个字符集合。在一个集合中：
    - 字符可以单独列出，比如 [amk] 匹配 'a'， 'm'， 或者 'k'。
    - 可以表示字符范围，通过用 '-' 将两个字符连起来。比如 [a-z] 将匹配任何小写ASCII字符， [0-5][0-9] 将匹配从 00 到 59 的两位数字， [0-9A-Fa-f] 将匹配任何十六进制数位。
    - 不在集合范围内的字符可以通过 取反 来进行匹配。如果集合首字符是 '^' ，所有 不 在集合内的字符将会被匹配，比如 [^5] 将匹配所有字符，除了 '5'， [^^] 将匹配所有字符，除了 '^'. ^ 如果不在集合首位，就没有特殊含义。
    - 在集合内要匹配一个字符 ']'，有两种方法，要么就在它之前加上反斜杠，要么就把它放到集合首位。比如， [()[\]{}] 和 []()[{}] 都可以匹配括号。

|
A|B， A 和 B 可以是任意正则表达式，创建一个正则表达式，匹配 A 或者 B. 任意个正则表达式可以用 '|' 连接。它也可以在组合（见下列）内使用。扫描目标字符串时， '|' 分隔开的正则样式从左到右进行匹配。当一个样式完全匹配时，这个分支就被接受。意思就是，一旦 A 匹配成功， B 就不再进行匹配，即便它能产生一个更好的匹配。或者说，'|' 操作符绝不贪婪。 如果要匹配 '|' 字符，使用 \|， 或者把它包含在字符集里，比如 [|].

(...)    
（组合），匹配括号内的任意正则表达式，并标识出组合的开始和结尾。匹配完成后，组合的内容可以被获取，并可以在之后用 \number 转义序列进行再次匹配，之后进行详细说明。要匹配字符 '(' 或者 ')', 用 \( 或 \), 或者把它们包含在字符集合里: [(], [)].

\A
只匹配字符串开始。
 
\b
匹配空字符串，但只在单词开始或结尾的位置。一个单词被定义为一个单词字符的序列。注意，通常 \b 定义为 \w 和 \W 字符之间，或者 \w 和字符串开始/结尾的边界， 意思就是 r'\bfoo\b' 匹配 'foo', 'foo.', '(foo)', 'bar foo baz' 但不匹配 'foobar' 或者 'foo3'。

\B
匹配空字符串，但 不 能在词的开头或者结尾。意思就是 r'py\B' 匹配 'python', 'py3', 'py2', 但不匹配 'py', 'py.', 或者 'py!'. \B 是 \b 的取非，所以Unicode样式的词语是由Unicode字母，数字或下划线构成的，虽然可以用 ASCII 标志来改变。如果使用了 LOCALE 标志，则词的边界由当前语言区域设置。

\d
对于 Unicode (str) 样式：
匹配任何Unicode十进制数（就是在Unicode字符目录[Nd]里的字符）。这包括了 [0-9] ，和很多其他的数字字符。如果设置了 ASCII 标志，就只匹配 [0-9] 。

对于8位(bytes)样式：
匹配任何十进制数，就是 [0-9]。

\D
匹配任何非十进制数字的字符。就是 \d 取非。 如果设置了 ASCII 标志，就相当于 [^0-9] 。

\s
对于 Unicode (str) 样式：
匹配任何Unicode空白字符（包括 [ \t\n\r\f\v] ，还有很多其他字符，比如不同语言排版规则约定的不换行空格）。如果 ASCII 被设置，就只匹配 [ \t\n\r\f\v] 。

对于8位(bytes)样式：
匹配ASCII中的空白字符，就是 [ \t\n\r\f\v] 。

\S
匹配任何非空白字符。就是 \s 取非。如果设置了 ASCII 标志，就相当于 [^ \t\n\r\f\v] 。

\w
对于 Unicode (str) 样式：
匹配Unicode词语的字符，包含了可以构成词语的绝大部分字符，也包括数字和下划线。如果设置了 ASCII 标志，就只匹配 [a-zA-Z0-9_] 。

对于8位(bytes)样式：
匹配ASCII字符中的数字和字母和下划线，就是 [a-zA-Z0-9_] 。如果设置了 LOCALE 标记，就匹配当前语言区域的数字和字母和下划线。

\W
匹配非单词字符的字符。这与 \w 正相反。如果使用了 ASCII 旗标，这就等价于 [^a-zA-Z0-9_]。如果使用了 LOCALE 旗标，则会匹配当前区域中既非字母数字也非下划线的字符。

\Z
只匹配字符串尾。
```

## 模块内容

模块定义了几个函数，常量，和一个例外。

### re.compile

```python
re.compile(pattern, flags=0)
```

`re.compile`将正则表达式的样式编译为一个 `正则表达式对象（正则对象`），可以用于匹配，通过这个对象的方法` match()`, `search()` 。

比如：

```python
import re

prog = re.compile('^a')
result = prog.match("abcd")
print(result)

result = re.match('^a', "abcd")
print(result)

# output
# <re.Match object; span=(0, 1), match='a'>
# <re.Match object; span=(0, 1), match='a'>
```

使用 `re.compile()` 保存正则对象以便复用，可以让程序更加高效。

### re.match

```python
re.match(pattern, string, flags=0)
```

如果 `string` 开始的0或者多个字符匹配到了正则表达式样式，就返回一个相应的 `匹配对象` 。 如果没有匹配，就返回 `None` ；注意它跟零长度匹配是不同的。

### re.search

```python
re.search(pattern, string, flags=0)
```

扫描整个 `字符串` 找到匹配样式的第一个位置，并返回一个相应的 `匹配对象` 。如果没有匹配，就返回一个 `None` ；

### re.split

```python
re.split(pattern, string, maxsplit=0, flags=0)
```

用 `pattern` 分开 `string` 。 如果在 `pattern` 中捕获到括号，那么所有的组里的文字也会包含在列表里。如果 `maxsplit` 非零， 最多进行 `maxsplit` 次分隔， 剩下的字符全部返回到列表的最后一个元素。

```python
>>> re.split(r'\W+', 'Words, words, words.')
['Words', 'words', 'words', '']
>>> re.split(r'(\W+)', 'Words, words, words.')
['Words', ', ', 'words', ', ', 'words', '.', '']
>>> re.split(r'\W+', 'Words, words, words.', 1)
['Words', 'words, words.']
>>> re.split('[a-f]+', '0a3B9', flags=re.IGNORECASE)
['0', '3', '9']
```

### re.findall

```python
re.findall(pattern, string, flags=0)
```

对 `string` 返回一个不重复的 `pattern` 的匹配列表， `string` 从左到右进行扫描，匹配按找到的顺序返回。如果样式里存在一到多个组，就返回一个组合列表；就是一个元组的列表（如果样式里有超过一个组合的话）。空匹配也会包含在结果里。

```python
import re

partern = re.compile(r"\d+")

res = partern.findall("我的电话号码是1760718283，你的电话是110吗？")
print(res)

# ['1760718283', '110']
```

###  re.finditer

```python
re.finditer(pattern, string, flags=0)
```

`pattern` 在 `string` 里所有的非重复匹配，返回为一个迭代器 `iterator` 保存了 匹配对象 。 string 从左到右扫描，匹配按顺序排列。空匹配也包含在结果里。

```python
import re

partern = re.compile(r"\d+")

res = partern.finditer("我的电话号码是1760718283，你的电话是110吗？")
for item in res:
    print(item.group())

# output
# 1760718283
# 110
```

## 匹配对象

匹配对象总是有一个布尔值 `True`。如果没有匹配的话 `match()` 和 `search()` 返回 `None` 所以你可以简单的用 `if` 语句来判断是否匹配

```python
match = re.search(pattern, string)
if match:
    process(match)
```

匹配对象支持以下方法和属性：

### Match.group

```python
Match.group([group1, ...])
```

返回一个或者多个匹配的子组。如果只有一个参数，结果就是一个字符串，如果有多个参数，结果就是一个元组（每个参数对应一个项），如果没有参数，组1默认到0（整个匹配都被返回）。如果一个组N 参数值为 0，相应的返回值就是整个匹配字符串；如果它是一个范围 [1..99]，结果就是相应的括号组字符串。

```python
import re

match = re.match(r"(\w+) (\w+)", "Hello World, heyhey!")
print(match)    # <re.Match object; span=(0, 11), match='Hello World'>

print(match.group())   # Hello World
print(match.group(0))   # Hello World
print(match.group(1))   # Hello
print(match.group(2))   # World
```

如果正则表达式使用了 `(?P<name>…)` 语法， `groupN` 参数就也可能是命名组合的名字。

```python
>>> m = re.match(r"(?P<first_name>\w+) (?P<last_name>\w+)", "Malcolm Reynolds")
>>> m.group('first_name')
'Malcolm'
>>> m.group('last_name')
'Reynolds'

```

命名组合同样可以通过索引值引用。

```python
>>> m.group(1)
'Malcolm'
>>> m.group(2)
'Reynolds'
```

如果一个组匹配成功多次，就只返回最后一个匹配。

```python
>>> m = re.match(r"(..)+", "a1b2c3")  # Matches 3 times.
>>> m.group(1)                        # Returns only the last match.
'c3'
```

### Match.groups

```python
Match.groups(default=None)
```

返回一个元组，包含所有匹配的子组，在样式中出现的从1到任意多的组合。 `default` 参数用于不参与匹配的情况，默认为 `None`。

```python
import re

match = re.match(r"(\w+) (\w+)", "Hello World, heyhey!")
print(match)    # <re.Match object; span=(0, 11), match='Hello World'>

print(match.groups())   # ('Hello', 'World')
```

[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>



