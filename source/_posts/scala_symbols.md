title: Scala Symbols
date: 2018/10/23
categories:
- Program Development
tags:
- Scala
---


## 1. 箭头 ##

### `<-` ###

```scala
for (arg <- args)

for (i <- 0 until 10) 
```

for循环中


### `->` ###

|||
|---|---|
|`val colors = Map("red" -> "#FF0000", "azure" -> "#F0FFFF")`|定义Map的键值对映射|
|`val t = "A" -> "B"    //("A", "B")`|直接生成Tuple（其实上面也是这个用法）|


### `<=` ###

没这玩意


### `=>` ###

|||
|---|---|
|`t: => Long`|参数声明，传名调用|
|`f: Int => String`|参数声明，传函数|
|`(x: Int) => x + 1`|定义匿名函数|
|`case 1 => "one"`|模式匹配match-case中|

