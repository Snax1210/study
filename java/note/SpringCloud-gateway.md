# SpringCloud-gateway

## SpringCloud GateWay简介

SpringCloud GateWay 是由spring官方基于Spring5.0、Spring Boot2.x、Project Reactor等技术开发的网关，目的是代替原先版本中得我Spring Cloud NetFlix Zuul。

![](.SpringCloud-gateway_images/9a176854.png)

## Gateway 网关特性

- 统一入口所有请求通过网关路由到内部其他服务
- 断言（Predicates）和过滤器（filters）特定路由。断言是根据具体的请求的规则有route去处理；过滤器用来对请求做各种判断和修改。
- 可集成熔断器：Hystrix熔断机制。Hystrix 是 spring cloud gateway 中是以 filter 的形式使用的。
- 路由重写（rewrite）：路径重写自定义路由转发规则。
- 能够自由设置任何请求属性的路由
- SpringCloud DiscoveryClient原生支持

## GateWay 流程架构

![](.SpringCloud-gateway_images/e0adaba1.png)

## 静态路由

所谓静态路由，就是指API网关启动前，通过配置文件或者代码的方式，静态的配置好API之间的路有关系，此后不需要二次维护， 大多数的内部API网关适用于这种方式。

### 方法一 配置文件方式

本质上是修改application.yml文件，相关修改方法，官方已经有详尽的描述了，[官方文档](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)
。 举例说明：

```yaml
spring:
  cloud:
    gateway:
      routes:
        # 认证中心
        - id: diaoyucheng-auth
          uri: lb://diaoyucheng-auth
          predicates:
            - Path=/auth/**
          filters:
            # 验证码处理
            - CacheRequestFilter
            - ValidateCodeFilter
            - StripPrefix=1 
```

### 方法二 代码构建路由

```java

public class GenerateRoute {
    // static imports from GatewayFilters and RoutePredicates
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
        return builder.routes()
            .route(r -> r.host("**.abc.org")
                .and()
                .path("/image/png")
                .filters(f -> f.addResponseHeader("X-TestHeader", "foobar"))
                .uri("http://snax.org:80"))
            .route(r -> r.path("/image/webp")
                .filters(f -> f.addResponseHeader("X-AnotherHeader", "baz"))
                .uri("http://snax.org:80")
                .metadata("key", "value"))
            .route(r -> r.order(-1)
                .host("**.throttle.org")
                .and()
                .path("/get")
                .filters(f -> f.filter(throttle.apply(1, 1, 10, TimeUnit.SECONDS)))
                .uri("http://snax.org:80")
                .metadata("key", "value"))
            .build();

    }

}
```

通过代码即可完成路由的创建，会在Spring Cloud Gateway启动时自动加载到运行态，本质上配置文件和代码的方式， 仅仅是两种加载形态，底层没有太大区别。然后，静态路由往往不能满足我们的需求。

## 断言(Predicates)

### After

匹配在某时间之后的请求

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: after_route
          uri: https://example.org
          predicates:
            - After=2021-12-10T00:00:00.000
```

### Before

匹配在某时间之前的请求

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: after_route
          uri: https://example.org
          predicates:
            - Before=2021-12-10T00:00:00.000
```

### Between

匹配在两时间之间的请求

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: after_route
          uri: https://example.org
          predicates:
            - Between=2021-12-10T00:00:00.000,2021-12-10T00:00:00.000
```

| 关键字        | 说明               |
|------------|------------------|
| Cookie     | Cookie属性正则表达式匹配  |
| Query      | 参数匹配             |
| Method     | 方法匹配             |
| Host       | 主机名正则表达式匹配       |
| Weight     | 权重匹配             |
| Header     | Header属性和正则表达式匹配 |
| Path       | 请求路径匹配           |   
| RemoteAddr | IP地址匹配（支持子网掩码）   |  

[官方参考地址](https://cloud.spring.io/spring-cloud-gateway/reference/html/#_after_route_predicate_factory)

## 过滤器(Filter)

路由器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路径过滤器的范围限定为特定路径，Spring Cloud Gateway包含许多内置的GatewayFilter工厂。

### GlobalFilter 全局过滤器

```java
public class Filter {
    private static final Logger LOGGER = LoggerFactory.getLogger(Filter.class);
    @Bean
    @Order(-1)
    public GlobalFilter a() {
        return (exchange, chain) -> {
            LOGGER.info("first pre filter");
            return chain.filter(exchange).then(Mono.fromRunnable(() -> LOGGER.info("third post filter")));
        };
    }

    @Bean
    @Order(0)
    public GlobalFilter b() {
        return (exchange, chain) -> {
            LOGGER.info("second pre filter");
            return chain.filter(exchange).then(Mono.fromRunnable(() -> LOGGER.info("second post filter")));
        };
    }
}
```

### CORS 跨域处理

例子：对于所有GET请求的路径，将允许来自docs.spring.io的请求的CORS请求

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
              - GET
```

### Gateway API

| 请求地址                            | 请求类型            | 描述             |
|---------------------------------|-----------------|----------------|
| /actuator/gateway/refresh       | POST            | 支持通过接口动态调整网关策略 |
| /actuator/gateway/routes        | GET             | 刷新路由缓存         |
| /actuator/gateway/globalfilters | GET             | 查询路由           |
| /actuator/gateway/routefilters  | GET             | 查询全局过滤器        |
| /actuator/gateway/routes/{id}   | GET、POST、DELETE | 查询指定路由信息       |

`无论是哪一种，在启动网关后将无法修改路由配置，如有新服务要上线，则需要先把网关下线，修改yml配置后，再重启网关。`

## 动态路由

Gateway网关启动时，路由信息默认会加载到内存中，路由信息被封装到RouteDefinition对象中， 配置多个RouteDefinition组成GateWay的路由系统。

```java
package org.springframework.cloud.gateway.route;

@Validated
public class RouteDefinition {

    @NotEmpty
    private String id = UUID.randomUUID().toString();

    @NotEmpty
    @Valid
    private List<PredicateDefinition> predicates = new ArrayList<>();

    @Valid
    private List<FilterDefinition> filters = new ArrayList<>();

    @NotNull
    private URI uri;

    private int order = 0;

    public RouteDefinition() {
    }

    //.....此处省略......

}
```

RouteDefinition中的字段与上边代码配置方式比较对应

```java
package org.springframework.cloud.gateway.route;

import reactor.core.publisher.Flux;

public interface RouteDefinitionLocator {
	Flux<RouteDefinition> getRouteDefinitions();
}
```

```java
package org.springframework.cloud.gateway.route;

import reactor.core.publisher.Flux;

public interface RouteDefinitionLocator{
    Flux<RouteDefinition> getRouteDefinitions();
}
```

RouteDefinitionLocator是个接口，在org.springframework.cloud.gateway.route包下，如果想查看网关中所有的路由信息，可以调用此方法；
后面还有另外一种查看方式，是Spring Cloud Gateway 的 Endpoint端点提供的方法。

### Gateway Endpoint 端点

Endpoint端点有暴露路由信息、获取所有路由、刷新路由、查看单个路由、删除路由等方法，源码在 org.springframework.cloud.gateway.actuate.GatewayControllerEndpoint 中，
访问端点中的方法需要修改pom和配置文件：  
添加pom文件依赖；

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

yaml配置：

```yaml
# 暴露端点
management:
 endpoints:
   web:
    exposure:
    include: '*'
 endpoint:
    health:
    show-details: always
```

### 动态路由实现

#### 创建路由模型

过滤器模型

```java
public class GatewayFilterDefinition{
    //Filter Name
    private String name;
    //对应路由规则
    private Map<String,String> args = new LinkedHashMap();
    //省略get、set方法
}
```

路由断言模型

```java
public class GatewayPredicateDefinition {
    //断言对应的name
    private String name;
    //配置的断言规则
    private Map<String,String> args = new LinkedHashMap<>();
    
    //省略get、set方法
}
```

路由模型

```java
public class GatewayRouteDefinition{
    //路由的Id
    private String id;

    //路由断言集合配置
    private List<GatewayPredicateDefinition> predicates = new ArrayList<>();

    //路由过滤器集合配置
    private List<GatewayFilterDefinition> filters = new ArrayList<>();

    //路由规则转发的目标uri
    private String uri;

    //路由执行的顺序
    private int order = 0;

    //......此处省略Get、Set方法......
}
```

编写态路由实现类，实现ApplicationEventPublisherAware接口

```java
@Service
public class DynamicRouteServiceImpl implements ApplicationEventPublisherAware{

    @Autowired
    private RouteDefinitionWriter       routeDefinitionWriter;

    private ApplicationEventPublisher   publisher;

  @Override
  public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
    this.publisher = applicationEventPublisher;
}

/**
 * 增加路由
 * @param definition
 * @return
 */
  public String add(RouteDefinition definition) {
      routeDefinitionWriter.save(Mono.just(definition)).subscribe();
      this.publisher.publishEvent(new RefreshRoutesEvent(this));
      return "success";
  }

/**
 * 更新路由
 * @param definition
 * @return
 */
public String update(RouteDefinition definition) {
    try {
        delete(definition.getId());
    } catch (Exception e) {
        return "update fail,not find route  routeId: "+definition.getId();
    }
    try {
        routeDefinitionWriter.save(Mono.just(definition)).subscribe();
        this.publisher.publishEvent(new RefreshRoutesEvent(this));
        return "success";
    } catch (Exception e) {
        return "update route  fail";
    }
}

/**
 * 删除路由
 * @param id
 * @return
 */
  public Mono<ResponseEntity<Object>> delete(String id) {
    return this.routeDefinitionWriter.delete(Mono.just(id)).then(Mono.defer(() -> {
        return Mono.just(ResponseEntity.ok().build());
    })).onErrorResume((t) -> {
        return t instanceof NotFoundException;
    }, (t) -> {
        return Mono.just(ResponseEntity.notFound().build());
    });
  }
}
```

### 编写API接口，实现动态路由功能

```java
@RestController
@RequestMapping("/api/route")
public class RouteRestController {

   @Autowired
   private DynamicRouteServiceImpl dynamicRouteService;

/**
 * 增加路由
 * @param gatewayRouteDefinition 路由模型
 *  报文:{"filters":[{"name":"StripPrefix","args":{"_genkey_0":"2"}}],"id":"ijep-service-sys-hyk11","uri":"lb://ijep-service-sys","order":2,"predicates":[{"name":"Path","args":{"_genkey_0":"/api/hyk11/**"}}]}
 * @return
 */
@PostMapping("/add")
public RestResponse add(@RequestBody GatewayRouteDefinition gatewayRouteDefinition) {
    String flag = "fail";
    try
    {
        RouteDefinition definition = assembleRouteDefinition(gatewayRouteDefinition);
        flag = this.dynamicRouteService.add(definition);
    } catch (Exception e) {
        e.printStackTrace();
    }

    return RestResponseBuilder.restResponse().data(flag).build();
}


/**
 * 更新路由
 * @param gatewayRouteDefinition 路由模型
 *  报文:{"filters":[{"name":"StripPrefix","args":{"_genkey_0":"2"}}],"id":"ijep-service-sys-hyk11","uri":"lb://ijep-service-sys","order":2,"predicates":[{"name":"Path","args":{"_genkey_0":"/api/hyk11/**"}}]}
 * @return
 */
@PostMapping("/update")
public RestResponse update(@RequestBody GatewayRouteDefinition gatewayRouteDefinition) {

    RouteDefinition definition = assembleRouteDefinition(gatewayRouteDefinition);
    String flag = this.dynamicRouteService.update(definition);

    return RestResponseBuilder.restResponse().data(flag).build();
}

/**
 * 删除路由
 * @param id 路由Id
 * @return
 */
@DeleteMapping("/routes/{id}")
public RestResponse delete(@PathVariable String id) {

    Mono<ResponseEntity<Object>> responseEntityMono = this.dynamicRouteService.delete(id);
    return RestResponseBuilder.restResponse().data(responseEntityMono).build();
}


/**
 * 把传递进来的参数转换成路由对象
 * @param gatewayRouteDefinition 路由模型
 * @return
 */
private RouteDefinition assembleRouteDefinition(GatewayRouteDefinition gatewayRouteDefinition) {
    RouteDefinition definition = new RouteDefinition();
    definition.setId(gatewayRouteDefinition.getId());
    definition.setOrder(gatewayRouteDefinition.getOrder());

    //设置断言
    List<PredicateDefinition> pdList = new ArrayList<>();
    List<GatewayPredicateDefinition> gatewayPredicateDefinitionList = gatewayRouteDefinition.getPredicates();
    for (GatewayPredicateDefinition gpDefinition: gatewayPredicateDefinitionList) {
        PredicateDefinition predicate = new PredicateDefinition();
        predicate.setArgs(gpDefinition.getArgs());
        predicate.setName(gpDefinition.getName());
        pdList.add(predicate);
    }

    definition.setPredicates(pdList);

    //设置过滤器
    List<FilterDefinition> filters = new ArrayList();
    List<GatewayFilterDefinition> gatewayFilters = gatewayRouteDefinition.getFilters();
    for(GatewayFilterDefinition filterDefinition : gatewayFilters){
        FilterDefinition filter = new FilterDefinition();
        filter.setName(filterDefinition.getName());
        filter.setArgs(filterDefinition.getArgs());
        filters.add(filter);
    }
    definition.setFilters(filters);

    URI uri = null;
    if(gatewayRouteDefinition.getUri().startsWith("http")){
        uri = UriComponentsBuilder.fromHttpUrl(gatewayRouteDefinition.getUri()).build().toUri();
    }else{
        // uri为 lb://consumer-service 时使用下面的方法
        uri = URI.create(gatewayRouteDefinition.getUri());
    }
    definition.setUri(uri);
    return definition;
}
}
```