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
