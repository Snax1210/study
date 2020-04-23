
[TOC]: # "ThreadLocal"

# ThreadLocal
- [1.1 官方介绍](#11-官方介绍)
- [1.2基本使用](#12基本使用)
  - [1.2.1常用方法](#121常用方法)
  - [1.2.2 使用案例](#122-使用案例)
- [1.3 ThreadLocal类和synchronized的区别](#13-threadlocal类和synchronized的区别)
- [ThreadLocal的内部结构](#threadlocal的内部结构)
- [ThreadLocal核心方法源码](#threadlocal核心方法源码)
- [ThreadLocalMap源码分析](#threadlocalmap源码分析)
- [弱引用和内存泄露](#弱引用和内存泄露)
- [Hash冲突的解决](#hash冲突的解决)


### 1.1 官方介绍

从Java官方文档中的描述：ThreadLocal类用来提供线程内部的局部变量。这种变量在多线程环境下访问（通过get和set方法访问）时能保证各个线程的变量相对独立于其他线程内的变量。ThreadLocal实例通常来说都是private static类型的，用于关联线程和线程上下文

ThreadLocal类的作用：

    提供线程内的局部变量，不同线程之间不会相互干扰，
    这种变量在线程的生命周期内起作用，
    减少同一个线程内多个函数或组件之间的一些公共变量传递的复杂度。

```text
总结：
    1. 线程并发：在多线程并发的场景下
    2. 传递数据：我们可以通过ThreadLocal在同一线程，不同组件中传递公共变量
    3. 线程隔离：每个线程的变量都是独立的，不会互相影响
```

### 1.2基本使用

#### 1.2.1常用方法


方法声明 | 描述
---|---
ThreadLocal() | 创建ThreadLocal对象
public void set (T value) | 设置当前线程绑定的局部变量
public T get() | 获取当前线程绑定的局部变量
public void remove() | 移除当前线程绑定的局部变量

#### 1.2.2 使用案例

```Java
package com.example.demo.test;

/**
 * 需求：线程隔离
 *      在多线程并发的场景下，每个线程中的变量都是相互独立
 *      线程A：设置（变量1） 获取（变量1）
 *      线程B：设置（变量2）获取（变量2）
 *
 *      ThreadLocal :
 *      1 set() : 将变量绑定到当前线程中
 *      2 get() : 获取当前线程绑定的变量
 */
public class MyDemo01
{
    ThreadLocal<String>  t1 = new ThreadLocal<>();

    public String getContent()
    {
        return t1.get();
    }

    public void setContent(String content)
    {
//        this.content = content;
        //绑定到当前线程
        t1.set(content);
    }

    public static void main(String[] args)
    {
        MyDemo01 demo = new MyDemo01();

        for (int i = 0; i < 5; i++)
        {
            Thread thread = new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    /**
                     * 每个线程：存一个变量，过一会，取出这个变量
                     */
                    demo.setContent(Thread.currentThread().getName() + "的数据");
                    System.out.println("-----------------------------------");
                    System.out.println(Thread.currentThread().getName() + "--->" + demo.getContent());
                }
            });
            thread.setName("线程" + i);
            thread.start();
        }
    }
}
```

### 1.3 ThreadLocal类和synchronized的区别

虽然ThreadLocal模式和synchronized关键字都用于处理多线程并发访问变量的问题，不过两者处理问题的角度和思路不同。


|             | synchronized                                                                                                             | ThreadLocal                                                                                                                                                                    |
|:-------|:----------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------|
| 原理     | 同步机制采用`以时间换空间`的方式，只提供了一份变量，让不同的线程排队访问 | ThreadLocal采用`以空间换时间`的方式，为每一个线程都提供了一份变量的副本，从而实现同时访问而不相互干扰 |
| 侧重点 | 多个线程之间资源的同步                                                                                           | 多线程中每个线程之间的数据互相隔离                                                                                                                           |

    总结：在刚刚的案例中，虽然使用ThreadLocal和synchronized都能解决问题，但是使用ThreadLocal更为合适，因为这样可以使程序拥有更高的并发性。


## ThreadLocal的内部结构

早期设计：每个ThreadLocal都创建一个Map，然后用线程作为Map的key，要存储的局部变量作为Map的value，这样就能达到各个线程的局部变量隔离的效果。这个最简单的设计方法，JDK最早期的ThreadLocal确实是这样设计的，但现在早已不是了。

现在的设计：JDK8中的ThreadLocal的设计是：每个Thread维护一个ThreadLocalMap，这个Map的key是ThreadLocal实例本身，value才是真正要存储的值Object。

具体过程：

    1. 每个Thread线程内部都有一个Map（ThreadLocalMap）
    2. Map里面存储的ThreadLocal对象（key）和线程的变量副本（value）
    3. Thread内部的Map是由ThreadLocal维护的，由ThreadLocal负责向map获取和设置线程的变量值。
    4. 对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰

JDK8的设计方案两个好处

1. 每个Map存储的Entry数量变少（ThreadLocal数量往往小于Thread）
2. 当Thread销毁的时候，ThreadLocalMap也会随之销毁，减少内存的使用


## ThreadLocal核心方法源码

除了基本的构造方法外，ThreadLocal对外暴露的方法有以下四个
方法声明 | 描述
---|---
protected T initialValue() | 返回当前线程局部变量的初始值
public void set (T value) | 设置当前线程绑定的局部变量
public T get() | 获取当前线程绑定的局部变量
public void remove() | 移除当前线程绑定的局部变量

- set方法代码执行流程

1. 首先获取当前线程，并根据当前线程获取一个Map
2. 如果获取的Map不为空，则将参数设置到Map中（当前ThreadLocal的引用作为key）
3. 如果Map为空，则给该线程创建Map，并设置初始值

- get方法代码执行流程

1. 首先获取当前线程，根据当前线程获取一个Map
2. 如果获取的Map不为空，则在Map中以ThreadLocal的引用作为key来在Map中获取对应的Entry e，否则转到4
3. 如果e不为空，则返回e.value，否则转到4
4. Map为空或者e为空，则通过initialValue函数获取初始值value，然后用ThreadLocal的引用和value作为firstKey和firstValue创建一个新的Map

总结：**先获取当前线程的ThreadLocalMap变量，如果存在则返回值，不存在则创建并返回初始值。**

- remove方法

1. 首先获取当前线程，并根据当前线程获取一个Map
2. 如果获取的Map不为空，则移除当前ThreadLocal对象对应的entry

- initialValue方法

1. 这是一个延迟调用方法，在set方法还未调用而先调用了get方法时才执行，并且仅执行一次
2. 这个方法缺省实现直接返回一个null
3. 如果想要一个出null之外的初始值，可以重写此方法。（备注：该方法是一个protected的方法，显然是为了让子类重写而设计的）


## ThreadLocalMap源码分析

ThreadLocalMap是ThreadLocal的内部类，没有实现Map接口，用独立的方式实现了Map的功能，其内部Entry也是独立实现的。

和HashMap类似，INITIAL_CAPACITY代表这个Map的初始容量；table是一个Entry类型的数组，用于存储数据；size代表表中的存储数目；threshold代表需要扩容时对应size的阈值。

在ThreadLocalMap中，也是用Entry保存K-V结构数据的。不过Entry中的key只能是ThreadLocal对象，这点在构造方法中已经限定死了。

另外，Entry集成WeakReference，也就是key（ThreadLocal）是弱引用，其目的是将ThreadLocal对象的生命周期和线程的声明周期解绑。


## 弱引用和内存泄露

**（1）内存泄露相关概念**

- Memory overflow：内存溢出，没有足够的内存提供申请者使用。
- Memory leak：内存泄露，指程序中已动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运营速度减慢甚至系统崩溃等严重后果。内存泄露的堆积终将导致内存溢出。

**（2）弱引用相关概念**

java中的引用有4种类型：强、软、弱、虚

**强引用：（“Strong Reference”）**，就是我们最常见的普通对象引用，只要还有强引用指向一个对象，就能表名对象还“活着”，垃圾回收器就不会回收这种对象。

**弱引用（WeakReference）**，垃圾回收器一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

不管ThreadLocalMap使用强引用还是弱引用都会可能引起内存泄露

出现内存泄露的真实原因：

1. 没有手动删除这个Entry
2. CurrentThread依然运行

根本原因：**由于ThreadLocalMap的声明周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄露**

为什么key使用弱引用：

事实上，在ThreadLocalMap中的set/getEntry方法中，会对key为null（也就是ThreadLocal为null）进行判断，如果为null的话，那么是会对value置为null的。

这就意味着使用完ThreadLocal，CurrentThread依然在运行的前提下，就算忘记调用remove方法，**弱引用比强引用可以多一层保障**：弱引用的ThreadLocal会被回收，对应的value在下一次ThreadLocalMap调用set、get、remove中的任一方法的时候会被清除，从而避免内存泄漏。

## Hash冲突的解决

构造函数首先创建一个长度为16的Entry数组，然后计算出firstKey对应的索引，然后存储到table中，并设置size和threshold。

每次获取当前值并加上HASH_INCREMENT，这个值跟斐波那契数列（黄金分割数）有关，其主要目的是为了让哈希码均匀的分组在2的n次方的数组里，也就是Entry[] table中，这样做可以避免hash冲突。

计算hash的时候里面采用了hashCode&(size-1)的算法，这相当于取模运算hashCode % size的一个更高效的实现。正式因为这种算法，我们要求size必须是2的整次幂，这也能保证在索引不越界的前提下，使得hash发生冲突的次数减小。

**set方法代码执行过程**

1. 首先还是根据key计算出索引i，然后查找i位置上的Entry，
2. 若是Entry已经存在并且key等于传入的key，那么这时候直接给这个Entry赋新的value值，
3. 若是Entry存在，但是key是null，则调用replaceStaleEntry来更换这个key为空的Entry。
4. 不断循环检测，直到遇到为null的地方，这时候要是还没在循环过程中return，那么就在这个null的位置新建一个Entry，并且插入，同时size增加1.
5. 最后调用cleanSomeSlots，清理key为null的Entry，最后返回是否清理了Entry，接下来判断size是否>=threshold达到了rehash的条件，达到的话就会调用rehash函数执行一次全表的扫描清理。

**重点分析：**ThreadLocalMap使用`线性探测法`来解决哈希冲突的。

该方法一次探测下一个地址，直到有空的地址后插入，若整个空间都找不到空余的地址，则产生溢出。

举个例子，假设当前table长度为16，也就是说如果计算出来key的hash值为14，如果table\[14\]上已经有值，并且其key与当前key不一致，那么就发生了hash冲突，这个时候将14加1得到15，取table\[15\]进行判断，这个时候如果还是冲突会回到0，取table\[0\]，以此类推，直到可以插入。

按照上面的描述，可以把Entry[] table看成一个环形数组。

