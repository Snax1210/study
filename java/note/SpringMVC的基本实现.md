# Spring



## 前沿

在Web应用程序设计中，MVC模式已经被广泛使用。SpringMVC以DispatcherServlet为核心，负责协调和组织不同组件以完成请求处理并返回响应的工作，实现了MVC模式。想要实现自己的SpringMVC框架，需要从以下几点入手：

- 了解SpringMVC运行流程及九大组件
- 梳理自己的SpringMVC的设计思路
- 实现自己的SpringMVC框架

## 了解SpringMVC运行流程及九大组件

![](https://avalon-zheng.xin/static/static-front/upload/cb7fe03a-0811-400f-8d88-956f456c50a4 "")

- 用户发送请求至前端控制器DispatcherServlet

- DispatcherServlet收到请求调用HandlerMapping处理器映射器。

- 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

- DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

- 执行处理器(Controller，也叫后端控制器)。

- Controller执行完成返回ModelAndView

- HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

- DispatcherServlet将ModelAndView传给ViewReslover视图解析器

- ViewReslover解析后返回具体View

- DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

- DispatcherServlet响应用户。


## 九大组件

大家知道spring对servlet进行的封装,DispatcherServlet对httpServlet的实现

```java
protected void initStrategies(ApplicationContext context) {
//用于处理上传请求。处理方法是将普通的request包装成MultipartHttpServletRequest，后者可以直接调用getFile方法获取File.
initMultipartResolver(context);
//SpringMVC主要有两个地方用到了Locale：一是ViewResolver视图解析的时候；二是用到国际化资源或者主题的时候。
initLocaleResolver(context);
//用于解析主题。SpringMVC中一个主题对应一个properties文件，里面存放着跟当前主题相关的所有资源、
//如图片、css样式等。SpringMVC的主题也支持国际化，
initThemeResolver(context);
//用来查找Handler的。
initHandlerMappings(context);
//从名字上看，它就是一个适配器。Servlet需要的处理方法的结构却是固定的，都是以request和response为参数的方法。
//如何让固定的Servlet处理方法调用灵活的Handler来进行处理呢？这就是HandlerAdapter要做的事情
initHandlerAdapters(context);
//其它组件都是用来干活的。在干活的过程中难免会出现问题，出问题后怎么办呢？
//这就需要有一个专门的角色对异常情况进行处理，在SpringMVC中就是HandlerExceptionResolver。
initHandlerExceptionResolvers(context);
//有的Handler处理完后并没有设置View也没有设置ViewName，这时就需要从request获取ViewName了，
//如何从request中获取ViewName就是RequestToViewNameTranslator要做的事情了。
initRequestToViewNameTranslator(context);
//ViewResolver用来将String类型的视图名和Locale解析为View类型的视图。
//View是用来渲染页面的，也就是将程序返回的参数填入模板里，生成html（也可能是其它类型）文件。
initViewResolvers(context);
//用来管理FlashMap的，FlashMap主要用在redirect重定向中传递参数。
initFlashMapManager(context);
}
```

## 梳理SpringMVC的设计思路

本文只实现自己的@Controller、@RequestMapping、@RequestParam注解起作用，其余SpringMVC功能可以尝试自己实现。

- 读取配置

SpringMVC本质上是一个Servlet,这个 Servlet 继承自 HttpServlet。FrameworkServlet负责初始化SpringMVC的容器，并将Spring容器设置为父容器。因为本文只是实现SpringMVC，对于Spring容器不做过多讲解。
为了读取web.xml中的配置，我们用到ServletConfig这个类，它代表当前Servlet在web.xml中的配置信息。通过web.xml中加载我们自己写的DispatcherServlet和读取配置文件。

- 初始化阶段

在前面我们提到DispatcherServlet的initStrategies方法会初始化9大组件，但是这里将实现一些SpringMVC的最基本的组件而不是全部，按顺序包括：

加载配置文件

扫描用户配置包下面所有的类

拿到扫描到的类，通过反射机制，实例化。并且放到ioc容器中(Map的键值对 beanName-bean) beanName默认是首字母小写

初始化HandlerMapping，这里其实就是把url和method对应起来放在一个k-v的Map中,在运行阶段取出

- 运行阶段

每一次请求将会调用doGet或doPost方法，所以统一运行阶段都放在doDispatch方法里处理，它会根据url请求去HandlerMapping中匹配到对应的Method，然后利用反射机制调用Controller中的url对应的方法，并得到结果返回。按顺序包括以下功能：

1. 异常的拦截
2. 获取请求传入的参数并处理参数
3. 通过初始化好的handlerMapping中拿出url对应的方法名，反射调用

## 代码实现

首先，新建一个maven项目，在pom.xml中导入以下依赖

```java
<dependency>
<groupId>javax.servlet</groupId>
<artifactId>javax.servlet-api</artifactId>
<version>3.0.1</version>
<scope>provided</scope>
</dependency>
```

- 添加5个注解

```java
//类注解
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Controller
{
String value() default "";
}
//路径注解
@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestMapping
{
String value() default "";
}
//参数注解
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestParam
{
String value() default "";
}
//service
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Service
{
String value() default "";
}
// 依赖注入
@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired
{
String value() default "";
}

```

- 模拟扫包

application.properties

```java
scanPackage=com.suntang
```

- web.xml配置

```java
<web-app>
<display-name>Archetype Created Web Application</display-name>

<servlet>
<servlet-name>springmvc</servlet-name>
<servlet-class>com.suntang.dispatch.DispatcherServlet</servlet-class>
<init-param>
<param-name>contextConfigLocation</param-name>
<param-value>application.properties</param-value>
</init-param>
<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
<servlet-name>springmvc</servlet-name>
<url-pattern>/*</url-pattern>
</servlet-mapping>
</web-app>
```

- 编写servlet

```java
public class DispatcherServlet extends HttpServlet
{
private Properties properties = new Properties();

private List<String> classNames = new ArrayList<>();

private Map<String, Object> beans = new HashMap<>();

private Map<String, Method> handlerMapping = new HashMap<>();

private Map<String, Object> controllerMap = new HashMap<>();

@Override
public void init(ServletConfig config) throws ServletException
{
//模拟九大组件初始化

//第1大组件 1.加载配置文件
initLoadConfig(config.getInitParameter("contextConfigLocation"));

//2.初始化所有相关联的类,扫描用户设定的包下面所有的类 (需要依赖注入的类)
doScanner(properties.getProperty("scanPackage"));

//3.拿到扫描到的类,通过反射机制,实例化,并且放到ioc容器中(k-v beanName-bean) beanName默认是首字母小写
doInstance();

// 4.将IOC容器中的service对象设置给controller层定义的field上
doIoc();

//第4大组件 .初始化HandlerMapping(将url和method对应上)
initHandlerMappings();
}

@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
{
this.doPost(req, resp);
}

@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
{
try
{
//处理请求
doDispatch(req, resp);
}
catch (Exception e)
{
resp.getWriter().write("500!! Server Exception");
}

}

private void doInstance()
{
if (classNames.isEmpty())
{
return;
}
for (String className : classNames)
{
try
{
//把类搞出来,反射来实例化(只有加@Controller需要实例化)
Class<?> clazz = Class.forName(className);
if (clazz.isAnnotationPresent(Controller.class))
{
Controller controller = clazz.getAnnotation(Controller.class);
if (controller.value().equals(""))
{
beans.put(toLowerFirstWord(clazz.getSimpleName()), clazz.newInstance());
}
else
{
beans.put(controller.value(), clazz.newInstance());
}
}
else if (clazz.isAnnotationPresent(Service.class))
{
Service service = clazz.getAnnotation(Service.class);
if (service.value().equals(""))
{
beans.put(toLowerFirstWord(clazz.getSimpleName()), clazz.newInstance());
}
else
{
beans.put(service.value(), clazz.newInstance());
}
}
else
{
continue;
}

}
catch (Exception e)
{
e.printStackTrace();
continue;
}
}
}

/**
* 依赖注入
*/
private void doIoc()
{
if (beans.isEmpty())
{
System.out.println("beans is null");
return;
}
for (Map.Entry<String, Object> map : beans.entrySet())
{
//IOC实例 , key是描述,也有可能是类名
Object instance = map.getValue();

//获取类
Class<?> clazz = instance.getClass();

if (clazz.isAnnotationPresent(Controller.class))
{
//获取成员变量
Field[] fields = clazz.getDeclaredFields();
for (Field field : fields)
{
if (field.isAnnotationPresent(Autowired.class))
{
Autowired requestParam = field.getAnnotation(Autowired.class);
String value = requestParam.value();
//未设置值则用类名
if ("".equals(value))
{
value = field.getName();
}
// 由于此类成员变量设置为private，需要强行设置
field.setAccessible(true);
try
{
field.set(instance, beans.get(value));
}
catch (IllegalAccessException e)
{
e.printStackTrace();
}
}
}
}
}
}

private void doScanner(String packageName)
{

//把所有的.替换成/
URL url = this.getClass().getClassLoader().getResource("/" + packageName.replaceAll("\\.", "/"));
File dir = new File(url.getFile());
for (File file : dir.listFiles())
{
if (file.isDirectory())
{
//递归读取包
doScanner(packageName + "." + file.getName());
}
else
{
String className = packageName + "." + file.getName().replace(".class", "");
classNames.add(className);
}
}

}

/**
* 处理请求
*
* @param req
* @param resp
* @throws Exception
*/
private void doDispatch(HttpServletRequest req, HttpServletResponse resp) throws Exception
{

if (handlerMapping.isEmpty())
{
return;
}

String url = req.getRequestURI();
String contextPath = req.getContextPath();

url = url.replace(contextPath, "").replaceAll("/+", "/");

System.out.println(" request url : " + url);
if (!this.handlerMapping.containsKey(url))
{
resp.getWriter().write("404 NOT FOUND!");
return;
}

Method method = this.handlerMapping.get(url);

//获取方法的参数列表
Class<?>[] parameterTypes = method.getParameterTypes();

//获取请求的参数
Map<String, String[]> parameterMap = req.getParameterMap();

//保存参数值
Object[] paramValues = new Object[parameterTypes.length];

//获取方法的参数注解
Annotation[][] parameterAnnotations = method.getParameterAnnotations();

//方法的参数列表
// 这是其实已经是适配器 HandlerAdapter , 这块比较复杂 , 这里简写了
for (int i = 0; i < parameterTypes.length; i++)
{
//根据参数名称，做某些处理
String requestParam = parameterTypes[i].getSimpleName();

//获取注解
Annotation[] annotations = parameterAnnotations[i];

if (requestParam.equals("HttpServletRequest"))
{
//参数类型已明确，这边强转类型
paramValues[i] = req;
continue;
}
if (requestParam.equals("HttpServletResponse"))
{
paramValues[i] = resp;
continue;
}
//请求的参数遍历
for (Entry<String, String[]> param : parameterMap.entrySet())
{
//方法参数列表
for (Annotation annotation : annotations)
{
//参数名
String paramName = "";
if (annotation instanceof RequestParam)
{
RequestParam p = (RequestParam)annotation;
paramName = p.value();

}
if("".equals(paramName)){
//如果没有匹配上注解, 则根据名字来 , 这里太复杂了涉及到JAVA字节码了. 不写了
}

//如果匹配上了
if (paramName.equals(param.getKey()))
{
if (requestParam.equals("String"))
{
//给定值
String value =
Arrays.toString(param.getValue()).replaceAll("\\[|\\]", "").replaceAll(",\\s", ",");
paramValues[i] = value;
}
}
}
}

}
//利用反射机制来调用
try
{
method.invoke(this.controllerMap.get(url), paramValues);//第一个参数是method所对应的实例 在ioc容器中
}
catch (Exception e)
{
e.printStackTrace();
}

}

/**
* 扫描所有的requestMapping注解,获取到请求路径
*/
private void initHandlerMappings()
{

if (beans.isEmpty())
{
return;
}
try
{
for (Entry<String, Object> entry : beans.entrySet())
{
Class<? extends Object> clazz = entry.getValue().getClass();
if (!clazz.isAnnotationPresent(Controller.class))
{
continue;
}
//获取控制器类名
String controllerName = toLowerFirstWord(clazz.getSimpleName());
if(!clazz.getAnnotation(Controller.class).value().equals("")){
controllerName = clazz.getAnnotation(Controller.class).value();
}

//拼url时,是controller头的url拼上方法上的url
String baseUrl = "";
if (clazz.isAnnotationPresent(RequestMapping.class))
{
RequestMapping annotation = clazz.getAnnotation(RequestMapping.class);
baseUrl = annotation.value();
}
Method[] methods = clazz.getMethods();
for (Method method : methods)
{
if (!method.isAnnotationPresent(RequestMapping.class))
{
continue;
}
RequestMapping annotation = method.getAnnotation(RequestMapping.class);
String url = annotation.value();

url = (baseUrl + "/" + url).replaceAll("/+", "/");
handlerMapping.put(url, method);
//这里必须从beans 里面拿 因为里面已经有service实例
controllerMap.put(url, beans.get(controllerName));
System.out.println(url + "," + method);
}

}

}
catch (Exception e)
{
e.printStackTrace();
}

}

/**
* 初始化配置文件
*
* @param contextConfigLocation
*/
private void initLoadConfig(String contextConfigLocation)
{
//把web.xml中的contextConfigLocation对应value值的文件加载到流里面
InputStream resourceAsStream = this.getClass().getClassLoader().getResourceAsStream(contextConfigLocation);
try
{
//用Properties文件加载文件里的内容
properties.load(resourceAsStream);
}
catch (IOException e)
{
e.printStackTrace();
}
finally
{
if (null != resourceAsStream)
{
try
{
resourceAsStream.close();
}
catch (IOException e)
{
e.printStackTrace();
}
}
}
}

/**
* 把字符串的首字母小写
*
* @param name
* @return
*/
private String toLowerFirstWord(String name)
{
return name.substring(0, 1).toLowerCase() + name.substring(1);
}
}
```

controller

```java
@RequestMapping("/hello")
@Controller
public class HelloController
{
@Autowired
HelloService helloService;

@RequestMapping("/index")
public String index(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
@RequestParam("test") String str) throws IOException
{
httpServletResponse.getWriter().write("str : " + str + ", service : " + helloService.hello());
return str;
}
}
```

service

```java
@Service
public class HelloService
{
public String hello(){
return "hello service";
}
}
```

整体架构如下

![](https://avalon-zheng.xin/static/static-front/upload/361ed0b2-57a9-478e-a476-5b79feab2319 "")

输入?test=1

![](https://avalon-zheng.xin/static/static-front/upload/65f15ce8-cb0b-4a56-a6b6-4b70522c878d "")

输入?str=1

![](https://avalon-zheng.xin/static/static-front/upload/f5c28f12-a35b-49b4-8dd9-cb3b04a90762 "")

可以看到.没有匹配上, 到这里,手写简单的springmvc已经完成了

## springmvc和struts2区别

Struts2的核心控制器是过滤器（filter），springmvc的核心控制器（Servlet）

Struts2的Action是多例的，springmvc的Handler（处理器）是单例的

Struts2用Action的属性接收客户端数据（必须多例才线程安全），springmvc使用方法的形参接收客户端数据（线程安全的）

Struts2是针对Action类型进行mapping（和url关联）

Springmvc是针对Handler中的处理请求的方法进行mappring（和url关联），

Springmvc在请求处理的控制上更精确，（粒度更小）

# Spring



## 前沿

在Web应用程序设计中，MVC模式已经被广泛使用。SpringMVC以DispatcherServlet为核心，负责协调和组织不同组件以完成请求处理并返回响应的工作，实现了MVC模式。想要实现自己的SpringMVC框架，需要从以下几点入手：

- 了解SpringMVC运行流程及九大组件
- 梳理自己的SpringMVC的设计思路
- 实现自己的SpringMVC框架

## 了解SpringMVC运行流程及九大组件

![](https://avalon-zheng.xin/static/static-front/upload/cb7fe03a-0811-400f-8d88-956f456c50a4 "")

- 用户发送请求至前端控制器DispatcherServlet

- DispatcherServlet收到请求调用HandlerMapping处理器映射器。

- 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

- DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

- 执行处理器(Controller，也叫后端控制器)。

- Controller执行完成返回ModelAndView

- HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

- DispatcherServlet将ModelAndView传给ViewReslover视图解析器

- ViewReslover解析后返回具体View

- DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

- DispatcherServlet响应用户。


## 九大组件

大家知道spring对servlet进行的封装,DispatcherServlet对httpServlet的实现

```java
protected void initStrategies(ApplicationContext context) {
//用于处理上传请求。处理方法是将普通的request包装成MultipartHttpServletRequest，后者可以直接调用getFile方法获取File.
initMultipartResolver(context);
//SpringMVC主要有两个地方用到了Locale：一是ViewResolver视图解析的时候；二是用到国际化资源或者主题的时候。
initLocaleResolver(context);
//用于解析主题。SpringMVC中一个主题对应一个properties文件，里面存放着跟当前主题相关的所有资源、
//如图片、css样式等。SpringMVC的主题也支持国际化，
initThemeResolver(context);
//用来查找Handler的。
initHandlerMappings(context);
//从名字上看，它就是一个适配器。Servlet需要的处理方法的结构却是固定的，都是以request和response为参数的方法。
//如何让固定的Servlet处理方法调用灵活的Handler来进行处理呢？这就是HandlerAdapter要做的事情
initHandlerAdapters(context);
//其它组件都是用来干活的。在干活的过程中难免会出现问题，出问题后怎么办呢？
//这就需要有一个专门的角色对异常情况进行处理，在SpringMVC中就是HandlerExceptionResolver。
initHandlerExceptionResolvers(context);
//有的Handler处理完后并没有设置View也没有设置ViewName，这时就需要从request获取ViewName了，
//如何从request中获取ViewName就是RequestToViewNameTranslator要做的事情了。
initRequestToViewNameTranslator(context);
//ViewResolver用来将String类型的视图名和Locale解析为View类型的视图。
//View是用来渲染页面的，也就是将程序返回的参数填入模板里，生成html（也可能是其它类型）文件。
initViewResolvers(context);
//用来管理FlashMap的，FlashMap主要用在redirect重定向中传递参数。
initFlashMapManager(context);
}
```

## 梳理SpringMVC的设计思路

本文只实现自己的@Controller、@RequestMapping、@RequestParam注解起作用，其余SpringMVC功能可以尝试自己实现。

- 读取配置

SpringMVC本质上是一个Servlet,这个 Servlet 继承自 HttpServlet。FrameworkServlet负责初始化SpringMVC的容器，并将Spring容器设置为父容器。因为本文只是实现SpringMVC，对于Spring容器不做过多讲解。
为了读取web.xml中的配置，我们用到ServletConfig这个类，它代表当前Servlet在web.xml中的配置信息。通过web.xml中加载我们自己写的DispatcherServlet和读取配置文件。

- 初始化阶段

在前面我们提到DispatcherServlet的initStrategies方法会初始化9大组件，但是这里将实现一些SpringMVC的最基本的组件而不是全部，按顺序包括：

加载配置文件

扫描用户配置包下面所有的类

拿到扫描到的类，通过反射机制，实例化。并且放到ioc容器中(Map的键值对 beanName-bean) beanName默认是首字母小写

初始化HandlerMapping，这里其实就是把url和method对应起来放在一个k-v的Map中,在运行阶段取出

- 运行阶段

每一次请求将会调用doGet或doPost方法，所以统一运行阶段都放在doDispatch方法里处理，它会根据url请求去HandlerMapping中匹配到对应的Method，然后利用反射机制调用Controller中的url对应的方法，并得到结果返回。按顺序包括以下功能：

1. 异常的拦截
2. 获取请求传入的参数并处理参数
3. 通过初始化好的handlerMapping中拿出url对应的方法名，反射调用

## 代码实现

首先，新建一个maven项目，在pom.xml中导入以下依赖

```java
<dependency>
<groupId>javax.servlet</groupId>
<artifactId>javax.servlet-api</artifactId>
<version>3.0.1</version>
<scope>provided</scope>
</dependency>
```

- 添加5个注解

```java
//类注解
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Controller
{
String value() default "";
}
//路径注解
@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestMapping
{
String value() default "";
}
//参数注解
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestParam
{
String value() default "";
}
//service
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Service
{
String value() default "";
}
// 依赖注入
@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired
{
String value() default "";
}

```

- 模拟扫包

application.properties

```java
scanPackage=com.suntang
```

- web.xml配置

```java
<web-app>
<display-name>Archetype Created Web Application</display-name>

<servlet>
<servlet-name>springmvc</servlet-name>
<servlet-class>com.suntang.dispatch.DispatcherServlet</servlet-class>
<init-param>
<param-name>contextConfigLocation</param-name>
<param-value>application.properties</param-value>
</init-param>
<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
<servlet-name>springmvc</servlet-name>
<url-pattern>/*</url-pattern>
</servlet-mapping>
</web-app>
```

- 编写servlet

```java
public class DispatcherServlet extends HttpServlet
{
private Properties properties = new Properties();

private List<String> classNames = new ArrayList<>();

private Map<String, Object> beans = new HashMap<>();

private Map<String, Method> handlerMapping = new HashMap<>();

private Map<String, Object> controllerMap = new HashMap<>();

@Override
public void init(ServletConfig config) throws ServletException
{
//模拟九大组件初始化

//第1大组件 1.加载配置文件
initLoadConfig(config.getInitParameter("contextConfigLocation"));

//2.初始化所有相关联的类,扫描用户设定的包下面所有的类 (需要依赖注入的类)
doScanner(properties.getProperty("scanPackage"));

//3.拿到扫描到的类,通过反射机制,实例化,并且放到ioc容器中(k-v beanName-bean) beanName默认是首字母小写
doInstance();

// 4.将IOC容器中的service对象设置给controller层定义的field上
doIoc();

//第4大组件 .初始化HandlerMapping(将url和method对应上)
initHandlerMappings();
}

@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
{
this.doPost(req, resp);
}

@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException
{
try
{
//处理请求
doDispatch(req, resp);
}
catch (Exception e)
{
resp.getWriter().write("500!! Server Exception");
}

}

private void doInstance()
{
if (classNames.isEmpty())
{
return;
}
for (String className : classNames)
{
try
{
//把类搞出来,反射来实例化(只有加@Controller需要实例化)
Class<?> clazz = Class.forName(className);
if (clazz.isAnnotationPresent(Controller.class))
{
Controller controller = clazz.getAnnotation(Controller.class);
if (controller.value().equals(""))
{
beans.put(toLowerFirstWord(clazz.getSimpleName()), clazz.newInstance());
}
else
{
beans.put(controller.value(), clazz.newInstance());
}
}
else if (clazz.isAnnotationPresent(Service.class))
{
Service service = clazz.getAnnotation(Service.class);
if (service.value().equals(""))
{
beans.put(toLowerFirstWord(clazz.getSimpleName()), clazz.newInstance());
}
else
{
beans.put(service.value(), clazz.newInstance());
}
}
else
{
continue;
}

}
catch (Exception e)
{
e.printStackTrace();
continue;
}
}
}

/**
* 依赖注入
*/
private void doIoc()
{
if (beans.isEmpty())
{
System.out.println("beans is null");
return;
}
for (Map.Entry<String, Object> map : beans.entrySet())
{
//IOC实例 , key是描述,也有可能是类名
Object instance = map.getValue();

//获取类
Class<?> clazz = instance.getClass();

if (clazz.isAnnotationPresent(Controller.class))
{
//获取成员变量
Field[] fields = clazz.getDeclaredFields();
for (Field field : fields)
{
if (field.isAnnotationPresent(Autowired.class))
{
Autowired requestParam = field.getAnnotation(Autowired.class);
String value = requestParam.value();
//未设置值则用类名
if ("".equals(value))
{
value = field.getName();
}
// 由于此类成员变量设置为private，需要强行设置
field.setAccessible(true);
try
{
field.set(instance, beans.get(value));
}
catch (IllegalAccessException e)
{
e.printStackTrace();
}
}
}
}
}
}

private void doScanner(String packageName)
{

//把所有的.替换成/
URL url = this.getClass().getClassLoader().getResource("/" + packageName.replaceAll("\\.", "/"));
File dir = new File(url.getFile());
for (File file : dir.listFiles())
{
if (file.isDirectory())
{
//递归读取包
doScanner(packageName + "." + file.getName());
}
else
{
String className = packageName + "." + file.getName().replace(".class", "");
classNames.add(className);
}
}

}

/**
* 处理请求
*
* @param req
* @param resp
* @throws Exception
*/
private void doDispatch(HttpServletRequest req, HttpServletResponse resp) throws Exception
{

if (handlerMapping.isEmpty())
{
return;
}

String url = req.getRequestURI();
String contextPath = req.getContextPath();

url = url.replace(contextPath, "").replaceAll("/+", "/");

System.out.println(" request url : " + url);
if (!this.handlerMapping.containsKey(url))
{
resp.getWriter().write("404 NOT FOUND!");
return;
}

Method method = this.handlerMapping.get(url);

//获取方法的参数列表
Class<?>[] parameterTypes = method.getParameterTypes();

//获取请求的参数
Map<String, String[]> parameterMap = req.getParameterMap();

//保存参数值
Object[] paramValues = new Object[parameterTypes.length];

//获取方法的参数注解
Annotation[][] parameterAnnotations = method.getParameterAnnotations();

//方法的参数列表
// 这是其实已经是适配器 HandlerAdapter , 这块比较复杂 , 这里简写了
for (int i = 0; i < parameterTypes.length; i++)
{
//根据参数名称，做某些处理
String requestParam = parameterTypes[i].getSimpleName();

//获取注解
Annotation[] annotations = parameterAnnotations[i];

if (requestParam.equals("HttpServletRequest"))
{
//参数类型已明确，这边强转类型
paramValues[i] = req;
continue;
}
if (requestParam.equals("HttpServletResponse"))
{
paramValues[i] = resp;
continue;
}
//请求的参数遍历
for (Entry<String, String[]> param : parameterMap.entrySet())
{
//方法参数列表
for (Annotation annotation : annotations)
{
//参数名
String paramName = "";
if (annotation instanceof RequestParam)
{
RequestParam p = (RequestParam)annotation;
paramName = p.value();

}
if("".equals(paramName)){
//如果没有匹配上注解, 则根据名字来 , 这里太复杂了涉及到JAVA字节码了. 不写了
}

//如果匹配上了
if (paramName.equals(param.getKey()))
{
if (requestParam.equals("String"))
{
//给定值
String value =
Arrays.toString(param.getValue()).replaceAll("\\[|\\]", "").replaceAll(",\\s", ",");
paramValues[i] = value;
}
}
}
}

}
//利用反射机制来调用
try
{
method.invoke(this.controllerMap.get(url), paramValues);//第一个参数是method所对应的实例 在ioc容器中
}
catch (Exception e)
{
e.printStackTrace();
}

}

/**
* 扫描所有的requestMapping注解,获取到请求路径
*/
private void initHandlerMappings()
{

if (beans.isEmpty())
{
return;
}
try
{
for (Entry<String, Object> entry : beans.entrySet())
{
Class<? extends Object> clazz = entry.getValue().getClass();
if (!clazz.isAnnotationPresent(Controller.class))
{
continue;
}
//获取控制器类名
String controllerName = toLowerFirstWord(clazz.getSimpleName());
if(!clazz.getAnnotation(Controller.class).value().equals("")){
controllerName = clazz.getAnnotation(Controller.class).value();
}

//拼url时,是controller头的url拼上方法上的url
String baseUrl = "";
if (clazz.isAnnotationPresent(RequestMapping.class))
{
RequestMapping annotation = clazz.getAnnotation(RequestMapping.class);
baseUrl = annotation.value();
}
Method[] methods = clazz.getMethods();
for (Method method : methods)
{
if (!method.isAnnotationPresent(RequestMapping.class))
{
continue;
}
RequestMapping annotation = method.getAnnotation(RequestMapping.class);
String url = annotation.value();

url = (baseUrl + "/" + url).replaceAll("/+", "/");
handlerMapping.put(url, method);
//这里必须从beans 里面拿 因为里面已经有service实例
controllerMap.put(url, beans.get(controllerName));
System.out.println(url + "," + method);
}

}

}
catch (Exception e)
{
e.printStackTrace();
}

}

/**
* 初始化配置文件
*
* @param contextConfigLocation
*/
private void initLoadConfig(String contextConfigLocation)
{
//把web.xml中的contextConfigLocation对应value值的文件加载到流里面
InputStream resourceAsStream = this.getClass().getClassLoader().getResourceAsStream(contextConfigLocation);
try
{
//用Properties文件加载文件里的内容
properties.load(resourceAsStream);
}
catch (IOException e)
{
e.printStackTrace();
}
finally
{
if (null != resourceAsStream)
{
try
{
resourceAsStream.close();
}
catch (IOException e)
{
e.printStackTrace();
}
}
}
}

/**
* 把字符串的首字母小写
*
* @param name
* @return
*/
private String toLowerFirstWord(String name)
{
return name.substring(0, 1).toLowerCase() + name.substring(1);
}
}
```

controller

```java
@RequestMapping("/hello")
@Controller
public class HelloController
{
@Autowired
HelloService helloService;

@RequestMapping("/index")
public String index(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
@RequestParam("test") String str) throws IOException
{
httpServletResponse.getWriter().write("str : " + str + ", service : " + helloService.hello());
return str;
}
}
```

service

```java
@Service
public class HelloService
{
public String hello(){
return "hello service";
}
}
```

整体架构如下

![](http://avalon-zheng.xin/static/static-front/upload/361ed0b2-57a9-478e-a476-5b79feab2319 "")

输入?test=1

![](http://avalon-zheng.xin/static/static-front/upload/65f15ce8-cb0b-4a56-a6b6-4b70522c878d "")

输入?str=1

![](http://avalon-zheng.xin/static/static-front/upload/f5c28f12-a35b-49b4-8dd9-cb3b04a90762 "")

可以看到.没有匹配上, 到这里,手写简单的springmvc已经完成了

## springmvc和struts2区别

Struts2的核心控制器是过滤器（filter），springmvc的核心控制器（Servlet）

Struts2的Action是多例的，springmvc的Handler（处理器）是单例的

Struts2用Action的属性接收客户端数据（必须多例才线程安全），springmvc使用方法的形参接收客户端数据（线程安全的）

Struts2是针对Action类型进行mapping（和url关联）

Springmvc是针对Handler中的处理请求的方法进行mappring（和url关联），

Springmvc在请求处理的控制上更精确，（粒度更小）
