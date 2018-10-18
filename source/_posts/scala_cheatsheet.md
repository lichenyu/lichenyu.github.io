title: Scala Cheatsheet
date: 2018/10/17
categories:
- Program Development
tags:
- Scala
---

## 2.1. 条件控制 ##

### 2.1.1. if条件 ###

与java相同

    if(condition1){
        statement(s);
    }else if(condition2){
        statement(s);
    }else {
        statement(s);
    }


### 2.1.2. 不支持switch ###


## 2.2. 循环 ##

### 2.2.1. while循环 ###

与java相同

    while(condition)
    {
        statement(s);
    }

    do {
        statement(s);
    } while(condition);


### 2.2.2. for循环 ###

普通形式：

    for(x <- Range){
        statement(s);
    }

Range可以是

  - 可迭代结构
  - `i until j`
  - `i to j`

`x`之前并没有`var`指定。其类型是由`Range`决定的。

多范围：

    for(x <- Range1; y <- Range2){
        statement(s);
    }


带守卫：

    for(x <- Range if condition_x1; if condition_x2...; y <- Range2 if condition_y1; if condition_y2...){
        statement(s);
    }


生成新List：

    var retVal = for{x <- List if condition1; if condition2...} yield x


### 2.2.3. 不支持continue、break ###

    // 导入以下包
    import scala.util.control._
    
    // 创建 Breaks 对象
    val loop = new Breaks;
    
    // 在 breakable 中循环
    loop.breakable{
        // 循环
        for(...){
            ....
            // 循环中断
            loop.break;
        }
    }


## 3.1. 方法和函数 ##

Scala中，方法是类的一部分，而函数是一个对象可以赋值给一个变量。
Scala的方法跟Java的类似，方法是组成类的一部分。
Scala的函数则是一个完整的对象，Scala的函数其实就是继承了Trait的类的对象。

Scala中使用`def`语句定义方法，`val`语句可以定义函数。

    class Test{
        def m(x: Int) = x + 3
        val f = (x: Int) => x + 3
    }

- 如果方法不写等号和主体，那么该方法会被隐式声明为抽象，包含它的类型也是一个抽象类型。
- 定义函数时，可以指定默认参数。
- 调用函数时，可以指定函数参数名传参，不一定要按顺序传参。
- 可变参数，在参数的类型之后放一个星号来设置，`def printStrings(args: String*)`。
- 函数中可以嵌套函数。
- 在Scala中无法直接操作方法，如果要操作方法，必须先将其转换成函数。方法转函数：`val f1 = m _`。


### 3.1.1. 函数传值/传名调用 ###

Scala的解释器在解析函数参数(function arguments)时有两种方式：

- 传值调用（call-by-value）：先计算参数表达式的值，再应用到函数内部。
- 传名调用（call-by-name）：将未计算的参数表达式直接应用到函数内部。

在进入函数内部前，传值调用方式就已经将参数表达式的值计算完毕，而传名调用是在函数内部进行参数表达式的值计算的。
这就造成了一种现象，每次使用传名调用时，解释器都会计算一次表达式的值。

    object Test {
        def main(args: Array[String]) {
            delayed(time());
        }

        def time() = {
            println("获取时间，单位为纳秒")
            System.nanoTime
        }
        def delayed( t: => Long ) = {
            println("在 delayed 方法内")
            println("参数： " + t) //计算t，打印"获取时间，单位为纳秒"
            t //计算t，打印"获取时间，单位为纳秒"
       }
    }

以上实例中我们声明的delayed方法，在变量名和变量类型使用`=>`符号来设置传名调用。


### 3.1.2. 高阶函数 ###

高阶函数可以使用其他函数作为参数，或者使用函数作为输出结果。

    object Test {
        def main(args: Array[String]) {
            println( apply( layout, 10) )
        }
        
        // 函数 f 和 值 v 作为参数，而函数 f 又调用了参数 v
        def apply(f: Int => String, v: Int) = f(v)

        def layout[A](x: A) = "[" + x.toString() + "]"
    }


`f: Int => String`中`Int`为入参类型，`String`为返回值类型。


### 3.1.3. 匿名函数 ###

箭头左边是参数列表，右边是函数体。

    var inc = (x: Int) => x + 1

    var userDir = () => { System.getProperty("user.dir") }


### 3.1.4. 偏应用函数 ###

不需要提供函数需要的所有参数，只需要提供部分，或不提供所需参数。

    import java.util.Date

    object Test {
        def main(args: Array[String]) {
            val date = new Date
            val logWithDateBound = log(date, _ : String)

            logWithDateBound("message1" )
            Thread.sleep(1000)
            logWithDateBound("message2" )
            Thread.sleep(1000)
            logWithDateBound("message3" )
        }

        def log(date: Date, message: String)  = {
            println(date + "----" + message)
        }
    }

`log()`方法接收两个参数：`date`和`message`。我们在程序执行时调用了三次，参数`date`值都相同，`message`不同。我们可以使用偏应用函数优化以上方法，绑定第一个`date`参数，第二个参数使用下划线(_)替换缺失的参数列表，并把这个新的函数值的索引的赋给变量。


### 3.1.5 函数柯里化 ###

柯里化（Currying）指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数为参数的函数。

首先定义一个函数:

    def add(x: Int, y: Int) = x + y

柯里化：

    def add(x: Int)(y: Int) = x + y

实质上最先演变成这样一个方法：

    def add(x: Int) = (y: Int) => x + y


### 3.1.6 闭包 ###

闭包是一个函数，返回值依赖于声明在函数外部的一个或多个变量。
闭包通常来讲可以简单的认为是可以访问一个函数里面局部变量的另外一个函数。

    object Test {
        def main(args: Array[String]) {
            println( "muliplier(1) value = " +  multiplier(1) )
            println( "muliplier(2) value = " +  multiplier(2) )
        }
        var factor = 3
        val multiplier = (i:Int) => i * factor
    }

函数变量`multiplier`成为一个“闭包”，因为它引用到函数外面定义的变量，定义这个函数的过程是将这个自由变量捕获而构成一个封闭的函数。

