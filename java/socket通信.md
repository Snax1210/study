
## Java 通信基础

<!-- TOC -->

- [Java 通信基础](#java-通信基础)
- [Java IO 通信基本示例](#java-io-通信基本示例)
    - [socket 指定长度发送数据](#socket-指定长度发送数据)
        - [半包](#半包)
        - [粘包](#粘包)
        - [分包](#分包)
        - [小示例](#小示例)
- [Java NIO](#java-nio)
    - [Channel](#channel)
    - [Buffer](#buffer)
    - [Selector](#selector)
    - [实战](#实战)
    - [Buffer的使用](#buffer的使用)

<!-- /TOC -->
socket 通信是基于TCP/IP 网络层上的一种传送方式，我们通常把TCP和UDP称为传输层

![](http://c.avalon-zheng.xin/20191105150316.png "")

七个层级关系中，我们将的socket属于传输层，其中UDP是一种面向无连接的传输层协议。UDP不关心对端是否真正收到了传送过去的数据。如果需要检查对端是否收到分组数据包，或者对端是否连接到网络，则需要在应用程序中实现。UDP常用在分组数据较少或多播、广播通信以及视频通信等多媒体领域。在这里我们不进行详细讨论，这里主要讲解的是基于TCP/IP协议下的socket通信。

socket是基于应用服务与TCP/IP通信之间的一个抽象，他将TCP/IP协议里面复杂的通信逻辑进行分装，对用户来说，只要通过一组简单的API就可以实现网络的连接。借用网络上一组socket通信图给大家进行详细讲解：

![](http://c.avalon-zheng.xin/20191105150457.png "")



## Java IO 通信基本示例

服务器端

```java
public static void main(String[] args) {

try {
            // 初始化服务端socket并且绑定9999端口
            ServerSocket serverSocket  =new ServerSocket(9999);
            //等待客户端的连接
            Socket socket = serverSocket.accept();
            //获取输入流
            BufferedReader bufferedReader =new BufferedReader(new InputStreamReader(socket.getInputStream()));
            //读取一行数据
            String str = bufferedReader.readLine();
            //输出打印
            System.out.println(str);

        }catch (IOException e) {

e.printStackTrace();

        }
    }
}
```

客户端

```java
public class ClientSocket {

public static void main(String[] args) {

try {
            Socket socket =new Socket("127.0.0.1",9999);
            BufferedWriter bufferedWriter =new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
            String str="你好，这是我的第一个socket";
            bufferedWriter.write(str);
        }catch (IOException e) {

e.printStackTrace();

        }
    }
}

```

![](http://c.avalon-zheng.xin/20191105144535.png "")

BlockIng I/O模式下，主要缺点如下：

1. 只能用于小规模下多个socket通信，因为客户端socket每次连接请求后，服务端ServerSocket都会创建一个线程来处理当前客户端的连接请求，如果连接数非常大，以千万级为单位，那么服务端的CPU资源开销会是一个非常庞大的数据。

2. Read、Write读写资源问题，由于是阻塞的读写模式，如果大量线程处于空闲状态没有数据可读写，则会造成空闲socket的Read 、Write操作大量阻塞，对系统资源线程的开销也会造成非常大的浪费。

### socket 指定长度发送数据

在实际应用中，socket发送的数据并不是按照一行一行发送的

在实际应用中，网络的数据在TCP/IP协议下的socket都是采用数据流的方式进行发送，那么在发送过程中就要求我们将数据流转出字节进行发送，读取的过程中也是采用字节缓存的方式结束。那么问题就来了，在socket通信时候，我们大多数发送的数据都是不定长的，所有接受方也不知道此次数据发送有多长，因此无法精确地创建一个缓冲区（字节数组）用来接收，在不定长通讯中，通常使用的方式时每次默认读取8*1024长度的字节，若输入流中仍有数据，则再次读取，一直到输入流没有数据为止。但是如果发送数据过大时，发送方会对数据进行分包发送，这种情况下或导致接收方判断错误，误以为数据传输完成，因而接收不全。在这种情况下就会引出一些问题，诸如半包，粘包，分包等问题，

#### 半包

接受方没有接受到一个完整的包，只接受了部分。

原因：TCP为提高传输效率，将一个包分配的足够大，导致接受方并不能一次接受完。

影响：长连接和短连接中都会出现

#### 粘包

发送方发送的多个包数据到接收方接收时粘成一个包，从接收缓冲区看，后一包数据的头紧接着前一包数据的尾。

分类：一种是粘在一起的包都是完整的数据包，另一种情况是粘在一起的包有不完整的包

出现粘包现象的原因是多方面的:

1)发送方粘包：由TCP协议本身造成的，TCP为提高传输效率，发送方往往要收集到足够多的数据后才发送一包数据。若连续几次发送的数据都很少，通常TCP会根据优化算法把这些数据合成一包后一次发送出去，这样接收方就收到了粘包数据。

2)接收方粘包：接收方用户进程不及时接收数据，从而导致粘包现象。这是因为接收方先把收到的数据放在系统接收缓冲区，用户进程从该缓冲区取数据，若下一包数据到达时前一包数据尚未被用户进程取走，则下一包数据放到系统接收缓冲区时就接到前一包数据之后，而用户进程根据预先设定的缓冲区大小从系统接收缓冲区取数据，这样就一次取到了多包数据。

#### 分包


分包（1）：在出现粘包的时候，我们的接收方要进行分包处理；

分包（2）：一个数据包被分成了多次接收；

原因：1. IP分片传输导致的；2.传输过程中丢失部分包导致出现的半包；3.一个包可能被分成了两次传输，在取数据的时候，先取到了一部分（还可能与接收的缓冲区大小有关系）。

影响：粘包和分包在长连接中都会出现

那么如何解决半包和粘包的问题，就涉及一个一个数据发送如何标识结束的问题，通常有以下几种情况

固定长度：每次发送固定长度的数据；

特殊标示：以回车，换行作为特殊标示；获取到指定的标识时，说明包获取完整。

字节长度：包头+包长+包体的协议形式，当服务器端获取到指定的包长时才说明获取完整；

#### 小示例

所以大部分情况下，双方使用socket通讯时都会约定一个定长头放在传输数据的最前端，用以标识数据体的长度，通常定长头有整型int，短整型short，字符串Strinng三种形式。

```java
public class ServerSocketTest {

public static void main(String[] args) {

try {
            ServerSocket serverSocket =new ServerSocket(9999);
                Socket client = serverSocket.accept();
                InputStream inputStream = client.getInputStream();
                DataInputStream dataInputStream =new DataInputStream(inputStream);
                while (true){
                    byte b = dataInputStream.readByte();
                    int len = dataInputStream.readInt();
                    byte[] data =new byte[len -5];
                    dataInputStream.readFully(data);
                    String str =new String(data);
                    System.out.println("获取的数据类型为："+b);
                    System.out.println("获取的数据长度为："+len);
                    System.out.println("获取的数据内容为："+str);

                }
    }catch (IOException e) {
    e.printStackTrace();
        }
    }
}
```

客户端

```java

public class ClientSocketTest {

public static void main(String[] args) {
try {
            Socket socket =new Socket("127.0.0.1",9999);
            OutputStream outputStream = socket.getOutputStream();
            DataOutputStream dataOutputStream =new DataOutputStream(outputStream);
            Scanner scanner =new Scanner(System.in);
            if(scanner.hasNext()){
                String str = scanner.next();
                int type =1;
                byte[] data = str.getBytes();
                int len = data.length +5;
                dataOutputStream.writeByte(type);
                dataOutputStream.writeInt(len);
                dataOutputStream.write(data);
                dataOutputStream.flush();
            }
    }catch (IOException e) {
        e.printStackTrace();
        }
    }
}
```



## Java NIO

![](http://c.avalon-zheng.xin/20191105144610.png "")

NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

### Channel

首先说一下Channel，国内大多翻译成`通道`。Channel和IO中的Stream(流)是差不多一个等级的。只不过Stream是单向的，譬如：InputStream, OutputStream.而Channel是双向的，既可以用来进行读操作，又可以用来进行写操作。
NIO中的Channel的主要实现有：

- FileChannel

文件IO

- DatagramChannel

UDP

- SocketChannel

Client

- ServerSocketChannel

Server



### Buffer

NIO中的关键Buffer实现有：ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBuffer, LongBuffer, ShortBuffer，分别对应基本数据类型: byte, char, double, float, int, long, short。当然NIO中还有MappedByteBuffer, HeapByteBuffer, DirectByteBuffer等。

### Selector

Selector运行单线程处理多个Channel，如果你的应用打开了多个通道，但每个连接的流量都很低，使用Selector就会很方便。例如在一个聊天服务器中。要使用Selector, 得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新的连接进来、数据接收等。

总结如下:

![](http://c.avalon-zheng.xin/20191105144653.png "")


### 实战

服务端

```java
public class ServerSocketChannel2 {
    public static void main(String[] args) {

        try {
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.bind(new InetSocketAddress(9000));
            serverSocketChannel.configureBlocking(false);

            while (true){
                SocketChannel socketChannel = serverSocketChannel.accept();
                if(socketChannel!=null){
                    System.out.println("有新的客户端进来连接");
                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                    while (socketChannel.read(byteBuffer)!=-1){
                            byteBuffer.flip();
                            System.out.println(Charset.defaultCharset().newDecoder().decode(byteBuffer).toString());
                            byteBuffer.clear();
                    }

                }


            }

        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}

```

客户端

```java
public class SocketChannel2 {
    public static void main(String[] args) {

        try {
            SocketChannel socketChannel = SocketChannel.open();
            socketChannel.connect(new InetSocketAddress("127.0.0.1",9000));
            socketChannel.configureBlocking(false);
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
            String str = null;
            while ((str=bufferedReader.readLine())!=null){
                ByteBuffer byteBuffer = ByteBuffer.allocate(str.length());
               byteBuffer.put(str.getBytes());
               byteBuffer.flip();
               socketChannel.write(byteBuffer);
               byteBuffer.clear();

            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```


### Buffer的使用

从案例2中可以总结出使用Buffer一般遵循下面几个步骤：

- 分配空间

ByteBuffer buf = ByteBuffer.allocate(1024);

- 写入数据到Buffer

int bytesRead = fileChannel.read(buf);

- 调用filp()方法

 buf.flip();

```
比如说我们在Buffer中有一个`helloword`字符串。并且我们Buffer的 capacity设置的为20，则：
当前buffer的各项值：
position = 9
limit = 20
capacity = 20
执行flip后：
position = 0
limit = 9
capacity = 20
执行clear 后：
position = 0
limit = 20
capacity = 20
```

 > 也就是说调用flip之后，读写指针指到缓存头部，并且设置了最多只能读出之前写入的数据长度(而不是整个缓存的容量大小)

- 从Buffer中读取数据

(char)buf.get();

- 调用clear()方法或者compact()方法

Buffer顾名思义：缓冲区，实际上是一个容器，一个连续数组。Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer。

- 从Channel写到Buffer

fileChannel.read(buf)

- 通过Buffer的put()方法

buf.put(…)

- 从Buffer读取到Channel

channel.write(buf)

- 使用get()方法从Buffer中读取数据

buf.get()


