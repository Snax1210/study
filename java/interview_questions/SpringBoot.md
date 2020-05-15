[TOC]: # "SpringBoot"

# SpringBoot
- [配置](#配置)
  - [什么是JavaConfig？](#什么是javaconfig)
  - [Spring Boot 自动配置原理是什么？](#spring-boot-自动配置原理是什么)
  - [你如何理解SpringBoot配置加载顺序？](#你如何理解springboot配置加载顺序)
  - [什么是YAML？](#什么是yaml)
  - [YAML配置的优势在哪里？](#yaml配置的优势在哪里)
  - [Spring Boot是否可以使用XML配置？](#spring-boot是否可以使用xml配置)
  - [spring boot 核心配置文件是什么？bootstrap.properties和application.properties有什么区别？](#spring-boot-核心配置文件是什么bootstrapproperties和applicationproperties有什么区别)
  - [什么是Spring Profiles？](#什么是spring-profiles)
  - [如何在自定义端口运行Spring Boot应用程序？](#如何在自定义端口运行spring-boot应用程序)
- [安全](#安全)
  - [如何实现SpringBoot应用程序的安全性？](#如何实现springboot应用程序的安全性)
  - [比较一下SpringSecurity 和Shiro各自的优缺点](#比较一下springsecurity-和shiro各自的优缺点)
  - [Spring Boot中如何解决跨域问题？](#spring-boot中如何解决跨域问题)
  - [什么CSRF攻击？](#什么csrf攻击)
- [监视器](#监视器)
  - [Spring Boot 中的监视器是什么？](#spring-boot-中的监视器是什么)
  - [如何在Spring Boot中禁用Actuator端点安全性？](#如何在spring-boot中禁用actuator端点安全性)
  - [如何监视所有Spring Boot 微服务？](#如何监视所有spring-boot-微服务)
- [整合第三方项目](#整合第三方项目)
  - [WebSockets？](#websockets)
  - [什么是Spring Data?](#什么是spring-data)
  - [什么是Spring Batch？](#什么是spring-batch)
  - [什么是FreeMarker模板？](#什么是freemarker模板)
  - [如何集成Spring boot和ActiveMQ？](#如何集成spring-boot和activemq)
  - [什么是Apache Kafka？](#什么是apache-kafka)
  - [什么是Swagger？你用Springboot实现了它吗？](#什么是swagger你用springboot实现了它吗)
  - [前后端分离，如何维护接口文档？](#前后端分离如何维护接口文档)
- [其他](#其他)
  - [如何重新加载Springboot上的更改，而无需重新启动服务器？Springboot项目如何热部署？](#如何重新加载springboot上的更改而无需重新启动服务器springboot项目如何热部署)
  - [使用了哪些starter maven依赖项？](#使用了哪些starter-maven依赖项)
  - [spring boot中的starter到底是什么？](#spring-boot中的starter到底是什么)
  - [spring-boot-start-parent有什么用？](#spring-boot-start-parent有什么用)
  - [springboot打成的jar和普通的jar有什么区别？](#springboot打成的jar和普通的jar有什么区别)
  - [运行springboot有哪几种方式？](#运行springboot有哪几种方式)
  - [SpringBoot需要独立的容器吗？](#springboot需要独立的容器吗)
  - [开启Spring boot特性有哪几种方式？](#开启spring-boot特性有哪几种方式)
  - [如何使用Spring boot实现异常处理？](#如何使用spring-boot实现异常处理)
  - [如何使用Springboot实现分页和排序？](#如何使用springboot实现分页和排序)
  - [微服务中如何实现session共享？](#微服务中如何实现session共享)
  - [springboot中如何实现定时任务？](#springboot中如何实现定时任务)




## 配置

### 什么是JavaConfig？

Spring JavaConfig是Spring社区的产品，它提供了配置Spring IOC 容器的纯Java方法。因此他有助于避免使用XML配置。使用JavaConfig的优点在于：

1. 面向对象的配置。由于配置被定义为JavaConfig中的类，因此用户可以充分利用Java中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean方法等。
2. 减少或消除XML配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在XML和Java之间来回切换。JavaConfig为开发人员提供了一种纯Java方法来配置与XML配置概念相似的Spring容器。从技术角度来讲，只使用JavaConfig配置类来配置容器是可行的，但实际上很多人认为将JavaConfig与XML混合匹配是理想的。
3. 类型安全和重构友好。JavaConfig提供了一种类型安全的方法来配置Spring容器。由于Java 5.0对泛型的支持，现在可以按类型而不是按名称检索bean，不需要任何强制转换或基于字符串的查找。

### Spring Boot 自动配置原理是什么？

注解@EnableAutoConfiguration，@Configuration，@ConfigurationOnClass就是自动配置的核心，

@EnableAutoConfiguration给容器导入META-INF/spring.factories里定义自动配置类。

筛选有效的自动配置类。

每一个自动配置类结合对应的xxxProperties.java读取配置文件进行自动配置功能

### 你如何理解SpringBoot配置加载顺序？

在SpringBoot里面，可以使用以下几种方式来加载配置。

1. properties文件；
2. YAML文件；
3. 系统环境变量；
4. 命令行参数等

### 什么是YAML？

YAML是一种人类可读的数据序列化语言。它通常用于配置文件。与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML文件就更加结构化，而且更少混淆。可以看出YAML具有分层配置数据。

###  YAML配置的优势在哪里？

1. 配置有序，在一些特殊的场景下，配置有序很关键
2. 支持数组，数组中的元素可以是基本数据类型也可以是对象
3. 简洁

相比properties配置文件，YAML还有一个缺点，就是不支持@PropertySource注解导入自定义的YAML配置。

### Spring Boot是否可以使用XML配置？

Spring Boot推荐使用Java配置而非XML配置，但是Spring Boot中也可以使用XML配置，通过@ImportResource注解可以已引入一个XML配置。

### spring boot 核心配置文件是什么？bootstrap.properties和application.properties有什么区别？

单纯做Spring Boot开发，可能不太容易遇到bootstrap.properties配置文件，但是在结合SpringCloud时，这个配置就会经常遇到了，特别是在需要加载一些远程配置文件的时候。

spring boot 核心的两个配置文件：
- bootstrap（.yml或者.properties）：bootstrap由父ApplicationContext加载的，比application优先加载，配置在应用程序上下文的引导阶段生效一般来说我们在Spring Cloud Config或者Nacos中会用到它。且bootstrap里面的属性不能被覆盖；
- application（.yml或者.properties）：由ApplicationContext加载，用于spring boot项目的自动化配置。

### 什么是Spring Profiles？

Spring Profiles允许用户根据配置文件（dev、prod、test等）来注册bean。因此，当应用程序在开发中运行时，只有某些bean可以加载，而在PRODUCTION中，某些其他bean可以加载。假设我们要求的是Swagger文档仅适用于QA环境，并且禁用所有其他文档。这可以使用配置文件来完成。SpringBoot使得使用配置文件非常简单。

### 如何在自定义端口运行Spring Boot应用程序？

为了在自定义端口运行Spring Boot应用程序，可以在application.properties中来指定端口。</br>
server.port=8080

## 安全

### 如何实现SpringBoot应用程序的安全性？

为了实现SpringBoot的安全性，我们使用spring-boot-start-security依赖项，并且必须添加安全配置。它只需要很少的代码。配置类将必须扩展WebSecurityConfigurerAdapter并覆盖其方法。

### 比较一下SpringSecurity 和Shiro各自的优缺点

由于Springboot官方提供了大量的非常方便的开箱即用的starter，包括时spring security的starter，使得在spring boot中使用spring security变得更加容易，甚至只需要添加一个依赖就可以保护所有的接口，所以，如果是springboot项目，一般选择spring security。当然这只是一个建议的组合，单纯从技术上来说，无论怎么组合，都是没有问题的。shiro和spring security相比，主要有如下一些特点：

1. spring security是一个重量级的安全管理框架；shiro则是一个轻量级的安全管理框架
2. spring security概念复杂，配置繁琐；shiro概念简单、配置简单
3. spring security功能强大，shiro功能简单。

### Spring Boot中如何解决跨域问题？

跨域可以在前端通过JSONP来解决，但是JSONP只可以发送GET请求，无法发送其他类型的请求，在RESTful风格的应用中，就显得非常鸡肋，因此我们推荐在后端通过（CORS，Cross-origin resource sharing) 来解决跨域问题。这种解决方法并非是spring boot特有的，在传统的ssm框架中，就可以CORS来解决跨域问题，只不过之前我们是在XML文件中配置CORS，现在可以通过实现WebMvcConfigurer接口然后重写addCorMappings方法解决跨域问题。

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);
    }

}
```

项目中前后端分离部署，所以需要解决跨域问题。

我们使用cookie存放用户登陆的信息，在spring拦截器进行去权限控制，当权限不符合时，直接返回给用户固定的json结果。

当用户登陆以后，正常使用；当用户退出登录状态或者token到期时，由于拦截器和跨域的顺序有问题，出现了跨域的现象。

我们知道一个http请求，先走filter，到达servlet后才进行 拦截器的处理，如果我们cors放在filter里，就可以优先于权限拦截器执行。

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource urlBasedCorsConfigurationSource = new UrlBasedCorsConfigurationSource();
        urlBasedCorsConfigurationSource.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsFilter(urlBasedCorsConfigurationSource);
    }

}
```

### 什么CSRF攻击？

CSRF代表跨站请求伪造。这是一种攻击，迫使最终用户在当前通过身份验证web应用程序上执行不需要的操作。CSRF攻击专门针对状态改变请求，而不是数据窃取，因为供给制无法查看对伪造请求的相应。

## 监视器

### Spring Boot 中的监视器是什么？

Spring boot actuator是spring启动框架中的重要功能之一。Spring boot监视器可帮助访问生产环境中正在运行的应用程序的当前状态。有几个指标必须在生产环境中惊醒检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为HTTP URL访问的REST端点来检查状态。

### 如何在Spring Boot中禁用Actuator端点安全性？

默认情况下，所有敏感的HTTP端点都是安全的，只有具有ACTUATOR角色的用户才能访问它们。安全性是使用标准的HttpServletRequest.isUserInRole方法实施的。我们可以使用来禁用安全性。只有在执行机构端点在防火墙后访问时，才建议禁用安全性。

### 如何监视所有Spring Boot 微服务？

springboot提供监视器端点以 监控各个微服务的度量。这些端点对于获取有关应用程序的信息（如他们是否已启动）以及他们的组件（数据库等）是否正常运行很有帮助。但是，使用监视器的一个缺点或困难是，我们必须单独打开应用程序的知识点以了解其状态或健康状况。想象一下涉及50个应用程序的微服务，管理员不得不几种所以50个应用程序的运行终端。为了帮助我们这种情况，我们将使用位于的开源项目。它建立在Spring Boot Actuator之上，它提供了一个Web UI，使我们能够可视化多个应用程序的度量。

## 整合第三方项目

### WebSockets？

WebSocket是一种计算机通信协议，通过单个TCP连接提供全双工通信信道。

1. WebSocket是双向的 ——使用WebSocket客户端或者服务端可以发起消息发送。
2. WebSocket是全双工的——客户端和服务器通信是相互独立的。
3. 单个TCP连接——初始连接使用HTTP，然后将此连接升级到基于套接字的连接。然后这个单一连接用于所有未来的通信。
4. Light——与http相比，WebSocket消息数据交换要轻得多。

### 什么是Spring Data?

Spring Data 是Spring的一个子项目。用于简化数据库访问，支持NoSQL和关系存储。其主要目标是使数据库的访问变得方便快捷。Spring Data具有如下特点：

- SpringData项目支持NoSQL存储：
  1. MongoDB（文档数据库）
  2. Neo4j（图形数据库）
  3. Redis（键/值存储）
  4. Hbase（列族数据库）
- SpringData项目所支持的关系数据存储技术：
  1. JDBC
  2. JPA

Spring Data Jpa致力于减少数据访问层（DAO）的开发量。开发者唯一要做的，就是声明持久层的接口，其他都交给Spring Data JPA完成。 Spring Data JPA通过规范方法的名字，根据符合规范的名字来确定方法需要实现什么样的逻辑。

### 什么是Spring Batch？

Spring Boot Batch提供可重用的函数，这些函数在处理大量记录时非常重要，包括日志/跟踪，事务管理，作业管理统计信息，作业重新启动，跳过和资源管理。它还提供了更先进的技术服务和功能，通过优化和分区技术，可以实现极高批量和高性能批处理作业。简单以及复杂的大批量批处理作业可以高度可扩展的方式利用框架处理重要大量的信息。

### 什么是FreeMarker模板？

FreeMaker是一个基于Java的模板引擎，最初专注于使用MVC软件架构进行动态网页生成。使用FreeMaker的主要优点是表示层和业务层的完全分离。程序员可以处理应用程序代码，而设计人员可以处理html页面设计。最后使用FreeMaker可以将这些结合起来，给出最终的输出页面。

### 如何集成Spring boot和ActiveMQ？

对于集成springboot和ActiveMQ，我么使用依赖关系。它只需要很少的配置，并且不需要样板代码。

### 什么是Apache Kafka？

Apache Kafka是一个分布式发布-订阅消息系统。它是一个可扩展，容错的发布-订阅消息系统，它使我们能够构建分布式应用程序。这是一个Apache顶级项目。Kafka适合离线和在线消费。

### 什么是Swagger？你用Springboot实现了它吗？

Swagger广泛用于可视化API，使用Swagger UI为前端开发人员提供在线沙箱。Swagger适用于生成RESTful Web服务的可视化表示的工具，规范和完整框架实现。它使文档能够以与服务期相同的速度更新。当通过Swagger正确定义时，消费者可以使用最少量的实现逻辑来理解远程服务并与其进行交互。因此，Swagger消除了调用服务时的猜测。

### 前后端分离，如何维护接口文档？

前后端分离开发日益流行，大部分情况下，我们都是通过Spring boot做前后端分离开发，前后盾分离一定会有接口文档，不然前后端会深深陷入到扯皮中。一个比较笨的方法就是使用word或者md维护接口文档，但是工作效率太低，接口一边，所有人手上的文档都得变。在springboot中，这个问题的常见解决方案就是Swagger，使用Swagger我们可以快速生成一个接口文档网站，接口一旦发生变化，文档就会自动更新，所有的开发工程师访问这一个在线网站就可以获取到最新的接口文档，非常方便。

## 其他

### 如何重新加载Springboot上的更改，而无需重新启动服务器？Springboot项目如何热部署？

这可以使用DEV工具来实现。通过这种依赖关系，可以节省修改，嵌入式tomcat将重新启动。Springboot有一个开发工具（DevTools）模块，它有助于提高开发人员的生产力。Java开发人员面临的一个主要挑战是将文件更改自动部署到服务器并自动重启服务器。开发人员可以重新加载Springboot上的更改，而无需重新启动无服务器。这将消除每次手动部署更改的需要。SpringBoot在发布它的第一个版本时没有这个功能。这是开发人员最需要的功能。DevTools模块完全满足开发人员的需求。该模块将在生产环境中被禁用。它还提供H2数据库控制台以更好地测试应用程序。

```maver
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

### 使用了哪些starter maven依赖项？

使用了下面一些依赖项

spring-boot-starter-activemq

spring-boot-start-security

这有助于增加更少的依赖关系，并减少版本的冲突。

### spring boot中的starter到底是什么？

首先，这个starter并非什么新的技术点，基本上还是基于spring已有功能来实现的。首先它提供了一个自动化配置类，一般命名为`XXXAutoConfiguration`，在这个配置类中通过条件注解来决定一个配置是否生效（条件注解就是Spring原本就有的），然后它还会提供一系列的默认配置，也允许开发者根据实际情况自定义相关配置，然后通过类型安全属性注入将这些配置属性注入进来，新注入的属性会代替掉默认属性。正因为如此，很多第三方框架，我们只需要引入依赖就可以直接使用了。当然开发者也可以自定义starter。

### spring-boot-start-parent有什么用？

我们都知道，新创建一个springboot项目，默认都是有parent的，这个parent就是spring-boot-start-parent,spring-boot-starter-parent主要如下作用：

1. 定义了Java编译版本为1.8
2. 使用了UTF-8格式编码。
3. 继承自spring-boot-dependencies，这个里边定义了依赖的版本，而正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。
4. 执行打包操作的配置。
5. 自动化的资源过滤。
6. 自动化的插件配置。
7. 针对application.properties和application.yml的资源过滤，包括通过profile定义不同的环境配置文件，例如application-dev.properties和application-dev.yml。

### springboot打成的jar和普通的jar有什么区别？

springboot项目最终打包成的jar是可执行的jar，这种jar可以通过java -jar xxx.jar命令来运行，这种jar不可以作为普通的jar被其他项目依赖，即使依赖了也无法使用其中的类。

springboot的jar无法被其他项目依赖，主要是和他和普通jar的结构不同。不同的jar包，解压后直接就是包名，包里就是我们的代码，而spring boot打包成的可执行jar解压后，在\BOOT-INF\classes目录下才是我们的代码，因此无法北直街引用。如果非要引用，可以在pom.xml文件中增加配置，将springboot项目打成两个jar，一个可执行，一个可引用。

### 运行springboot有哪几种方式？

1. 打包用命令或者放到容器中运行
2. 用maven/Gradle插件运行
3. 直接执行main方法执行

###  SpringBoot需要独立的容器吗？
可以不需要，内置了Tomcat/Jetty等容器

### 开启Spring boot特性有哪几种方式？
1. 继承spring-boot-starter-parent项目
2. 导入spring-boot-dependencies项目依赖

### 如何使用Spring boot实现异常处理？

Spring提供了一种使用ControllerAdvice处理异常的非常有效的方法。我们通过实现一个ControllerAdvice类，来处理控制类抛出的所有异常。

### 如何使用Springboot实现分页和排序？

使用SpringBoot实现分页非常简单。使用Spring-Data-JPA可以实现将可分页传递给存储库方法。

### 微服务中如何实现session共享？

在微服务中，一个完整的项目被拆分成多个不相同的独立的服务，各个服务独立部署在不同的服务器上，各自的session被从物理空间上隔离开了，但是经常，我们需要在不同微服务之间共享session，常见的方案就是spring session+redis来实现session共享。将所有的微服务的session统一保存在redis上，当各个微服务对session有相关读写操作的时候，都去操作redis上的session。这样就实现了session的共享，spring session基于spring中的代理过滤器实现，使得session的同步操作对开发人员而言是透明的，非常方便。

### springboot中如何实现定时任务？

定时任务也是一个常见的需求，springboot中对于定时任务的支持主要还是来自spring框架。

在springboot中使用定时任务主要有两种不同的方式，一个就是使用spring中的@Scheduled注解，另一个则是使用第三方框架Quartz。

使用 Quartz ，则按照 Quartz 的方式，定义 Job 和 Trigger 即可。



