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
当外部属性值绑定到`@ConfigurationProperties` 的 beans 时，Spring Boot 会将值进行正确的转换；

自定属性值类型转换的方式:
1. 提供name 为 `conversionService` 的 `ConversionService` 类型 bean
2. 通过 `CustomEditorConfigurer` 自定义 property edtiors
3. 自定义 `Converters`(在类上用`@ConfigurationPropertiesBinding`)

##### Converting durations
暴露了 `java.time.Duration` 属性时，以下格式的程序属性值是允许的:
1. `long` ,除开`@DurationUint`指定时间单位，默认为毫秒(milliseconds)
2. ISO-8601标准的`java.time.Duration`
3. 当值和时间单位组合在一起时更加易读

支持的时间单位:
- `ns`
- `us`
- `ms`
- `s`
- `m`
- `h`
- `d`

##### Converting Data Sizes
暴露 `DataSize` 时，可以使用以下单位:
- `long` 默认单位:byte ，可以用 `@DataSize`指定
- 当值和时间单位组合在一起时更加易读

支持的数据大小类型的单位:
- `B`
- `KB`
- `MB`
- `GB`
- `TB`

#### `@ConfigurationProperties` Validation
当`@Validated` 和 `@ConfigurationProperties` 一起注解一个类时，SpringBoot会去对类中的属性值进行校验

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

	@NotNull
	private InetAddress remoteAddress;

	@Valid
	private final Security security = new Security();

	public static class Security {

		@NotEmpty
		public String username;

		// ... getters and setters

	}
	// ... getters and setters

}
```

创建自定义的 `Validator`,调用`configurationPropertiesValidator` 创建一个bean definition,`@Bean`方法需要声明为`static` ;此校验器在程序声明周期开始阶段创建，将方法声明为静态的则会使校验器在 `@Configuration`类实例化前被创建。

```java
@SpringBootApplication
public class SamplePropertyValidationApplication implements CommandLineRunner {

	private final SampleProperties properties;

	public SamplePropertyValidationApplication(SampleProperties properties) {
		this.properties = properties;
	}

	//自定义的校验器
	@Bean
	public static Validator configurationPropertiesValidator() {
		return new SamplePropertiesValidator();
	}

	@Override
	public void run(String... args) {
		System.out.println("=========================================");
		System.out.println("Sample host: " + this.properties.getHost());
		System.out.println("Sample port: " + this.properties.getPort());
		System.out.println("=========================================");
	}

	public static void main(String[] args) {
		new SpringApplicationBuilder(SamplePropertyValidationApplication.class).run(args);
	}

}
```

#### `@ConfigurationProperties` vs `@Value`

Feature | `@ConfigurationProperties` | `@Value`
------|------|------
Relaxed binding | Yes | No
Meta-data support | Yes  | NO
`SpEL` evaluation | Yes  | NO


### Profiles
```java
@Configuration
@Profile("production")
public class ProductionConfiguration {

	// ...

}
```

用`spring.profiles.active` 指定激活哪个环境:
1. `application.properties` 文件的 `spring.profiles.active=dev,hsqldb`
2. `--spring.profiles.active=dev,hsqldb` 命令行

#### Adding Active Profiles
`spring.profiles.include` 会将增加激活的环境(多个配置)

```properties
---
my.property:fromyamlfile
---
spring.profiles:production
spring.profiles.include:
	- proddb
	- prodmq
```

#### Programmatically Setting Profiles
调用 `SpringApplication.setAdditionalProfiles(…​)`

#### Profile-specific Configuration Files
- `application.properties` 文件
- `application.yml`
- `@ConfigurationProperties` 引用的文件

### Logging
Spring Boot 内部使用 Commons Logging,但是底层Log实现是对外开放的。

#### Log Format
```
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```
输出内容如下:
- 日期和时间:精确到毫秒，便排序
- 日志级别:`ERROR` `WARN` `INFO` `DEBUG` `TRACE`
- 进程ID
- `---` 分离真正日志信息的开始
- 线程名称:`[]`包括
- 日志名称
- 日志信息

**注意:Logbak 没有 `FATAL`级别，会映射到`ERROR`**


#### Console Output
```
$ java -jar myapp.jar --debug # 命令行设置日志级别
```

##### Color-coded Output
终端支持ANSI，彩色日志输出可以增加日志的可读性。`spring.output.ansi.enabled`设定支持的值而覆盖自动检测的值。

色彩码可以使用 `%clr` 转换词

Level | Color
------|------
`FATAL` | Red
`ERROR` | Red
`WARN` | Yellow
`INFO` | Green
`DEBUG` | Green
`TRACE` | Greee

`%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}` 自定义色彩

支持的格式和色彩
- `blue`
- `cyan`
- `faint`
- `green`
- `magenta`
- `red`
- `yellow`

#### File Output
需要将日志输出到日志文件则需要设置:`logging.file` `logging.path` 属性值

`logging.file` | `logging.path` | Example | Description
------|------|------|------
(none) | (none) | | 只有终端日志
Specific file | (none) | `my.log` | 日志输出到指定文件，路径可以相对或绝对
(none) | Specific directory | `/var/log` | 日志输出到指定目录的`spring.log`文件中


`logging.file.max-size` 每个文件的最大值

`logging.file.max-history` 每个文件保存的时间

日志系统比应用生命周期前实例化，日志的属性值不能使用`property` 文件加载

#### Log Levels
```properties
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

#### Log Groups

```properties
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat

logging.level.tomcat=TRACE
```


#### Custom Log Configuration
1. 在`classpath`中放置合适的类包
2. 在类路径下或Spring `Environment` 属性`logging.config`指定的位置提供合适的文件

通过`org.springframework.boot.logging.LoggingSystem` 系统属性，可以强制SpringBoot使用特定的日志，值必须为`LoggingSystem`实现的全类名

因为日志的初始化在`ApplicationContext`前，因此不能通过Spring的`@Configuration`文件的 `@PropertySources`来控制，只能通过系统变量来控制日志

 Logging System | Customization
------|------
Logback | `logback-spring.xml`,`logback-spring.groovy`,`logback.xml`, or `logback.groovy`
Log4j2 | `log4j2-spring.xml` `log4j2.xml`
JDK (Java Util Logging) | `logging.properties`

Spring `Environment` 与系统属性的转换关系：

Spring Environment|	System Property	| Comments
------|------|------
`logging.exception-conversion-word`| `LOG_EXCEPTION_CONVERSION_WORD`|The conversion word used when logging exceptions.
`logging.file` | `LOG_FILE` | If defined, it is used in the default log configuration.
`logging.file.max-size` | `LOG_FILE_MAX_SIZE` |Maximum log file size (if LOG_FILE enabled). (Only supported with the default Logback setup.)
`logging.file.max-history` | `LOG_FILE_MAX_HISTORY` |Maximum number of archive log files to keep (if LOG_FILE enabled). (Only supported with the default Logback setup.)
`logging.path` | `LOG_PATH` | If defined, it is used in the default log configuration.
`logging.pattern.console` | `CONSOLE_LOG_PATTERN` | The log pattern to use on the console (stdout). (Only supported with the default Logback setup.)
`logging.pattern.dateformat` | `LOG_DATEFORMAT_PATTERN` | Appender pattern for log date format. (Only supported with the default Logback setup.)
`logging.pattern.file` | `FILE_LOG_PATTERN` | The log pattern to use in a file (if LOG_FILE is enabled). (Only supported with the default Logback setup.)
`logging.pattern.level` | `LOG_LEVEL_PATTERN`|The format to use when rendering the log level (default %5p). (Only supported with the default Logback setup.)
`PID` | `PID` | The current process ID (discovered if possible and when not already defined as an OS environment variable).

#### Logback Extensions -Logback 扩展插件
SpringBoot提供多扩展可以在`logback-spring.xml`中使用

##### Profile-specific Configuration
logback中暴露SpringProfile的属性

```xml
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
	<!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
	<!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

##### Environment Properties
logback中暴露环境属性值

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
		defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
	<remoteHost>${fluentHost}</remoteHost>
	...
</appender>
```

### JSON

`Jackson` 是默认的推荐的库；

#### Jackson
`ObjectMapper` 会在Jackson类路径中包含时自动配置，`ObjectMapper` 有几个可以自定义的属性

#### Gson
`Gson`在类路径中包含Gson时会自动配置；`spring.gson.*`的几个配置项能够对进行自定义，控制一个或多的`GsonBuilderCustomizer`的使用

#### JSON-B

### Developing Web Applications

#### The “Spring Web MVC Framework”
```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

	@RequestMapping(value="/{user}", method=RequestMethod.GET)
	public User getUser(@PathVariable Long user) {
		// ...
	}

	@RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
	List<Customer> getUserCustomers(@PathVariable Long user) {
		// ...
	}

	@RequestMapping(value="/{user}", method=RequestMethod.DELETE)
	public User deleteUser(@PathVariable Long user) {
		// ...
	}

}
```

#####  Spring MVC Auto-configuration
SpringBoot的SpringMVC自动配置Bean：
- `ContentNegotiatingViewResolver` 和 `BeanNameViewResolver`
- 静态资源的获取，包括webjars
- `Converter` `GenericConverter` `Formatter` 自动注册
- `HttpMessageConverters` 支持
- `MessageCodesResolver` 自动注册
- `index.html` 支持
- `Favicon` 定制
- `ConfigurableWebBindingInitializer` 自动使用



##### HttpMessageConverter
`HttpMessageConverter` 转换HTTP请求和响应，默认String以UTF-8为编码

自定义`HttpMessageConverter`：
```java
@Configuration
public class MyConfiguration {

	@Bean
	public HttpMessageConverters customConverters() {
		HttpMessageConverter<?> additional = ...
		HttpMessageConverter<?> another = ...
		return new HttpMessageConverters(additional, another);
	}

}
```

所有的`HttpMessageConverters` 会被自动添加到Converterslist中

##### Custom JSON Serializers and Deserializers
`@JsonComponent` 可以用于类上，也可以用在内部类中有`serializers` `deserializers` 的类上:
```java
@JsonComponent
public class Example {

	public static class Serializer extends JsonSerializer<SomeObject> {
		// ...
	}

	public static class Deserializer extends JsonDeserializer<SomeObject> {
		// ...
	}
}
```
所有的`@JsonComponent`会自动被`ApplicationContext`注册

##### MessageCodesResolver
`MessageCodesResolver` 会为从错误信息中渲染出错误信息生成错误码

##### Static Content

默认的SpringBoot的静态内容会在类路径的 `/static` `/public` `resources` `/META-INF/resources` 中，或者`ServletContext` 的根路径中。


```properties
spring.mvc.static-path-pattern=/resources/** #修改请求路径
spring.resources.static-locations=/myresources/**           #修改静态资源位置

# 缓存静态资源 需要修改静态资源文件名
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**

# 固定资源版本号
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

##### Welcome Page
先找静态资源路径下的`index.html`,然后再找`index`的模板；

##### Custom Favicon
在静态资源下找`favicon.ico`,然后在类路径下找这个文件

##### Path Matching and Content Negotiation

SpringBoot默认会禁用后缀匹配模式:`GET /projects/spring-boot.json` 不会匹配到`@GetMapping("/projects/spring-boot")`

兼容HackSkill：
`GET /projects/spring-boot?format=json` 可以映射到 `@GetMapping("/projects/spring-boot")`
```properties
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```


使用后缀匹配
```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```
开放特定后缀的匹配:
```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true

# You can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc
```

##### ConfigurableWebBindingInitializer
SpringMVC使用 `WebBindingInitializer` 初始化`WebDataBinder`,也可以`ConfigurableWebBindingInitializer` 创建自定义的`@Bean`

##### Template Engines
- FreeMarker
- Groovy
- Thymeleaf
- Mustache

将文件放在`src/main/resources/templates` 或者 `classpath*:/templates/`

##### Error Handling

自定义`/error`请求
1. 实现`ErrorController`
2. 注册bean

或者  
注册`ErrorAttributes` beans

继承`BasicErrorController` 自定消费的特定的`Content-Type`:
```java
public CustomerErrorController extends BasicErrorController{
	@RequestMapping(produces="text/html")
	public String error(){
		return '';
	}
}
```

`@ControllerAdvice` 自定义JSON或特定请求的或异常的处理:
```java
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {

	@ExceptionHandler(YourException.class)
	@ResponseBody
	ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
	}

	private HttpStatus getStatus(HttpServletRequest request) {
		Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
		if (statusCode == null) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
		return HttpStatus.valueOf(statusCode);
	}

}
```

复杂的映射方式
```java
public class MyErrorViewResolver implements ErrorViewResolver {

	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request,
			HttpStatus status, Map<String, Object> model) {
		// Use the request or status to optionally return a ModelAndView
		return ...
	}

}
// ...

private static class MyErrorPageRegistrar implements ErrorPageRegistrar {

	@Override
	public void registerErrorPages(ErrorPageRegistry registry) {
		registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
	}

}
```


##### SPring HATEOAS (Hypermedia as the engine of application state)
`@EnableHypermediaSupport` 开启了，则`ObjectMapper`之前自定义的会失效

##### CORS
```java
@Configuration //全局CORS配置
public class MyConfiguration {

	@Bean
	public WebMvcConfigurer corsConfigurer() {
		return new WebMvcConfigurer() {
			@Override
			public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/api/**");
			}
		};
	}
}
```

### JAX-RS and Jersey

### Embedded Servlet Container Support
`org.springframework.boot.web.servlet.ServletContextInitializer` Servlet上下文初始化接口，如果需要这个很适合作为现有`WebApplicationInitializer`的适配器

`@WebServlet` `@WebFilter` `@WebListener` 能够被`@ServletComponentScan`扫到


##### ServletWebServerApplicationContext
`JettyServletWebServerFactory`  `TomcatServletWebServerFactory` `UndertowServletWebServerFactory`

##### Customizing Embedded Servlet Containers
```properties
server.port=8080
server.address=127.0.0.1
server.servlet.session.persistence=boolean
server.servlet.session.timeout=30
server.servlet.session.store-dir=/temp
server.servlet.session.cookie.*=
server.error.path=Location of the error page
```
ServerProperties 类作用

编程式自定义:
```java
@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

	@Override
	public void customize(ConfigurableServletWebServerFactory server) {
		server.setPort(9000);
	}

}

@Bean
public ConfigurableServletWebServerFactory webServerFactory() {
	TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
	factory.setPort(9000);
	factory.setSessionTimeout(10, TimeUnit.MINUTES);
	factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
	return factory;
}
```

#### Reactive(反应式) Server Resources Configuration
默认:
- 同样的技术会提供给服务端和客户端
- 客户端实例使用`WebClient.Builder`-SpringBoot自动配置的Bean构建的


## Security
`DefaultAuthenticationEventPublisher`发布验证事件

### MVC Security
`SecurityAutoConfiguration` `UserDetailsServiceAutoConfiguration` 实现的默认安全配置

`WebSecurityConfigurerAdapter` 覆盖默认的安全认证，而不会让其他的安全认证失效

覆盖`UserDetailsService` 配置:提供 `UserDetailsService` `AuthenticationProvider` `AuthenticationManager` 类型Bean

访问规则可以由`WebSecurityConfigurerAdapter`自定义

### WebFlux Security
`spring-boot-starter-security` 加入依赖

默认安全配置:`ReactiveSecurityAutoConfiguration` `UserDetailsServiceAutoConfiguration`;`WebFilterChainProxy`可以用于关闭默认的安全配置，而`UserDetailsService` 不会受影响;`ReactiveUserDetailsService` 和 `ReactiveAuthenticationManager`可以关闭掉 `UserDetailsService`

`SecurityWebFilterChain` 访问规则
```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
	return http
		.authorizeExchange()
			.matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
			.pathMatchers("/foo", "/bar")
				.authenticated().and()
			.formLogin().and()
		.build();
}
```


### OAuth2
`OAuth2ClientProperties`

```properties
spring.security.oauth2.client.registration.my-client-1.client-id=abcd
spring.security.oauth2.client.registration.my-client-1.client-secret=password
spring.security.oauth2.client.registration.my-client-1.client-name=Client for user scope
spring.security.oauth2.client.registration.my-client-1.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-1.scope=user
spring.security.oauth2.client.registration.my-client-1.redirect-uri-template=http://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-1.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-1.authorization-grant-type=authorization_code

spring.security.oauth2.client.registration.my-client-2.client-id=abcd
spring.security.oauth2.client.registration.my-client-2.client-secret=password
spring.security.oauth2.client.registration.my-client-2.client-name=Client for email scope
spring.security.oauth2.client.registration.my-client-2.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-2.scope=email
spring.security.oauth2.client.registration.my-client-2.redirect-uri-template=http://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-2.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-2.authorization-grant-type=authorization_code

spring.security.oauth2.client.provider.my-oauth-provider.authorization-uri=http://my-auth-server/oauth/authorize
spring.security.oauth2.client.provider.my-oauth-provider.token-uri=http://my-auth-server/oauth/token
spring.security.oauth2.client.provider.my-oauth-provider.user-info-uri=http://my-auth-server/userinfo
spring.security.oauth2.client.provider.my-oauth-provider.user-info-authentication-method=header
spring.security.oauth2.client.provider.my-oauth-provider.jwk-set-uri=http://my-auth-server/token_keys
spring.security.oauth2.client.provider.my-oauth-provider.user-name-attribute=name
```

`OAuth2LoginAuthenticationFilter`默认只处理 `/login/oauth2/code/*` 路劲请求;可以使用`WebSecurityConfigurerAdapter`自定义Banner

```java
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.anyRequest().authenticated()
				.and()
			.oauth2Login()
				.redirectionEndpoint()
					.baseUri("/custom-callback");
	}
}
```
Google Provider
```properties
spring.security.oauth2.client.registration.my-client.client-id=abcd
spring.security.oauth2.client.registration.my-client.client-secret=password
spring.security.oauth2.client.registration.my-client.provider=google

spring.security.oauth2.client.registration.google.client-id=abcd
spring.security.oauth2.client.registration.google.client-secret=password
```
### Actuator Security
默认下`/health` `/info` 外的请求会被要求验证权限

`management.endpoints.web.exposure.include`



















----
