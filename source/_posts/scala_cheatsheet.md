title: Scala Cheatsheet
date: 2018/10/17
categories:
- Program Development
tags:
- Scala
---


## 1.1. 变量 ##

使用关键词`var`声明变量，使用关键词`val`声明常量。

```scala
var myVar : String = "aaa"
val myVal : String = "bbb"
```


## 1.2. 运算符 ##

基本与java相同，但无`++`。


## 2.1. 条件控制 ##

### 2.1.1. if条件 ###

与java相同

```scala
if(condition1){
    statement(s);
}else if(condition2){
    statement(s);
}else {
    statement(s);
}
```


### 2.1.2. 不支持switch ###


## 2.2. 循环 ##

### 2.2.1. while循环 ###

与java相同

```scala
while(condition)
{
    statement(s);
}

do {
    statement(s);
} while(condition);
```


### 2.2.2. for循环 ###

普通形式：

```scala
for(x <- Range){
    statement(s);
}
```

Range可以是

  - 可迭代结构
  - `i until j`
  - `i to j`

`x`之前并没有`var`指定。其类型是由`Range`决定的。

多范围：

```scala
for(x <- Range1; y <- Range2){
    statement(s);
}
```


带守卫：

```scala
for(x <- Range if condition_x1; if condition_x2...; y <- Range2 if condition_y1; if condition_y2...){
    statement(s);
}
```


生成新List：

```scala
var retVal = for{x <- List if condition1; if condition2...} yield x
```


### 2.2.3. 不支持continue、break ###

```scala
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
```


## 3.1. 方法和函数 ##

Scala中，方法是类的一部分，而函数是一个对象可以赋值给一个变量。
Scala的方法跟Java的类似，方法是组成类的一部分。
Scala的函数则是一个完整的对象，Scala的函数其实就是继承了Trait的类的对象。

Scala中使用`def`语句定义方法，`val`语句可以定义函数。

```scala
class Test{
    def m(x: Int) = x + 3
    val f = (x: Int) => x + 3
}
```

- 如果方法不写等号和主体，那么该方法会被隐式声明为抽象，包含它的类型也是一个抽象类型。
- 定义函数时，可以指定默认参数。
- 调用函数时，可以指定函数参数名传参，不一定要按顺序传参。
- 可变参数，在参数的类型之后放一个星号来设置，`def printStrings(args: String*)`。
- 函数中可以嵌套函数。
- 在Scala中无法直接操作方法，如果要操作方法，必须先将其转换成函数。方法转函数：`val f1 = m _`。
- 如果函数不带参数，可以不写括号。


### 3.1.1. 函数传值/传名调用 ###

Scala的解释器在解析函数参数(function arguments)时有两种方式：

- 传值调用（call-by-value）：先计算参数表达式的值，再应用到函数内部。
- 传名调用（call-by-name）：将未计算的参数表达式直接应用到函数内部。

在进入函数内部前，传值调用方式就已经将参数表达式的值计算完毕，而传名调用是在函数内部进行参数表达式的值计算的。
这就造成了一种现象，每次使用传名调用时，解释器都会计算一次表达式的值。

```scala
object Test {
    def main(args: Array[String]) {
        delayed(time());
    }

    def time() = {
        println("获取时间，单位为纳秒")
        System.nanoTime
    }
    def delayed(t: => Long) = {
        println("在 delayed 方法内")
        println("参数： " + t) //计算t，打印"获取时间，单位为纳秒"
        t //计算t，打印"获取时间，单位为纳秒"
   }
}
```

以上实例中我们声明的delayed方法，在变量名和变量类型使用`=>`符号来设置传名调用。


### 3.1.2. 高阶函数 ###

高阶函数可以使用其他函数作为参数，或者使用函数作为输出结果。

```scala
object Test {
    def main(args: Array[String]) {
        println( apply( layout, 10) )
    }
    
    // 函数 f 和 值 v 作为参数，而函数 f 又调用了参数 v
    def apply(f: Int => String, v: Int) = f(v)

    def layout[A](x: A) = "[" + x.toString() + "]"
}
```

`f: Int => String`中`Int`为入参类型，`String`为返回值类型。


### 3.1.3. 匿名函数 ###

箭头左边是参数列表，右边是函数体。

```scala
var inc = (x: Int) => x + 1

var userDir = () => { System.getProperty("user.dir") }
```

**总结一下：**

|||
|---|---|
|**t: => Long**|参数声明，传名调用|
|**f: Int => String**|参数声明，传函数|
|**(x: Int) => x + 1**|定义匿名函数|


### 3.1.4. 偏应用函数 ###

不需要提供函数需要的所有参数，只需要提供部分，或不提供所需参数。

```scala
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
```

`log()`方法接收两个参数：`date`和`message`。我们在程序执行时调用了三次，参数`date`值都相同，`message`不同。我们可以使用偏应用函数优化以上方法，绑定第一个`date`参数，第二个参数使用下划线(_)替换缺失的参数列表，并把这个新的函数值的索引的赋给变量。


### 3.1.5 函数柯里化 ###

柯里化（Currying）指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数为参数的函数。

首先定义一个函数:

```scala
def add(x: Int, y: Int) = x + y
```

柯里化：

```scala
def add(x: Int)(y: Int) = x + y
```

实质上最先演变成这样一个方法：

```scala
def add(x: Int) = (y: Int) => x + y
```


### 3.1.6 闭包 ###

闭包是一个函数，返回值依赖于声明在函数外部的一个或多个变量。
闭包通常来讲可以简单的认为是可以访问一个函数里面局部变量的另外一个函数。

```scala
object Test {
    def main(args: Array[String]) {
        println( "muliplier(1) value = " +  multiplier(1) )
        println( "muliplier(2) value = " +  multiplier(2) )
    }
    var factor = 3
    val multiplier = (i:Int) => i * factor
}
```

函数变量`multiplier`成为一个“闭包”，因为它引用到函数外面定义的变量，定义这个函数的过程是将这个自由变量捕获而构成一个封闭的函数。


## 4.1. 数组 ##

```scala
var z: Array[String] = new Array[String](3)
var z = new Array[String](3)
var z = Array("aaa", "bbb", "ccc")
```

创建多维数组：

```scala
var myMatrix = ofDim[Int](3,3)
```

import Array._ 引入包
- `def ofDim[T]( n1: Int ): Array[T]`，创建（指定维度长度）的一维数组
- `def ofDim[T]( n1: Int, n2: Int ): Array[Array[T]]`，创建二维数组
- `def ofDim[T]( n1: Int, n2: Int, n3: Int ): Array[Array[Array[T]]]`，创建三维数组


## 4.2. 集合 ##

### 4.2.1 List ###

列表是不可变的，值一旦被定义了就不能改变。
列表具有链表结构。

```scala
val site: List[String] = List("Runoob", "Google", "Baidu")
// 空列表
val empty: List[Nothing] = List()
```

在Scala中，列表要么是Nil（及空表），要么是一个head元素加上一个tail，而tail是一个列表（或者一个init加上一个last元素，而init是一个列表）

- `head`返回列表第一个元素
- `tail`返回一个列表，包含除了第一元素之外的其他元素
- `isEmpty`在列表为空时返回`true`

|||
|---|---|
|`::`|连接元素至列表|
|`:::`|连接列表至列表|
|`def +:(elem: A): List[A]`|元素加列表前面，生成新列表|
|`def :+(elem: A): List[A]`|元素加列表后面，生成新列表|
|`List.fill()`|创建重复元素的列表，例如`val site = List.fill(3)("Runoob")`|
|`List.tabulate()`|通过给定函数来创建列表，例如`val l = List.tabulate(6)(n => n * n)`|
|`def apply(n: Int): A`|通过列表索引获取元素，同`site(3)`|
|`def drop(n: Int): List[A]`|丢弃前n个元素，并返回新列表|
|`def dropRight(n: Int): List[A]`|丢弃最后n个元素，并返回新列表|
|`def exists(p: (A) => Boolean): Boolean`|某元素是否存在，例如`site.exists(s => s == "Baidu")`|
|`def filter(p: (A) => Boolean): List[A]`|输出符号指定条件的所有元素，例如`site.filter(s => s.length == 3)`|
|`def forall(p: (A) => Boolean): Boolean`|检测所有元素，例如`site.forall(s => s.startsWith("H"))`|
|`def foreach(f: (A) => Unit): Unit`|将函数应用到列表的所有元素|


### 4.2.2. Set ###

集合分为可变的和不可变的集合。
默认情况下，Scala使用的是scala.collection.immutable.Set，不可变集合。
如果想使用可变集合，需要引用scala.collection.mutable.Set。

```scala
val set = Set(1,2,3)
println(set.getClass.getName) // 

println(set.exists(_ % 2 == 0)) //true
println(set.drop(1)) //Set(2,3)


import scala.collection.mutable.Set // 可以在任何地方引入 可变集合

val mutableSet = Set(1,2,3)
println(mutableSet.getClass.getName) // scala.collection.mutable.HashSet

mutableSet.add(4)
mutableSet.remove(1)
mutableSet += 5
mutableSet -= 2

println(mutableSet) // Set(5, 3, 4)

val another = mutableSet.toSet
println(another.getClass.getName) // scala.collection.immutable.Set
```

虽然可变Set和不可变Set都有添加或删除元素的操作，但是有一个非常大的差别。
对不可变Set进行操作，会产生一个新的set，原来的set并没有改变，这与List一样。 
而对可变Set进行操作，改变的是该Set本身，与ListBuffer类似。


### 4.2.3. Map ###

映射分为可变的和不可变的映射。
默认情况下为不可变映射。
如果想使用可变映射，需要引用scala.collection.mutable.Map。

```scala
// 空哈希表，键为字符串，值为整型
var A:Map[Char,Int] = Map()

// Map 键值对演示
val colors = Map("red" -> "#FF0000", "azure" -> "#F0FFFF")
```


### 4.2.4. Tuple ###

与列表一样，元组也是不可变的

```scala
object Test {
    def main(args: Array[String]) {
        val t = (4,3,2,1)
        val t2 = new Tuple3(1, 3.14, "Fred")

        val sum = t._1 + t._2 + t._3 + t._4

        println( "元素之和为: "  + sum )
   }
}
```

`A->B`,`->`方法调用的结果是返回一个二元的元组`(A,B)`。


### 4.2.5. Option ###

Option（选项）类型用来表示一个值是可选的（有值或无值)。
`Option[T]`是一个类型为`T`的可选值的容器：如果值存在，`Option[T]`就是一个`Some[T]`；如果不存在，`Option[T]`就是对象`None`。

```scala
object Test {
    def main(args: Array[String]) {
        val sites = Map("runoob" -> "www.runoob.com", "google" -> "www.google.com")

    println("show(sites.get( \"runoob\")) : " + show(sites.get( "runoob")) )
    println("show(sites.get( \"baidu\")) : " + show(sites.get( "baidu")) )
    }

    def show(x: Option[String]) = x match {
        case Some(s) => s
        case None => "?"
    }
}
```

可以使用 isEmpty() 方法来检测元组中的元素是否为 None

```scala
object Test {
    def main(args: Array[String]) {
        val a:Option[Int] = Some(5)
        val b:Option[Int] = None 

        println("a.isEmpty: " + a.isEmpty)  // false
        println("b.isEmpty: " + b.isEmpty)  // true
   }
}
```


### 4.2.6. 迭代器 ###

```scala
object Test {
    def main(args: Array[String]) {
        val it = Iterator("Baidu", "Google", "Runoob", "Taobao")

        while (it.hasNext){
            println(it.next())
        }
    }
}
```


## 5.1 继承 ##

```scala
import java.io._

class Point(val xc: Int, val yc: Int) {
    var x: Int = xc
    var y: Int = yc
    def move(dx: Int, dy: Int) {
        x = x + dx
        y = y + dy
        println ("x 的坐标点 : " + x);
        println ("y 的坐标点 : " + y);
    }
}

class Location(override val xc: Int, override val yc: Int,
    val zc :Int) extends Point(xc, yc){
    var z: Int = zc

    def move(dx: Int, dy: Int, dz: Int) {
        x = x + dx
        y = y + dy
        z = z + dz
        println ("x 的坐标点 : " + x);
        println ("y 的坐标点 : " + y);
        println ("z 的坐标点 : " + z);
    }
}

object Test {
    def main(args: Array[String]) {
        val loc = new Location(10, 20, 15);
        // 移到一个新的位置
        loc.move(10, 10, 5);
    }
}
```

- 类定义可以有参数，称为类参数，如上面的`xc`,`yc`。类参数在整个类中都可以访问。
- 子类只有主构造函数才可以往父类的构造函数里写参数。
- 子类重写一个非抽象方法必须使用override修饰符；重写抽象方法时不需要。


## 5.2 单例/伴生对象 ##

Scala不能定义静态成员，而是定义单例对象。以`object`关键字定义。 
对象定义了某个类的单个实例，包含了“静态”特性：

```scala
object Accounts {
    private var lastNumber = 0
    def newUniqueNumber() = {lastNumber += 1; lastNumber}
}
```

当单例对象与某个类共享同一个名称时，它就被称为是这个类的伴生对象。
类被称为是这个单例对象的伴生类。
类和它的伴生对象必须定义在同一个源文件中。
类和它的伴生对象可以互相访问其私有成员。

```scala
class Account {
    val id = Account.newUniqueNumber()
    private var balance = 0.0
    def deposit(amount: Double){ balance += amount }
    ...
}

object Account {
    private var lastNumber = 0
    def newUniqueNumber() = { lastNumber += 1; lastNumber}
}
```

- 类的伴生对象可以被访问，但并不在作用域当中。`Account`类必须通过`Account.newUniqueNumber()`来调用伴生对象的方法。


## 5.3. 特征 ##

相当于抽象类。

```scala
trait Equal {
    def isEqual(x: Any): Boolean
    def isNotEqual(x: Any): Boolean = !isEqual(x)
}
```

通过`with`关键字，一个类可以扩展多个特质：

```scala
class BMW extends Car with Shiny {
    val brand = "BMW"
    val shineRefraction = 12
}
```


## 5.4. 匹配 ##

### 5.4.1. 选择器 ###

```scala
object Test {
    def main(args: Array[String]) {
        println(matchTest("two"))   # 2
        println(matchTest("test"))  # many
        println(matchTest(1))       # one
        println(matchTest(6))       # scala.Int

    }
    def matchTest(x: Any): Any = x match {
        case 1 => "one"
        case "two" => 2
        case y: Int => "scala.Int"
        case _ => "many"
    }
}
```

### 5.4.2. 样例类 ###

使用了`case`关键字的类定义就是就是样例类。
样例类是种特殊的类，经过优化以用于模式匹配。

```scala
object Test {
    def main(args: Array[String]) {
        val alice = new Person("Alice", 25)
        val bob = new Person("Bob", 32)
        val charlie = new Person("Charlie", 32)
   
        for (person <- List(alice, bob, charlie)) {
            person match {
                case Person("Alice", 25) => println("Hi Alice!")
                case Person("Bob", 32) => println("Hi Bob!")
                case Person(name, age) => println("Age: " + age + " year, name: " + name + "?")
            }
        }
    }
   
    // 样例类
    case class Person(name: String, age: Int)
}
```


