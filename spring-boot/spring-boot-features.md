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



--
