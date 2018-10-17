title: Scala Cheatsheet
date: 2018/10/17
categories:
- Program Development
tags:
- Scala
---


### for循环 ###

**普通形式**

    for(x <- Range){
    	statement(s);
    }

Range可以是

  - 可迭代结构
  - `i until j`
  - `i to j`

`x`之前并没有`var`指定。其类型是由`Range`决定的。

**多范围**

    for(x <- Range1; y <- Range2){
    	statement(s);
    }


**带守卫**

    for(x <- Range if condition_x1; if condition_x2...; y <- Range2 if condition_y1; if condition_y2...){
    	statement(s);
    }


**生成新List**

    var retVal = for{x <- List if condition1; if condition2...} yield x

