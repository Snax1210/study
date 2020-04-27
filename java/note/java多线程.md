# **JAVA多线程**
<!-- TOC -->

- [**JAVA多线程**](#java多线程)
    - [多线程](#多线程)
        - [多线程的应用场景](#多线程的应用场景)
    - [java并发包](#java并发包)
        - [Vector和ArrayList的区别：](#vector和arraylist的区别)
        - [HashTable和HashMap的区别](#hashtable和hashmap的区别)
        - [ConcurrentHashMap原理分析](#concurrenthashmap原理分析)
    - [CountDownLatch ---计数器](#countdownlatch----计数器)
        - [CyclicBarrier --- 计数](#cyclicbarrier-----计数)
        - [Semaphore](#semaphore)
        - [并发队列](#并发队列)

<!-- /TOC -->

## 多线程
使用多线程提高程序执行效率，
每个线程互不影响，因为都在自己独立运行

什么是进程，进程就是正在运行的程序，它是线程的集合

什么是线程，线程是操作系统能够进行运算调度的最小单位

什么是多线程？就是为了提高程序的执行效率的方式

==注意：多线程并不是提高速度而是提高效率==

多线程不是交替执行，而是同一时刻同时进行

### 多线程的应用场景
多线程下载、Ajax异步上传、分布式job（需要同时执行多个任务调度）、归根结底多线程提升程序的效率。

多线程的创建形式<br>

1. 使用继承Thread类方式（继承Thread类重写run方法，run方法是就是线程需要执行的任务或者执行的代码）

2. 使用实现runnable接口方式

3. 使用匿名内部类方式

4. callable

5. 使用线程池创建线程

使用开启多线程之后，代码不会从上往下进行而是交替进行（异步执行）

使用继承方式创建线程好还是实现接口好？

使用接口来说，一般来说开发都是以面向接口编程

守护线程：和主线程一起销毁
t.setDaemon(true);
非守护线程：和主线程互不影响

多线程的运行状态</br>
新建new Thread()</br>
准备 start()</br>
运行 cpu开始执行run方法</br>
阻塞 sleep()方法或者wait方法 从运行状态到休眠状态 当时间到期不会立马到运行状态而是到就绪状态</br>
停止

什么是线程安全问题？  
保证在多个线程之间共享同一个全局变量或者静态变量，保证数据一致性，和原子性

线程同步有哪些方式？  
synchronized、lock、volatile类型的变量、原子变量 

线程同步提高效率吗？  
降低了程序效率、阻塞、抢锁的资源、效率并不高

## java并发包

原子类、lock锁、并发包。  
并发类。  

线程安全的类：  
vector、

### Vector和ArrayList的区别：

实现原理都是通过数组实现--查询速度快；增加、修改、删除、速度慢  
区别：线程安全问题
Vector线程安全(上锁的集合synchronized)、ArrayList线程不安全
ArrayList效率高。

### HashTable和HashMap的区别

实现原理都是链表(增加、修改、删除快)+数组实现的  put  通过HashCode取模得到下标位置 一致性取模算法
HashTable是线程安全的(synchronized关键字)  
HashMap线程不安全  
Collections.synchronizedMap();将不安全的集合转换成安全的集合

### ConcurrentHashMap原理分析

始于jdk1.5  
分段锁，怎么进行分段？16段  
将一个整体拆分成多个小的HashTable()，默认分成16段

某一个ConcurrentHashMap长度为12，分成3段  
第一段：0-3、第二段4-7、第三段8-11 分成了三个HashTable  
当某一多线程：线程1查询下标2、线程2查询下标5、线程3查询下标10时  
没有用到同一个同步锁，因为查询的是不同的HashTable提高了效率  
当某一个线程：线程1查询下标2、线程2查询下标3   
用到同一个同步锁，还是会等待  
但是总体来说，减少了锁的资源竞争

ConcurrentMap接口下有两个重要的实现：
    ConcurrentHashMap；
    ConcurrentskipListMap(支持并发排序功能。弥补ConcurrentHashMap)
    ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的HashTable，他们有自己的锁，只要多个修改操作发生在不同的段上，他们就可以并发进行。把一个整体分成了16个段(Segment，也就是最高支持16个线程的并发修改操作。)这也是在多线程场景时减小锁的粒度从而降低锁竞争的一种方案。并且代码中大多共享变量使用volatile关键字声明，目的是第一时间获取修改的内容，性能非常好。


## CountDownLatch ---计数器

始于jdk1.5并发包 位于java.util.concurrent包下，利用它可以实现类似于计数器的功能。比如有一个任务A，他要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

```java
import java.util.Map;
import java.util.concurrent.CountDownLatch;

/**
 * @description: 描述
 * <br>
 * @date: 2020/4/7 11:47
 * @author: m00000061/maotianhang
 * @version: SmartCityManager V1.0
 * @since: JDK 1.8
 */
public class TestCountDownLatch
{
    public static void main(String[] args) throws InterruptedException
    {
        final CountDownLatch latch = new CountDownLatch(2);
        System.out.println("我是主线程");
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                System.out.println("我是子线程开始执行任务..........................");
                try
                {
                    Thread.sleep(10);
                }
                catch (Exception e)
                {
                    e.printStackTrace();
                }
                latch.countDown();
                System.out.println("我是子线程开始执行任务..........................");
            }
        }).start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                System.out.println("我是子线程开始执行任务..........................");
                try
                {
                    Thread.sleep(10);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                latch.countDown();
                System.out.println("我是子线程开始执行任务..........................");
            }
        }).start();
        latch.await();
        System.out.println("主线程开始执行任务");//如果不为0的时候，一直等待
        for (int i = 0; i < 10; i++)
        {
            System.out.println("main i: " +i);
        }
    }
}

```

结果：

    我是主线程
    我是子线程开始执行任务..........................
    我是子线程开始执行任务..........................
    我是子线程开始执行任务..........................
    我是子线程开始执行任务..........................
    主线程开始执行任务
    main i: 0
    main i: 1
    main i: 2
    main i: 3
    main i: 4
    main i: 5
    main i: 6
    main i: 7
    main i: 8
    main i: 9

### CyclicBarrier --- 计数

CyclicBarrier初始化时规定一个数目，然后计算调用了CyclicBarrier.await()进入等待的线程数。当线程数达到了这个数目时，所有进入等待状态的线程被唤醒并继续。  
CyclicBarrier就像它名字的意思一样，可看成是个障碍，所有的线程必须到齐后才能一起通过这个障碍。  
CyclicBarrier初始时还可带一个Runnable的参数，此Runnable任务在CyclicBarrier的数目达到后，所有其他线程被唤醒前被执行

```java
import java.util.concurrent.CyclicBarrier;

/**
 * @description: 描述
 * <br>
 * @date: 2020/4/7 14:31
 * @author: m00000061/maotianhang
 * @version: SmartCityManager V1.0
 * @since: JDK 1.8
 */
public class Test3
{
    public static void main(String[] args)
    {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
        for (int i = 0; i < 5; i++)
        {
            new Writer(cyclicBarrier).start();
        }
    }
}
class Writer extends Thread
{
    CyclicBarrier cyclicBarrier;
    public Writer(CyclicBarrier cyclicBarrier)
    {
        this.cyclicBarrier = cyclicBarrier;
    }
    @Override
    public void run()
    {
        System.out.println(Thread.currentThread().getName()+"开始写入数据....");
        try
        {
            Thread.sleep(30);
            //并行执行成功
            System.out.println(Thread.currentThread().getName()+"写入数据成功....");
            cyclicBarrier.await();
            System.out.println(Thread.currentThread().getName()+"所有数据执行完毕....");
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}

```

结果：

    Thread-0开始写入数据....
    Thread-1开始写入数据....
    Thread-2开始写入数据....
    Thread-3开始写入数据....
    Thread-4开始写入数据....
    Thread-2写入数据成功....
    Thread-2所有数据执行完毕....
    Thread-1写入数据成功....
    Thread-1所有数据执行完毕....
    Thread-0写入数据成功....
    Thread-0所有数据执行完毕....
    Thread-4写入数据成功....
    Thread-4所有数据执行完毕....
    Thread-3写入数据成功....
    Thread-3所有数据执行完毕....

### Semaphore

Semaphore是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。Semaphore可以用来构建一些线程池，资源池之类的，比如数据库线程池，我们也可以创建计数为1的Semaphore，将其作为一种类似互斥锁的机制，这也叫二元信号量，表示两种互斥状态。它的用法如下：

    availablePermits函数用来获取当前可用的资源数量
    wc.acquire();//申请资源
    wc.release();//释放资源

```java
package com.example.demo;

import java.util.Random;

import java.util.concurrent.Semaphore;


class Parent extends Thread
{

    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see Thread#run()
     */
    Semaphore wc;

    String name;

    public Parent(Semaphore wc, String name)
    {
        this.wc = wc;
        this.name = name;
    }

    @Override
    public void run()
    {
        //获取到资源，减去1
        int availablePermits = wc.availablePermits();
        if (availablePermits > 0)
        {
            System.out.println(name + "天助我也");
        }
        else
        {
            System.out.println(name + "nope!!!!");
        }
        try
        {
            //获取到资源权限
            wc.acquire();
            System.out.println(name + "爽");
            Thread.sleep(new Random().nextInt(1000));
            System.out.println(name + "完");
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            //释放
            wc.release();
        }
    }
}

public class SemaphoreTest
{
    public static void main(String[] args) throws InterruptedException
    {
        //        Semaphore semaphore = new Semaphore(5);
        //        //获取到资源，减去1
        //        semaphore.availablePermits();
        //        //获取到资源权限
        //        semaphore.acquire();
        //        //释放资源
        //        semaphore.release();
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 10; i++)
        {
            new Parent(semaphore, "第" + i + "个").start();
        }
    }
}
```

### 并发队列

在并发队列上JDK提供了两套实现，一个是ConcurrentLinkedQueue为代表的高性能队列，一个是以BlockingQueue接口为代表的阻塞队列，无论哪种都继承自Queue。

在并发队列中，分为有界和无界队列，区别在于一个支持有限制，一个无限制
