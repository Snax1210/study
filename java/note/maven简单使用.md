**Maven简单使用**

Maven是基于项目对象模型（POM），可以通过一小段描述信息来管理项目的构建、报告和文档软件项目管理工具。

**项目中未使用maven情况：**

1、如果使用了spring，去spring的官网下载jar包；如果使用hibernate，去hibernate的官网下载Jar包；如果使用Log4j，去log4j的官网下载jar包.....

2、当某些jar包有依赖的时候，还要去下载对应的依赖jar包

3、当jar包依赖有冲突时，不得不一个一个的排查

4、当新人加入开发时，需要拷贝大量的jar包，然后重复进行构建等等

**使用maven情况：**

1 依赖的管理：仅仅通过jar包的几个属性，就能确定唯一的jar包，在指定的文件pom.xml中，只要写入这些依赖属性，就会自动下载并管理jar包。

2 项目的构建：内置很多的插件与生命周期，支持多种任务，比如校验、编译、测试、打包、部署、发布...

3 项目的知识管理：管理项目相关的其他内容，比如开发者信息，版本等等 

**Maven主要做了两件事：**

1.  统一开发规范与工具
2.  统一管理jar包

**M****aven****本地仓储配置**

 ![](/download/attachments/7733270/image2019-9-6_18-43-22.png?version=1&modificationDate=1567737803705&api=v2)

**Maven****中央仓库配置**

 ![](/download/attachments/7733270/image2019-9-6_18-43-34.png?version=1&modificationDate=1567737816334&api=v2)

**Maven远程****仓库配置**

JBoss的远程仓库

 **![](/download/attachments/7733270/image2019-9-6_18-43-50.png?version=1&modificationDate=1567737832124&api=v2)**

这样，Maven搜索依赖的顺序就是：

1）搜索本地仓库，没有找到，就去第2步，否则退出

2）搜索中央仓库【配置镜像】，没有找到，就去第3步，否则退出

3）去远程仓库获取，没有找到，就报错，否则退出

**M****aven目录骨架**：

 ![](/download/attachments/7733270/image2019-9-6_18-44-2.png?version=1&modificationDate=1567737843901&api=v2)

**命令行创建maven 项目**

mvn archetype:generate 如果是第一次使用，需要下载插件，稍等几分钟即可

 ![](/download/attachments/7733270/image2019-9-6_18-44-12.png?version=1&modificationDate=1567737854165&api=v2)



**常用的maven命令：**

显示版本信息：

mvn -version、mvn -v

项目转化为idea项目；

mvn idea:idea

编译,将Java 源程序编译成 class 字节码文件；

mvn compile

测试，并生成测试报告；

mvn test

将以前编译得到的旧的 class 字节码文件删除;

mvn clean

打包,动态 web工程打 war包，Java工程打 jar 包

mvn package

将项目生成 jar 包放在仓库中，以便别的模块调用，安装到本地

mvn install



**POM(Project Object Model)项目对象模型**

**坐标**

**<groupId>com.suntang.scm</groupId>**  
**<artifactId>scm-parent</artifactId>**  
**<version>1.0-SNAPSHOT</version>**

groupId , artifactId , version 三个元素是项目的坐标，唯一的标识这个项目。

groupId 项目所在组，一般是组织或公司

artifactId 是当前项目在组中的唯一ID，一般为模块名

version  该构件件的版本号

版本管理：

**SNAPSHOT**  
译为快照版，这个版本一般用于开发过程中，表示不稳定的版本。

**LATEST**  
指某个特定构件的最新发布，这个发布可能是一个发布版，也可能是一个snapshot版，具体看哪个时间最后。

**RELEASE**  
是指仓库中最后的一个非快照版本

**依赖范围**

**S****cope:**

1\. compile : 编译，测试，运行都有效，默认的选择  
2\. test : 测试有效，例如junit  
3\. provided : 编译，测试有效，例如 servlet ，运行时容器会提供实现  
4\. runtime : 运行和测试有效，例如 jdbc，编译时只需相应的接口，测试和运行时才需要具体的实现  
5\. system : 编译，测试有效。

<dependency>

      <groupId>junit</groupId>

      <artifactId>junit</artifactId>

      <version>4.11</version>

      <scope>test</scope>

    </dependency>

**生命周期**

maven的生命周期包含多个阶段，每个阶段由对应的插件去完成对应的动作；其中后面的插件执行会先执行之前的生命周期；

Clean

clean插件是一个独立的阶段，功能是删除当前项目的target目录；这个自己也可以测试下，执行之后就是把target目录删除了而已；

resources 

resources插件的功能是把src/main/resources目录的文件拷贝到target/classes目录下；如果不存在src/main/resources目录，则不会做任何处理，

Compile

compile插件执行时，会先执行resources插件，主要的作用就是把src/main/java代码编译成字节码生成class文件，输出到targe/classes目录下

Test

只不过这两个类主要用于对单元测试的资源文件和代码进行编译；生成的文件位于target/test-classes下面；

Package

这个插件是把class文件，resources文件打包成一个jar包，依赖包是不在里面包含的；常用的打包的插件有maven-jar-plugin、maven-assembly-plugin、maven-shade-plugin三种；生成的jar包位于target目录下；

Install

install是把构建好的artifact部署到本地仓库中；这样本地的其他项目依赖于本项目的jar时可以直接从本地仓库去获取，

Deploy等

**最常用****mvn clean install -Dmaven.test.skip=true** **跳过单元测试**



**依赖插件：**

 **![](/download/attachments/7733270/image2019-9-6_18-45-27.png?version=1&modificationDate=1567737929329&api=v2)**

**聚合**

modules用于聚合，把执行的项目都放到同一的地方用module包括，可以省去一个个项目去mvn install,这样可以所有项目一次聚合 mvn install

![](/download/attachments/7733270/image2019-9-6_18-45-42.png?version=1&modificationDate=1567737943732&api=v2)

**继承：**

dependencyManagement的使用 就是方便管理版本,子项目中要引入group id和atifact id

![](/download/attachments/7733270/image2019-9-6_18-45-55.png?version=1&modificationDate=1567737956682&api=v2)

**子类pom.xml**

 **![](http://doc.suntang.com/pages//download/attachments/7733270/image2019-9-6_18-46-20.png?version=1&modificationDate=1567737981718&api=v2)**

**依赖：**

**直接依赖：**

A-->B

A-->C

**传递依赖：**

A->B  B-->C  ：  A-->C

依赖原则：

1.路径最近者优先  
2.路径相同，第一声明者优先
