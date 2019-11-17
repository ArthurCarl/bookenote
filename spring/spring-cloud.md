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

## Spring Cloud Config

### 服务端
Config资源:
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

Spring Cloud Config 配置信息来源:

```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
```

### 客户端使用
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>{spring-boot-docs-version}</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>

<dependencyManagement>
<dependencies>
 <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-dependencies</artifactId>
   <version>{spring-cloud-version}</version>
   <type>pom</type>
   <scope>import</scope>
 </dependency>
</dependencies>
</dependencyManagement>

<dependencies>
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-test</artifactId>
 <scope>test</scope>
</dependency>
</dependencies>

<build>
<plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
</plugins>
</build>

```

在 `bootstrap.properties` 或者 `application.properties` 中改变配置地址:  
`spring.cloud.config.uri: http://myconfigserver.com`

## Spring Cloud Config Server
`@EnableConfigServer`

`application.properties` 改变配置:  
```
server.port: 8888
spring.cloud.config.server.git.uri: file://${user.home}/config-repo #配置文件仓库
```

### Environment Repository
`EnvironmentRepository` 的三个参数:
- `{application}` 对应于`spring.application.name`
- `{profile}` 对应于`spring.profiles.active` 逗号分隔
- `{label}` 对应服务端`versioned`

`Label` - `foo/bar`使用 `foo(_)bar`在URL中。

#### SSL
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://example.com/my/repo
          skipSslValidation: true # 校验SSL
```

#### HTTP连接超时设置
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://example.com/my/repo
          timeout: 4
```

#### 模式匹配和多仓库
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            development:
              pattern:
                - '*/development'
                - '*/staging'
              uri: https://github.com/development/config-repo
            staging:
              pattern:
                - '*/qa'
                - '*/production'
              uri: https://github.com/staging/config-repo
              cloneOnStart: true
```

#### Authentication
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          username: trolley
          password: strongpassword
```
##### GIT SSH
```
spring:
  cloud:
    config:
      server:
        git:
          uri: git@gitserver.com:team/repo1.git
          ignoreLocalSshSettings: true
          hostKey: someHostKey
          hostKeyAlgorithm: ssh-rsa
          privateKey: |
                       -----BEGIN RSA PRIVATE KEY-----
                       MIIEpgIBAAKCAQEAx4UbaDzY5xjW6hc9jwN0mX33XpTDVW9WqHp5AKaRbtAC3DqX
                       IXFMPgw3K45jxRb93f8tv9vL3rD9CUG1Gv4FM+o7ds7FRES5RTjv2RT/JVNJCoqF
                       ol8+ngLqRZCyBtQN7zYByWMRirPGoDUqdPYrj2yq+ObBBNhg5N+hOwKjjpzdj2Ud
                       1l7R+wxIqmJo1IYyy16xS8WsjyQuyC0lL456qkd5BDZ0Ag8j2X9H9D5220Ln7s9i
                       oezTipXipS7p7Jekf3Ywx6abJwOmB0rX79dV4qiNcGgzATnG1PkXxqt76VhcGa0W
                       DDVHEEYGbSQ6hIGSh0I7BQun0aLRZojfE3gqHQIDAQABAoIBAQCZmGrk8BK6tXCd
                       fY6yTiKxFzwb38IQP0ojIUWNrq0+9Xt+NsypviLHkXfXXCKKU4zUHeIGVRq5MN9b
                       BO56/RrcQHHOoJdUWuOV2qMqJvPUtC0CpGkD+valhfD75MxoXU7s3FK7yjxy3rsG
                       EmfA6tHV8/4a5umo5TqSd2YTm5B19AhRqiuUVI1wTB41DjULUGiMYrnYrhzQlVvj
                       5MjnKTlYu3V8PoYDfv1GmxPPh6vlpafXEeEYN8VB97e5x3DGHjZ5UrurAmTLTdO8
                       +AahyoKsIY612TkkQthJlt7FJAwnCGMgY6podzzvzICLFmmTXYiZ/28I4BX/mOSe
                       pZVnfRixAoGBAO6Uiwt40/PKs53mCEWngslSCsh9oGAaLTf/XdvMns5VmuyyAyKG
                       ti8Ol5wqBMi4GIUzjbgUvSUt+IowIrG3f5tN85wpjQ1UGVcpTnl5Qo9xaS1PFScQ
                       xrtWZ9eNj2TsIAMp/svJsyGG3OibxfnuAIpSXNQiJPwRlW3irzpGgVx/AoGBANYW
                       dnhshUcEHMJi3aXwR12OTDnaLoanVGLwLnkqLSYUZA7ZegpKq90UAuBdcEfgdpyi
                       PhKpeaeIiAaNnFo8m9aoTKr+7I6/uMTlwrVnfrsVTZv3orxjwQV20YIBCVRKD1uX
                       VhE0ozPZxwwKSPAFocpyWpGHGreGF1AIYBE9UBtjAoGBAI8bfPgJpyFyMiGBjO6z
                       FwlJc/xlFqDusrcHL7abW5qq0L4v3R+FrJw3ZYufzLTVcKfdj6GelwJJO+8wBm+R
                       gTKYJItEhT48duLIfTDyIpHGVm9+I1MGhh5zKuCqIhxIYr9jHloBB7kRm0rPvYY4
                       VAykcNgyDvtAVODP+4m6JvhjAoGBALbtTqErKN47V0+JJpapLnF0KxGrqeGIjIRV
                       cYA6V4WYGr7NeIfesecfOC356PyhgPfpcVyEztwlvwTKb3RzIT1TZN8fH4YBr6Ee
                       KTbTjefRFhVUjQqnucAvfGi29f+9oE3Ei9f7wA+H35ocF6JvTYUsHNMIO/3gZ38N
                       CPjyCMa9AoGBAMhsITNe3QcbsXAbdUR00dDsIFVROzyFJ2m40i4KCRM35bC/BIBs
                       q0TY3we+ERB40U8Z2BvU61QuwaunJ2+uGadHo58VSVdggqAo0BSkH58innKKt96J
                       69pcVH/4rmLbXdcmNYGm6iu+MlPQk4BUZknHSmVHIFdJ0EPupVaQ8RHT
                       -----END RSA PRIVATE KEY-----
```

#####  Force  Pull/Deleting untracked branches/refreshRate Git Repository
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://git/common/config-repo.git
          force-pull: true
          repos:
            team-a:
                pattern: team-a-*
                uri: http://git/team-a/config-repo.git
                force-pull: true
            team-b:
                pattern: team-b-*
                uri: http://git/team-b/config-repo.git
                force-pull: true
            team-c:
                pattern: team-c-*
                uri: http://git/team-a/config-repo.git
          deleteUntrackedBranches: true
          refreshRate:30 #30秒刷新一次
```

#### Accessing Backends Through a Proxy
```
spring:
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          proxy:
            https:
              host: my-proxy.host.io
              password: myproxypassword
              port: '3128'
              username: myproxyusername
              nonProxyHosts: example.com
```

#### JDBC Backends
`JdbcEnvironmentRepository`

#### 组合
```
spring:
  profiles:
    active: composite
  cloud:
    config:
      server:
        composite:
        -
          type: svn
          uri: file:///path/to/svn/repo
        -
          type: git
          uri: file:///path/to/rex/git/repo
        -
          type: git
          uri: file:///path/to/walter/git/repo
```

#### 属性值覆盖
```
spring:
  cloud:
    config:
      server:
        overrides:
          foo: bar
```

### Health Indicator
```
spring:
  cloud:
    config:
      server:
        health:
          repositories:
            myservice:
              label: mylabel
            myservice-dev:
              name: myservice
              profiles: development
```
### Security

### 秘钥管理
对称秘钥:
`encrypt.key` 或者 `ENCRYPT_KEY`


## 提供可替换格式
可以使用`.yaml` `.yml` `.properties` 文件，不如Json，Json有数组`yaml`的值被映射为一个Map没有顺序


## Push Notifications and Spring Cloud Bus
`RefreshRemoteApplicationEvent` 只有在 Spring Cloud Bus 在配置中心服务和客户端都激活了才可以。

## Spring Cloud Config Client
### 配置 Bootstrap
`bootstrap.yml` 中设置 `spring.cloud.config.uri`

### 客户端配置立即失败
`spring.cloud.config.fail-fast=true`

### 配置重连
1. `spring.cloud.config.fail-fast=true`
2. 添加依赖`spring-retry`和`spring-boot-starter-aop`
3. `spring.cloud.config.retry.*` 重连配置

### 远程的配置文件
服务端 `/{name}/{profile}/{label}` ，和客户端以下方式对应:  
- name = `${spring.application.name}`
- profile = `${spring.profiles.active}`(`Environment.getActiveProfiles()`)
- label = 'master'

以上都可以使用`spring.cloud.config.*`进行自定义

### 高可用
`spring.cloud.config.uri` 配置多个URL，高可用

服务端返回 `500` `401` 时，客户端不会更换请求URL，这2种响应都不是和高可用相关

### 超时时长
`spring.cloud.config.request-read-timeout` 默认秒

### 安全
```
spring:
  cloud:
    config:
     uri: https://myconfig.mycompany.com
     username: user
     password: secret
--------
spring:
  cloud:
    config:
     uri: https://user:secret@myconfig.mycompany.com     
```

#### Health Indicator
`health.config.enabled=false` 配置心跳关闭  

`health.config.time-to-live` 缓存时间(毫秒)

#### 自定义 ConfigServicePropertySourceLocator
```java
@Configuration
public class CustomConfigServiceBootstrapConfiguration {
    @Bean
    public ConfigServicePropertySourceLocator configServicePropertySourceLocator() {
        ConfigClientProperties clientProperties = configClientProperties();
       ConfigServicePropertySourceLocator configServicePropertySourceLocator =  new ConfigServicePropertySourceLocator(clientProperties);
        configServicePropertySourceLocator.setRestTemplate(customRestTemplate(clientProperties));
        return configServicePropertySourceLocator;
    }
}
```

`resources/META-INF/spring.factories` 文件添加：  
```
org.springframework.cloud.bootstrap.BootstrapConfiguration = com.my.config.client.CustomConfigServiceBootstrapConfiguration
```

##  Spring Cloud Netflix
- Service Discovery (Eureka)
- Circuit Breaker (Hystrix)
- Intelligent Routing (Zuul)
- Client Side Load Balancing (Ribbon)

## Service Discovery: Eureka Clients

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

客户端配置文件中添加配置:  
```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/ # 默认Euroka地址
```

### Eureka 服务认证
复杂认证方式:
1. `DiscoveryClientOptionalArgs` 的 `@Bean`
2. 注入`ClientFilter`

### Status Page and Health Indicator
```
eureka:
  instance:
    statusPageUrlPath: ${server.servletPath}/info
    healthCheckUrlPath: ${server.servletPath}/health
```

### Registering a Secure Application
HTTPS的支持,在`EurekaInstanceConfig`中增加 :  
```
eureka.instance.[nonSecurePortEnabled]=[false]
eureka.instance.[securePortEnabled]=[true]
```
### 健康检查
开启客户端健康检查：  
```
eureka:
  client:
    healthcheck:
      enabled: true
```
以上最好在`application.yml`中配置，在`bootstrap.yaml`会出现一些问题
### Eureka Metadata for Instances and Clients
1. hostname
2. IP地址
3. 端口
4. 状态页面(status page)
5. 健康检查

通过`eureka.instance.metadataMap`添加新的源信息

#### 改变Eureka 实例ID
application.yml
```
eureka:
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

### Using the EurekaClient-Use to discover service instance
原生EurekaClient:

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
    return instance.getHomePageUrl();
}
```
** 不要在`@PostConstruct`中使用 **

### Netflix EurekaClient的替代方案
1. Feign 和 Spring RestTemplate 通过Eureka服务的ID获取服务
2. `DiscoveryClient`
```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri();
    }
    return null;
}
```

### 服务注册慢
- 心跳检查
- RenewInterval

### Zone
同Zone的客户端消费同Zone服务

1. Eureka服务的部署
2. `metadataMap`配置服务

```
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```


## Service Discovery: Eureka Server
groupId:`org.springframework.cloud`  
artifactId:`spring-cloud-starter-netflix-eureka-server`


### High Availability, Zones and Regions
发送心跳保持注册状态更新，客户端有注册中心的缓存-不必每次请求都访问注册；

每个Eureka服务器都是一个Eureka客户端，需要注册，不然会有大量日志


### 单节点
关闭客户端注册行为
```
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 同伴感知
Eureka 互相注册服务
```
---
spring:
 profiles: peer1
eureka:
 instance:
   hostname: peer1
 client:
   serviceUrl:
     defaultZone: http://peer2/eureka/

---
spring:
 profiles: peer2
eureka:
 instance:
   hostname: peer2
 client:
   serviceUrl:
     defaultZone: http://peer1/eureka/
```

##  Circuit Breaker: Hystrix Clients

![Hystrix Stream:Sample Apps](pic/Hystrix.png)

1. 特定服务调用超过`circuitBreaker.requestVolumeThreshold`(默认:20)
2. 故障百分比大于`circuitBreaker.errorThresholdPercentage`（默认值：> 50％）

满足以上则断路器起作用，调用终止；开放人员可以提供断路的回调

![Hystrix fallback prevents cascading failures](pic/HystrixFallback.png)

断路器阻止级联失败且给高负荷、失败服务时间恢复；而回调也可以是Hystrix的另一种对调用、静态数据、空值敏感的保护，回调可以是链式的，这样第一个回调可以调用其他的服务-回到静态数据。

### How to Include Hystrix
groupID:`org.springframework.cloud`  
artifactID:`spring-cloud-starter-netflix-hystrix`

```java
@SpringBootApplication
@EnableCircuitBreaker
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}

@Component
public class StoreIntegration {

    @HystrixCommand(fallbackMethod = "defaultStores")
    public Object getStores(Map<String, Object> parameters) {
        //do stuff that might fail
    }

    public Object defaultStores(Map<String, Object> parameters) {
        return /* something useful */;
    }
}
```

### 广播安全上下文/使用SpringScopes
`@HystrixCommand`中使用ThreadLocal上下文，按照如下处理

```java
@HystrixCommand(fallbackMethod = "stubMyService",
    commandProperties = {
      @HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
    }
)
...
```

### Health Indicator
`/health`
```
{
    "hystrix": {
        "openCircuitBreakers": [
            "StoreIntegration::getStoresByLocationLink"
        ],
        "status": "CIRCUIT_OPEN"
    },
    "status": "UP"
}
```

### Hystrix Metrics Stream
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```


## Hystrix Timeouts And Ribbon Clients

覆盖Ribbon的超时时间

### Hystrix Dashboard
groupID:`org.springframework.cloud`  
artifactID:`spring-cloud-starter-netflix-hystrix-dashboard`

`@EnableHystrixDashboard` 访问 `/hystrix` 和 `/hystrix.stream`

### Turbine
Turbine 将相关的 `/hystrix.stream` 聚集到 `/turbine.stream`

`@EnableTurbine` 需要加在 `main` 上

```
eureka:
  instance:
    metadata-map:
      management.port: ${management.port:8081}
```

#### Clusters Endpoint
`turbine.endpoints.clusters.enabled=true` 访问 `/clusters`
```
[
  {
    "name": "RACES",
    "link": "http://localhost:8383/turbine.stream?cluster=RACES"
  },
  {
    "name": "WEB",
    "link": "http://localhost:8383/turbine.stream?cluster=WEB"
  }
]
```

### Turbine Stream
Hystrix 向Turbin推送metrics：
1. client添加依赖 `spring-cloud-netflix-hystrix-stream` 和 `spring-cloud-starter-stream-*`
2. Server端,添加 `@EnableTurbineStream` ,依赖 `spring-boot-starter-webflux`

------
