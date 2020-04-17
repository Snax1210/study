Dubbo官网地址：http://dubbo.apache.org/zh-cn/

1.1 RPC
RPC 【Remote Procedure Call】 是指远程过程调用， 是一种进程间通信方式，是一种技术思想，而不是规范。 它允许程序调用另一个地址空间（网络的另一台机器上）的过程或函数，而不用开发人员显式编码这个调用的细节。 调用本地方法和调用远程方法一样。

RPC 的实现方式可以不同。例如 java 的 rmi, spring 远程调用等。

使用 PRC 可以将本地的调用扩展到远程调用（分布式系统的其他服务器）。
RPC 的特点
1. 简单：使用简单，建立分布式应用更容易。
2. 高效：调用过程看起来十分清晰，效率高。
3. 通用：进程间通讯的方式，有通用的规则。

1.2 RPC架构
一个完整的RPC架构里面包含了四个核心的组件，分别是Client，Client Stub，Server以及Server Stub，这个Stub可以理解为存根。

客户端(Client)，服务的调用方。

客户端存根(Client Stub)，存放服务端的地址消息，再将客户端的请求参数打包成网络消息，然后通过网络远程发送给服务方。

服务端(Server)，真正的服务提供者。

服务端存根(Server Stub)，接收客户端发送过来的消息，将消息解包，并调用本地的方法。

1.3 RPC调用过程

suntang > 9.第九期.dubbo.20190910 > image2019-9-10_20-24-38.png

(1) 客户端（client）以本地调用方式（即以接口的方式）调用服务；

(2) 客户端存根（client stub）接收到调用后，负责将方法、参数等组装成能够进行网络传输的消息体（将消息体对象序列化为二进制）；

(3) 客户端通过sockets将消息发送到服务端；

(4) 服务端存根( server stub）收到消息后进行解码（将消息对象反序列化）；

(5) 服务端存根( server stub）根据解码结果调用本地的服务；

(6) 本地服务执行并将结果返回给服务端存根( server stub）；

(7) 服务端存根( server stub）将返回结果打包成消息（将结果消息对象序列化）；

(8) 服务端（server）通过sockets将消息发送到客户端；

(9) 客户端存根（client stub）接收到结果消息，并进行解码（将结果消息发序列化）；

(10) 客户端（client）得到最终结果。

RPC的目标是要把2、3、4、7、8、9这些步骤都封装起来。

注意：无论是何种类型的数据，最终都需要转换成二进制流在网络上进行传输，数据的发送方需要将对象转换为二进制流，而数据的接收方则需要把二进制流再恢复为对象。

2.1 dubbo 架构
suntang > 9.第九期.dubbo.20190910 > image2019-9-10_20-24-52.png

1、Provider在启动时，向Registry注册自己提供的服务

2、Consumer在启动时，想Registry订阅自己所需的服务

3、Registry给Consumer返回Provider的地址列表，如果Provider地址有变更（上线/下线机器），Registry将基于长连接推动变更数据给Consumer

4、Consumer从Provider地址列表中，基于软负载均衡算法，选一台进行调用，如果失败，重试另一台调用

5、Consumer和Provider，在内存中累计调用次数和时间，定时每分钟一次将统计数据发送到Monitor

2.2、特性：
suntang > 9.第九期.dubbo.20190910 > image2019-9-10_20-25-2.png

面向接口代理：调用接口的方法，在 A 服务器调用 B 服务器的方法， 由 dubbo 实现对 B 的调用，无需关心实现的细节，就像 MyBatis 访问 Dao 的接口，可以操作数据库一样。不用关心 Dao 接口方法的实现。

 

2.3 dubbo 支持的协议
支持多种协议： dubbo , hessian , rmi , http, webservice , thrift , memcached , redis。dubbo 官方推荐使用 dubbo 协议。

 dubbo 协议默认端口 20880使用 dubbo 协议， spring 配置文件加入：<dubbo:protocol name="dubbo" port="20880" />

3.1dubbo应用
 

Pom.xml配置

<!--spring的依赖，spring-web.jar-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>4.3.16.RELEASE</version>
</dependency>
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>dubbo</artifactId>
  <version>2.6.2</version>
</dependency>

 

<dependency>

    <groupId>org.apache.zookeeper</groupId>

    <artifactId>zookeeper</artifactId>

    <version>3.3.3</version>

</dependency>

<!--zookeeper客户端-->
<dependency>
  <groupId>org.apache.curator</groupId>
  <artifactId>curator-framework</artifactId>
  <version>4.1.0</version>
</dependency>

 

 

 

公用配置

<!--声明服务名称 供方应用信息，用于计算依赖关系 -->
<dubbo:application name="dubbo-dist-dap4 "/>

<!-- 用dubbo协议在20882暴露服务，端口可自定义 -->
<dubbo:protocol name="dubbo" port="20882" />

<!-- 使用zookeeper注册中心暴露服务地址 -->

 <dubbo:registry address="zookeeper://192.168.1.166:2181" check="false"/>

 
<!-- 定义监控中心 -->

<dubbo:monitor address="192.168.1.166:7070" />

 

 

提供者配置
<!--暴露服务-->

version="1.0":指定接口的不同实现
<dubbo:service interface="com.bjpowernode.service.OrderService"
               ref="orderService" version="1.0"/>

 

<dubbo:service interface="com.bjpowernode.service.OrderService"
               ref="orderService" version="2.0"/>

 

 

消费者配置

<!--consumer -->

声明要使用的远程接口   check:检查项，dubbo默认在消费者启动时，要检查提供者是否存在，不存在项目不能启动，报错。 默认是true,

version:指定要使用哪个版本的服务提供者

 

<dubbo:reference interface="com.bjpowernode.service.OrderService" id="orderService"
 version="2.0"  check="false" />

3.2dubbo常用标签
Dubbo 中常用标签。分为三个类别：公用标签，服务提供者标签，服务消费者标签
公用标签
<dubbo:application/> 和 <dubbo:registry/>
A、配置应用信息
<dubbo:application name=”服务的名称”/> 
B、配置注册中心
<dubbo:registry address=”ip:port” protocol=”协议”/>

C、配置监控中心

<dubbo:monitor address=”ip:port” />

服务提供者标签
<dubbo:service interface=”服务接口名” ref=”服务实现对象 bean”>

服务消费者
<dubbo:reference id=”服务引用 bean 的 id” interface=”服务接口名”/>

 

4、dubbo的高可用
高可用性（High Availability）： 通常来描述一个系统经过专门的设计，从而减少不能提供服
务的时间，而保持其服务的高度可用性。
Zookeeper 是高可用的，健壮的。

Zookeeper 宕机，正在运行中的 dubbo 服务仍然可以正常访问。

监控中心宕掉不影响使用，只是丢失部分采样数据

数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务

注册中心对等集群，任意一台宕掉后，将自动切换到另一台

注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯

服务提供者无状态，任意一台宕掉后，不影响使用

服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复
