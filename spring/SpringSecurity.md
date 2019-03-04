# Spring Security
## Authentication
 Principal : userName   -> UserDetail
 Credentials : Password -> ---
 Authorities:           -> ROLE_USER
 Authenticated:         -> True

## Authentication
- `AuthenticationFilter` 创建AuthenticationRequest，传给`AuthenticationManager`
- `AuthenticationManager` 委托`AuthenticationProvider`
- `AuthenticationProvider` 使用`UserDetailsService`获取用户认证信息
- `AuthenticationFilter`将`Authentication`放置在ThreadLocal的`SecurityContext`中。

## Authorization
- `FilterSecurityInterceptor`通过请求获取`SecurityMetadata`
- `FilterSecurityInterceptor`通过`SecurityContext`获取`Authentication`
- `Authentication` `Security Metadata` `Request` 传给 `AccessDecisionManager`
- `AccessDecisionManager`委托`AccessDecisionVoter`

## Spring Security Filter Chain
![SpringSecurityFilterChain](pic/SpringSecurityFilterChain.png)



------


# spring-security-architecture
安全涉及的2个方面:认证(Authentication)和授权(Authorization)

## 用户认证和访问控制
认证主要接口 `AuthenticationManager`

认证失败需要返回HTTP服务需要返回`401`,`WWW-Authenticate`响应中带不带则根据上下文。

`ProviderManager`是 `AuthenticationManager`接口常用实现类。

`ProviderManager` 将认证工作委托给 `AuthenticationProvider` 链处理。

`ProviderManager` 有父类，这样认证具有层次感，还可以提供全局认证管理器。

### 认证定制化
`AuthenticationManagerBuilder` 工具类快速进行定制化认证.

`UserDetailsService` 提供自定义的设置

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

  // web stuff here

  @Autowired //全局 AuthenticationManager
  public initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

  @Autowired
  DataSource dataSource;

   // web stuff here

  @Override //本地应用 AuthenticationManager
  public configure(AuthenticationManagerBuilder builder) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

### 授权/访问控制
`AccessDecisionManager` 核心授权策略接口，框架提供了三个实现类，每个委托给`AccessDecisionVoter`链处理。

`AccessDecisionManager` 考虑 `Authentication`代表主体，`ConfigAttributes` 安全对象需要使用的

```java
boolean supports(ConfigAttribute attribute);

boolean supports(Class<?> clazz);

int vote(Authentication authentication, S object,
        Collection<ConfigAttribute> attributes);
```

## Web Security
Spring Security 拦截器顺序的实现机制:
1. `@Bean` 类型的 `Filter` 用 `@Order` 注释或者实现`Ordered`接口。
2. `FilterRegistrationBean`的一部分，这个自带有顺序API

## 自定义拦截器
```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
     ...;
  }
}
```

## 请求分发和认证
```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
      .authorizeRequests()
        .antMatchers("/foo/bar").hasRole("BAR")
        .antMatchers("/foo/spam").hasRole("SPAM")
        .anyRequest().isAuthenticated();
  }
}
```

## 异步方法的安全
```java
@Configuration
public class ApplicationConfiguration extends AsyncConfigurerSupport {

  @Override
  public Executor getAsyncExecutor() {
    return new DelegatingSecurityContextExecutorService(Executors.newFixedThreadPool(5));
  }

}
```


# Spring Security
## 依赖
### Spring Boot
```xml
<dependencies>
    <!-- ... other dependency elements ... -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```
### Maven
```xml
<dependencyManagement>
    <dependencies>
        <!-- ... other dependency elements ... -->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-bom</artifactId>
            <version>5.2.0.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## Servlet Applications 配置
### JAVA配置
1. `@EnableWebSecurity` 与 `WebSecurityConfiguration`
2. `AbstractSecurityWebApplicationInitializer`	
	- 非Spring项目:继承`AbstractSecurityWebApplicationInitializer` 构造方法传递`super(WebSecurityConfig.class)`
	- Spring项目:直接继承`AbstractSecurityWebApplicationInitializer`;`getRootConfigClasses()`中添加`WebSecurityConfig.class`类
	

## Architecture and Implementation

#### `SecurityContextHolder` `SecurityContext` `Authentication`
`SecurityContextHolder` 核心:存储应用安全上下文,默认为`ThreadLocal`

改变策略的2中方式: `SecurityContextHolder` 的JavaDoc;

`Authentication` - principal用户信息  

`GrantedAuthority` -权限

Summary
- `SecurityContextHolder` 访问 `SecurityContext`
- `SecurityContext` 持有 `Authentication`
- `Authentication` 用户信息
- `GrantedAuthority` 用户的权限
- `UserDetails` 自定义DAO用户的必要认证信息
- `UserDetailsService` 创建`UserDetails`

### `Authentication`
- 用户名、密码实例化一个`UsernamePasswordAuthenticationToken`(Authentication 接口实现类)对象
- `AuthenticationManager`校验，返回`Authentication`成功的认知信息
- `SecurityContextHolder.getContext().setAuthentication(…​)` 认证成功

### `Authentication` 网站中应用
- `ExceptionTranslationFilter` -处理认证时可能的异常
- `AuthenticationEntryPoint` -
- 认证机制(`AuthenticationManager`调用)

### `SecurityContextPersistenceFilter`
在每个请求间将 `SecurityContext` 作为 `HttpSession`的属性保存；请求完成时,清除`SecurityContextHolder`;

在无状态的Rest服务中，也需要将`SecurityContextPersistenceFilter`包含在拦截器链中，负责每个请求结束时清除`SecurityContextHolder`;

### Access-Control
安全对象-需要保护的方法、web请求；每个安全对象类型都有对应得拦截器(`interceptor`)

`AbstractSecurityInterceptor` 处理请求:
- 寻找当前请求的配置属性
- 安全对象、`Authentication`、配置属性值提交给`AccessDecisionManager`,进行认证
- 或许改变`Authentication`在调用时
- 安全对象的通过认证继续执行
- `AfterInvocationManager`配置的话，执行

`Configuration Attributes` - `AbstractSecurityInterceptor` 有特殊含义的`String`,由`ConfigAttribute`接口呈现;可以是角色名、可以有更多含义取决于`AccessDecisionManager`的实现理念。

`RunAsManager`-伪装身份

`AfterInvocationManager`-方法完成后

支持三种安全对象类型:`MethodInvocation` `JoinPoint` `FilterInvocation`;

## Core Service
`AuthenticationManager` `UserDetailsService` `AccessDecisionManager`

`AuthenticationManager` 委托 `ProviderManager` 进行认证

`DaoAuthenticationProvider` 是 `ProviderManager` 接口的实现类

### `Password Encoding`
`PasswordEncoder`单向加密，不提解密功能

`DelegatingPasswordEncoder` 密码代理加密

``

