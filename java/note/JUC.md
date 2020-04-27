# JUC #
<!-- TOC -->

- [JUC](#juc)
    - [一、Java并发基本概念](#一java并发基本概念)
        - [1、原子性](#1原子性)
        - [2、可见性](#2可见性)
        - [3 有序性](#3-有序性)
    - [二、Volatile关键字](#二volatile关键字)
    - [三、CAS算法](#三cas算法)
        - [1. 概述](#1-概述)
            - [3.1 循环时间太长](#31-循环时间太长)
            - [3.2 只能保证一个共享变量原子操作](#32-只能保证一个共享变量原子操作)
            - [3.3 ABA问题](#33-aba问题)
                - [3.3.1 影响](#331-影响)
    - [四、CountDownLatch闭锁](#四countdownlatch闭锁)
    - [五、Condition线程通信](#五condition线程通信)
    - [六、ReadWriteLock读写锁](#六readwritelock读写锁)
        - [1. 简介](#1-简介)

<!-- /TOC -->

## 一、Java并发基本概念 ##


在并发编程中我们一般都会遇到这三个基本概念：原子性、可见性、有序性
### 1、原子性 ###
原子性：即一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

原子性就像数据库里面的事务，volatile 是无法保证复合操作的原子性。
### 2、可见性 ###

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

在多线程环境下，一个线程对共享变量的操作对其他线程是不可见的。

**Java提供了 volatile 来保证可见性**

当一个变量被 volatile 修饰后，表示着线程本地内存无效。当一个线程修改共享变量后他会立即被更新到主内存中；当其他线程读取共享变量时，它会直接从主内存中读取
### 3 有序性 ###
有序性：即程序执行的顺序按照代码的先后顺序执行

在 Java 内存模型中，为了效率是允许编译器和处理器对指令进行重排序，当然重排序它不会影响单线程的运行结果，但是对多线程会有影响。
## 二、Volatile关键字 ##


Volatile在多线程开发中保证了共享变量的“可见性”。如果一个变量使用 volatile ，则它比使用 synchronized 的成本更加低，因为它不会引起线程上下文的切换和调度

Java 编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量 

通俗点讲就是说一个变量如果用 volatile 修饰了，则 Java 可以确保所有线程看到这个变量的值是一致的。如果某个线程对 volatile 修饰的共享变量进行更新，那么其他线程可以立马看到这个更新，这就是所谓的线程可见性。



## 三、CAS算法 ##

### 1. 概述 ###

CAS ，Compare And Swap ，即比较并交换，Doug Lea 大神在实现同步组件时，大量使用CAS 技术，鬼斧神工地实现了Java 多线程的并发操作。整个 AQS 同步组件、Atomic 原子类操作等等都是基 CAS 实现的，甚至 ConcurrentHashMap 在 JDK 1.8 的版本中，也调整为 CAS + synchronized 。可以说，CAS 是整个 J.U.C 的基石  
![](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120816001.png)

 ### 2. CAS分析 ###
 在 CAS 中有三个参数：内存值 V、旧的预期值 A、要更新的值 B ，当且仅当内存值 V 的值等于旧的预期值 A 时，才会将内存值V的值修改为 B ，否则什么都不干

 J.U.C 下的 Atomic 类，都是通过 CAS 来实现的。

    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
    
    static {
    try {
    valueOffset = unsafe.objectFieldOffset
    (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
    }
    
    private volatile int value;
   
 Unsafe 是 CAS 的核心类，Java 无法直接访问底层操作系统，而是通过本地 native 方法来访问。不过尽管如此，JVM还是开了一个后门：Unsafe ，它提供了硬件级别的原子操作。


valueOffset 为变量值在内存中的偏移地址，Unsafe 就是通过偏移地址来得到数据的原值的。



value 当前值，使用 volatile 修饰，保证多线程环境下看见的是同一个。


 

    //AtomicInteger.java
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
    // Unsafe.java
    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }


    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
 ### 3. CAS缺陷 ###
 CAS 虽然高效地解决了原子操作，但是还是存在一些缺陷的，主要表现在三个方面：

循环时间太长  
只能保证一个共享变量原子操作  
ABA 问题
#### 3.1 循环时间太长 ####
如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋 CAS 长时间地不成功，则会给 CPU 带来非常大的开销。在 J.U.C 中，有些地方就限制了 CAS 自旋的次数，例如： BlockingQueue 的 SynchronousQueue 。

#### 3.2 只能保证一个共享变量原子操作 ####
看了 CAS 的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了，当然如果你有办法把多个变量整成一个变量，利用 CAS 也不错。例如读写锁中 state 的高低位。

#### 3.3 ABA问题 ####
CAS 需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是 A，变成了 B，然后又变成了 A，那么在 CAS 检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。对于 ABA 问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加 1 ，即 A —> B —> A ，变成1A —> 2B —> 3A 。

##### 3.3.1 影响 #####

用一个例子来阐述 ABA 问题所带来的影响。

有如下链表如下图：  
![](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120816002.png)

假如我们想要把 A 替换为 B ，也就是 #compareAndSet(this, A, B) 。线程 1 执行 A 替换 B 操作之前，线程 2 先执行如下动作，A 、B 出栈，然后 C、A 入栈，最终该链表如下：

![](https://gitee.com/chenssy/blog-home/raw/master/image/sijava/2018120816003.png)

完成后，线程 1 发现仍然是 A ，那么 #compareAndSet(this, A, B) 成功，但是这时会存在一个问题就是 B.next = null，因此 #compareAndSet(this, A, B) 后，会导致 C 丢失，改栈仅有一个 B 元素，平白无故把 C 给丢失了。

 ##### 3.3.2 解决方案 AtomicStampedReference #####

 CAS 的 ABA 隐患问题，解决方案则是版本号，Java 提供了 AtomicStampedReference 来解决。AtomicStampedReference 通过包装 [E,Integer] 的元组，来对对象标记版本戳 stamp ，从而避免 ABA 问题。对于上面的案例，应该线程 1 会失败。

AtomicStampedReference 的 #compareAndSet(...) 方法，代码如下


    
    public boolean compareAndSet(V   expectedReference,
     V   newReference,
     int expectedStamp,
     int newStamp) {
    Pair<V> current = pair;
    return
    expectedReference == current.reference &&
    expectedStamp == current.stamp &&
    ((newReference == current.reference &&
      newStamp == current.stamp) ||
     casPair(current, Pair.of(newReference, newStamp)));
    }
  

  有四个方法参数，分别表示：预期引用、更新后的引用、预期标志、更新后的标志。源码部分很好理解：   
1、预期的引用 == 当前引用  
2、预期的标识 == 当前标识  
3、如果更新后的引用和标志和当前的引用和标志相等，则直接返回 true  
4、通过 Pair#of(T reference, int stamp) 方法，生成一个新的 Pair 对象，与当前 Pair CAS 替换。  
Pair 为 AtomicStampedReference 的内部类，主要用于记录引用和版本戳信息（标识），定义如下：  

    // AtomicStampedReference.java
    
    private static class Pair<T> {
    final T reference; // 对象引用
    final int stamp; // 版本戳
    private Pair(T reference, int stamp) {
    this.reference = reference;
    this.stamp = stamp;
    }
    static <T> Pair<T> of(T reference, int stamp) {
    return new Pair<T>(reference, stamp);
    }
    }
    
    private volatile Pair<V> pair;

   
   

 Pair 记录了对象的引用和版本戳，版本戳为 int 型，保持自增。同时 Pair 是一个不可变对象，其所有属性全部定义为 final 。  
 对外提供一个 #of(T reference, int stamp) 方法，该方法返回一个新建的 Pair 对象。  
pair 属性，定义为 volatile ，保证多线程环境下的可见性。在AtomicStampedReference 中，大多方法都是通过调用 Pair 的 #of(T reference, int stamp) 方法，来产生一个新的 Pair 对象，然后赋值给变量 pair 。如 set(V newReference, int newStamp)方法，代码如下：

    // AtomicStampedReference.java
    public void set(V newReference, int newStamp) {
    Pair<V> current = pair;
    if (newReference != current.reference || newStamp != current.stamp)
    this.pair = Pair.of(newReference, newStamp);
    }
    
## 四、CountDownLatch闭锁 ##
CoundDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

CoundDownLatch类之中的常用方法有如下几个：

构造方法:  
` public CountDownLatch(int count);  //要设置一个等待的线程个数；`  
减少等待个数  ：  
 `public void countDown(); `  
 等待countDownLatch为0：  
`public void await() throws InterruptedException;`

```java
public class TestCountDownLatch
{
    public static void main(String[] args)
    {
        CountDownLatch latch = new CountDownLatch(5);
        LatchDemo latchDemo = new LatchDemo(latch);
        long start = System.currentTimeMillis();
        ArrayList<Thread> threadArrayList = new ArrayList<>();
        for (int i = 0; i <5 ; i++)
        {
            Thread thread = new Thread(latchDemo);
            thread.start();
            threadArrayList.add(thread);
        }
//        for (Thread t:threadArrayList)
//        {
//            try
//            {
//                t.join();
//            }
//            catch (InterruptedException e)
//            {
//                e.printStackTrace();
//            }
//        }
        try
        {
            //在latch计数器变为0前一直阻塞
            latch.await();
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();
        System.out.println("计算5遍结束时间:"+(end-start));
    }
}
class LatchDemo implements Runnable{
    private  CountDownLatch latch;

    public  LatchDemo(CountDownLatch latch){
        this.latch=latch;
    }
    @Override
    public void run()
    {
        try
        {
            for(int i=1;i<500000;i++){
                if(i%2==0){
                    System.out.println(i);
                }
            }
        }catch(Exception e){

        }finally
        {
            latch.countDown();
        }
    }
}
```
计算5遍结束时间:6387
## 五、Condition线程通信 ##
```java
 * 编写一个程序，开启 3 个线程，这三个线程的 ID 分别为 A、B、C，每个线程将自己的 ID 在屏幕上打印 10 遍，要求输出的结果必须按顺序显示。如：ABCABCABC…… 依次递归
 */
public class TestABCThread
{
    public static void main(String[] args)
    {
        ABCThreadDemo  demo=new  ABCThreadDemo();
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 1; i <=20 ; i++)
                {
                    demo.loopA(i);
                }
            }
        },"A").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 1; i <=20 ; i++)
                {
                    demo.loopB(i);
                }
            }
        },"B").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 1; i <=20 ; i++)
                {
                    demo.loopC(i);
                }
            }
        },"C").start();
    }
}

class ABCThreadDemo{

    private int number=1;

    private Lock lock=new ReentrantLock();

    private Condition condition1=lock.newCondition();

    private Condition condition2=lock.newCondition();

    private Condition condition3=lock.newCondition();

    public void loopA(int totalLoop){
        lock.lock();
        try
        {

            if (number!=1){
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName()+":"+totalLoop);
            number=2;
            condition2.signal();
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            lock.unlock();
        }
    }

    public void loopB(int totalLoop){
        lock.lock();
        try
        {
            if (number!=2){
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName()+":"+totalLoop);
            number=3;
            condition3.signal();
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            lock.unlock();
        }
    }

    public void loopC(int totalLoop){
        lock.lock();
        try
        {
            if (number!=3){
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName()+":"+totalLoop);
            System.out.println("============================");
            number=1;
            condition1.signal();
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            lock.unlock();
        }
    }

}

打印结果：...
A:16
B:16
C:16
============================
A:17
B:17
C:17
============================
A:18
B:18
C:18
============================
A:19
B:19
C:19
============================
A:20
B:20
C:20
============================
```
## 六、ReadWriteLock读写锁 ##
### 1. 简介 ###
重入锁 ReentrantLock 是排他锁，排他锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而，读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。

读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：
* 在同一时间，可以允许多个读线程同时访问。  
* 但是，在写线程访问时，所有读线程和写线程都会被阻塞

读写锁的主要特性：
* 公平性：支持公平性和非公平性。
* 重入性：支持重入。读写锁最多支持 65535 个递归写入锁和 65535 个递归读取锁。
* 锁降级：遵循获取写锁，再获取读锁，最后释放写锁的次序，如此写锁能够降级成为读锁。
 ### 2. ReadWriteLock ###
 java.util.concurrent.locks.ReadWriteLock ，读写锁接口。定义方法如下：
 ```java
 Lock readLock();

Lock writeLock();
```
* 一对方法，分别获得读锁和写锁 Lock 对象
 ### 3. ReentrantReadWriteLock ###
 java.util.concurrent.locks.ReentrantReadWriteLock ，实现 ReadWriteLock 接口，可重入的读写锁实现类。在它内部，维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 Writer 线程，读取锁可以由多个 Reader 线程同时保持。也就说说，写锁是独占的，读锁是共享的。

 读写锁 ReentrantReadWriteLock 内部维护着一对读写锁，如果要用一个变量维护多种状态，需要采用“按位切割使用”的方式来维护这个变量，将其切分为两部分：高16为表示读，低16为表示写。

分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过位运算。假如当前同步状态为S，那么：

写状态，等于 S & 0x0000FFFF（将高 16 位全部抹去）
读状态，等于 S >>> 16 (无符号补 0 右移 16 位)。

 FROM 《Java并发编程的艺术》的 「5.4 读写锁」 章节
 ![](http://static.iocoder.cn/images/JUC/%E8%AF%BB%E5%86%99%E9%94%81-01.png)
 ## 七、线程池 ##

 ![](https://upload-images.jianshu.io/upload_images/8702776-d702e9ab12c7c685?imageMogr2/auto-orient/strip|imageView2/2/w/815/format/webp)
### 1、Executor任务提交接口 ### 
java.util.concurrent.Executor ，任务的执行者接口，线程池框架中几乎所有类都直接或者间接实现 Executor 接口，它是线程池框架的基础。

Executor 提供了一种将“任务提交”与“任务执行”分离开来的机制，它仅提供了一个 #execute(Runnable command) 方法，用来执行已经提交的 Runnable 任务。代码如下：
```java
public interface Executor {

    void execute(Runnable command);
    
}
```
 ### 2 ExcutorService ### 
 java.util.concurrent.ExcutorService ，继承 Executor 接口，它是“执行者服务”接口，它是为”执行者接口 Executor “服务而存在的。准确的地说，ExecutorService 提供了”将任务提交给执行者的接口( submit 方法)”，”让执行者执行任务( invokeAll , invokeAny 方法)”的接口等等。
 ###  3 AbstractExecutorService ### 

 java.util.concurrent.AbstractExecutorService ，抽象类，实现 ExecutorService 接口，为其提供默认实现。

AbstractExecutorService 除了实现 ExecutorService 接口外，还提供了 #newTaskFor(...) 方法，返回一个 RunnableFuture 对象，在运行的时候，它将调用底层可调用任务，作为 Future 任务，它将生成可调用的结果作为其结果，并为底层任务提供取消操作。

###  4、 Executors ### 

Executors为Executor，ExecutorService，ScheduledExecutorService，ThreadFactory和Callable类提供了一些工具方法，类似于集合中的Collections类的功能。Executors方便的创建线程池。

1>newCachedThreadPool ：该线程池比较适合没有固定大小并且比较快速就能完成的小任务，它将为每个任务创建一个线程。第三个参数60L和第四个参数TimeUnit.SECONDS,好处就在于60秒内能够重用已创建的线程  
下面是Executors中的newCachedThreadPool()的源代码：
```java
publicstaticExecutorServicenewCachedThreadPool() {returnnewThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,newSynchronousQueue());      }
```
2> newFixedThreadPool使用的Thread对象的数量是有限的,如果提交的任务数量大于限制的最大线程数，那么这些任务讲排队，然后当有一个线程的任务结束之后，将会根据调度策略继续等待执行下一个任务。下面是Executors中的newFixedThreadPool()的源代码：
```java
publicstaticExecutorServicenewFixedThreadPool(intnThreads) {returnnewThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,newLinkedBlockingQueue());    }
```
3>newSingleThreadExecutor就是线程数量为1的FixedThreadPool,如果提交了多个任务，那么这些任务将会排队，每个任务都会在下一个任务开始之前运行结束，所有的任务将会使用相同的线程。下面是Executors中的newSingleThreadExecutor()的源代码：
```java
publicstaticExecutorServicenewSingleThreadExecutor() {returnnewFinalizableDelegatedExecutorService              (newThreadPoolExecutor(1,1,0L, TimeUnit.MILLISECONDS,newLinkedBlockingQueue()));      }
```
4>newScheduledThreadPool创建一个固定长度的线程池，而且以延迟或定时的方式来执行任务。 

通过如上配置的线程池的创建方法源代码，我们可以发现： 

1> 除了CachedThreadPool使用的是直接提交策略的缓冲队列以外，其余两个用的采用的都是无界缓冲队列，也就说，FixedThreadPool和SingleThreadExecutor创建的线程数量就不会超过 corePoolSize。 

2> 我们可以再来看看三个线程池采用的ThreadPoolExecutor构造方法都是同一个，使用的都是默认的ThreadFactory和handler:
```
privatestaticfinalRejectedExecutionHandler defaultHandler = newAbortPolicy();

public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
```
也就说三个线程池创建的线程对象都是同组，优先权等级为正常的Thread.NORM_PRIORITY（5）的非守护线程，使用的被拒绝任务处理方式是直接抛出异常的AbortPolicy策略

###  4、 模仿Executors自定义线程池 ###

```java
public class ScmThreadPool
{
    static final Logger LOGGER = LogManager.getLogger(ScmThreadPool.class);

    /**
     * 默认最大并发数<br>
     */
    private static final int DEFAULT_MAX_CONCURRENT = Runtime.getRuntime().availableProcessors() * 2;

    /**
     * 线程池拒绝策略
     */
    private static final RejectedExecutionHandler SCM_HANDLER = new ScmPolicy();

    /**
     * 默认队列大小
     */
    private static final int DEFAULT_SIZE = 500;

    /**
     * 默认线程空闲后存活时间
     */
    private static final long DEFAULT_KEEP_ALIVE = 60L;

    /**
     * Executor
     */
    private static ExecutorService executor;

    /**
     * 等待执行的任务的阻塞队列（ArrayBlockingQueue：基于数组结构的有界阻塞队列）
     */
    private static BlockingQueue<Runnable> executeQueue = new ArrayBlockingQueue<>(DEFAULT_SIZE);

    /**
     * 防止重复创建同种命名的线程池
     */
    private static Map<ThreadFactory, ExecutorService> executorMap = new ConcurrentHashMap<>(16);

    /**
     * 私有构造
     */
    private ScmThreadPool()
    {
    }

    public static ExecutorService initThreadPool(ThreadFactory factory)
    {
        if (executorMap.containsKey(factory))
        {
            executor = executorMap.get(factory);
        }
        else
        {
            // 创建 Executor
            // 此处默认最大值改为处理器数量的 4 倍
            try
            {
                executor = new ThreadPoolExecutor(DEFAULT_MAX_CONCURRENT,
                    DEFAULT_MAX_CONCURRENT * 4,
                    DEFAULT_KEEP_ALIVE,
                    TimeUnit.SECONDS,
                    executeQueue,
                    factory,
                    SCM_HANDLER);
                executorMap.put(factory, executor);
                // 关闭事件的挂钩
                Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                    //                    @Override
                    //                    public void run()
                    //                    {
                    LOGGER.info("AsyncProcessor shutting down.");
                    executor.shutdown();
                    try
                    {
                        // 等待1秒执行关闭
                        if (!executor.awaitTermination(1, TimeUnit.SECONDS))
                        {
                            LOGGER.error("AsyncProcessor shutdown immediately due to wait timeout.");
                            executor.shutdownNow();
                        }
                    }
                    catch (InterruptedException e)
                    {
                        LOGGER.error("AsyncProcessor shutdown interrupted.");
                        executor.shutdownNow();
                    }

                    LOGGER.info("AsyncProcessor shutdown complete.");
                    //                    }
                }));
            }
            catch (Exception e)
            {
                LOGGER.error("AsyncProcessor init error.", e);
                throw new ExceptionInInitializerError(e);
            }
        }
        return executor;
    }

    /**
     * 自定义线程拒绝策略
     */
    public static class ScmPolicy implements RejectedExecutionHandler
    {
        /**
         * Creates an {@code AbortPolicy}.
         */
        public ScmPolicy()
        {
        }

        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e)
        {
            LOGGER.info("当前被拒绝任务为：{}", r.toString());
            LOGGER.info("当前线程池大小为：{}", e.getPoolSize());
        }
    }
}
```

```java
public class ScmThreadFactory implements ThreadFactory
{

    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private String poolName;
    private boolean isDaemon;

    private static ScmThreadFactory factory;

    /**
     * 防止创建同种命名的线程工厂
     */
    private static Map<String,ScmThreadFactory> factoryMap=new ConcurrentHashMap<>(16);

    private ScmThreadFactory() {
        this("SCM_POOL",false);
    }

    private ScmThreadFactory(String poolName) {
        this(poolName, false);
    }

    private ScmThreadFactory(String poolName, boolean isDaemon) {
        this.poolName = poolName;
        this.isDaemon = isDaemon;
    }

    public static ScmThreadFactory getThreadFactory(String poolName){
        if(factoryMap.containsKey(poolName)){
            factory=factoryMap.get(poolName);
        }else{
            factory= new ScmThreadFactory(poolName);
            factoryMap.put(poolName,factory);
        }
        return factory;
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setName(this.poolName + "-thread-" + threadNumber.getAndIncrement());
        t.setDaemon(isDaemon);
        return t;
    }
}
```
 ## 八、AQS ##
### 1. 简介 ### 
AQS ，AbstractQueuedSynchronizer ，即队列同步器。它是构建锁或者其他同步组件的基础框架（如 ReentrantLock、ReentrantReadWriteLock、Semaphore 等），J.U.C 并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。

它是 J.U.C 并发包中的核心基础组件

### 2. 优势 ###
AQS 解决了在实现同步器时涉及当的大量细节问题，例如获取同步状态、FIFO 同步队列。基于 AQS 来构建同步器可以带来很多好处。它不仅能够极大地减少实现工作，而且也不必处理在多个位置上发生的竞争问题。

在基于 AQS 构建的同步器中，只能在一个时刻发生阻塞，从而降低上下文切换的开销，提高了吞吐量。同时在设计 AQS 时充分考虑了可伸缩性，因此 J.U.C 中，所有基于 AQS 构建的同步器均可以获得这个优势。

 ### 3. 同步状态 ###

 AQS 的主要使用方式是继承，子类通过继承同步器，并实现它的抽象方法来管理同步状态。

AQS 使用一个 int 类型的成员变量 state 来表示同步状态：

* 当 state > 0 时，表示已经获取了锁。
* 当 state = 0 时，表示释放了锁。

它提供了三个方法，来对同步状态 state 进行操作，并且 AQS 可以确保对 state 的操作是安全的：

* #getState()
* #setState(int newState)
* #compareAndSetState(int expect, int update)
 ### 4. 同步队列 ###
 AQS 通过内置的 FIFO 同步队列来完成资源获取线程的排队工作：

* 如果当前线程获取同步状态失败（锁）时，AQS 则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程
* 当同步状态释放时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。
 ### 5. 主要内置方法 ###
 AQS 主要提供了如下方法：

* getState()：返回同步状态的当前值。
* setState(int newState)：设置当前同步状态。
* compareAndSetState(int expect, int update)：使用 CAS 设置当前状态，该方法能够保证状态设置的原子性。
* 【可重写】tryAcquire(int arg)：独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获取同步状态。
* 【可重写】tryRelease(int arg)：独占式释放同步状态。
* 【可重写】tryAcquireShared(int arg)：共享式获取同步状态，返回值大于等于 0 ，则表示获取成功；否则，获取失败。
* 【可重写】tryReleaseShared(int arg)：共享式释放同步状态。
* 【可重写】isHeldExclusively()：当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占。
* acquire(int arg)：独占式获取同步状态。如果当前线程获取同步状态成功，则由该方法返回；否则，将会进入同步队列等待。该方法将会调用可重写的 #tryAcquire(int arg) 方法；
* acquireInterruptibly(int arg)：与 #acquire(int arg) 相同，但是该方法响应中断。当前线程为获取到同步状态而进入到同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException 异常并返回。
* tryAcquireNanos(int arg, long nanos)：超时获取同步状态。如果当前线程在 nanos 时间内没有获取到同步状态，那么将会返回 false ，已经获取则返回 true 。
* acquireShared(int arg)：共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式的主要区别是在同一时刻可以有多个线程获取到同步状态；
* acquireSharedInterruptibly(int arg)：共享式获取同步状态，响应中断。
* tryAcquireSharedNanos(int arg, long nanosTimeout)：共享式获取同步状态，增加超时限制。
* release(int arg)：独占式释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒。
* releaseShared(int arg)：共享式释放同步状态。

从上面的方法看下来，基本上可以分成 3 类：

* 独占式获取与释放同步状态
* 共享式获取与释放同步状态
* 查询同步队列中的等待线程情况
