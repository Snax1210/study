# ZooKeeper框架Curator分布式锁实现及源代码分析

curator中有各种分布式锁，本文挑选其中一个---InterProcessMutex进行讲解。

我们先看下Curator代码中对于InterProcessMutex的注释：

    可重入的互斥锁，跨JVM工作。使用ZooKeeper来控制锁。
    所有JVM中的任何进程，只要使用同样的锁路径，将会成为跨进程的一部分。
    此外，这个排他锁是“公平的”，每个用户按照申请的顺序得到排他锁。

可见InterProcessMutex是一个排他锁，此外还可以重入

## 如何使用InterProcessMutex

在分析InterProcessMutex代码前，我们先看一下它是如何使用的，下面代码简单展示了InterProcessMutex的使用：

```java
    public static void soldTickWithLock(CuratorFramework client) throws Exception {
        //创建分布式锁, 锁空间的根节点路径为/curator/lock
        InterProcessMutex mutex = new InterProcessMutex(client, "/curator/locks");
        mutex.acquire();

        //获得了锁, 进行业务流程
        //代表复杂逻辑执行了一段时间
        int sleepMillis = (int) (Math.random() * 2000);
        Thread.sleep(sleepMillis);

        //完成业务流程, 释放锁
        mutex.release();
    }
``` 

首先通过mutex.acquire()获取锁，该方法会阻塞进程，直到获取锁，然后执行你的业务方法，最后通过 mutex.release()释放锁。

下面展开分析Curator关于分布式锁的实现：

## 实现思路

1. 创建有序临时节点

2. 触发"尝试获取锁逻辑"，如果自己是临时锁节点序列的第一个，则取得锁，获取锁成功。

3. 如果自己不是序列中的第一个，则监听前一个锁节点变更。同时阻塞进程。

4. 当前一个锁节点变更时，通过watcher恢复线程，然后再次到步骤2"尝试获取锁逻辑"

如下图所示：
![avatar](https://img-blog.csdnimg.cn/20181109130232331.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpeWltaW5nMjAxNw==,size_16,color_FFFFFF,t_70)

## 代码实现概述

Curator对于排它锁的顶层实现逻辑是在InterProcessMutex类中，它对客户端暴露锁的使用方法，如获取锁和释放锁等。但锁的上述实现逻辑，是由他持有的LockInternals对象来具体实现的。LockInternals使用StandardLockInternalsDriver类中的方法来做一些处理。

简单点解释，Curator好比一个公司承接各种业务，InterProcessMutex是老板，收到自己客户（client）的需求后，分配给自己的下属LockInternals去具体完成，同时给他一个工具StandardLockInternalsDriver,让他在做任务的过程中使用。如下图所示：

![avatar](https://img-blog.csdnimg.cn/20181109130458808.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpeWltaW5nMjAxNw==,size_16,color_FFFFFF,t_70)

## InterProcessMutex源码分析

InterProcessMutex类是Curator中的排它锁类，客户端直接打交道的就是InterProcessMutex。从顶层开始分析InterProcessMutex。

## 实现接口

InterProcessMutex实现了两个接口

```java
public class InterProcessMutex implements InterProcessLock, Revocable<InterProcessMutex>

```

InterProcessLock是分布式锁接口，分布式锁必须实现接口中的如下方法

1. 获取锁，直到锁可用

```java
public void acquire() throws Exception;
```

2. 指定等待的时间内获取锁

```java
public boolean acquire(long time, TimeUnit unit) throws Exception;
```

3. 释放锁

```java
public void release() throws Exception;
```

4. 当前线程是否获取了锁

```java
boolean isAcquiredInThisProcess();
```

以上方法也是InterProcessMutex暴露出来，供客户端在使用分布式锁时调用。

## 属性

InterProcessMutex属性如下：
|  类型   | 名称  |  说明   |
|  ----  | ----  |  ----------  |
| LockInternals  | internals | 锁的实现都在该类中，InterProcessMutex通过此类的方法实现锁  |
| String  | basePath | 锁节点在zk中的根路径  |
| ConcuurentMap<Thread,LockData>  | threadData | 线程和自己的锁相关数据映射  |
| String  | LOCK_NAME | 常量，值为"lock-"。表示锁节点的前缀  |

它还有一个内部静态类LockData，也是threadData中保存的value，它定义了锁的相关数据，包括锁所属线程，锁的全路径，和该线程加锁的次数（InterProcessMutex为可重入锁）。代码如下：

```java
private static class LockData
{
    final Thread owningThread;
    final String lockPath;
    final AtomicInteger lockCount = new AtomicInteger(1);

    private LockData(Thread owningThread, String lockPath)
    {
        this.owningThread = owningThread;
        this.lockPath = lockPath;
    }
}
```

## 构造方法

InterProcessMutex有三个构造方法，根据入参不同，嵌套调用，最终调用的构造方法如下：

```java
InterProcessMutex(CuratorFramework client, String path, String lockName, int maxLeases, LockInternalsDriver driver)
{
    basePath = PathUtils.validatePath(path);
    internals = new LockInternals(client, driver, path, lockName, maxLeases);
}
```

可见构造方法最终初始化了两种属性，basePath被设置为我们传值"/curator/lock",这是锁的根节点。此外就是初始化了internals，前面说过internals是真正实现锁功能的对象。

构造完InterProcessMutex对象：

## 方法

InterProcessMutex实现InterProcessLock接口，关于分布式锁的几个方法都在这个接口中，具体实现方法如下：

## 获得锁:

获得锁有两个方法，区别为是否限定了等待锁的时间长度。其实最终都是调用私有方法internalLock()。不限定等待时长的代码如下：

```java
public void acquire() throws Exception
{
    if ( !internalLock(-1, null) )
    {
        throw new IOException("Lost connection while trying to acquire lock: " + basePath);
    }
}
```

可以看到internalLock()返回false时，只可能因为连接超时，否则会一直等待获取锁。

internalLock逻辑如下：

1. 取得当前线程在threadData中的lockData

2. 如果存在该线程的锁数据，说明锁重入，lockData.lockCount加1，直接返回true。获取锁成功

3. 如果不存在该线程的锁数据，则通过internals.atttmptLock()获取锁，此时线程被阻塞，直至获得到锁。

4. 锁获取成功后，把锁的信息保存到threadData中

5. 如果没能获取到锁，则返回false。

完整代码如下：

```java
private boolean internalLock(long time, TimeUnit unit) throws Exception
{
    /*
       Note on concurrency: a given lockData instance
       can be only acted on by a single thread so locking isn't necessary
    */

    Thread currentThread = Thread.currentThread();

    LockData lockData = threadData.get(currentThread);
    if ( lockData != null )
    {
        // re-entering
        lockData.lockCount.incrementAndGet();
        return true;
    }

    String lockPath = internals.attemptLock(time, unit, getLockNodeBytes());
    if ( lockPath != null )
    {
        LockData newLockData = new LockData(currentThread, lockPath);
        threadData.put(currentThread, newLockData);
        return true;
    }

    return false;
}
```

可以看到获取锁的核心代码是internals.attemptLock

## 释放锁

释放锁的方法为release()，逻辑如下：

从threadData中取得当前线程的锁数据，有如下情况：

1. 不存在，抛出无此锁的异常

2. 存在，而且lockCount-1后大于零，说明该线程锁重入了，所以直接返回，并不在zk中释放

3. 存在，而且lockCount-1后小于零，说明有某种异常发生，直接抛异常

4. 存在，而且lockCount-1后等于零，这是无重入的正确状态，需要做的就是从zk中删除临时节点，通过internals.releaseLock(),不管结果如何，在threadData汇总移除该线程的数据。

## LockInternals源码分析

Curator通过zk实现分布式锁的核心逻辑都在LockInternals中，按获取锁到释放锁的流程为指引，逐步分析LockInternals的源码。

## 获取锁

在InterProcessMutex获取锁的代码分析中，可以看到它是通过internals.attemptLock(time,unit,getLockNodeBytes());来获取锁的，以这个方法为入口分析逻辑如下：

1. 通过driver在zk上创建锁节点，获取锁节点路径。

2. 通过internalLockLoop()方法阻塞进程，直到获取锁成功。

核心代码如下：

```java
ourPath = driver.createsTheLock(client, path, localLockNodeBytes);
hasTheLock = internalLockLoop(startMillis, millisToWait, ourPath);
```

继续分析internalLockLoop方法，获取锁的核心逻辑在此方法中。

internalLockLoop中通过while自旋，判断锁如果没有被获取，将不断的尝试去获取锁。

while循环中的逻辑如下：

1. 通过driver查看当前锁节点序号是否排在第一位，如果排在第一位，说明取锁成功，跳出循环

2. 如果没有排在第一位，则监听自己的前序锁节点，然后阻塞线程。

当前序节点释放了锁，监听会被触发，恢复线程，此时主线程又回到了while中的第一步。

重复以上逻辑，直至获取到锁(自己锁的序号排在首位)。

internalLockLoop方法核心代码如下:

```java
while ( (client.getState() == CuratorFrameworkState.STARTED) && !haveTheLock )
{
    List<String>        children = getSortedChildren();
    String              sequenceNodeName = ourPath.substring(basePath.length() + 1); // +1 to include the slash

    PredicateResults    predicateResults = driver.getsTheLock(client, children, sequenceNodeName, maxLeases);
    if ( predicateResults.getsTheLock() )
    {
        haveTheLock = true;
    }
    else
    {
        String  previousSequencePath = basePath + "/" + predicateResults.getPathToWatch();

        synchronized(this)
        {
            try
            {
                // use getData() instead of exists() to avoid leaving unneeded watchers which is a type of resource leak
                client.getData().usingWatcher(watcher).forPath(previousSequencePath);
                if ( millisToWait != null )
                {
                    millisToWait -= (System.currentTimeMillis() - startMillis);
                    startMillis = System.currentTimeMillis();
                    if ( millisToWait <= 0 )
                    {
                        doDelete = true;    // timed out - delete our node
                        break;
                    }

                    wait(millisToWait);
                }
                else
                {
                    wait();
                }
            }
            catch ( KeeperException.NoNodeException e )
            {
                // it has been deleted (i.e. lock released). Try to acquire again
            }
        }
    }
}
```

## 释放锁

释放锁的逻辑很简单，移除watcher，删除锁节点。代码如下：

```java
final void releaseLock(String lockPath) throws Exception

{

client.removeWatchers();

revocable.set(null);

deleteOurPath(lockPath);

}
```
