title: Python Cheatsheet
date: 2018/10/08
categories:
- Program Development
tags:
- Python
---


## 1.1. 标识符 ##

- 标识符由字母、数字、下划线组成，不能以数字开头。
- 以下划线开头的标识符是有特殊意义的。
  - 以单下划线开头`_foo`的代表不能直接访问的类属性protected，需通过类提供的接口进行访问，不能用`from xxx import *`而导入。
  - 以双下划线开头的`__foo`代表类的私有成员private。
  - 以双下划线开头和结尾的`__foo__`代表Python里特殊方法专用的标识，如`__init__()`代表类的构造函数。


## 1.2. 变量 ##

- Python 中的变量赋值不需要类型声明。每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。
- 多变量赋值
  - `a = b = c = 1`
  - `a, b, c = 1, 2, "john"`
- 删除对象的引用
  - `del var_a, var_b`


## 1.3. 标准数据类型 ##

### Number数字 ###

- 整型 int，例如：`10`。
- 长整型 long，例如：`10L`。
  - `long`类型只存在于Python2.X版本中，`int`型数据溢出后会自动转为`long`型。在Python3.X版本中`long`类型被移除，使用`int`替代。
- 浮点型 float，例如：`10.0`。
- 复数型 complex，例如：`10+3.14j`。
  -  复数的实部和虚部都是浮点型


### String字符串 ###

- 引号
  - Python 可以使用引号`'`、双引号`"`、三引号`'''`或`"""`来表示字符串，引号的开始与结束必须的相同类型的。
  - 其中三引号可以由多行组成（字符串中换行）。
  - 原始字符串：例如`r"\n"`，不对字符串进行转义。
- 注释
  - 单行注释采用`#`开头。
  - 多行注释使用三个单引号`'''`或三个双引号`"""`。
  - `# -*- coding: UTF-8 -*-`。
- 截取与拼接
  - 索引下标：从左到右索引从0开始，最大范围是n-1；从右到左索引从-1开始，最大范围-n。
  - 使用`[头下标:尾下标]`来截取相应的字符串，截取区间左闭右开[)。
  - 使用加号`+`来拼接字符串，使用乘号`*`来生成重复拼接的字符串。
- 格式化
  - 例如：`a = "%d, %s" % (111, "test")`。


### List列表 ###

- 用`[]`标识，元素类型可以不同，支持列表嵌套。
- 支持截取、拼接，参考字符串截取。
- 用`[下标]`取值。


### Tuple元组 ###

- 用`()`标识，元组不能二次赋值（指定下标赋值），相当于只读列表。
- 新建元组中只包含一个元素时，需要在元素后面添加逗号`tup1 = (50,)`。


### Dictionary字典 ###

- 用`{}`标识，key-value结构。
- 用`[key]`取值。
- 字典值可以没有限制地取任何python对象，既可以是标准的对象，也可以是用户定义的。但键必须不可变（可以用数字、字符串或元组充当，用列表就不行）。


## 1.4. 运算符 ##

- 算术运算符：`+`、`-`、`*`、`/`、次幂`**`、取模`%`、地板除`//`（即处理浮点时，`1.//2=0.0`）。
  - `=`及运算符与`=`连用。
  - 无`++`，因为在 Python 里的数值和字符串之类的都是不可变对象，对不可变对象操作的结果都会生成一个新的对象，而不是把某内存地址数据值增加 。
- 位运算符：`&`、`|`、`^`、`~`、`<<`、`>>`。
- 比较运算符：`==`、`!=`、另一种不等于`<>`、`>`、`<`、`>=`、`<=`。
- 逻辑运算符：`and`、`or`、`not`。
  - 可加括号，表达更清楚。
- 成员运算符：`in`、`not in`。
  - 字符串，列表或元组中是否包含某元素。
- 身份运算符：`is`、`is not`。
  - 是否引用同一内存。
  - 例如a为List，b=a与b=a[:]的区别，前者为传递引用，后者为拷贝。


## 2.1. 缩进 ##

Python的代码块不使用大括号`{}`来控制类，函数以及其他逻辑判断。Python最具特色的就是**用缩进来写模块**。


## 2.2. 条件控制 ##

    if expression : 
       suite 
    elif expression :  
       suite  
    else :  
       suite


## 2.3. 循环 ##

- 支持`continue`、`break`。
- `pass`为空语句，占位语句。


### while循环 ###

    while expression : 
       suite 
    else :  
       suite

- `else`为非`break`退出循环的分支。


### for循环 ###

    for iterating_var in sequence:
       suite
    else :  
       suite

- 直接迭代元素：`for letter in 'Python'`；或迭代下标：`for index in range(0， 10)`。
- 使用`for index, item in enumerate(sequence)`，同时获取下标和值。


## 2.4. 函数 ##

    def functionname( parameters ):
       "函数_文档字符串"
       function_suite
       return [expression]

- 参数传递
  - 数字、字符串、元组是不可变类型，而列表、字典等则是可变类型。参数传递时，不可变类型的对象传值，可变类型的对象传引用。
  - 默认参数，例如`def printinfo( name, age = 35 ):`。
  - 不定长参数，加了星号`*`的变量名会存放所有未命名的变量参数，例如`def printinfo( arg1, *vartuple ):`。
- 全局变量想作用于函数内，需加关键字`global`。
- 匿名函数lambda
  - `lambda [arg1 [,arg2,.....argn]]:expression`。
  - lambda函数拥有自己的命名空间，且不能访问自有参数列表之外或全局命名空间里的参数。


## 2.5. 异常处理 ##

	raise [Exception [, args [, traceback]]]
    try:
    	<语句>
    except：
    	<语句>
    else:
    	<语句>
    finally:
    	<语句>


## 2.6. 模块 ##

- 模块导入
  - `import module1`，使用函数：`module1.func1()`。
  - `from module1 import func1`，使用函数：`func1()`。
  - `from module1 import *`，使用函数：`func1()`。


## 3.1. 面向对象 ##

### 定义类 ###

    #!/usr/bin/python
    # -*- coding: UTF-8 -*-
     
    class Employee:
    	'所有员工的基类'
    	empCount = 0
    
    	def __init__(self, name, salary):
    		self.name = name
    		self.salary = salary
    		Employee.empCount += 1
    
    	def displayCount(self):
    		print "Total Employee %d" % Employee.empCount
    
    	def displayEmployee(self):
    		print "Name : ", self.name,  ", Salary: ", self.salary

- `empCount`变量是一个类变量，它的值将在这个类的所有实例之间共享。在内部类或外部类使用`Employee.empCount`访问。
- 方法`__init__()`是构造函数。
- `self`代表类的实例，`self`在定义类的方法时是必须有的，但在调用时不必传入相应的参数。
  - 类的方法与普通的函数只有一个特别的区别——它们必须有一个额外的第一个参数名称, 按照惯例它的名称是`self`。


### 使用实例 ###

    emp1 = Employee("Zara", 2000)
    emp1.displayEmployee()
    print "Total Employee %d" % Employee.empCount

- 无`new`关键字，使用的是构造函数`__init__`。
- 用点号`.`访问属性、方法。


### 继承 ###

    class SubClassName (ParentClass1[, ParentClass2, ...]):
        ...

- 在调用基类的方法时，需要加上基类的类名前缀，且需要带上`self`参数变量。
- 调用方法时，先在本类中查找调用的方法，找不到才去基类中逐个查找。

