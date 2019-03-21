title: Java - 反射示例
date: 2019/03/21
categories:
- Program Development
tags:
- Java
---


```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * Reflection机制允许程序在正在执行的过程中，利用Reflection APIs取得任何“已知名称”的类的内部信息，并动态生成instances、变更fields内容、调用methods等。     
 */
public class ReflectionDemo
{
    private int myInt;
    private String myString;
    
    public ReflectionDemo()
    {
        this.myInt = 0;
        this.myString = "zzz";
    }
    
    public ReflectionDemo(int myInt, String myString)
    {
        this.myInt = myInt;
        this.myString = myString;
    }
    
    public void printMyInt()
    {
        System.out.println(myInt);
    }
    
    public boolean isMyString(String s)
    {
        return myString.equals(s);
    }
    
    @Override
    public String toString()
    {
        return "[" + myInt + ", " + myString + "]";
    }
    
    public static void main(String[] args) throws ReflectiveOperationException
    {
        // create instance
        /**
         * 几种获取Class对象的方式：
         * 1. 类名称.class
         *        Class demoClass = ReflectionDemo.class;
         * 2. 实例.class
         *        Class demoClass = new ReflectionDemo().getClass();
         * 3. Class.forName
         *        Class demoClass = Class.forName("ReflectionDemo");
         * 4. ClassLoader.loadClass
         *        Class demoClass = ClassLoader.getSystemClassLoader().loadClass("ReflectionDemo");
         */
        //
        Class demoClass = Class.forName("ReflectionDemo");
        
        ReflectionDemo demo1 = (ReflectionDemo) demoClass.newInstance();// <----- default constructor
        System.out.println(demo1);
        
        Constructor ctor = demoClass.getDeclaredConstructor(int.class, String.class);
        ReflectionDemo demo2 = (ReflectionDemo) ctor.newInstance(666, "hahaha");// <----- constructor with arguments
        System.out.println(demo2);
        
        // change field
        Field field = demoClass.getDeclaredField("myInt");
        field.setAccessible(true);
        field.set(demo1, 999);
        
        // invoke method
        Method method1 = demoClass.getDeclaredMethod("printMyInt");
        method1.invoke(demo1);
        method1.invoke(demo2);
        Method method2 = demoClass.getDeclaredMethod("isMyString", String.class);
        System.out.println(method2.invoke(demo1, "hahaha"));
        System.out.println(method2.invoke(demo2, "hahaha"));
        
        // return instance for usage
        // return demo1;
        // return demo2;
    }
}
```
