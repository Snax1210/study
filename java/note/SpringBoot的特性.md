
# SpringBoot 的特性


SpringBoot 的 3 大特性

- 组件自动装配: WEB mvc , web flux , jdbc 等

1. 激活@EnableAutoConfiguration

2. 配置/META-INF/spring.facotories

3. 实现:xxxAutoConfiguration

- 嵌入式容器: Tomcat , Jetty 以及 Undertow

web servlet : Tomcat , Jetty 以及 Undertow

web reactive : netty web server

- 生产准备特性: 指标 admin ,健康检查 actuator

## springboot 自动装配

先快速过一下注解小知识

- @Retention: 定义注解的保留策略

@Retention(RetentionPolicy.SOURCE) //注解仅存在于源码中，在 class 字节码文件中不包含

@Retention(RetentionPolicy.CLASS) // 默认的保留策略，注解会在 class 字节码文件中存在，但运行时无法获得，

@Retention(RetentionPolicy.RUNTIME) // 注解会在 class 字节码文件中存在，在运行时可以通过反射获取到
首 先要明确生命周期长度 SOURCE < CLASS < RUNTIME ，所以前者能作用的地方后者一定也能作用。一般如果需要在运行时去动态获取注解信息，那只能用 RUNTIME 注解；如果要在编译时进行一些预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS 注解；如果只是做一些检查性的操作，比如 @Override 和 @SuppressWarnings，则可选用 SOURCE 注解。

* @Target：定义注解的作用域

@Target(ElementType.TYPE) //接口、类、枚举、注解

@Target(ElementType.FIELD) //字段、枚举的常量

@Target(ElementType.METHOD) //方法

@Target(ElementType.PARAMETER) //方法参数

@Target(ElementType.CONSTRUCTOR) //构造函数

@Target(ElementType.LOCAL_VARIABLE)//局部变量

@Target(ElementType.ANNOTATION_TYPE)//注解

@Target(ElementType.PACKAGE) ///包

* @Document：说明该注解将被包含在 javadoc 中

* @Inherited：说明子类可以继承父类中的该注解

只有在类上使用时才会有效，对方法，属性等其他无效。

### spring Framework 手动装配

#### 模式注解

现在来了解一下 spring 自带的装配的

| Spring Framework 注解 | 场景说明 | 起始版本 |
| --------------------- | ------------------ | -------- |
| @Repository | 数据仓储模式注解 | 2.0 |
| @Component | 通用组件模式注解 | 2.5 |
| @Service | 服务模式注解 | 2.5 |
| @Controller | Web 控制器模式注解 | 2.5 |
| @Configuration | 配置类模式注解 | 3.0 |

#### 装配方式

装配方式有两种, 一种是基于 XML 一种是 JAVA 方式.

- xml

激活特性

```java
<context:annotation-config />
```

扫描指定的包

```java
<context:component-scan base-package="xxxx" />
```

- java

```java
@ComponentScan(basePackages = "xxxxxx")
public class SpringConfiguration{
....
}
```

#### 注解的"派生性"

- 什么 是派生性?

@Component 作为一种由 Spring 容器托管的通用模式组件，任何被 @Component 标准的组件均为组件扫描的候选对象。类似地，凡是被 @Component 元标注（meta-annotated）的注解，如 @Service ，当任何组件标注它时，也被视作组件扫
描的候选对

仓库对象

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repository
public @interface FirstLevelRepository {

String value() default "";
}

```

类

```java
@FirstLevelRepository(value = "myFirstLevelRepository")
public class MyFirstLevelRepository {
}
```

启动

```java
@ComponentScan(basePackages = "com.zx.diveinspringboot")
public class RepositoryBoostrap {

public static void main(String[] args) {
ConfigurableApplicationContext context = new SpringApplicationBuilder(RepositoryBoostrap.class).web(WebApplicationType.NONE).run(args);

//判断myFirstLevelRepository 是否存在, 不存在就会报错
MyFirstLevelRepository myFirstLevelRepository = context.getBean("myFirstLevelRepository", MyFirstLevelRepository.class);

System.out.println(" bean " + myFirstLevelRepository);
context.close();
}
}
```

输出

```java
bean com.zx.diveinspringboot.repository.MyFirstLevelRepository@4ebff610
```

#### 注解的"层次性"

结构

- @Component
- @Repository
- FirstLevelRepository

### Spring @Enable 模块装配

Spring Framework 3.1 开始支持”@Enable 模块驱动“。所谓“模块”是指具备相同领域的功能组件集合， 组合所形成一个独立
的单元。比如 Web MVC 模块、AspectJ 代理模块、Caching（缓存）模块、JMX（Java 管 理扩展）模块、Async（异步处
理）模块等。

简单概括一下就是，借助@Import 的支持，收集和注册特定场景相关的 bean 定义。

#### spring 常用的注解

- @Controller
- @Service
- @Autowired
- @Resource
- @RequestMapping
- @RequestParam
- @ModelAttribute
- @Repository
- @Component
- @Qualifier
- @Scope

singleton：定义 bean 的范围为每个 spring 容器一个实例（默认值）

prototype：定义 bean 可以被多次实例化（使用一次就创建一次）

request： 定义 bean 的范围是 http 请求（springMVC 中有效）

session： 定义 bean 的范围是 http 会话（springMVC 中有效）

global-session：定义 bean 的范围是全局 http 会话（portlet 中有效）

#### @Enable 举例

| 框架实现 | @Enable 注解模块 | 激活模块 |
| ---------------- | ------------------------------- | ------------------- |
| Spring Framework | @EnableWebMvc | Web MVC 模块 |
| | @EnableTransactionManagement | 事务管理模块 |
| | @EnableCaching | Caching 模块 |
| | @EnableMBeanExport | JMX 模块 |
| | @EnableAsync | 异步处理模块 |
| | @EnableWebFlux | Web Flux 模块 |
| | @EnableAspectJAutoProxy AspectJ | 代理模块 |
| Spring Boot | @EnableAutoConfiguration | 自动装配模块 |
| | @EnableManagementContext | Actuator 管理模块 |
| | @EnableConfigurationProperties | 配置属性绑定模块 |
| | @EnableOAuth2Sso | OAuth2 单点登录模块 |
| Spring Cloud | @EnableEurekaServer | Eureka 服务器模块 |
| | @EnableConfigServer | 配置服务器模块 |
| | @EnableFeignClients | Feign 客户端模块 |
| | @EnableZuulProxy | 服务网关 Zuul 模块 |
| | @EnableCircuitBreaker | 服务熔断模块 |

##### @Enable 实现方式

我们先看看 spring 是怎么帮我们实现的.

- 注解驱动方式

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

```java
@Configuration
public class DelegatingWebMvcConfiguration extends
WebMvcConfigurationSupport {
...
}
```

接口编程方式

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(CachingConfigurationSelector.class)
public @interface EnableCaching {
...
}
```

```java
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {
   @Override
    public String[] selectImports(AdviceMode adviceMode) {
        switch (adviceMode) {
            case PROXY:
                return getProxyImports();
            case ASPECTJ:
                return getAspectJImports();
            default:
                return null;
        }
    }
}
```

手动实现一下. .

- 基于注解驱动

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(HelloWorldConfiguration.class)
public @interface EnableHelloWorld {
}
```

```java
@Configuration
public class HelloWorldConfiguration {
@Bean
public String helloWorld(){
return "hello, World 2018";
}
}

```

- 接口编程方式

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(HelloWorldImportSelector.class)
public @interface EnableHelloWorld2 {
}
```

```java
public class HelloWorldImportSelector implements ImportSelector {
@Override
public String[] selectImports(AnnotationMetadata importingClassMetadata) {
return new String[]{HelloWorldConfiguration.class.getName()};
}
}
```

输出

```java
@EnableHelloWorld2
public class EnableAutoConfigurationBootstrap {

public static void main(String[] args) {
ConfigurableApplicationContext context = new SpringApplicationBuilder(EnableAutoConfigurationBootstrap.class).web(WebApplicationType.NONE).run(args);
String helloword = context.getBean("helloWorld", String.class);
System.out.println(" hello world : " + helloword);
context.close();
}
}
hello world : hello, World 2018
```

<!--
- 问题1

为什么spring能够识别自定义的注解?

importselector 是导入筛选器. 对spring管理的bean 进行筛选装配, DeferredImportselector是延迟装配

这个接口在ConfigurationClassParser这个类的processImports方法
-->

### Spring 条件装配

spring3.1 开始，spring 引入了@Profile 注解，可以根据环境 Environment 的不同引入不同的配置。spring4.0 开始，Conditional 注解可以更灵活的根据不同条件引入不同的配置。

#### 条件注解举例

| Spring 注解 | 场景说明 | 起始版本 |
| ------------ | -------------- | -------- |
| @Profile | 配置化条件装配 | 3.1 |
| @Conditional | 编程条件装配 | 4.0 |

#### 编程方式 - @Conditional

举个例子

##### 基于配置方式 Profile

先看一下 Profile 实现

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {
    String[] value();
}
```

```java
class ProfileCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(Profiles.of((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
        return true;
    }

}
```

```java
@FunctionalInterface
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}
```

依照上述,我们可以自己写一个基于 JAVA8 还是 JAVA7 的计算求和接口

```java
public interface CalculateService {
Integer sum(Integer... values);
}
```

```java
@Service
@Profile("java7")
public class JAVA7CalculateService implements CalculateService {

@Override
public Integer sum(Integer... values) {
int sum = 0;
for (Integer value : values) {
sum += value;
}
return sum;
}
public static void main(String[] args){
CalculateService calculateService = new JAVA7CalculateService();
System.out.println(calculateService.sum(1,2,3,4,5,56,6,7,7,8,8));
}
}
```

```java
@Service
@Profile("java8")
public class JAVA8CalculateService implements CalculateService {

@Override
public Integer sum(Integer... values) {
return Stream.of(values).reduce(0, Integer::sum);
}


public static void main(String[] args){
CalculateService calculateService = new JAVA8CalculateService();
System.out.println(calculateService.sum(1,2,3,4,5,56,6,7,7,8,8));
}
}
```

启动类

```java
@SpringBootApplication(scanBasePackages = "com.zx.diveinspringboot.service")
public class CalculateServcieBootstrap {

public static void main(String[] args) {
ConfigurableApplicationContext context =
new SpringApplicationBuilder(CalculateServcieBootstrap.class).web(WebApplicationType.NONE).profiles("java7").run(args);
CalculateService calculateService = context.getBean(CalculateService.class);
System.out.println(" sum : " + calculateService.sum(1, 2, 3, 4, 5, 56, 6, 7, 7, 8, 8));
context.close();
}
}
```

##### 基于编程方式配置-自定义条件装配

首先来熟悉一下每个的含义

- ConditionalOnBean:
存在某个 bean 已经被 spring 容器装配的时候，当前 bean 也自动装配。

- ConditionalOnMissingBean:
某个 bean 没有被 spring 容器装配的时候自定装配当前类。

- ConditionalOnClass:
当 classpath 路径下有这个类依赖的时候自动装配，没有的时候不装配

- ConditionalOnMissingClass
当 classpath 路径下没有这个类依赖的时候自动装配

- ConditionalOnExpression ：
当指定的 SpEL 表达式返回 true 的时候自动装配，

- ConditionalOnJava：
当前 jvm 的版本号是\*\*的时候进行自动装配。

- ConditionalOnNotWebApplication：
当前环境不是 web 环境的时候装配

- conditionConditionalOnWebApplication：
当前环境是 web 环境的时候自动装配.

- ConditionalOnResource：
当某个资源存在的时候自动装配。

- ConditionalOnProperty
当配置文件存在的时候，或者当配置文件等于某个值的时候自动装配。

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnClassCondition.class)
public @interface ConditionalOnClass {
Class<?>[] value() default {};
String[] name() default {};
}
```

```java
@Order(Ordered.HIGHEST_PRECEDENCE)
class OnClassCondition extends FilteringSpringBootCondition {
    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext context,
            AnnotatedTypeMetadata metadata) {
        ClassLoader classLoader = context.getClassLoader();
        ConditionMessage matchMessage = ConditionMessage.empty();
        List<String> onClasses = getCandidates(metadata, ConditionalOnClass.class);
        if (onClasses != null) {
            List<String> missing = filter(onClasses, ClassNameFilter.MISSING,
                    classLoader);
            if (!missing.isEmpty()) {
                return ConditionOutcome
                        .noMatch(ConditionMessage.forCondition(ConditionalOnClass.class)
                                .didNotFind("required class", "required classes")
                                .items(Style.QUOTE, missing));
            }
            matchMessage = matchMessage.andCondition(ConditionalOnClass.class)
                    .found("required class", "required classes").items(Style.QUOTE,
                            filter(onClasses, ClassNameFilter.PRESENT, classLoader));
        }
        List<String> onMissingClasses = getCandidates(metadata,
                ConditionalOnMissingClass.class);
        if (onMissingClasses != null) {
            List<String> present = filter(onMissingClasses, ClassNameFilter.PRESENT,
                    classLoader);
            if (!present.isEmpty()) {
                return ConditionOutcome.noMatch(
                        ConditionMessage.forCondition(ConditionalOnMissingClass.class)
                                .found("unwanted class", "unwanted classes")
                                .items(Style.QUOTE, present));
            }
            matchMessage = matchMessage.andCondition(ConditionalOnMissingClass.class)
                    .didNotFind("unwanted class", "unwanted classes")
                    .items(Style.QUOTE, filter(onMissingClasses, ClassNameFilter.MISSING,
                            classLoader));
        }
        return ConditionOutcome.match(matchMessage);
    }

private List<String> getCandidates(AnnotatedTypeMetadata metadata,
            Class<?> annotationType) {
        MultiValueMap<String, Object> attributes = metadata
                .getAllAnnotationAttributes(annotationType.getName(), true);
        if (attributes == null) {
            return null;
        }
        List<String> candidates = new ArrayList<>();
        addAll(candidates, attributes.get("value"));
        addAll(candidates, attributes.get("name"));
        return candidates;
    }

}
```

自己实现一个

条件注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnSystemPropertyCondition.class)
public @interface ConditionOnSystemProperty {

String name();

String value();
}
```

condition 条件判断类

```java
public class OnSystemPropertyCondition implements Condition {

@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {

Map<String , Object> attr = metadata.getAnnotationAttributes(ConditionOnSystemProperty.class.getName());

String name = String.valueOf(attr.get("name"));
String value = String.valueOf(attr.get("value"));
String javaPropertyValue = System.getProperty(name);
return value.equals(javaPropertyValue);
}
}
```

main

```java
public class ConditionalOnSystemPropertyBootstrap {

@Bean
@ConditionOnSystemProperty(name="user.name" , value="zx")
public String helloWorld(){
return "hello , world ";
}

public static void main(String[] args){
ConfigurableApplicationContext context =
new SpringApplicationBuilder(ConditionalOnSystemPropertyBootstrap.class).web(WebApplicationType.NONE).run(args);

String helloWorld = context.getBean("helloWorld" ,String.class);
System.out.println(" hello world bean : " + helloWorld);
context.close();
}
}
```

### 自动装配原理

前面介绍了

- 注解基本语法
- spring手动装配
- @enable装配
- conditional条件装配

但是springboot的一套流程还没有介绍

1. @SpringBootApplication 注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

其中最重要的就是@EnableAutoConfiguration

这个注解其实就是上面的条件装配注解@EnableXXXX

在看看 @EnableAutoConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

这里利用了 import 导入了 AutoConfigurationImportSelector.class

这个类实现了这个方法

```java
@Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
//1加载META-INF/spring-autoconfigure-metadata.properties文件
        AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
                .loadMetadata(this.beanClassLoader);
//2. 看下面
        AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
                autoConfigurationMetadata, annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }

    protected AutoConfigurationEntry getAutoConfigurationEntry(
            AutoConfigurationMetadata autoConfigurationMetadata,
            AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        }
////2获取注解的属性及其值（PS：注解指的是@EnableAutoConfiguration注解）
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
//3.在classpath下所有的META-INF/spring.factories文件中查找org.springframework.boot.autoconfigure.EnableAutoConfiguration的值，并将其封装到一个List中返回 SpringFactoriesLoader.loadFactoryNames
        List<String> configurations = getCandidateConfigurations(annotationMetadata,
                attributes);
//4.对上一步返回的List中的元素去重、排序
        configurations = removeDuplicates(configurations);
//5.依据第2步中获取的属性值排除一些特定的类(EnableAutoConfiguration可以配置排除的类)
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
//6对上一步中所得到的List进行过滤，过滤的依据是条件匹配。这里用到的过滤器是
//org.springframework.boot.autoconfigure.condition.OnClassCondition最终返回的是一个ConditionOutcome[]
//数组。（PS：很多类都是依赖于其它的类的，当有某个类时才会装配，所以这次过滤的就是根据是否有某个
//class进而决定是否装配的。这些类所依赖的类都写在META-INF/spring-autoconfigure-metadata.properties文件里）
        checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = filter(configurations, autoConfigurationMetadata);
        fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationEntry(configurations, exclusions);
    }

```

1. 轻扫 AutoConfigurationMetadataLoader.class

```java
final class AutoConfigurationMetadataLoader {

    protected static final String PATH = "META-INF/"
            + "spring-autoconfigure-metadata.properties";

    private AutoConfigurationMetadataLoader() {
    }
```

里面有一堆这样的
META-INF/spring.factories

```java
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener
```

META-INF/spring-autoconfigure-metadata.properties

```java
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration.AutoConfigureAfter=org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration.Configuration=
org.springframework.boot.autoconfigure.data.neo4j.Neo4jBookmarkManagementConfiguration.Configuration=
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration=
org.springframework.boot.autoconfigure.kafka.KafkaAnnotationDrivenConfiguration.Configuration=
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration.ConditionalOnClass=org.influxdb.InfluxDB
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration=
org.springframework.boot.autoconfigure.session.SessionRepositoryFilterConfiguration.Configuration=
```

## 三、自动依赖过程总结

1.通过各种注解实现了类与类之间的依赖关系，容器在启动的时候 Application.run，会调用 EnableAutoConfigurationImportSelector.class 的 selectImports 方法（其实是其父类的方法）--这里需要注意，调用这个方法之前发生了什么和是在哪里调用这个方法需要进一步的探讨

2.selectImports 方法最终会调用 SpringFactoriesLoader.loadFactoryNames 方法来获取一个全面的常用 BeanConfiguration 列表

3.loadFactoryNames 方法会读取 FACTORIES_RESOURCE_LOCATION（也就是 spring-boot-autoconfigure.jar 下面的 spring.factories），获取到所有的 Spring 相关的 Bean 的全限定名 ClassName，大概 120 多个

4.selectImports 方法继续调用 filter(configurations, autoConfigurationMetadata);这个时候会根据这些 BeanConfiguration 里面的条件，来一一筛选，最关键的是
@ConditionalOnClass，这个条件注解会去 classpath 下查找，jar 包里面是否有这个条件依赖类，所以必须有了相应的 jar 包，才有这些依赖类，才会生成 IOC 环境需要的一些默认配置 Bean

5.最后把符合条件的 BeanConfiguration 注入默认的 EnableConfigurationPropertie 类里面的属性值，并且注入到 IOC 环境当中

底层装配技术

Spring 模式注解装配


Spring @Enable 模块装配



Spring 条件装配

Spring 工厂加载机制

### 手动实现自动装配

还记得之前学过的Spring模式注解吗？

还记得之前学过的@Enable模块装配吗?

还记得之前学过的条件装配吗？

不记得自己从头看.....

- resources下新建-META-INF目录---新建spring.factories文件

内容如下:

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.spring_source.configure.HelloWorldAutoConfiguration
```

- HelloWorldAutoConfiguration.java

```java

/**
* Hello World 自动装配
*/
@Configuration //Spring 模式注解装配
@EnableHelloWorld //Spring @Enable模块装配
@ConditionalOnClass(B.class) //条件装配
public class HelloWorldAutoConfiguration
{
}
```

- B.java

```java
public class B
{
}
```

- @EnableHelloWorld.java

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(HelloWorldConfiguration.class)
public @interface EnableHelloWorld
{
}
```

- HelloWorldConfiguration.java

```java
public class HelloWorldConfiguration
{
@Bean
public String helloworld(){
return " hello world";
}
}
```

- EnableAutoConfigurationBootstrap.java

```java
@EnableAutoConfiguration
public class EnableAutoConfigurationBootstrap {

public static void main(String[] args) {
ConfigurableApplicationContext context = new SpringApplicationBuilder(EnableAutoConfigurationBootstrap.class)
.web(WebApplicationType.NONE)
.run(args);
//helloWorld Bean是否存在
String helloWorld = context.getBean("helloworld", String.class);
System.out.println("helloWorld Bean : " + helloWorld);
//关闭上下文
context.close();
}
}
```

- 输出

```java
. ____ _ __ _ _
/\\ / ___'_ __ _ _(_)_ __ __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/ ___)| |_)| | | | | || (_| | ) ) ) )
' |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot :: (v2.1.8.RELEASE)

2019-09-17 16:02:43.249 INFO 212656 --- [ restartedMain] c.e.s.EnableAutoConfigurationBootstrap : Starting EnableAutoConfigurationBootstrap on DESKTOP-50REN51 with PID 212656 (E:\test\spring_source\target\classes started by DELL in E:\test\spring_source)
2019-09-17 16:02:43.252 INFO 212656 --- [ restartedMain] c.e.s.EnableAutoConfigurationBootstrap : No active profile set, falling back to default profiles: default
2019-09-17 16:02:43.288 INFO 212656 --- [ restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2019-09-17 16:02:45.488 INFO 212656 --- [ restartedMain] o.s.b.d.a.OptionalLiveReloadServer : LiveReload server is running on port 35729
2019-09-17 16:02:45.520 INFO 212656 --- [ restartedMain] c.e.s.EnableAutoConfigurationBootstrap : Started EnableAutoConfigurationBootstrap in 2.676 seconds (JVM running for 3.261)
helloWorld Bean : hello world
```

【说明】
HelloWorldAutoConfiguration

条件判断 ：条件是满足的-----是否有B类

模式注解： @Configuration

@Enable模块：

@EnableHelloWorld ->加载 HelloWorldImportSelector ->Selector 加载HelloWorldConfiguration - > HelloWorldConfiguration会自动生成 helloWorld bean
引导类启动后这个bean就会被装载

也就是说：

模块装配的时候，前面的条件都满足就会引导出@EnableHelloWorld ->加载 HelloWorldImportSelector ->加载HelloWorldConfiguration - > helloWorld bean
