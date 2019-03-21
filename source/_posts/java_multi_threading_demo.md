title: Java - 多线程示例
date: 2019/03/21
categories:
- Program Development
tags:
- Java
---


```java
/**
 * 定义一个类A实现Runnable接口；然后通过new Thread(new R())方式新建线程。
 */
public class MultiThreadingDemo implements Runnable
{
    private static final Object lock = new Object();
    
    private int tickets = 50;
    

    @Override
    public void run()
    {
        for(int i = 0; i < 10; ++i)
        {
            synchronized(lock)
            {
                --tickets;
                System.out.println(Thread.currentThread().getName() + " sell 1 ticket (" + i + "st), " + tickets + " left");
            }            
            
            try
            {
                Thread.sleep((long) Math.random() * 1000);
            }
            catch(InterruptedException e)
            {
                e.printStackTrace();
            }
        }

    }

    public static void main(String[] args)
    {
        MultiThreadingDemo demo = new MultiThreadingDemo();
        // new Thread(R)
        Thread t1 = new Thread(demo);
        // set thread name
        Thread t2 = new Thread(demo, "thr2");
        Thread t3 = new Thread(demo, "thr3");
        Thread t4 = new Thread(demo, "thr4");
        Thread t5 = new Thread(demo, "thr5");
        
        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
    }
}
```
