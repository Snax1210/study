[TOC]: # "JVM是怎样执行main方法的"

# JVM是怎样执行main方法的
- [1\. JVM启动：](#1-jvm启动)
- [2\. Loading, Linking, and Initializing](#2-loading-linking-and-initializing)
  - [2.1 Loading](#21-loading)
  - [2.2Linking](#22linking)
    - [2.2.1 verifying](#221-verifying)
    - [2.2.2 preparing](#222-preparing)
    - [2.2.3 resolution](#223-resolution)
  - [2.3Initializing](#23initializing)
- [3\. 启动main线程](#3-启动main线程)
- [4\. 执行main函数](#4-执行main函数)
- [5\. JVM退出](#5-jvm退出)


对java coder来说，经常接触JVM，可能不需要熟悉JVM工作原理，也能根据业务需求，通过代码实现其功能模块，一般不需要对JVM有特别的了解。但是，如果想精通java开发，需要对JVM的工作原理有一定的理解。本来JVM的工作原理浅到可以泛泛而谈，但如果真的想把JVM工作机制弄清楚，实在是很难，涉及到的知识领域太多。所以，本文通过简单的mian方法执行，浅谈JVM工作原理，看看JVM里面都发生了什么。

先上代码：

```java
package org.test;

class Test {
    private int invar = 1;

    public static String concat(String str1, String str2) {
        return str1 + str2;
    }

    public int f() {
        return invar;
    }

    public static void main(String[] args) {
        Test test = new Test();
        test.f();
        int localInt1 = 1;
        int localInt2 = 2;
        int sum = localInt1 + localInt2;
        String str = concat("string ", "concat");
        System.out.println(str);
    }
}
```
再看看JVM内部结构：

![](https://img-blog.csdn.net/20160112170239495?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

    上图是对《The Java® VirtualMachine SpecificationJava SE 7 Edition》中JVM内部结构的一个描述简图，“The Java® VirtualMachine Specification”是JVM的一个抽象规范，主流JVM实现的主要有Oracle HotSpot、IBM J9等。

    现在来看看启动org.test.Test的main方法，JVM（本文涉及到的JVM implementation是针对JDK 7的HotSpot，下同）会做些什么：

## 1\. JVM启动：

    根据JVM的启动参数分配JVM runtime data area内存空间，如根据-Xms、-Xmx分配Heap大小；根据-XX:PermSize、-XX:MaxPermSize分配Method area大小；根据-Xss分配JVM Stack大小。注意，Method area、Heap是所有JVM线程都共享的，在JVM启动时就会创建且分配内存空间；JVM Stack、PC Register、Native Method Stack是每个线程私有的，都是在线程创建时才分配。在HotSpot中，没有JVM Stacks和Native Method Stacks之分，功能上已经合并。官方说明：Both Java programming language methods and native methods share the same stack.

## 2\. Loading, Linking, and Initializing

   所有这些工作都是Class Loader Subsystem来完成，对org.test.Test.class进行加载、链接、初始化。

### 2.1 Loading

   将Test.class加载为Method area的Test Class data，Test.class是一个二进制文件，里面是JVM编译器对org.test.Test.java编译成的字节码，关于.class字节码的解读，请看[《实例分析Java Class的文件结构》](http://coolshell.cn/articles/9229.html)，讲得非常透彻。Test.class在Method area是如何存储的？这个问题的解答，首先还是要对Method area有一个认识。先看看《The Java® Virtual Machine Specification Java SE 7 Edition》中对ClassFile的定义：

![](https://img-blog.csdn.net/20160112095106490?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

    也就是说Test.class里面的二进制码是按照ClassFile的结构一个一个字节来存储相应的Test.java编译后的信息。所有这些信息被类加载器加载会对应地存储到Method area中，尽管体现的方式不一样，例如：Test.class中的Constant pool对应Method area中的Runtime constant pool，Runtime constant pool中的Constant中的信息都是从Constant pool中获取到的，Runtime constant pool里面都是些符号引用、字符串字面值以及整形、浮点型常量。下面是“JVM Method Area Class Data结构示意图”：

 ![](https://img-blog.csdn.net/20160110165546292?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 实际上，在JVM的method area中，一个class或interface是由其全限定名称和真正加载该类或接口的类加载器（即the defining classloader）来唯一确定的，因为一个类（如果无特殊说明，“类”表示class或interface，下同）可以由不同的类加载器加载。

 这里要弄清楚《The Java® Virtual Machine Specification Java SE 7 Edition》中关于类加载器的两个概念：the defining loader和the initiating loader。假设有两个类加载器classloader1和classloader2，classloader1是classloader2的parent classloader（委派的），一个待加载的类A，现在classloader2加载类A，首先委派classloader1去加载类A，结果classloader1加载类A成功了，那这里classloader1就是类A的“the defining loader”即![](https://img-blog.csdn.net/20160110203804272?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，classloader2就是类A的the initiating loader即![](https://img-blog.csdn.net/20160112100916175?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，因为类加载动作由classloader2发起的。关于类的可见性：对于类A、类B，类A是否对类B可见（即类B能找到类A）取决于类B的“the defining loader”能否自己或通过委派加载类A成功。

 所以在method area中，首先根据“the defining loader”来划分所加载的类的范围，这是逻辑意义上的概念，而对于每一个Class Data里面存储什么内容，上图已经比较清晰给出答案。这里需要注意的是Class的static变量是存储在Field data中，而final static则存储在Run-time constant pool中，Code的JVM指令存储在Method data中，另外，Class Data中还有两个引用：一个是指向“the defining loader”的引用，该引用在Class的动态链接解析时需要用到，如果该Class中引用了其他类，那么在动态链接解析时，就用该类的“the defining loader”去找其他类，完成其加载、链接、初始化动作；另一个是指向Class实例（即java.lang.Class对象）的引用，而JDK提供java.lang.Class出来，这样java开发者就可以通过该引用获取到存储在method area中的Class Data，即有哪些Field、哪些Method，这就是大家熟悉的反射。

 JVM在加载Test.class到Method area后，在使用其之前，还需要做一些工作，就是接下来的Linking和Initializing。

### 2.2Linking

 链接一个类包括对该类、其直接超类、其直接superinterfaces进行验证（verifying）、准备（preparing），如果该类为数组类型，还包括对数组元素类型的链接。Loading, Linking, and Initializing的时间顺序遵循以下两个原则：

（1）一个类一定要在链接之前加载完成；

（2）一个类的验证和准备一定要在初始化之前完成。

 关于类中的符号引用什么时候进行解析，这个要看JVM的实现。例如有的JVM实现采用懒解析（"lazy"or "late" resolution），即在需要进行该符号引用的解析时才去解析它，这样的话，可能该类都已经初始化完成了，如果其他的类链接到该类中的符号引用，需要进行解析，这个时候才会去解析；还有的JVM实现采用静态解析（"eager" or "static" resolution），即在该类在进行验证时一次性将其中的符号引用全部解析完成。在JVM的实现中，一般采用懒解析。

![](https://img-blog.csdn.net/20160110193427261?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

#### 2.2.1 verifying

 验证.class文件在结构上满足JVM规范的要求，验证工作可能会引起其他类的加载但不进行验证和准备。

#### 2.2.2 preparing

 给Class static变量分配内存空间，初始化为默认值即将内存空间清零。

 关于对类的方法声明，需要满足以下加载约束条件：

假设![](https://img-blog.csdn.net/20160110202404111?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)为类C的“the defining loader”即![](https://img-blog.csdn.net/20160110202609078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，![](https://img-blog.csdn.net/20160110202704204?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)为类D的“the defining loader”即![](https://img-blog.csdn.net/20160110202811115?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，D为C的超类或superinterface，对于C中所Override D的方法m，m的返回类型为![](https://img-blog.csdn.net/20160110203005505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，参数类型为![](https://img-blog.csdn.net/20160110203046334?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，如果![](https://img-blog.csdn.net/20160110203005505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)不是数组类型，假设![](https://img-blog.csdn.net/20160110203136428?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)为![](https://img-blog.csdn.net/20160110203005505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，否则![](https://img-blog.csdn.net/20160110203136428?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)为![](https://img-blog.csdn.net/20160110203005505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)的元素类型，对于i=1,...,n，如果![](https://img-blog.csdn.net/20160110203224486?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)不是数组类型，假设![](https://img-blog.csdn.net/20160110203309311?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)为![](https://img-blog.csdn.net/20160110203224486?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，否则![](https://img-blog.csdn.net/20160110203309311?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)为![](https://img-blog.csdn.net/20160110203224486?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)的元素类型，则对于i=0,...,n满足：![](https://img-blog.csdn.net/20160110201623824?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)，即![](https://img-blog.csdn.net/20160110202404111?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)、![](https://img-blog.csdn.net/20160110202704204?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)都能自己或通过委派成功加载![](https://img-blog.csdn.net/20160110203309311?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)。

   更进一步，假设![](https://img-blog.csdn.net/20160110210008403?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)为C的superinterface，![](https://img-blog.csdn.net/20160110202811115?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)为C的superclass，![](https://img-blog.csdn.net/20160110210008403?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)中声明了方法m，![](https://img-blog.csdn.net/20160110202811115?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)中声明且实现了方法m，则对于i=0,...,n满足：![](https://img-blog.csdn.net/20160110211023638?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)。

#### 2.2.3 resolution

 详细见：[Resolution in《The Java® Virtual Machine Specification Java SE 7 Edition》](http://blog.csdn.net/architect0719/article/details/50499736)

### 2.3Initializing

 详细见：[Initialization in《The Java® Virtual Machine Specification Java SE 7 Edition》](http://blog.csdn.net/architect0719/article/details/50501746)

## 3\. 启动main线程

   在JVM实现中，线程为Execution Engine的一个实例，main函数是JVM指令执行的起点，JVM会创建main线程来执行main函数，以触发JVM一系列指令的执行，真正地把JVM run起来。在创建main线程时，会为其分配私有的PC Register、JVM Stack、Native Method Stack，当然在HotSpot的实现中，JVM Stack、Native Method Stack功能上已经合并，下面以HotSpot为例来说说main函数的执行。

## 4\. 执行main函数

   先用javap -c Test.class，通过反汇编把Test.class对应的JVM机器码弄出来：

    Compiled from "Test.java"public class org.test.Test {  public org.test.Test();    Code:       0: aload_0       1: invokespecial #10                 // Method java/lang/Object."<init>":()V       4: aload_0       5: iconst_1       6: putfield      #12                 // Field invar:I       9: return   public static java.lang.String concat(java.lang.String, java.lang.String);    Code:       0: new           #20                 // class java/lang/StringBuilder       3: dup       4: aload_0       5: invokestatic  #22                 // Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;       8: invokespecial #28                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V      11: aload_1      12: invokevirtual #31                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;      15: invokevirtual #35                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;      18: areturn   public int f();    Code:       0: aload_0       1: getfield      #12                 // Field invar:I       4: ireturn   public static void main(java.lang.String[]);    Code:       0: new           #1                  // class org/test/Test       3: dup       4: invokespecial #46                 // Method "<init>":()V       7: astore_1       8: aload_1       9: invokevirtual #47                 // Method f:()I      12: pop      13: iconst_1      14: istore_2      15: iconst_2      16: istore_3      17: iload_2      18: iload_3      19: iadd      20: istore        4      22: ldc           #49                 // String string      24: ldc           #51                 // String concat      26: invokestatic  #52                 // Method concat:(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;      29: astore        5      31: getstatic     #54                 // Field java/lang/System.out:Ljava/io/PrintStream;      34: aload         5      36: invokevirtual #60                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V      39: return}

    JVM指令后面的#index表示ClassFile中常量池“数组”的索引，实际上线程中每一个函数的执行都对应一个帧在JVM Stack中的压入弹出，帧包含局部变量数组（用来存储函数参数、局部变量等）、操作数栈（用于配合JVM指令的执行，存储来自局部变量数组和类属性的值及中间结果，其中操作数栈中的值可以为直接引用）、一个指向当前方法所在类的runtime constant pool（用于符号引用解析）：

![](https://img-blog.csdn.net/20160112105517076?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

main线程调用main函数时，先创建一个main帧，根据编译时期就已经确定的局部变量数组和操作数栈的大小分配内存空间，将内存空间清零，将main帧压入main线程 Stack中：

![](https://img-blog.csdn.net/20160112152406318?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

看看main中的指令：

    0: new  #1    // class org/test/Test3: dup


0: new #1：#1表示Test.class中常量池“数组”的索引，该索引位置为CONSTANT\_Class\_info常量，表示Test class，这里的new指令表示new一个org/test/Test对象，且将其引用压入操作数栈中；3: dup：在操作数栈中，复制栈顶的操作数，同时将其压入栈顶。

在执行JVM指令的过程中，main线程的PC register会记录当前所执行的JVM指令的地址。执行完这两条指令后，只有操作数栈有变化：

![](https://img-blog.csdn.net/20160112154725332?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

    4: invokespecial #46                 // 调用Method "<init>":()V

这条指令操作是：弹出操作数栈顶的元素，在该元素所指向的Test对象上，调用<init>方法，其实就是对该对象进行初始化，<init>的参数为空，返回为V的类型即为空，执行完这条指令后，main帧如下：

![](https://img-blog.csdn.net/20160112155405379?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

    7: astore_1 // 弹出操作数栈顶元素，将该引用存放到局部变量“数组”索引1的位置8: aload_1  // 将局部变量“数组”索引1的位置的引用压入操作数栈

执行完后，main帧如下：

![](https://img-blog.csdn.net/20160112160257842?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

    9: invokevirtual #47     // 在Test对象上，调用Method f:()I

在执行invokevirtual指令时，在main函数中又调用了函数f()，所以main线程会创建一个帧即f帧，为f帧分配相应的内存空间，保存main帧的状态，即保存局部变量数组、操作数栈中的值，以便f()调用 完成后再恢复，，将f帧压入main线程的Stack，将f帧切换成当前帧，执行f()函数的字节码，main线程PC register记录当前执行的指令地址：

![](https://img-blog.csdn.net/20160112173537091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

f帧中局部变量“数组”索引0的位置为什么是reference 1？这是因为在invokevirtual指令执行过程中，首先会将main帧中操作数栈中的栈顶元素弹出，将其传给f帧，f帧将其存放到局部变量”数组“中，下面是执行f()函数中的指令：

    0: aload_0 // 将局部变量“数组”索引0位置的引用压入操作数栈中

![](https://img-blog.csdn.net/20160112162618780?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

    1: getfield      #12   // 弹出操作数栈顶元素即Test对象引用，在该引用指向的对象上获取属性Field invar:I的值，再将该值压入操作数栈中

![](https://img-blog.csdn.net/20160112163426680?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

4: ireturn   // 弹出f帧中操作数栈顶元素即1，将其压入main帧中的操作数栈

执行ireturn指令，实际上表明f()函数调用完成返回，main线程会释放f帧及其内存空间，将main帧切换成当前帧，恢复main帧的状态，main线程的PC
register记录main帧中当前执行指令的地址，继续执行完main帧后面的指令。

## 5\. JVM退出

    释放main线程所占用资源及内存空间，如PC register、JVM Stack等，释放JVM所占用的内存空间，如Heap、Method area，JVM退出。

虽然只是一个简单的main方式执行，但通过这个简单的示例可以看到JVM完整的工作流程。

