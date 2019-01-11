# Spring Cloud

特性:
1. 分布式/版本控制配置
2. 服务注册和发现
3. 路由
4. 服务对服务调用
5. 负载均衡
6. 断路器
7. 分布式消息

## 云原生应用
Spring Cloud Context为`ApplicationContext`的启动、加密、刷新scope，环境终端提供工具类和服务

Spring Cloud Commons 一个抽象和普通类，使用于Spring Cloud的实现。


### Spring Cloud Context : 应用程序上下问题服务
Bootstrap上下文和主应用程序上下问题共享 `Environment`;Bootstrap属性优先级别高，不能被本地配置覆盖。

### 应用上下文的层级
1. Bootstrap
2. applicationConfig:[classpath:Bootstrap.yml]

### 改变Bootstrap属性位置
`spring.cloud.bootstrap.name` `spring.cloud.bootstrap.localtion` 改变配置文件位置

### 覆盖远程配置值
`spring.Cloud.config.overrideNone=true` 用本地覆盖任何远程配置值

`spring.cloud.config.overrideSystemProperties=false` 只有系统变量、命令行参数值环境变量值可以覆盖远程配置

### 自定义引导配置
`/META-INF/Spring.factories` 文件中 `org.springframework.cloud.bootstrap.BootstrapConfiguration` key中添加自定义的值

Bootstrap过程在将初始化器注入`SpringApplication`后结束:Bootstrap上下文利用`spring.factories`中的类创建，然后所有的`ApplicationContextInitializer`被增加到主`SpringApplication`

### 自定义Bootstrap Property Sources
`PropertySourceLocator` 可以添加SpringCloudConfigServer 以外的配置


```java
@Configuration
public class CustomPropertySourceLocator implements PropertySourceLocator {

    @Override
    public PropertySource<?> locate(Environment environment) {
        return new MapPropertySource("customProperty",
                Collections.<String, Object>singletonMap("property.from.sample.custom.source", "worked as intended"));
    }

}
```

在`META-INF/spring.factories` 添加如下:
`org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator`

### Logging Configuration

`bootstrap.yam[properties]` 中的日志配置会应用于所有的事件


### Environment Changes

应用监听到 `EnvironmentChangeEvent` 事件，应用的动作:
1. `@ConfigurationProperties`的重新绑定
2. `logging.levele.*`的改变

### Refresh Scope
`@RefreshScope` 解决动态改变Bean状态值

`spring.cloud.refresh.extra-refreshable`

`RefreshScope` bean在上下文中刷新Bean

`/refresh` 路径

```
management:
  endpoints:
    web:
      exposure:
        include: refresh
```

### 加解密
`Environment` 有个预处理-解密本地属性值

### EndPoint
- `/actuator/env`
- `/actuator/refresh`
- `/actuator/restart`
- `/actuator/pause` 和 `/actuator/resume`


## Spring Cloud Commons: Common Abstractions
服务发现，负载平衡、断路器等公共抽象层

### `@EnableDiscoveryClient`
在 `META-INF/spring.factories` 中找`DiscroveryClient`接口实现类。

客户端在`META-INF`的`spring.factories`中的 `org.springframework.cloud.client.discovery.EnableDiscoveryClient` key增加实现的类名。

#### Health Indicator
- `spring.cloud.discovery.client.composite-indicator.enabled=false` - 禁用复合`HealthIndicator`
- `spring.cloud.discovery.client.health-indicator.enabled=false` - 禁用`DiscoveryClientHealthIndicator`
- `spring.cloud.discovery.client.health-indicator.include-description=false` -禁用`DiscoveryClientHealthIndicator`的字段描述

### 服务注册
`ServiceRegistry` 服务注册接口抽象


```java
@Configuration
@EnableDiscoveryClient(autoRegister=false)
public class MyConfiguration {
    private ServiceRegistry registry;

    public MyConfiguration(ServiceRegistry registry) {
        this.registry = registry;
    }

    // called through some external process, such as an event or a custom actuator endpoint
    public void register() {
        Registration registration = constructRegistration();
        this.registry.register(registration);
    }
}
```

每个 `ServiceRegistry` 实现都有自己的`Registry`实现
- `ZookeeperRegistration` 和 `ZookeeperServiceRegistry`
- `EurekaRegistration` 和 `EurekaServiceRegistry`
- `ConsulRegistration` 和 `ConsulServiceRegistry`

#### ServiceRegistry 自动注册
`ServiceRegistry` 实现会自动注册运行中的服务

`@EnableDiscoveryClient(autoRegister=false)` 关闭自动注册

`spring.cloud.service-registry.auto-registration.enabled=false`

自动注册的事件:
1. `InstancePreRegisteredEvent`注册前触发
2. `InstanceRegisteredEvent` 注册后触发

#### Service Registry 监控终端
`/service-registry` 监控终端

支持`POST`和`GET`请求

### RestTemplate 负载均衡客户端
```java
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    RestTemplate loadBalanced() {
        return new RestTemplate();
    }

    @Primary
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

public class MyClass {
    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    @LoadBalanced
    private RestTemplate loadBalanced;

    public String doOtherStuff() {
        return loadBalanced.getForObject("http://stores/stores", String.class);
    }

    public String doStuff() {
        return restTemplate.getForObject("http://example.com", String.class);
    }
}
```
### Spring WebFlux WebClient as a Load Balancer Client
在`spring-webflux`存在时 `LoadBalancerExchangeFilterFunction` 会自动配置
```java
public class MyClass {
    @Autowired
    private LoadBalancerExchangeFilterFunction lbFunction;

    public Mono<String> doOtherStuff() {
        return WebClient.builder().baseUrl("http://stores")
            .filter(lbFunction)
            .build()
            .get()
            .uri("/stores")
            .retrieve()
            .bodyToMono(String.class);
    }
}
```

### Ignore Network Interfaces
```yml
# application.yml
spring:
  cloud:
    inetutils:
      ignoredInterfaces:
        - docker0
        - veth.*
```
以上忽略`docker0` 和 `veth`开头的网关

```yml
# bootstrap.yml
spring:
  cloud:
    inetutils:
      preferredNetworks:
        - 192.168
        - 10.0
```
以上只允许以上开头的IP

```yml
# application.yml
spring:
  cloud:
    inetutils:
      useOnlySiteLocalInterfaces: true
```
以上只允许本地

###  HTTP Client Factories
支持`ApacheHttpClientFactory` 和 `OkHttpClientFactory`

HTTP 管理器:
- `ApacheHttpClientConnectionManagerFactory` - Apaache HTTP Client  
- `OkHttpClientConnectionPoolFactory` - Ok Http Client

禁止创建以上bean:

```
spring.cloud.httpclientfactories.apache.enabled=false
# or
# spring.cloud.httpclientfactories.ok.enabled=false
```

### 特性开启
`/features` 返回开启的特性

#### 特性类型
abstract 和 named 2种类型特性

abstract 指接口抽象类的具体实现类在上下文中的`Bean`

named 指没有特定的实现类比如断路器、API网关、SpringCloudBus；需要name和BeanType

#### Declaring features
```java
@Bean
public HasFeatures commonsFeatures() {
  return HasFeatures.abstractFeatures(DiscoveryClient.class, LoadBalancerClient.class);
}

@Bean
public HasFeatures consulFeatures() {
  return HasFeatures.namedFeatures(
    new NamedFeature("Spring Cloud Bus", ConsulBusAutoConfiguration.class),
    new NamedFeature("Circuit Breaker", HystrixCommandAspect.class));
}

@Bean
HasFeatures localFeatures() {
  return HasFeatures.builder()
      .abstractFeature(Foo.class)
      .namedFeature(new NamedFeature("Bar Feature", Bar.class))
      .abstractFeature(Baz.class)
      .build();
}
```
