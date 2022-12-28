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

避坑指南：defer函数紧跟在资源打开后面，否则defer可能得不到执行，导致内存泄露。

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

