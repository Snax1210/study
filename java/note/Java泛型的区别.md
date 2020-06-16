[TOC]: # "JAVA泛型通配符T，E，K，V区别，T以及Class<T>，Class<?>的区别"

# JAVA泛型通配符T，E，K，V区别，T以及Class<T>，Class<?>的区别
- [1.1. 先解释下泛型概念](#11-先解释下泛型概念)
- [1.2. 下来说说泛型通配符T，E，K，V区别](#12-下来说说泛型通配符tekv区别)
- [1.3. 接下来说说List\<T>，List\<Object>，List\<?>区别](#13-接下来说说listtlistobjectlist区别)
- [1.4. 最后来说说T，Class\<T>，Class<?>区别](#14-最后来说说tclasstclass区别)
  - [1.4.1. 那么问题来了？Class类是创建出来了，但是Class<T>和Class<?>适用于什么时候呢？？?](#141-那么问题来了class类是创建出来了但是class和class适用于什么时候呢)



## 1.1. 先解释下泛型概念

> 泛型是Java SE 1.5的新特性，泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。这种参数类型可以用在类、接口和方法的创建中，分别称为[泛型类](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F2104244.htm)、泛型接口、泛型方法。[Java语言](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F229611.htm)引入泛型的好处是安全简单。  
> 在Java SE 1.5之前，没有泛型的[情况](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F780206.htm)的下，通过对类型Object的引用来实现参数的“任意化”，“任意化”带来的缺点是要做显式的[强制类型转换](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F2886403.htm)，而这种转换是要求开发者对[实际参数](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F2245196.htm)类型可以预知的情况下进行的。对于强制类型转换错误的情况，[编译器](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F487018.htm)可能不提示错误，在运行的时候才出现异常，这是一个安全隐患。  
> 泛型的好处是在编译的时候检查[类型安全](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F1965709.htm)，并且所有的[强制转换](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F965170.htm)都是自动和[隐式](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fview%2F2852863.htm)的，以提高代码的重用率。

以上内容摘自百度百科

举个栗子:  
Box类定义为一个泛型类

```java
    public class Box<T> {
        private T object;
    
        public void set(T object) { this.object = object; }
        public T get() { return object; }
    }
```

创建一个Box对象，不带泛型参数，发现获取对象的时候需要强制转换

```java
    Box box2 = new Box();
    box2.set(new Apple());
    Apple apple = (Apple) box2.get();
```

创建一个Box对象，带泛型参数，获取对象的时候就不需要强制转换

```java
    Box<Apple> box = new Box<Apple>();
    box.set(new Apple());
    Apple apple = box.get();
```

总结下泛型的好处就是  
**省去了强制转换，可以在编译时候检查类型安全，可以用在类，方法，接口上**

但是我们定义泛型类，泛型方法，泛型接口的时候经常会碰见很多不同的通配符T，E，K，V等等，这些通配符又都是什么意思呢？继续往下看

## 1.2. 下来说说泛型通配符T，E，K，V区别


这些全都属于java泛型的通配符，刚开始我看到这么多通配符，一下晕了，这几个其实**没什么区别**，只不过是一个约定好的代码，也就是说

> **使用大写字母A,B,C,D......X,Y,Z定义的，就都是泛型，把T换成A也一样，这里T只是名字上的意义而已**
>
> *   **？** 表示不确定的java类型
> *   **T (type)** 表示具体的一个java类型
> *   **K V (key value)** 分别代表java键值中的Key Value
> *   **E (element)** 代表Element

举个栗子：

```java
    public class Test<T> {    
      public List<T> list = new ArrayList<T>();   
      public static void main(String[] args) {
            Test<String> test = new Test<String>();
            test.list.add("hello");
            System.out.println(test.list);
        }}
```

和

```java
    public class Test<A> {    
      public List<A> list = new ArrayList<A>();   
      public static void main(String[] args) {
            Test<String> test = new Test<String>();
            test.list.add("hello");
            System.out.println(test.list);
        }}
```

将T换成了A，在执行效果上是没有任何区别的，只不过我们约定好了T代表type，所以还是**按照约定规范来比较好**，增加了代码的可读性。

如果要定义**多个**泛型参数，比如说两个泛型参数  
很典型的一个栗子是Map的key,value泛型，我们也可以定义一个这样的

```java
    public interface Mymap<K, V> {
        public K getKey();
        public V getValue();
    }
    
    public class MymapImpl<K, V> implements Mymap<K, V> {
    
        private K key;
        private V value;
    
        public MymapImpl(K key, V value) {
        this.key = key;
        this.value = value;
        }
    
        public K getKey()   { return key; }
        public V getValue() { return value; }
    }
```

下来就可以传入任意类型，创建实例了，不用转化类型

```java
    Mymap<String, Integer> mp1= new MymapImpl<String, Integer>("Even", 8);
    Mymap<String, String>  mp2= new MymapImpl<String, String>("hello", "world");
    Mymap<Integer, Integer> mp3= new MymapImpl<Integer, Integer>(888, 888);
```

如果要定义**超过两个，三个或三个以上**的泛型参数可以使用**T1, T2, ..., Tn**，像这样子

```java
    public class Test<T1,T2,T3> {
       public void print(T1 t1,T2 t2,T3 t3){
            System.out.println(t1.getClass());
            System.out.println(t2.getClass());
            System.out.println(t3.getClass());
        }
    }
```

## 1.3. 接下来说说List\<T>，List\<Object>，List\<?>区别

> **ArrayList\<T> al=new ArrayList\<T>();** 指定集合元素只能是T类型

**ArrayList<?> al=new ArrayList<?>();** 集合元素可以是任意类型，这种没有意义，一般是方法中，只是为了说明用法

> **ArrayList<? extends E> al=new ArrayList<? extends E>();**  
泛型的限定：  
**? extends E**:接收E类型或者E的子类型。  
**? super E**:接收E类型或者E的父类型

* Object和T不同点在于，Object是一个实打实的类,并没有泛指谁，而T可以泛指Object，比方**public void printList(List\<T> list){}**方法中可以传入**List\<Object> list**类型参数，也可以传入**List\<String> list**类型参数，但是**public void printList(List\<Object> list){}**就只可以传入**List\<Object> list**类型参数，因为Object类型并没有泛指谁，是一个确定的类型
* ?和T区别是？是一个不确定类，？和T都表示不确定的类型 ，但如果是T的话，函数里面可以对T进行操作，比方 T car = getCar()，而不能用？ car = getCar()。

下面举个栗子比较下这三种：

```java
    package com.lyang.demo.fanxing;
    
    import java.util.Arrays;
    import java.util.List;
    
    /**
     * 测试泛型参数Object和T的区别
     * Created by yanglu on 2017/04/20.
     */
    public class TestDifferenceBetweenObjectAndT {
        public static void printList1(List<Object> list) {
            for (Object elem : list)
                System.out.println(elem + " ");
            System.out.println();
        }
    
        public static <T> void printList2(List<T> list) {
            for (T elem : list)
                System.out.println(elem + " ");
            System.out.println();
        }
    
        public static  void printList3(List<?> list) {
            for (int i = 0;i<list.size();i++)
                System.out.println(list.get(i) + " ");
            System.out.println();
        }
    
        public static void main(String[] args) {
            List<Integer> test1 = Arrays.asList(1, 2, 3);
            List<String>  test2 = Arrays.asList("one", "two", "three");
            List<Object> test3 = Arrays.asList(1, "two", 1.23);
            List<Fruit> test4 = Arrays.asList(new Apple(), new Banana());
            /*
            * 下面这句会编译报错，因为参数不能转化成功
            * */
            printList1(test4);
            /**/
            printList1(test3);
            printList1(test3);
            printList2(test1);
            printList2(test2);
            printList2(test3);
            printList3(test1);
            printList3(test2);
            printList3(test3);
        }
    }
```

![](.Java泛型的区别_images/f8102e85.png)

## 1.4. 最后来说说T，Class\<T>，Class<?>区别

T是一种具体的类，例如String,List,Map......等等，这些都是属于具体的类，这个比较好理解  
\*\* Class是什么呢，Class也是一个类，但Class是存放上面String,List,Map......类信息的一个类\*\*，有点抽象，我们一步一步来看 。

如何获取到Class类呢，有三种方式：  
**1\. 调用Object类的getClass()方法来得到Class对象，这也是最常见的产生Class对象的方法。**  
例如：

```java
    List list = null;
    Class clazz = list.getClass();
```

\*\*2. 使用Class类的中静态forName()方法获得与字符串对应的Class对象。  
\*\*  
例如：

```java
    Class clazz = Class.forName("com.lyang.demo.fanxing.People");
```

**3.获取Class类型对象的第三个方法非常简单。如果T是一个Java类型，那么T.class就代表了匹配的类对象。**

```java
    Class clazz = List.class;
```

### 1.4.1. 那么问题来了？Class类是创建出来了，但是Class<T>和Class<?>适用于什么时候呢？？?

使用Class\<T>和Class<?>多发生在反射场景下，先看看如果我们不使用泛型，反射创建一个类是什么样的。

```java
    People people = (People) Class.forName("com.lyang.demo.fanxing.People").newInstance();
```

看到了么，需要强转，如果反射的类型不是People类，就会报  
**java.lang.ClassCastException**错误。

使用Class\<T>泛型后，不用强转了

```java
    public class Test {
        public static <T> T createInstance(Class<T> clazz) throws IllegalAccessException, InstantiationException {
            return clazz.newInstance();
        }
    
        public static void main(String[] args)  throws IllegalAccessException, InstantiationException  {
                Fruit fruit= createInstance(Fruit .class);
                People people= createInstance(People.class);
        }
    }
```

那Class\<T>和Class<?>有什么区别呢？  
**Class\<T>在实例化的时候，T要替换成具体类  
Class<?>它是个通配泛型，?可以代表任何类型，主要用于声明时的限制情况**  
例如可以声明一个

```java
    public Class<?> clazz;
```

但是你不能声明一个

```java
    public Class<T> clazz;
```

因为T需要指定类型  
所以当，不知道定声明什么类型的Class的时候可以定义一个Class<?>,Class<?>可以用于参数类型定义，方法返回值定义等。
