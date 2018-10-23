title: Java Cheatsheet
date: 2018/10/22
categories:
- Program Development
tags:
- Java
---


## 1.1. 数据类型 ##

- 8种基本类型：
  - 4种整型：`int` (4)、`short` (2)、`long` (8, B)、`byte` (1)
  - 2种浮点：`float` (4, F)、`double` (8, D, default)
  - 1种字符：`char`
  - 1种布尔：`boolean`
- 枚举类型：`enum`
- 数组：
```java                
int[] a = new int [100];
int[] b = {0, 1, 2, 3, 4, 5}
a = new int[] {0, 2, 4, 6}      //使用匿名数组，在不创建新变量情况下，重新初始化a
// 允许运行时确定大小
int size = 10;
int[] arr = new int[size];
// 引用同一个
int[] c = a;
// 拷贝数值
int[] d = Arrays.copyOf(a, 2 * a.length);
```


## 1.2. 运算符 ##

`=` `+` `-` `*` `/` `%` `+=` `++`
`==` `!=` `>` `<` `>=` `<=` `&&` `||` `!` `?:`
`&` `|` `^` `~` `>>`（用符号位补位） `>>>`（用0补位） `<<`（用0补位）


## 2.1. 控制流程 ##
- 条件分支：

```java
if ()
{}
else if ()
{}
else
{}

switch ()
{
    case xxx:
        break;
    default:
        break;
}
```

- 循环：

```java
while ()
{}

do
{}
while ()

for （；；）
{}

for ( : )
{}
```

- 中断：

```java
break;

continue;

lable:
while ()
{
    for ( ; ; )
    {
        if ()
        {
            break lable;
        }
    }
}
// jump to here
```


## 2.2. 异常处理 ##

```java
try
{}
catch ()
{}
catch ( | )
{}
finally
{
    // if "throw" exists in catch, or the exception is not catched, execute here then throw
}
```


## 3.1. 修饰符 ##

|final||
|---|---|
|变量|常量，不可更改|
|方法|不允许覆盖|
|类|不允许继承|

|static||
|---|---|
|变量|类变量|
|方法|类方法|
|类|内部类不引用外围类对象。不加static的内部类可以引用外围类对象，但如在外围类static方法中定义的内部类，就需要不引用外围类对象。|

|||
|---|---|
|public|被任意类使用|
|private|被定义自身的类使用|
|protected|被子类或同包中的类使用|
|default|被同包中的类使用|


