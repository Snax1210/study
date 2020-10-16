[TOC]: # "SpringCloud阿里巴巴"

# SpringCloud阿里巴巴
- [1.SpringCloud阿里巴巴简介](#1springcloud阿里巴巴简介)
  - [1.1 概述](#11-概述)
  - [1.2 主要功能](#12-主要功能)
  - [1.3 组件](#13-组件)
- [2. Spring Cloud Alibaba 创建依赖管理项目](#2-spring-cloud-alibaba-创建依赖管理项目)
  - [创建文件夹](#创建文件夹)
  - [用idea打开](#用idea打开)
  - [创建子模块](#创建子模块)
  - [创建pom](#创建pom)


## 1.SpringCloud阿里巴巴简介

### 1.1 概述

2018年10月31日,SpringCloudAlibaba正式入职SpringCloud官方孵化器，并在Maven中央仓库发布第一个版本。

[Spring Cloud for Alibaba 0.2.0 released](https://spring.io/blog/2018/10/30/spring-cloud-for-alibaba-0-2-0-released)

>The Spring Cloud Alibaba project,
>consisting of Alibaba’s open-source
>components and several Alibaba Cloud
>products, aims to implement and expose
>well known Spring Framework patterns
>and abstractions to bring the benefits of
>Spring Boot and Spring Cloud to Java
>developers using Alibaba products.

>Spring Cloud for
>Alibaba，它是由一些阿里巴巴的开源组件和云产品组成的。这个项目的目的是为了让大家所熟知的
>Spring
>框架，其优秀的设计模式和抽象理念，以给使用阿里巴巴产品的
>Java 开发者带来使用 Spring Boot 和 Spring
>Cloud 的更多便利。

Spring Cloud
Alibaba，您只需要添加一些注解和少量配置，就可以将SpringCloud应用接入阿里微服务解决方案，通过阿里中间件来迅速单间分布式应用系统。

### 1.2 主要功能

- **服务限流降级：**
  默认支持Servlet、Feign、RestTemplate、Dubbo和RocketMQ限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，可以支持查看限流降级Metric监控。
- **服务注册与发现：**
  适配SpringCloud服务注册和发现标准，默认集成了Ribbon的支持。
- **分布式配置管理：**
  支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力：**
  基于SpringCloudStream为微服务应用构建消息驱动能力。
- **阿里云对象存储：**
  阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用，任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度：**
  提供秒级、精准、高可靠、高可用的地夯实（基于cron表达式）任务调度服务。同事提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有Worker（schedulerx-client）上执行。

### 1.3 组件

- **Sentinel：**
  面向分布式服务架构的轻量级流量控制产品，主要以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度来帮助您保护服务的稳定性。
- **Nacos：**
  阿里巴巴推出来的一个新开源项目，这是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- **RocketMQ：**
  分布式消息系统，基于高可用分布式集群技术，提供低延时、高可靠的消息发布订阅与服务。
- **Alibaba Cloud ACM：**
  一款分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
- **Alibaba Cloud OSS：**
  阿里云对象存储服务（Object Storage
  Service，简称OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以用任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **Alibaba Cloud SchedulerX：**
  阿里中间件团队开发的一款
  分布式调度产品，提供秒级、精准、高可靠、高可用的定时（基于Cron表达式）任务调度服务。

## 2. Spring Cloud Alibaba 创建依赖管理项目:

> 当前Spring Cloud Alibaba 的2.1.0RELEASE
> 版本基于Spring Cloud
> Greenwich开发，SpringCloudAlibaba项目都是基于Spring
> Cloud，而Spring Cloud项目又是基于Spring
> Boot进行开发，并且都是使用Maven做项目管理工具。在实际开发中，我们一般都会创建一个依赖管理项目作为Maven的Parent项目使用，这样做可以极大地方便我们对Jar包版本的统一管理。

### 创建文件夹

在电脑上创建一个文件夹`hello-spring-cloud-alibaba`当做项目的根目录

![创建文件夹](.springcloud阿里巴巴_images/1975882c.png)

### 用idea打开

用idea打开刚刚创建的文件夹：

![用idea打开文件夹](.springcloud阿里巴巴_images/60712067.png)

### 创建子模块

选择hello-spring-cloud-alibaba跟目录鼠标右键  
创建子模块`hello-spring-cloud-alibaba-dependencies`

![创建模块](.springcloud阿里巴巴_images/a0bd3bd6.png)

### 创建pom

在hello-spring-cloud-alibaba-dependencies下创建pom.xml文件  
pom.xml内容如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
    </parent>
    <groupId>com.wsl</groupId>
    <artifactId>hello-spring-cloud-alibaba-dependencies</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>hello-spring-cloud-alibaba-dependencies</name>
    <inceptionYear>2019-Now</inceptionYear>
    <description>Demo project for Spring Boot</description>
    <packaging>pom</packaging>

    <properties>
        <!-- Environment Settings -->
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

        <!-- Spring Settings -->
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
        <spring-cloud-alibaba.version>2.2.0.RELEASE</spring-cloud-alibaba.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Compiler 插件, 设定 JDK 版本 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <showWarnings>true</showWarnings>
                </configuration>
            </plugin>

            <!-- 打包 jar 文件时，配置 manifest 文件，加入 lib 包的 jar 依赖 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <addMavenDescriptor>false</addMavenDescriptor>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <configuration>
                            <archive>
                                <manifest>
                                    <!-- Add directory entries -->
                                    <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                                    <addDefaultSpecificationEntries>true</addDefaultSpecificationEntries>
                                    <addClasspath>true</addClasspath>
                                </manifest>
                            </archive>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- resource -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
            </plugin>

            <!-- install -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-install-plugin</artifactId>
            </plugin>

            <!-- clean -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-clean-plugin</artifactId>
            </plugin>

            <!-- ant -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
            </plugin>

            <!-- dependency -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
            </plugin>
        </plugins>

        <pluginManagement>
            <plugins>
                <!-- Java Document Generate -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-javadoc-plugin</artifactId>
                    <executions>
                        <execution>
                            <phase>prepare-package</phase>
                            <goals>
                                <goal>jar</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>

                <!-- YUI Compressor (CSS/JS压缩) -->
                <plugin>
                    <groupId>net.alchim31.maven</groupId>
                    <artifactId>yuicompressor-maven-plugin</artifactId>
                    <version>1.5.1</version>
                    <executions>
                        <execution>
                            <phase>prepare-package</phase>
                            <goals>
                                <goal>compress</goal>
                            </goals>
                        </execution>
                    </executions>
                    <configuration>
                        <encoding>UTF-8</encoding>
                        <jswarn>false</jswarn>
                        <nosuffix>true</nosuffix>
                        <linebreakpos>30000</linebreakpos>
                        <force>true</force>
                        <includes>
                            <include>**/*.js</include>
                            <include>**/*.css</include>
                        </includes>
                        <excludes>
                            <exclude>**/*.min.js</exclude>
                            <exclude>**/*.min.css</exclude>
                        </excludes>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>

        <!-- 资源文件配置 -->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <excludes>
                    <exclude>**/*.java</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>

    <repositories>
        <repository>
            <id>aliyun-repos</id>
            <name>Aliyun Repository</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>

        <repository>
            <id>sonatype-repos</id>
            <name>Sonatype Repository</name>
            <url>https://oss.sonatype.org/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>sonatype-repos-s</id>
            <name>Sonatype Repository</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>

        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>aliyun-repos</id>
            <name>Aliyun Repository</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</project>
```

- parent: 继承了Spring Boot的Parent，表示我们是以一个Spring Boot工程
- package：pom，表示该项目仅当做依赖项目，没有具体实现的代码
- spring-cloud-alibaba-dependencies: 在properties配置中预定义了版本号为2.1.0RELEASE，表示我们的Spring Cloud Alibaba对应的是Spring Cloud Greenwich版本
- build：配置了项目所需的各种插件
- repositories：配置项目下载依赖时的第三方库

### 将pom加到maven

![添加pom文件到maven](.springcloud阿里巴巴_images/8516c7bb.png)

![目录结构](.springcloud阿里巴巴_images/77c356d3.png)

### 依赖版本简介

项目的最新版本是2.2.1.RELEASE
1.5.x版本使用于Spring Boot 1.5.x
2.0.x版本使用于Spring Boot 2.0.x
2.1.x版本使用于Spring Boot 2.1.x

> 截止目前时间2019年8月

