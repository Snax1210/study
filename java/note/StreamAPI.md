
### Java8新特性——StreamAPI
<!-- TOC -->

- [Java8新特性——StreamAPI](#java8新特性streamapi)
    - [1. 流的基本概念](#1-流的基本概念)
        - [1.1 什么是流？](#11-什么是流)
        - [1.2 流的特点](#12-流的特点)
        - [1.3 流的操作种类](#13-流的操作种类)
    - [2. 流的使用](#2-流的使用)
        - [2.1 获取流](#21-获取流)
        - [2.2 筛选 filter](#22-筛选-filter)
        - [2.3 去重 distinct](#23-去重-distinct)
        - [2.4 截取 limit](#24-截取-limit)
        - [2.5 跳过 skip](#25-跳过-skip)
        - [2.6 映射 map](#26-映射-map)
        - [2.7 合并多个流 flagmap](#27-合并多个流-flagmap)
        - [2.8 是否匹配任意元素：anyMatch](#28-是否匹配任意元素anymatch)
        - [2.9 是否匹配所有元素 allMatch](#29-是否匹配所有元素-allmatch)
        - [2.10 是否未匹配所有元素：noneMatch](#210-是否未匹配所有元素nonematch)
        - [2.11获取任意一个元素findAny](#211获取任意一个元素findany)
        - [Optional介绍](#optional介绍)
        - [2.12 获取第一个元素findFirst](#212-获取第一个元素findfirst)
        - [2.13 归约 reduce](#213-归约-reduce)
        - [2.13.1元素求和：自定义Lambda表达式实现求和](#2131元素求和自定义lambda表达式实现求和)
        - [2.13.2元素求和：使用Integer.sum函数求和](#2132元素求和使用integersum函数求和)
        - [2.14数值流的使用](#214数值流的使用)
        - [2.14.1将普通流转换成数值流](#2141将普通流转换成数值流)
        - [2.14.2数值计算](#2142数值计算)
    - [3. 收集器简介](#3-收集器简介)
    - [4. 收集器的使用](#4-收集器的使用)
        - [4.1 归约](#41-归约)
        - [4.1.1 计数](#411-计数)
        - [4.1.2最值](#412最值)
        - [4.1.3 求和](#413-求和)
        - [4.1.4求平均值](#414求平均值)
        - [4.1.5连接字符串](#415连接字符串)
        - [4.1.6一般性的归约操作](#416一般性的归约操作)
        - [4.2分组](#42分组)
        - [4.2.1一级分组](#421一级分组)
        - [4.2.2多级分组](#422多级分组)
        - [4.2.3对分组进行统计](#423对分组进行统计)
        - [4.2.4将收集器的结果转换成另一种类型](#424将收集器的结果转换成另一种类型)
    - [总结：](#总结)

<!-- /TOC -->
#### 1. 流的基本概念
##### 1.1 什么是流？
流是Java8一如的全新概念，它与java.io包里的InputSteam和OutoutStream是完全不同的概念。用来处理集合中的数据，暂且可以把它理解为一种高级集合。


##### 1.2 流的特点
1.只能遍历一次
我们可以将流想象成一条流水线，流水线的源头是我们的数据源（一个集合），数据源中的元素依次被输送到流水线上，我们可以在流水线上对元素进行各种操作。一旦元素走到了流水线的另一头，那么这些元素就被“消费掉了”，我们无法再对这个流进行操作。当然我们可以从数据源那里再获得一个新的流重新遍历一遍。 
2.采用内部迭代方式
若要对集合进行处理，则需要我们手写处理代码，这就叫做外部迭代。而要对流进行处理，我们只需告诉流我们需要什么结果，处理过程由流自行完成，这就称为内部迭代。

##### 1.3 流的操作种类
流的操作分为两种，分别为中间操作和终端操作 
1.中间操作（Intermediate） 
map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered 
一个流可以后边跟随零个或多个中间操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化得（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历 
2.终端操作（Terminal） 
forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator 
一个流只能一个终端操作，当这个操作执行后，流就被消耗了，无法再被操作，所以这必定是流的最后一个操作。Terminal操作的执行，才回真正开始流的遍历，并且会生成一个结果。 
在对于一个Stream进行多次中间操作，每次都对Stream的每个元素进行转换，而且是执行多次，这样时间复杂度就是N（转换次数）个for循环里把所有操作都执行的总和吗？其实不是，转换操作是lazy的，多个中间操作只会在Terminal操作的时候融合起来，一次循环完成。简单的将就是，Stream里有个操作函数的集合，每次中间操作就是把转换函数放入这个集合中，在Terminal操作的时候循环Stream对应的集合，然后对每个元素执行所有的函数。
```Java
int sum = widgets.stream()
.filter(w -> w.getColor() == RED)
.mapToInt(w -> w.getWeight())
.sum();
```

#### 2. 流的使用
##### 2.1 获取流
在使用流之前，首先需要拥有一个数据源，并通过StreamAPI提供的一些方法获取该数据源的流对象。数据源可以有多种形式： 
集合 
这种数据源较为常用，通过stream()方法即可获取流对象： 
```Java
List<Person> list = new ArrayList<Person>();
Stream<Person> stream = list.stream();
```
数组 
通过Arrays类提供的静态函数stream()获取数组的流对象：
```Java
String[] names = {"chaimm","peter","john"};
Stream<String> stream = Arrays.stream(names);
```
值 
直接将几个值变成流对象：
```Java
Stream<String> stream = Stream.of("chaimm","peter","john");
```
文件 
```Java
try(Stream lines = Files.lines(Paths.get(“文件路径名”),Charset.defaultCharset())){
//可对lines做一些操作
}catch(IOException e){
}
```
PS:Java7简化了IO操作，把打开IO操作放在try后的括号中即可省略关闭IO的代码。
##### 2.2 筛选 filter
filter函数接收一个lambda表达式作为参数，该表达式返回boolean，在执行过程中，流将元素逐一输送给filter，并筛选出执行结果为true的元素。 
如，筛选出说有学生：
```Java
List<Person> studentList = list.stream()
.filter(Person::getCareer=="student")
.collect(toList());
```
##### 2.3 去重 distinct
去掉重复的结果：
```Java
List<Person> result = list.stream()
.distinct(Person::getName)
.collect(toList());
```
##### 2.4 截取 limit
截取流的前N个元素：
```Java
List<Person> result = list.stream()
.limit(3)
.collect(toList());
```

##### 2.5 跳过 skip
跳过流的前N个元素
```Java
List<Person> result = list.stream()
.skip(3)
.collect(toList());
```

##### 2.6 映射 map
对流的每个元素执行一个函数，使得元素转换成另一种类型输出。流会将每一个元素输送给map函数，并执行map种的Lambda表达式，最后将执行结果存入一个新的流中。 
如，获取每个人的姓名（实则是将Person类型的List中的某一属性提取，装入到一个新的List中）:
```Java
List<Person> result = list.stream()
.map(Person::getName)
.collect(toList());
```

##### 2.7 合并多个流 flagmap
例如：列出List中各不相同的单词，List集合如下：
```Java
List<String> list = new ArrayList<String>();
list.add("I am a boy");
list.add("I love the girl");
list.add("But the girl loves another girl");
```
思路如下： 
首先将list变成流：
```Java
list.stream();
```
按空格分词：
```Java
list.stream()
.map(line->line.split(" "));
```
分词之后，每个元素变成了一个String[]数组。 
将每个String[]变成流：
```Java
list.stream()
.map(line->line.split(" "))
.map(Arrays::stream);
```
此时一个大流里面包含了一个个小流，我们需要将这些小流合并成一个流。 
将小流合并成一个大流：（用flagmap替换刚才的map）
```Java
list.stream()
.map(line->line.split(" "))
.flagmap(Arrays::stream)
```
去重：
```Java
list.stream()
.map(line->line.split(" "))
.flagmap(Arrays::stream)
.distinct()
.collect(toList());
```

##### 2.8 是否匹配任意元素：anyMatch
anyMatch用于判断流中是否存在至少一个元素满足指定的条件，这个判断条件通过Lambda表达式传递给anyMatch，执行结果为boolean类型。 
如，判断list中是否有学生：
```Java
boolean result = list.stream()
.anyMatch(Person::Career=="student");
```

##### 2.9 是否匹配所有元素 allMatch
allMatch用于判断流中的所有元素是否都满足指定条件，这个判断条件通过Lambda表达式传递给anyMatch，执行结果为boolean类型。 
如,判断是否所有人都是学生：
```Java
boolean result = list.stream()
.allMatch(Person::Career=="student");
```
##### 2.10 是否未匹配所有元素：noneMatch
nonMatch与allMatch恰恰相反，它用于判断流中的所有元素是否都不满足指定条件： 
```java
boolean result = list.stream()
.noneMatch(Person::isStudent);
```
##### 2.11获取任意一个元素findAny
findAny能够从流中随便选一个元素出来，它返回一个Optional类型的元素。
```Java
Optional<Person> person = list.stream()
.findAny();
```

##### Optional介绍
Optional是Java8新加入的一个容器，这个容器只存1个或0个元素，他用于防止出现NullPointerException，它提供如下方法：
- isPresent()
判断容器中是否有值。
- ifPresent(Consume lambda) 
容器若不为空则执行括号中的Lambda表达式。
- T get() 
获取容器中的元素，若容器为空则抛出NoSuchElement异常。
- T orElse(T other) 
获取容器中的元素，若容器为空则返回括号中的默认值。

##### 2.12 获取第一个元素findFirst
```Java
Optional<Person> person = list.stream()
.findFirst();
```
##### 2.13 归约 reduce
归约是将集合中的所有元素经过指定运算，折叠成一个元素输出，如：求最值、平均数等，这些操作都是将一个集合的元素折叠成一个元素输出。 
在流中，reduce函数能实现归约 
reduce函数接收两个参数：
-  初始值
- 进行归约操作的Lambda表达式

##### 2.13.1元素求和：自定义Lambda表达式实现求和
例：计算所有人的年龄总和
```java
int age = list.stream().reduce(0, (person1,person2)->person1.getAge()+person2.getAge());
```
reduce的第一个参数表示初始值为0；
reduce的第二个参数为需要进行的归约操作，它接收一个拥有两个参数的Lambda表达式，reduce会吧流中的元素两两输送给Lambda表达式，最后将计算出累加之和。
##### 2.13.2元素求和：使用Integer.sum函数求和
上面的方法我们自己定义了Lambda表达式实现求和运算，如果当前流的元素为数值类型，那么可以使用Integer提供的sum函数替代自定义的Lambda表达式,如：
```java
int age = list.stream().reduce(0, Integer::sum);
```
Integer类还提供了min、max等一系列数值操作，当流中的元素为数值类型时可以直接使用。
##### 2.14数值流的使用
采用reduce进行数值操作会设置到基本数值类型和引用数值类型之间的装箱、拆箱操作，因此效率较低。 
当流操作为纯数字操作时，使用数值流能或得较高的效率。
##### 2.14.1将普通流转换成数值流
StreamAPI提供了三种数值流IntStream、DoubleStream、LongStream，也提供了将普通流转换成数值流的三种方法：mapToInt、mapToDouble、mapToLong。 
如，将Person中的age转换成数值流：
```Java
IntStream stream = list.stream()
.mapToInt(Person::getAge);
``` 
##### 2.14.2数值计算
每种数值流都提供了数值计算函数，如max、min、sum等。 
如，找出最大的年龄：
```Java
OptionalInt maxAge = list.stream()
.mapToInt(Person::getAge).max();
```
由于数值流可能为空，并且给空的数值流计算最大值是没有意义的，因此max函数返回OptionalInt，他是Optional的一个子类，能够判断流是否为空，并对流为空的情况做相应的处理。 
此外mapToInt、mapToDouble、mapToLong进行数值操作后的返回结果分别为：OptionalInt、OptionalDouble、OptionalLong

#### 3. 收集器简介
收集器用来将经过筛选、映射的流进行最后的处理，可以使得最后的结果以不同的形式展现。 
collect方法即为收集器，它接收Collector接口的实现作为具体收集器的收集方法。 
Collector接口提供了很多默认实现的方法，我们可以直接使用它们格式化流的结果；也可以自定义Collector接口的实现，从而定制自己的收集器。

#### 4. 收集器的使用
##### 4.1 归约
流由一个个元素组成，归约就是将一个个元素“折叠”成一个值，如求和、求最值、求平均值都是归约操作。
##### 4.1.1 计数
```Java
long count = list.stream()
.collect(Collectors.counting());
```
也可以不使用收集器的计数函数：
```Java
long count = list.stream().count();
```
注意：计数结果一定是long类型。
##### 4.1.2最值
例如：找出所有人中年龄最大的人
```Java
Optional<Person> oldPerson = list.stream()
.collect(Collectors.maxBy(Comparator.comparingInt(Person::getAge)));
```
计算最值需要使用Collector.maxBy和Collector.minBy,这两个函数需要传入一个比较器Comparator.comparingInt，这个比较器又要接收需要比较的字段。这个接收器将会返回一个Optional类型的值。
##### 4.1.3 求和
例如：计算所有人年龄的总和
```Java
int summing = list.stream()
.collect(Collectors.summingInt(Person::getAge));
```
java8还提供了summingLong、summingDouble。
##### 4.1.4求平均值
例如：计算所有人的年龄平均值
```Java
double avg = list.stream()
.collect(Collectors.averagingInt(Person::getAge));
```
注意：计算平均值时，不论对象是int、long、double，计算结果一定都是double。

##### 4.1.5连接字符串
```Java
String names = list.stream()
.collect(Collectors.joining(","));
```
设置指定分隔符为","，默认为空格
##### 4.1.6一般性的归约操作
若需要自定义一个归约操作，那么需要使用Collector.reducing函数，该函数接收三个参数：
- 第一个参数为归约的初始值
- 第二个参数为归约操作进行的字段
- 第三个参数为归约操作的过程 
例如：计算所有人年龄的总和 

```Java
Optional<Integer> sumAge = list.stream()
.collect(Collectors.reducing(0,Person::getAge,(i,j)->i+j));
```
Collectors.reducing方法还提供了一个单参数的重载形式。 
你只需传一个归约的操作过程给该方法即可(即第三个参数)，其他两个参数均使用默认值。

- 第一个参数默认为流的第一个元素
- 第二个参数默认为流的元素 
这就意味着，当前流的元素类型为数值类型，并且是你要进行归约的对象。
例如：采用单参数的reducing计算所有人的年龄总和

```Java
Optional<Integer> sumAge = list.stream()
.filter(Person::getAge)
.collect(Collectors.reducing((i,j)->i+j));
```
##### 4.2分组
分组就是讲流中的元素按照指定类别进行划分，类似于Sql语句中的GroupBy 
例如：将所有人按照职业分组 
```Java
Map<String, List<Person>> collect = list.stream().collect(Collectors.groupingBy(Person::getCareer));
```
groupingby函数接收一个Lambda表达式，该表达式返回String类型的字符串，groupingby会将当前流中的元素按照Lambda返回的字符串进行分组。 
分组结果是一个Map< String,List< Person>>，Map的键就是职业名，Map的值就是该组的Perosn集合。

##### 4.2.1一级分组
例如：将所有人分为老年人、中年人、青年人 
```Java
Map<String,List<Person>> result = list.stream()
.collect(Collectors.groupingby((person)->{
if(person.getAge()>60)
return "老年人";
else if(person.getAge()>40)
return "中年人";
else
return "青年人";
}));
```
##### 4.2.2多级分组
多级分组可以支持在完成一次分组之后，分别对每个小组再次进行分组。 
使用具有两个参数的groupingby重载方法即可实现多级分组。
- 第一个参数：一级分组的条件
- 第二个参数：一个新的groupingby函数，该函数包含二级分组的条件
例如：将所有人分为老年人、中年人、青年人，并且将每个小组再分成：男女两组。 

```Java
Map<String,Map<String,List<Person>>> result = list.stream()
.collect(Collectors.groupingby((person)->{
if(person.getAge()>60)
return "老年人";
else if(person.getAge()>40)
return "中年人";
else
return "青年人";
},
groupingby(Person::getSex)));
```
##### 4.2.3对分组进行统计
拥有两个参数的groupingby函数不仅仅能够实现多几分组，还能对分组的结果进行统计 
例如：将所有人分为老年人、中年人、青年人统计每一组的人数

```Java
Map<String,Long> result = list.stream()
.collect(Collectors.groupingby((person)->{
if(person.getAge()>60)
return "老年人";
else if(person.getAge()>40)
return "中年人";
else
return "青年人";
},
counting()));
```
##### 4.2.4将收集器的结果转换成另一种类型
当使用maxBy、minBy统计最值时，结果会封装在Optional中，该类型是为了避免流为空时计算的结果也为空的情况。在单独使用maxBy、minBy函数时确实需要返回Optional类型，这样能确保没有空指针异常。然而当我们使用groupingBy进行分组时，若一个组为空，则该组将不会被添加到Map中，从而Map中的所有值都不会是一个空集合。既然这样，使用maxBy、minBy方法计算每一组的最值时，将结果封装在optional对象中就显得有些多余。 
我们可以使用collectingAndThen函数包裹maxBy、minBy，从而将maxBy、minBy返回的Optional对象进行转换。 
例如：将所有人按性别划分，并计算出每组最大的年龄。

```java
Map<String,Integer> map = list.stream()
.collect(groupingBy(Person::getSex,
collectingAndThen(
maxBy(comparingInt(Person::getAge)),
Optional::get
)));
```
此时返回的是一个Map< String,Integer>，String表示每组的组名(男、女)，Integer为每组最大的年龄。 
如果不用collectingAndThen包裹maxBy，那么最后返回的结果为Map< String,Optional< Person>>。 
使用collectingAndThen包裹maxBy后，首先会执行maxBy函数，该函数执行完后便会执行Optional::get，从而将Optional中的元素取出来

#### 总结：
Stream的特性可以归纳为：
- 不是数据结构
- 它没有内部存储，它只是用操作管道从 source（数据结构、数组、generator function、IO channel）抓取数据。
- 它也绝不修改自己所封装的底层数据结构的数据。例如 Stream 的 filter 操作会产生一个不包含被过滤元素的新 Stream，而不是从 source 删除那些元素。
- 所有 Stream 的操作必须以 lambda 表达式为参数
- 不支持索引访问
- 很容易生成数组或者 List
- 惰性化(很多 Stream 操作是向后延迟的，一直到它弄清楚了最后需要多少数据才会开始。)
- 当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的。
- 可以是无限的(集合有固定大小，Stream 则不必。limit(n) 和 findFirst() 这类的 short-circuiting 操作可以对无限的 Stream 进行运算并很快完成。)



