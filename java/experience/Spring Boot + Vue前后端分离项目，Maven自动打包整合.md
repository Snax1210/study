[TOC]: # "Spring Boot + Vue前后端分离项目，Maven自动打包整合"
# Spring Boot + Vue前后端分离项目，Maven自动打包整合
- [前言](#前言)
- [前后端项目结构要求](#前后端项目结构要求)
- [配置pom.xml文件](#配置pomxml文件)
  - [1、父工程的pom.xml文件](#1父工程的pomxml文件)
  - [2、Vue前端工程的pom.xml文件](#2vue前端工程的pomxml文件)
  - [3、后端工程的pom.xml文件](#3后端工程的pomxml文件)
- [打包部署](#打包部署)


## 前言

现在各类项目为了降低项目、模块间的高度耦合性，提出了“**前后端分离**”，而前后端分离的项目该如何打包呢？

一般的做法是前端项目打包完，将打包文件手动复制到后端项目工程的src\main\resources\static目录下，再进行后端工程项目打包，
这样手动来回复制、多次打包总是让人觉得麻烦。（即使采用Jenkins打包部署，也会存在上面2次打包过程）

为了解决上述问题，需要通过Maven自动打包整合

## 前后端项目结构要求

以Spring Boot + Vue 的前后端项目举例说明

通过Maven构建项目，针对子父项目结构创建前端、后端工程，结构如下：

```
spring-boot-vue-parent
    |---spring-boot  # spring boot后端工程
        |---src
            |---main
                |---java
                    |---...
                |---resources
                    |---static    # 存放前端资源的目录
        |---pom.xml   # spring-boot后端工程的pom.xml文件
    |---vue-ui       # Vue前端工程
       |---...
       |---dist    # 打包编译时，自动创建的目录，无需手动创建该目录
       |---pom.xml # Vue前端工程的pom.xml文件，此文件可不要
 pom.xml 父工程的pom.xml文件
```

## 配置pom.xml文件

### 1、父工程的pom.xml文件

满足Maven父子项目结构配置要求，配置spring-boot-maven-plugin插件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>com.xcbeyond.demo</groupId>
    <artifactId>demo</artifactId>
    <packaging>pom</packaging>
    <version>1.0.0</version>
    
    <modules>
        <!-- spring boot后端工程，作为子工程 -->
        <module>spring-boot</module>
        <!-- Vue前端工程，作为子工程 -->
        <module>vue-ui</module>
    </modules>
 
    <dependencies>
        # 配置项目依赖包
        ……
    </dependencies>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.0.7.RELEASE</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal><!--可以把依赖的包都打包到生成的Jar包中-->
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build> 
</project>
```

### 2、Vue前端工程的pom.xml文件

对应Vue项目而言，pom.xml对它而言是不存在的，也是毫无意义的，此文件可以不要。在此体现出来，只是为了配置父子工程而已，凸显出Vue工程属于
父工程的子工程而已，便于IDE导入呈现展示而已

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>demo</artifactId>
        <groupId>com.xcbeyond.demo</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>com.xcbeyond.demo.vue.ui</groupId>
    <artifactId>vue-ui</artifactId>
 
</project>
```

### 3、后端工程的pom.xml文件

该pom.xml文件是需要终端关注配置的，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>demo</artifactId>
        <groupId>com.xcbeyond.demo</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>com.xcbeyond.demo.spring.boot</groupId>
    <artifactId>spring-boot</artifactId>
 
    <dependencies>
        # 配置项目依赖包
        ……
    </dependencies>
 
    <build>
        <plugins>
            <!-- 插件maven-clean-plugin，用于在编译前，清除之前编译的文件、文件夹等，避免残留之前的内容 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-clean-plugin</artifactId>
                <version>3.1.0</version>
                <configuration>
                    <filesets>
                        <fileset>
                            <!-- 前端资源目录，即：存放前端包目录-->
                            <directory>src/main/resources/static</directory>
                        </fileset>
                        <fileset>
                            <!-- Vue项目打包自动生成的dist目录 -->
                            <directory>${project.parent.basedir}/vue-ui/dist</directory>
                        </fileset>
                    </filesets>
                </configuration>
            </plugin>
 
            <!--frontend-maven-plugin为项目本地下载/安装Node和NPM，运行npm install命令-->
            <plugin>
                <groupId>com.github.eirslett</groupId>
                <artifactId>frontend-maven-plugin</artifactId>
                <version>1.6</version>
                <configuration>
                    <workingDirectory>${project.parent.basedir}/vue-ui</workingDirectory>
                </configuration>
                <executions>
                    <execution>
                        <id>install node and npm</id>
                        <goals>
                            <goal>install-node-and-npm</goal>
                        </goals>
                        <configuration>
                            <nodeVersion>v8.12.0</nodeVersion>
                            <npmVersion>6.4.1</npmVersion>
                        </configuration>
                    </execution>
                    <!-- Install all project dependencies -->
                    <execution>
                        <id>npm install</id>
                        <goals>
                            <goal>npm</goal>
                        </goals>
                        <phase>generate-resources</phase>
                        <configuration>
                            <arguments>install</arguments>
                        </configuration>
                    </execution>
                    <!-- Build and minify static files -->
                    <execution>
                        <id>npm run build</id>
                        <goals>
                            <goal>npm</goal>
                        </goals>
                        <configuration>
                            <arguments>run build</arguments>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
 
            <!--资源插件，主要为了从前端项目里复制打包好的文件到springboot项目-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <id>copy static</id>
                        <phase>generate-resources</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <!-- 复制前端打包文件到这里 -->
                            <outputDirectory>src/main/resources/static</outputDirectory>
                            <overwrite>true</overwrite>
                            <resources>
                                <resource>
                                    <!-- 从前端打包的目录dist进行指定文件、文件夹内容的复制-->
                                    <directory>${project.parent.basedir}/vue-ui/dist</directory>
                                    <includes>
                                        <!-- 具体根据实际前端代码、及目录结构进行配置-->
                                        <include>css/</include>
                                        <include>fonts/</include>
                                        <include>img/</include>
                                        <include>js/</include>
                                        <include>favicon.ico</include>
                                        <include>index.html</include>
                                    </includes>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
 
</project>
```

## 打包部署

上述的pom.xml配置，已经整合了前后端项目的Maven自动打包，打包时，只需关注后端项目(spring-boot子工程)打包即可，就会将前端、后端一起打包到后端成功中。

在子工程后端工程中，执行打包命令mvn clean package -Dmaven.test.skip=true, 或采用IDE中相应的Maven直接打包。



至此，只需一次打包，即可完成前后端项目的Maven自动打包了，再也不用担心多次打包、漏打包的情况
