title: Java - 正则表达式示例
date: 2019/03/22
categories:
- Program Development
tags:
- Java
---


```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegExDemo
{
    public static void main(String[] args)
    {
        String content = "this is a test";
        
        // 整句：.*is\s+.*
        String pattern1 = ".*is\\s+.*";
        // 空白符：\s
        String pattern2 = "\\s";
        // 部分：is
        String pattern3 = "is";
        
        
        // --------------- Pattern ---------------
        // 快速是否匹配
        boolean isMatch = Pattern.matches(pattern1, content);
        System.out.println(isMatch);
        
        Pattern p1 = Pattern.compile(pattern1);
        Pattern p2 = Pattern.compile(pattern2);
        Pattern p3 = Pattern.compile(pattern3);
        
        // 分割字符串
        String[] fields = p2.split(content);
        for(String s : fields)
        {
            System.out.println(s);
        }
        
        // 获取Matcher
        Matcher m1 = p1.matcher(content);
        Matcher m2 = p2.matcher(content);
        Matcher m3 = p3.matcher(content);
        
        
        // --------------- Matcher ---------------
        // 匹配整句 matches
        isMatch = m1.matches();
        System.out.println(isMatch);
        isMatch = m2.matches();
        System.out.println(isMatch);
        
        // 匹配任意部分 find。如想匹配多次，则反复调用
        isMatch = m3.find();
        System.out.println(isMatch);
        // 匹配到的子串，在整句的[)idx及内容
        System.out.println(m3.start() + ", " + m3.end() + ", " + m3.group());
        
        /* 分组操作，捕获组是通过从左至右数括号的方式来编号。例如，表达式((A)(B(C)))有四个组：
         * ((A)(B(C))), (A), (B(C)), (C)*/
        Pattern pg = Pattern.compile("([a-z]+)(\\s+)"); 
        Matcher mg = pg.matcher(content);
        mg.find();
        // group(0)总是代表整个表达式。该组不包括在 groupCount的返回值中。
        for(int i = 1; i <= mg.groupCount(); ++i)
        {
            System.out.println(mg.start(i) + ", " + mg.end(i) + ", " + mg.group(i));
        }
    }
}

/*
 * \        转义
 * ^        字符串开始
 * $        字符串结尾
 * .        除换行符外所有字符
 * 
 * *        0次以上，{0,}
 * +        1次以上，{1,}
 * ?        0次或1次，{0,1}
 * {n}      n次
 * {n,}     n次以上
 * {n,m}    至少n次，至多m次
 * ?        当此字符紧随任何其他限定符（*、+、?、{n}、{n,}、{n,m}）之后时，匹配模式是“非贪心的”最小匹配。
 *          例如，在字符串“oooo”中，o+匹配所有“o”，而o+?只匹配单个“o”。
 *          
 * x|y      匹配x或y
 * [xyz]    字符集合。匹配所包含的任意一个字符。[a-z]任意字符。[0-9]任意数字。
 * [^xyz]   负值字符集合。匹配未包含的任意一个字符。[^a-z]任意非字符。[^0-9]任意非数字。
 * (p)      匹配pattern并获取这一匹配。
 * (?:p)    匹配pattern但不获取匹配结果。
 * 
 * \b       匹配一个单词边界，也就是指单词和空格间的位置。
 *          例如，er\b可以匹配“never”中的“er”，但不能匹配“verb”中的“er”。
 * \B       匹配非单词边界。
 * \w       任意英文字符。等价于[a-zA-Z_0-9]。
 * \W       任意非英文字符。等价于[^a-zA-Z_0-9]。
 * \d       匹配一个数字。[0-9]
 * \D       匹配一个非数字。[^0-9]
 * \s       匹配任何不可见字符，包括空格、制表符、换页符等等。等价于[ \f\n\r\t\v]。
 * \S       匹配任何可见字符。等价于[^ \f\n\r\t\v]。
 */
```
