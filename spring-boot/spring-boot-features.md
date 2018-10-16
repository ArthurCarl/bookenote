# spring-boot-features
## SpringApplication

### `FailureAnalyzer` - 启动失败分析;也可以在 `debug` 属性，`DEBUG` log  
```
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### 自定义Banner
1. 在类路径下添加`banner.txt` 文件
2. 设置 `spring.banner.location` 设置banner文件路劲

文件的形式可以是: `banner.gif` `banner.jpg` `banner.png`

Banner 变量
- `${application.version}`
- `${application.formatted-version}`
- `${spring-boot.version}`
- `${spring-boot.formatted-version}`
- `${Ansi.NAME}` 
- `${application.title}`

`spring.main.banner-mode` banner的模式:`System.out` `log` `off`

`SpringBootBanner` 类就是`Banner` 接口的实现；

### 自定义 SpringApplication
`SpringApplication` 提供很多API供自定义，如：  
```java
	SpringApplication app = new SpringApplication(MySpringConfiguration.class);
	app.setBannerMode(Banner.Mode.OFF);
	app.run(args);
```

### Fluent Builder API
`SpringApplicationBuilder` 可以构建有上下级关系或者使用流式的API，如:
```java
new SpringApplicationBuilder()
		.sources(Parent.class)
		.child(Application.class)
		.bannerMode(Banner.Mode.OFF)
		.run(args);
```
### 应用事件和监听器
`ContextRefreshedEvent`-`SpringApplication` 发送的额外的应用事件。
有些事件实在`ApplicationContext` 创建前，就被自动被触发了；因此这些事件不可以注册为`@Bean`,可以使用`SpringApplication.addListeners(…)` 或者 `SpringApplicationBuilder.listeners(…)`;

想不论应用的创建方式，事件监听器总是自动注册；可以在`META-INF/spring.factories` 文件添加:  
```
org.springframework.context.ApplicationListener=com.example.project.MyListener
```


1. `ApplicationStartingEvent` 运行的开始，但是在其他进程前，除开监听器的注册和实例化
2. `ApplicationEnvironmentPreparedEvent` - `Environment` 上线使用的被发现，在上下文创建前；
3. `ApplicationPreparedEvent` 在刷新开始之前，bean 的定义加载之后；
4. `ApplicationStartedEvent` 在上下文刷新后，应用和命令行运行器被调用前；
5. `ApplicationReadyEvent` 应用和命令行运行器被调用后，这表明应用已经可以响应请求
6. `ApplicationFailedEvent` 启动时发生异常造成启动失败

### Web Environment
1. `AnnotationConfigServletWebServerApplicationContext` SpringMVC
2. `AnnotationConfigReactiveWebServerApplicationContext` Spring
3. `AnnotationConfigApplicationContext` Otherwise

### Accessing Application Arguments
`org.springframework.boot.ApplicationArguments`

### Using the ApplicationRunner or CommandLineRunner
`ApplicationRunner` 和 `CommandLineRunner` 接口方法在 `SpringApplication.run(…)` 启动后完成前；

在应用中需要使用多个这样的bean时，则需要`Order`

### Application Exit
每个`SpringApplication` 都在 JVM 上注册一个`shutdown hook`,确保`ApplicationContext` 能优雅推出;其他的Spring生命周期中的回调函数；

```java
@SpringBootApplication
public class ExitCodeApplication {

	@Bean
	public ExitCodeGenerator exitCodeGenerator() {
		return () -> 42;
	}

	public static void main(String[] args) {
		System.exit(SpringApplication
				.exit(SpringApplication.run(ExitCodeApplication.class, args)));
	}

}
```

### Admin Features
`spring.application.admin.enabled` 属性启用Admin特性；这个将`SpringApplicationAdminMXBean` 暴露在 `MBeanServer`上

`local.server.port` 服务端口

## Externalized Configuration
SpringBoot支持 `properties` `YAML` `environment variable` 和命令行参数 配置方式；

代码注入`properties`文件属性方式:
1. `@Value`
2. `Environment`的抽象
3. `@ConfigurationProperties` 绑定

`PropertySource` 顺序，确保文件的加载顺序

#### `SPRING_APPLICATION_JSON` 
1. 环境变量:`$ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar`
2. java命令行参数:`spring.application.json` `$ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar`
3. 命令行擦拭:`$ java -jar myapp.jar --spring.application.json='{"name":"test"}'`
4. JNDI:`java:comp/env/spring.application.json`

### Configuring Random Values -随机数
`RandomValuePropertySource` 生产随机数、UUID、String 的核心类

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

### Accessing Command Line Properties
`SpringApplication` 的命令行参数会被转换为 `property` ,然后添加到Spring的`Environment`中，优先级最高

`SpringApplication.setAddCommandLineProperties(false)` 不将命令行参数添加到Springd的`Environment`

### Application Property Files -应用properties文件
`SpringApplication` 加载`application.properties` 的顺序:
1. 当前目录下的`/config`文件 ，命令行所在的目录位置
2. 当前目录的文件
3. 类路径下的`/config` 下的文件
4. 类路径的更目录

`spring.config.name` 环境变量可以改变 `application.properties` 的名称  
`$ java -jar myproject.jar --spring.config.name=myproject`

`spring.config.location` 制定文件的位置  
`$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties`

`spring.config.name` 和 `spring.config.location` 必须作为环境变量

**`spring.config.location` 按照书写顺序的反序方式加载文件**

#### Profile-specific Propertie
`spring.config.location` 配置在使用 `profile` 时使用文件夹

#### Loading YAML
`YamlPropertiesFactoryBean` 加载YAML文件为`Properties`，`YamlMapFactoryBean` 加载YAML文件为`Map`

```yaml
my:
servers:
	- dev.example.com
	- another.example.com
```
将以上属性绑定到Spring环境中;
```java
@ConfigurationProperties(prefix="my")
public class Config {

	private List<String> servers = new ArrayList<String>();

	public List<String> getServers() {
		return this.servers;
	}
}
```
####  Exposing YAML as Properties in the Spring Environment
`YamlPropertySourceLoader` 将YAML文件在Spring的环境暴露为`PropertySource`

#### Multi-profile YAML Documents
```yaml
server: #默认没有配置时
	address: 192.168.1.100
---
spring:
	profiles: development
server:
	address: 127.0.0.1
---
spring:
	profiles: production
server:
	address: 192.168.1.120
```

#### YAML Shortcomings
`@PropertySource` 支持YAML 文件

### Type-safe Configuration Properties -properties文件属性值得校验
```java
@ConfigurationProperties("acme") // 将文件中`acme.enabled` 映射到，此对象的属性`enabled`
public class AcmeProperties {

	private boolean enabled;

	private InetAddress remoteAddress;

	private final Security security = new Security();

	public boolean isEnabled() { ... }

	public void setEnabled(boolean enabled) { ... }

	public InetAddress getRemoteAddress() { ... }

	public void setRemoteAddress(InetAddress remoteAddress) { ... }

	public Security getSecurity() { ... }

	public static class Security {

		private String username;

		private String password;

		private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

		public String getUsername() { ... }

		public void setUsername(String username) { ... }

		public String getPassword() { ... }

		public void setPassword(String password) { ... }

		public List<String> getRoles() { ... }

		public void setRoles(List<String> roles) { ... }

	}
}


@Configuration
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}

```

#### Third-party Configuration
`@ConfigurationProperties` 也可以放在 `@Bean` 的方法上

```java
@ConfigurationProperties(prefix = "another")
@Bean
public AnotherComponent anotherComponent() {
}
```

#### Relaxed Binding -宽松的绑定
```java
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {

	private String firstName;

	public String getFirstName() {
		return this.firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

}
```

可以使用以下的属性值:
- `acme.my-project.person.first-name`
- `acme.myProject.person.firstName`
- `acme.my_project.person.first_name`
- `ACME_MYPROJECT_PERSON_FIRSTNAME`


`Map` 属性值映射使用`[]` 
```yaml
acme:
  map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3
```
`Map` `/key1` `/key2` `key3`

####  Merging Complex Types 合并复杂的类型
```java
@ConfigurationProperties("acme")
public class AcmeProperties {

	private final List<MyPojo> list = new ArrayList<>();

	public List<MyPojo> getList() {
		return this.list;
	}

}
```

```yaml
acme:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
############分割线#############
acme:
  list:
    - name: my name
      description: my description
    - name: another name
      description: another description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

	private final Map<String, MyPojo> map = new HashMap<>();

	public Map<String, MyPojo> getMap() {
		return this.map;
	}

}
```

```yaml
acme:
  map:
    key1:
      name: my name 1
      description: my description 1
---
spring:
  profiles: dev
acme:
  map:
    key1:
      name: dev name 1
    key2:
      name: dev name 2
      description: dev description 2
```

#### Properties Conversion














--























