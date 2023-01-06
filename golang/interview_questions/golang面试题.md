# golang面试题

## 一、基础部分

### 1、golang中make和new的区别

**共同点：** 给变量分配内存

**不同点：**

1. 作用变量类型不同，new给string，int和数组分配内存，make给切片，map，channel分配内存；
2. 返回类型不一样，new返回指向变量的指针，make返回变量本身；
3. new分配的空间被清零。make分配空间后，会进行初始化。
4. new是在堆上分配内存，make是在栈上分配内存。创建make函数的时候，
   会将make函数的参数压入栈中，然后调用make函数，make函数执行完毕后，
   会将make函数的参数出栈。而new函数不会将参数压入栈中，而是直接在堆上分配内存。

### 2、数组和切片的区别

**相同点：**

1. 只能存储一组相同类型的数据结构
2. 都是通过下标来访问，并且有容量长度，长度通过len获取，容量通过cap获取

**区别：**

1. 数组是定长，访问和复制都不能超过数组定义的长度，否则就会下标越界，
   切片长度和容量可以自动扩容。
2. 数组是值类型，切片是引用类型，每个切片都引用了一个底层数组，切片本身不能
   存储任何数据，都是这底层数组存储数据，所以修改切片的时候修改的是底层数组中的
   数据。切片一旦扩容，指向一个新的底层数组，内存地址也就随之改变。

**简洁的回答：**

1. 定义方式不一样
2. 初始化方式不一样，数组需要指定大小，大小不改变
3. 在函数传递中，数组切片都是值传递。

**数组的定义**

```go
var a1 [3]int
var a2 [...]int{1,2,3}
```

**切片的定义**

```go
var a1 []int
var a2 := make([]int,3,5)
```

**数组的初始化**

```go
a1 := [...]int{1,2,3}
a2 := [3]int{1,2,3}
```

**切片的初始化**

```go
b := make([]int,3,5)
```

### 3、for range的时候它的地址会发生变化么？

在for a,b := range c 遍历中，a和b在内存中只会存在一份，即之后每次循环时遍历到的
数据都是以值覆盖的方式赋给a和b，a，b的内存地址始终不变。由于有这个特性，for
循环里面如果开协程，不要直接把a或者b的地址传给协程。解决办法：在每次循环时，
创建一个临时变量。

### 4、go defer ， 多个defer的顺序，defer在什么时机会修改返回值？

作用： defer延迟函数，释放资源，收尾工作；如释放锁，关闭文件，关闭连接；捕获panic；

避坑指南：defer函数紧跟在资源打开后面，否则defer可能得不到执行，导致内存泄漏。

多个defer调用顺序是LIFO（后入先出），defer后的操作可以理解为压入栈中

defer，return，return value（函数返回值）执行顺序：首先return，其次return value，
最后defer。defer可以修改函数最终返回值，修改时机：**有名返回值或者函数返回指针**，参考：

[](https://link.zhihu.com/?target=https%3A//blog.csdn.net/Cassie_zkq/article/details/10856720533)

**有名返回值**

```go
func b()(i int) {
    defer func() {
        i++
        fmt.println("defer2:", i)
    }()
    defer func() {
        i++
        fmt.println("defer1:", i)
    }()
    return i
}

func main(){
    fmt.println("return:", b())
}
```

**函数返回指针**

```go
func c() *int {
	var i int
	defer func() {
		i++
		fmt.Println("defer2:", i)
	}()
	defer func() {
		i++
		fmt.Println("defer1:", i)
	}()
	return &i
}
func main() {
	fmt.Println("return:", *(c()))
}
```

### 5、 uint类型溢出问题

超过最大存储值如uint8最大是255

var a uint8 = 255

var b uint8 = 1

a+b = 0 总之类型溢出会出现难以预料的结果

| 类型 | 有无符号 | 占用存储空间 | 表数范围 | 备注 |
|------|------|-------------------------|---------------------------------------|--|
| int | 有 | 32位系统4个字节<br/>64位系统8个字节 | -2^31^ ~ 2^31^-1<br/>-2^63^ ~ 2^63^-1 | |
| uint | 无 | 32位系统4个字节<br/>64位系统8个字节 | 0 ~ 2^32^-1<br/> 0 ~ 2^64^-1 | |
| rune | 有 | 与int32一样 | -2^31^ ~ 2^31^-1 | 等价int32，表示一个Unicode码 |
| byte | 无 | 与uint8等价 | 0~255 |当要存储字符时用 |

### 6、能介绍下rune类型吗？

相当int32

golang中的字符串底层实现是通过byte数组实现的，中文字符在Unicode下占2个字节，
在utf-8编码下占3个字节，而golang默认编码正好是utf-8。

byte等同于int8，常用来处理ascii字符；

rune等同于int32，常用来处理unicode或utf-8字符。

```go
package main
import (
    "fmt"
    "unicode/utf8"
)

func main() {
   var str = "hello 你好“
   //golang中string的底层是通过byte数组实现的，所以直接求len 实际是在按字节计算
   //一个汉字占3个字节算了3个长度
   fmt.Println("len(str):", len(str)) //12
   //以下两种都可以得到str的字符串长度
   //golang中的unicode/utf8包提供了用utf-8获取长度的方法
    fmt.Println("RuneCountInString(str):", utf8.RuneCountInString(str)) //8
    //通过rune类型处理unicode字符
    fmt.Println("len([]rune(str)):", len([]rune(str))) //8
}
```

### 7、golang中解析tag是怎么实现的？反射原理是什么？

```go
type User struct {
   name string `json:name-field`
   age int
}

func main() {
   user := &User{"John Doe The Fourth", 30}
   
   field, ok := reflect.TypeOf(user).Elem().FieldByName("name")
    if !ok {
        panic("Field not found")
    }
    fmt.Println(getStructTag(field))
}

func getStructTag(f reflect.StructField) string {
    return string(f.Tag)
}
```

Go中解析的tag是通过反射实现的，反射是指计算机程序运行时（Run time）可以访问
、检测和修改它本身状态或行为的一种能力或动态获取给定数据对象的类型和结构，并
有机会修改它。反射将接口变量转换成反射对象Type和Value；反射可以通过反射对象
Value还原成原先的接口变量；反射可以用来修改一个变量的值，前提是这个值可以被
修改；tag是啥：结构体支持标记，name string `json:name-field`就是`json:name-field`
这部分。

**gorm json yaml gRPC protobuf gin.Bind() 都是通过反射来实现的**

### 8、调用函数传入结构体时，应该传值还是指针？（Golang都是传值）

Go函数参数传递都是值传递。所谓值传递，指在调用函数时将实际阐述复制一份传递到
函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。参数传递还有引用传递，
所以引用传递是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数
所进行的修改，将影响到实际参数。

因为Go里面的map，slice，chan是引用类型。变量区分值类型和引用类型。所谓值类型：
变量和变量的值存在同一个位置。所谓引用类型：变量和变量的值是不同的位置，变量
的值存储的是对值的引用。但并不是map，slice，chan的所有的变量在函数内都能被修改，
不同数据类型的底层存储结构和实现可能不太一样，情况也就不一样。

### 9、 讲讲Go的slice底层数据结构和一些特性

答：Go的slice底层数据结构是由一个array指针指向底层数组，len表示切片长度，cap表示
切片容量。slice的主要实现是扩容。对于append向slice添加元素时，假如slice容量够用，
则追加新的元素进去，slice.len++，返回原来的slice。当原容量不够，则slice先扩容，
扩容之后slice得到新的slice，将元素追加进新的slice，slice.len++，返回新的slice。对
于切片的扩容规则，当切片比较小时（容量小于1024），则采用较大的扩容倍数进行
扩容（新的扩容回事原来的2倍），避免频繁扩容，从而减少内存分配次数和数据拷贝的
代价。当切片较大的时候（原来的slice容量大于或者等于1024），采用较少的扩容倍数
（新的扩容将扩大大于或者冬雨原来的1.25倍），主要是避免空间浪费，网上其实很多
总结的是1.25倍，那是在不考虑内存对齐的情况下，实际上还要考虑内存对齐，扩容是
大于或者等于1.25倍。

（关于刚才问的slice为什么传到函数内可能被修改，如果slice在函数内没有出现扩容，
函数外和函数内slice变量指向是同一个数组，则函数内复制的slice变量值出现修改，函
数外这个slice变量值也会被修改。如果slice在函数内出现扩容，则函数内变量的值会新
生成一个数组（也就是新的slice）而函数外的slice指向的还是原来的slice，则函数内的
修改不会影响函数外的slice。）

### 10、 讲讲Go的select底层数据结构和一些特性？

答，go 的select为golang提供了多路IO复用机制，和其他IO复用一样，用于检测是否
有读写时间是否ready。linux的系统IO模型有select，poll，epoll，go的select和linux
系统select非常相似。

select结构组成主要是由case语句和执行的函数组成的select实现的多路复用是：每个
线程或者进程都先到注册和接收的channel（装置）注册，然后阻塞，然后只有一个线程
在运输，当注册的线程和进程准备好数据后，装置会根据注册的信息得到相应的数据。

**select的特性**

1. select操作至少要有一个case语句，出现读写nil的channel该分支会忽略，在nil的
channel上操作则会报错。
2. select仅支持管道，而且是单协程操作。
3. 每个case语句仅能处理一个管道，要么读要么写。
4. 多个case语句的执行顺序是随机的。
5. 存在default语句，select将不会阻塞，但存在default会影响性能。

### 11、讲讲Go的defer层数据结构和一些特性？

答： 每个defer语句都对应一个_defer实例，多个实例使用指针连接起来形成一个单链表，
保存在goroutine数据结构中，每次插入_defer实例，均插入到链表的头部，函数结束再
一次从头部取出，从而形成后进先出的效果。

**defer的规则总结：**

1. 延迟函数的参数是defer语句出现的时候就已经确定了的。
2. 延迟函数执行的按照后进先出的顺序执行，即先出现的defer最后执行。
3. 延迟函数可能操作主函数的返回值。
4. 申请资源后立即使用defer关闭资源是个好习惯。

### 12、单引号、双引号、反引号的区别？

单引号，表示byte类型或者rune类型，对应uint8和int32类型，默认是rune类型。byte
用来强调数据是raw data，而不是数字；而rune用来表示Unicode的code point。

双引号，才是字符串，实际上是字符数组，可以用索引号访问某字节， 也可以用len\()
函数来获取字符串所占字节长度。

反引号，表示字符串字面量，但不支持任何转义序列。字面量 raw literal string的意思
是，你定义时写的啥样，它就啥样，你有换行，它就换行。你写转义字符，它也就展示
转义字符。

## 二、map相关

### 1、 map使用注意的点，是否并发安全？

map的类型是map\[key]，key的类型必须是可比较的，通常情况，会选择内建的基本类型，
比如整数、字符串做key的类型。如果要使用struct作为key，要保证struct对象在逻辑上
是不可变的。在Go语言中，map\[key]函数返回的结果可以是一个值，也可以是两个值。
map是无序的，如果我们想要保证遍历map时元素有序，可以使用辅助的数据结构，例如
orderedmap。

**第一，** 一定要先初始化，否则panic。

**第二，** map类型是容易发生并发访问问题的。不注意就容易发生程序运行时并发
读写导致的panic。Go语言内建的map对象不是线程安全的，并发读写的时候运行时会有
检查，遇到并发问题就会导致panic。

### 2、map循环是有序的还是无序的？

无序的，map因扩张而重新哈希时，各键值项存储位置都可能会发生改变，顺序自然也
没法保证了，所以官方避免大家依赖顺序，直接打乱处理。就是 for range map在开始
处理循环逻辑的时候，就做了随机播种。

### 3、map中删除一个key，它的内存能释放吗？

如果删除的元素是值类型，如int，float，bool，string以及数组和struct，map 的内
存不会自动释放

如果删除的元素是引用类型，如指针，slice，map，chan等，map的内存会自动释放，但
释放的内存是子元素引用类型的内存占用

将map设置为nil，内存被回收。

### 4、 怎么处理对map进行并发访问？有没有其他方案，区别是什么？

使用sync.Map，它是一个并发安全的map，可以使用sync.Map的Store、Load、LoadOrStore、
Delete、Range方法来进行操作。

 [](https://studygolang.com/articles/22778)

### 5、 nil map和空map有何不同？

1. 可以对未初始化的map进行取值，但取出来的东西是空：

```go
   var m1 map[string]string
   fmt.Println(m1["1"])
```

2. 不能对未初始化的map进行赋值，会报错：

```go
   var m1 map[string]string
   m1["1"] = "1" // panic: assignment to entry in nil map
```
3. 通过fmt打印map时，空map和nil map结果是一样的，都为map[]。所以，这个时候别
断定map是空还是nil，而应该通过map==nil来判断。

nil map 未初始化，空map是长度为空

### 6、 map的数据结构时什么？是怎么实现扩容？

答： golang中map是一个kv对集合。底层使用hash table，用链表来解决冲突，出现冲
突时，不是每一个key都申请一个结构通过链表串起来，而是以bmap为最小粒度挂载，
一个bmap可以放8个kv。在哈希函数的选择上，会在程序启动时，检测cpu是否支持aes
，如果支持，则使用aes hash，否则使用memhash。每个map的底层结构是hmap，是由
若干个结构为bmap的bucket组成的数组。每个bucket底层都采用链表结构。

**hmap的结构如下：**

```go
type hmap struct {
    count     int                  // 元素个数
    flags     uint8
    B         uint8                // 扩容常量相关字段B是buckets数组的长度的对数 2^B
    noverflow uint16               // 溢出的bucket个数
    hash0     uint32               // hash seed
    buckets    unsafe.Pointer      // buckets 数组指针
    oldbuckets unsafe.Pointer      // 结构扩容的时候用于赋值的buckets数组
    nevacuate  uintptr             // 搬迁进度
    extra *mapextra                // 用于扩容的指针
}
```

**map的容量大小**

底层调用makemap函数，计算得到合适的B，map的容量最多可容纳6.5^B个元素，6.5为
装载因子阈值常量。装载因子的计算公式是：装载因子=填入表中的元素个数/散列表的
长度，装载因子越大，说明空闲位置越少，冲突越多，散列表的性能会下降。

**触发map扩容的条件**

1. 装载因子超过阈值，源码里定义的阈值是6.5
2. overflow 的bucket 数量过多map的bucket定位和key的定位高八位用于定位bucket，
低八位用于定位key，快速试错后再进行完整对比

### 7、 slices能作为map的类型的key吗？

其实是这个问题的变种： golang那些类型可以作为map的key？

答案是：**在golang规范中，可比较的类型都可以作为map key；** 这个问题又延伸
到：在golang规范中，哪些数据类型可以比较？

**不能作为map key 的类型包括：**

- slices
- maps
- functions

详细参考：

[](https://blog.csdn.net/lanyang123456/article/details/123765745)

## 三、context相关

### 1、context结构是什么样的？context使用场景和用途？

参考链接：

[](https://www.cnblogs.com/juanmaofeifei/p/14439957.html)

答： Go的Context的数据结构包含Deadline，Done，Err，Value，Deadline方法返回
一个time.Time，表示当前Context应该结束的时间，ok则表示有结束的时间，Done方
法当Context被取消或者超时的时候返回一个close 的channel，告诉给context相关的
函数要停止当前工作然后返回了，Err表示context被取消的原因，Value方法表示context
实现共享数据存储的地方，是协程安全的。context在业务中是经常被使用的。

**其主要的应用：**

1. 上下文控制
2. 多个goroutine之间的数据交互等
3. 超时控制：到某个时间点超时，多久超时。

## 四、channel相关

### 1、channel是否线程安全？锁用在什么地方？

线程安全：不同协程通过channel进行通信，本身使用场景就是多线程，为了保证数据的
一致性，必须实现线程安全。

channel的底层实现中，hchan结构体中采用Mutex锁来保证数据读写安全。在对循环数
组buf的数据进行入队和出队操作时，必须先获取互斥锁，才能操作channel数据。

### 2、go channel的底层实现原理（数据结构）

### 5、讲讲Go的chan底层数据结构和主要使用场景

答：channel的数据结构包含qccount当前队列中剩余元素个数，dataqsiz环形队列长度
，即可以存放的元素个数，buf环形队列指针，elemsize每个元素的大小，closed标识
关闭转台，elemtype元素类型，sendx队列下标，指示元素写入时存放到队列中的位置，
recvx队列下标，指示元素从队列的该位置读出。recvq等待读消息的goroutine队列，
sendq等待写消息的goroutine队列，lock互斥锁，chan不允许并发读写。

**无缓冲和有缓冲区别：** 管道没有缓冲区，从管道读数据会阻塞，直到有协程向管道
中写入数据。同样，向管道写入数据也会阻塞，知道有协程从管道读取数据。管道有缓
冲区，但缓冲区没有数据，从管道读取数据也会阻塞，直到协程写入数据，如果管道满了
，写数据也会阻塞，直到协程从缓冲区读取数据。

**channel的一些特点：**

1. 读写值nil管道会永久阻塞
2. 关闭的管道读数据仍然可以读数据
3. 关闭的管道写数据会panic
4. 关闭为nil 的管道会panic
5. 关闭已经关闭的管道会panic

**向channel写数据的流程：** 如果等待接收队列recvq不为空，说明缓冲区中没有数
据或者没有缓冲区，此时直接从recvq取出G，并把数据写入，最后把该G唤醒，结束发
送过程；如果缓冲区中有空余位置，将数据写入缓冲区，结束发送过程；如果缓冲区没
有空余位置，将待发送的数据写入G，将当前G加入sendq，进入睡眠，等待被读goroutine
唤醒；

**从channel读数据的流程：** 如果等待发送队列sendq不为空，且没有缓冲区，直接从
sendq中取出G，把G中数据读出，最后把G唤醒，结束读取过程；等待发送队列sendq不
为空，此时说明缓冲区已满，从缓冲区中首部读出数据，把G中数据写入缓冲区尾部，把
G唤醒，结束读取过程；如果缓冲区中有数据，则从缓冲区取出数据，结束读取过程；
将当前goroutine加入recvq，进入睡眠，等待被写goroutine唤醒；

**使用场景：** 消息传递、消息过滤、信号广播，事件订阅与广播，请求、响应转发、
任务转发、结果汇总，并发控制，限流，同步与异步。


## 五、GMP相关

### 1、什么是GMP？

答：G代表goroutine，P代表着上下文处理器，M代表thread线程，在GPM模型，有一个
全局队列（Global Queue）：存放等待运行的G，还有一个P的本地队列：也是存放等待
运行的G，但数量有限，不超过256个。GPM的调度流程从 go func\()开始创建一个goroutine，
新建的goroutine优先保存在P的本地队列汇总，如果P的本地队列已经满了，则会保存到
全局队列中。M会从P的队列中取一个可执行状态的G来执行，如果P的本地队列为空，就
会从其他的MP组合偷取一个可执行的G来执行，当M执行某一个G的时候发生系统调用
或者阻塞，M阻塞，如果这个时候G在执行，runtime会把这个线程M从P中摘除，然后
创建一个新的操作系统线程来服务于这个P，当M系统调用结束时，这个G会尝试获取一
个空闲的P来执行，并放入到这个P的本地队列，如果这个线程M变成休眠状态，加入到
空闲线程中，然后整个G就会被放入到全局队列中。

关于GPM的个数问题，G的个数理论上是无限制的，但是受内存限制，P的数量一般建议
是逻辑CPU数量的两倍，M的数据默认启动的时候是10000，内核很难支持这么多线程数，
所以整个限制客户忽略，M一般不做设置，设置好P，M一般都是要大于P。

### 2、进程、线程、协程有什么区别？

进程：是应用程序的启动实例，每个进程都有独立的内存空间，不同的进程通过进程间
的通信方式来通信。

线程：从数据进程，每个进程至少包含一个线程，线程是CPU调度的基本单位，多个线
程之间可以共享进程的资源并通过共享内存等线程间的通信方式来通信。

协程：为轻量级线程，与线程相比，协程不受操作系统的调度，协程的调度器由用户应
用程序提供，协程调度器按照调度策略把协程调度到线程中运行

### 3、抢占式调度是如何抢占的？

**基于协作的抢占式调度：**

- 编译器会在调用函数前插入 `runtime.morestack` ， 可能会调用 `runtime.newstack`
进行抢占。
- Go语言运行时会在垃圾回收暂停程序、系统监控返现Goroutine运行超过10ms时发出
抢占请求 `StackPreempt` 。
- 当发生函数调用时，可能会执行编译器插入的 `runtime.morestack` ，它调用的`runtime.newstack` 
会检查Goroutine的 `stackguard0` 字段是否为 `StackPreempt`。
- 如果是，就会触发抢占让出当前线程；

但是这种调度并不完备，比如一个goroutine运行了很久，但是它并没有调用另一个函数
，则它不会被抢占。

**基于信号的抢占式调度：**

- M注册一个SIGURG信号的处理函数：sighandler。
- sysmon线程检测到执行时间过长的goroutine，GC stw时，会向相应的M（或者说线
程，每一个线程对应一个M）发送SIGURG信号。
- 收到信号后，内核执行sighandler函数，通过pushCall插入asyncPreempt函数调用。
- 回到当前goroutine执行asyncPreempt函数，通过mcall切到g0栈执行gopreempt_m函数。
- 将单枪goroutine插入到全局可运行队列，M则继续寻找其他goroutine来运行。
- 被抢占的goroutine再次调度过来执行时，会继续原来的执行流。

### 4、M和P的数量问题？

p默认cpu内核数

M与P的数量没有绝对关系，一个M阻塞，P就会去创建或者切换另一个M，所以即使P的
默认数量是1，也有可能创建很多个M出来。

## 六、锁相关

### 1、除了mutex意外还有哪些方式安全读写共享变量？

- 将共享变量的读写放到一个goroutine中，其他goroutine通过channel进行读写操作。
- 可以用个数为1的信号量（Semaphore）实现互斥。
- 通过Mutex锁实现

### 2、Go如何实现原子操作？

答：原子操作就是不可中断操作，外界是看不到原子操作的中间状态，要么看到原子操作
已经完成，要么看到原子操作已经结束。在某个值的原子操作执行的过程中，CPU绝对
不会再去执行其他针对该值的操作，那么其他操作也是原子操作。

Go语言的标准库代码包sync/atomic提供了原子读取的操作（Load为前缀的函数）或
写入（Store为前缀的函数）某个值

**原子操作与互斥锁的区别：**

1. 互斥锁是一种数据结构，用来让一个线程执行程序的关键部分，完成互斥的多个操作。
2. 原子操作是针对某个值的单个互斥操作。

### 3、Mutex是悲观锁还是乐观锁？悲观锁、乐观锁是什么？

**悲观锁：**

当要对数据库中的一条数据进行修改的时候，为了避免同时被其他人修改，最好的办法
就是直接对数据进行加锁以防止并发。这种借助数据库锁机制，在修改数据之前先锁定
，再修改的方式被称为悲观并发控制【Pessimistic Concurrency Control，PCC】。

**乐观锁：**

乐观锁是相对悲观锁而言的，乐观锁假设数据一般不会造成冲突，所以在数据进行提交
更新的时候，才会正式对数据的冲突与否进行检测，如果冲突，则返回给用户异常信息，
让用户决定如何去做。乐观锁适用于读多写少的场景，这样可以提高程序的吞吐量。

### 4、Mutex有几种模式？

**正常模式：**

1. 当前的mutex只有一个goroutine来获取，那么没有竞争，直接返回。
2. 新的goroutine进来，如果当前mutex已经被获取了，则该goroutine进入一个先入先
出的waiter队列，在mutex被释放后，waiter按照先进先出的方式获取锁。该goroutine
会处于自选状态（不挂起，继续占有cpu）。
3. 新的goroutine进来，mutex处于空闲状态，将参与竞争。新来的goroutine有先天的
优势，它们正在CPU中运行，可能它们的数量还不少，所以，在高并发情况下，被唤醒
的waiter可能比较悲剧的获取不到锁，这时，它会被插入到队列的前面。如果waiter
获取不到锁的时间超过阈值1毫秒，那么这个Mutex就进入到了饥饿模式。

**饥饿模式：**

在饥饿模式下，Mutex的拥有者将直接把锁交给队列最前面的waiter。新来的goroutine
不会尝试获取锁，及时看起来锁没有被持有，它也不会去抢，也不会自旋（spin），它
会乖乖的加入到等待队列的尾部。如果拥有mutex的waiter发现下面两种情况之一，它
就会把这个Mutex转换成正常模式：

1. 此waiter已经是队列中的最后一个waiter了，没有其他的等待锁的goroutine了。
2. 此waiter的等待时间小于1毫秒。

### 5、goroutine的自旋占用资源如何解决？

自旋锁是指当一个线程在获取锁的时候，如果锁已经被其他线程获取，那么该线程将
循环等待，然后不断地判断是否能够被成功获取，直到获取到锁才会退出循环。

**自旋的条件如下：**

1. 还没自旋超过4次。
2. 多核CPU。
3. GOMAXPROCS大于1。
4. p上本地goroutine队列为空。

mutex会让当前的goroutine去空转CPU，在空转完后再次调用CAS方法去尝试性的占用
锁资源，直到不满足自旋条件，则最终会加入到等待队列里。

## 七、并发相关

### 1、怎么控制并发数？

**第一，有缓冲通道**

根据通道中没有数据时读取操作陷入阻塞和通道已满时继续写入操作陷入阻塞的特性，
正好实现了控制并发数的功能。

```go
func main() {
	count := 10 // 最大支持并发
	sum := 100 // 任务总数
	wg := sync.WaitGroup{} //控制主协程等待所有子协程执行完之后再退出。

	c := make(chan struct{}, count) // 控制任务并发的chan
	defer close(c)

	for i:=0; i<sum;i++{
		wg.Add(1)
		c <- struct{}{} // 作用类似于waitgroup.Add(1)
		go func(j int) {
			defer wg.Done()
			fmt.Println(j)
			<- c // 执行完毕，释放资源
		}(i)
	}
	wg.Wait()
}
```

**第二、三方库实现的协程池**

panjf2000/ants（比较火）

Jeffail/tunny

```go
import (
	"log"
	"time"

	"github.com/Jeffail/tunny"
)
func main() {
	pool := tunny.NewFunc(10, func(i interface{}) interface{} {
		log.Println(i)
		time.Sleep(time.Second)
		return nil
	})
	defer pool.Close()

	for i := 0; i < 500; i++ {
		go pool.Process(i)
	}
	time.Sleep(time.Second * 4)
}
```

### 2、多个goroutine对同一个map写会panic，异常是否可以用defer捕获？

可以捕获异常，但是只能捕获一次，Go语言，可以使用多值返回来返回错误。不要用异
常代替错误，更不要用来控制流程。在极个别的情况下，才使用Go中引用的Exception
处理：defer，panic，recover。Go中，对异常处理的原则是：多用error包，少用panic

```go
defer func() {
		if err := recover(); err != nil {
			// 打印异常，关闭资源，退出此函数
			fmt.Println(err)
		}
	}()
```

## 八、GC相关

### 1、 go gc是怎么实现的？

**细分常见的三个问题：**

1. GC机制随着golang版本变化如何变化的？
2. 三色标记法的流程？
3. 插入屏障、删除屏障，混合写屏障（具体的实现比较难描述，但你要知道屏障的作用
：避免运行过程中，变量被误回收；减少STW的时间）

Go的GC回收有三次演进过程，Go 1.3之前普通标记清除（mark and sweep）方法，整
体过程需要启动STW，效率极低。1.5 三色标记法，堆空间启动写屏障，栈空间不启动，
全部扫描之后，需要重新扫描一次栈（需要STW），效率普通。 1.8三色标记法，混合
写屏障机制：栈空间不启动（全部标记为褐色），堆空间启用写屏障，整个过程不要
STW，效率高。

Go 1.3之前的版本所谓的标记清除是先启动STW暂停，然后执行标记，再执行数据回收
，最后停止STW。1.3版本标记清除做了点优化，流程是：先启动STW暂停，然后执行
标记，停止STW，最后再执行数据回收。

1.5 三色标记主要是插入屏障和删除屏障，写入屏障的流程：程序开始，标记为白色
1. 所有的对象放到白色集合
2. 遍历一次根节点，得到灰色节点
3. 遍历灰色节点，将可达的对象，从白色标记灰色，遍历之后的灰色标记成黑色
4. 由于并发特性，此刻外界向在堆中的对象发生添加对象，以及在栈中的对象添加对象
，在队中的对象会触发插入屏障机制，栈中的对象不触发。
5. 由于堆中对象插入屏障，则会把堆中黑色对象添加的白色对象改成灰色，栈中的黑色
对象添加的白色对象依然是白色。
6. 循环第五步，直到没有灰色节点。
7. 在准备回收白色之前，重新遍历扫描一次栈空间，加上STW暂停保护栈，防止外界
干扰（有新的白色会被添加成黑色）在STW中，将栈中的对象一次三色标记，直到没有
灰色
8. 停止STW，清除白色。至于删除写屏障，则是遍历灰色节点的时候出现可达的节点
被删除，这个时候触发删除写屏障，这个可达的被删除的节点也是灰色，等循环三色标记
之后，直到没有灰色节点，然后清理白色，删除写屏障会造成一个对象即使被删除了最
后一个指向它的指针也依旧可以活过这一轮，在下一轮GC中被清理掉。

1.8混合写屏障的规则是：

1. GC开始将栈上的对象全部扫描并标记为黑色（之后不再进行第二次重复扫描，无需
STW）
2. 在GC期间，任何在栈上创建的新对象，均为黑色
3. 被删除的对象标记为灰色
4. 被添加的对象标记为灰色

### 2、go的gc算法是怎么实现的？

```go
func GC() {
	n := atomic.Load(&amp;work.cycles)
	gcWaitOnMark(n)

	gcStart(gcTrigger{kind: gcTriggerCycle, n: n + 1})
	gcWaitOnMark(n + 1)

	for atomic.Load(&amp;work.cycles) == n+1 &amp;&amp; sweepone() != ^uintptr(0) {
		sweep.nbgsweep++
		Gosched()
	}
	for atomic.Load(&amp;work.cycles) == n+1 &amp;&amp; atomic.Load(&amp;mheap_.sweepers) != 0 {
		Gosched()
	}
	mp := acquirem()
	cycle := atomic.Load(&amp;work.cycles)
	if cycle == n+1 || (gcphase == _GCmark &amp;&amp; cycle == n+2) {
		mProf_PostSweep()
	}
	releasem(mp)
}
```

### 3、GC中stw时机，各阶段是如何解决的？

1. 在开始新的一轮GC周期前，需要调用gcWaitOnMark方法上一轮的标记结束（含扫描
终止、标记、或标记终止等）
2. 开始新的一轮GC周期，调用gcStart方法触发GC行为，开始扫描标记阶段。
3. 需要调用gcWaitOnMark方法等待，直到当前GC的扫描、标记、标记终止完成。
4. 需要调用sweepone方法，扫描未扫除的堆跨度，并持续扫除，保证清理完成。在等待
扫除完毕前的阻塞时间，会调用Gosched让出
5. 在本轮GC基本完成后，会调用mProf_PostSweep方法。以此记录最后一次标记终止
时堆配置文件快照。
6. 结束，释放M。

### 4、 GC的触发时机？

分为系统触发和主动触发。

1. gcTriggerHeap：当所分配的堆大小达到阈值（由控制器计算的触发堆的大小）时，
将会触发。
2. gcTriggerTime：当距离上一个GC周期的时间超过一定时间时，将会触发。时间周期
以runtime.forcegcperiod变量为准，默认为2分钟
3. gcTriggerCycle：如果没有开启GC，则启动GC。
4. 手动触发的runtime.GC方法。

## 九、内存相关

### 1、谈谈内存泄漏，什么情况下会内存泄漏？怎么定位排查内存泄漏问题？

答： go中的内存泄漏一般都是goroutine泄漏，就是goroutine没有被关闭，或者没有添
加超时控制，让goroutine一直处于阻塞状态，不能被GC。

**内存泄漏有下面一些情况：**

1. 如果goroutine在执行时被阻塞而无法退出，就会导致goroutine的内存泄漏，一个
goroutine的最低栈大小是2KB，在高并发的场景下，对内存的消耗也是非常恐怖的。
2. 互斥锁未释放或者造成死锁会造成内存泄漏
3. time.Ticker是每隔指定的时间就会向通道内写数据。作为循环触发器，必须调用
stop方法才会停止，从而被GC掉，否则会一直占用内存空间。
4. 字符串的截取引发的临时性的内存泄漏

```go
func main() {
	var str0 = "12345678901234567890"
	str1 := str0[:10]
}
```

5. 切片截取引起的子切片内存泄漏

```go
func main() {
	var s0 = []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := s0[:3]
}
```

6. 函数数组传参引发的内存泄漏（如果我们在函数传参的时候用到了数组传参，且这个
数组足够大（假设数组大小为100万，64位机上消耗的内存约为800w字节，即8M），
或者函数短时间内被调用N次，那么可想而知，会消耗大量内存，对性能产生极大的影响
，如果短时间内分配大量内存，而又来不及GC，那么就会产生临时性内存泄漏，对于
高并发场景厢房可怕。）

**排查方式：**

pprof是Go的性能分析工具，在程序运行过程中，可以记录程序的运行信息，可以是
CPU使用情况、内存使用情况、goroutine运行情况等，当需要性能调优或者定位Bug时
候，这些记录的信息是相当重要。

### 2、知道golang的内存逃逸吗？什么情况下会发生内存逃逸？

1. 本该分配到栈上的变量，跑到了堆上，这就导致了内存逃逸
2. 栈是高地址到低地址，栈上的变量，函数结束后变量会跟着回收掉，不会有额外的性
能开销
3. 变量从栈逃逸到堆上，如果要回收掉，需要进行GC，这就会导致额外的性能开销。
编程语言不断优化GC算法，主要目的都是为了减少gc带来的额外性能开销，变量一旦
逃逸会导致性能开销变大。

**内存逃逸的情况如下：**

1. 方法内返回局部变量指针。
2. 向channel发送指针或者包含指针的结构体。
3. 在闭包中引用包外的值。
4. 在slice或者map中存储指针。
5. 切片（扩容后）长度太大。
6. 在interface类型上调用方法。

### 3、请简述Go是如何分配内存的？

mcache mcentral mheap mspan

Go程序启动的时候申请一大块内存，并且划分spans，bitmap，areana区域；arena
区域按照页划分成一个个小块，span管理一个或者多个页，mcentral管理多个span供
线程申请试用；mcahe作为线程私有资源，来源于mcentral。

### 4、Channel分配在栈上还是堆上？哪些对象分配在堆上，哪些对象分配在栈上？

Channel被设计用来实现协程间通信的组件，其作用域和生命周期不可能仅限于某个函数
内部，所以golang直接将其分配在堆上。

golang中的变量只要被引用就一直会存活，存储在堆上还是栈上由内部实现决定而和具
体的语法没有关系。

当前情况下，如果一个变量被取址，那么它就有可能被分配到堆上，然而，还要对这些
变量做逃逸分析，如果函数return之后，变量不再被引用，则将其分配到栈上。

### 5、介绍一下大对象小对象，为什么小对象多了造成GC压力？

小于等于32k的对象就是小对象，其它都是大对象。一般小对象通过mspan分配内存；
大对象则直接由mheap分配内存。通常小对象过多会导致GC三色法消耗过多的CPU，优化
思路是，减少对象的分配。

小对象：如果申请小对象时，发现当前内存空间不存在空间跨度时，将会需要调用
nextFree方法获取新的可用的对象，可能会触发GC行为。

大对象：如果申请大于32K以上的大对象时，可能会触发GC行为。

## 十、其他问题

### 1、Go多返回值怎么实现的？

答：Go传参和返回值是通过FP+offset实现，并且存储在函数调用的栈帧中，FP栈底寄
存器，指向一个函数栈的顶部；PC程序计数器，指向下一条执行指令；SB指向静态数据
的基指针，全局符号；SP指向栈顶寄存器。

### 2、讲讲GO中主协程如何等待其余协程退出？

Go的sync.WaitGroup是等待一组协程结束，sync.WaitGroup只有3个方法，Add\()是
添加计数，Done\(\)是减少计数，Wait\(\)是阻塞等待计数为0。Go里面还能通过有
缓冲的channel实现其阻塞等待一组协程结束，这个能保证一组goroutine按照顺序执行，
但是不能并发执行。

### 3、Go语言中不同的类型如何比较是否相等？

答：像string，int，float，interface等可以通过reflect.DeepEqual\(\)和==来比较，
像slice，map，channel等不能通过==来比较，只能通过reflect.DeepEqual\(\)来比较。

### 4、Go中init函数的特征？

答：一个包下可以有多个init函数，每个文件也可以有多个init函数。多个init函数按照
它们的文件名顺序逐个初始化。应用初始化时初始化工作的顺序是：从被导入的最深层
包开始进行初始化，层层递出最后到main包。不管包被导入多少次，包内的init函数只
会执行一次。但包级别变量的初始化先于包内init函数的执行。

### 5、Go中uintptr和unsafe.Pointer的区别？

答：unsafe.Pointer是通用指针类型，不能参与计算，任何类型的指针都可以转化成
unsafe.Pointer，unsafe.Pointer可以转化成任何类型的指针。uintptr可以转换为unsafe.
Pointer，unsafe.Pointer可以转换为uintptr。uintptr是指针运算的工具，但是它不能持
有指针对象（意思就是它跟指针对象不能相互转换），unsafe.Pointer是指针对象进行
运算（也就是uintptr）的桥梁。
