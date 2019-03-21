title: Java - File IO示例
date: 2019/03/21
categories:
- Program Development
tags:
- Java
---


```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class FileIoDemo
{

    public static void main(String[] args)
    {
        // 先声明文件对象
        File file = new File("D:" + File.separator + "out.txt");
        try
        {
            System.out.println("CanonicalPath = " + file.getCanonicalPath());
        }
        catch(IOException e1)
        {
            e1.printStackTrace();
        }  

        // FileWriter外面套一层buffer
        try(BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter(file)))
        {
            bufferedWriter.write("line1\n");
            bufferedWriter.write("line2\n");
            bufferedWriter.write("line3\n");
            bufferedWriter.write("line4\n");
            bufferedWriter.write("line5\n");
        }
        catch(IOException e)
        {
            e.printStackTrace();
        }
        
        // FileReader外面套一层buffer
        try(BufferedReader bufferedReader = new BufferedReader(new FileReader(file)))
        {
            String curLine = null;
            while((curLine = bufferedReader.readLine()) != null)
            {
                System.out.println(curLine);
            }
        }
        catch(IOException e)
        {
            e.printStackTrace();
        }
    }
}
```
