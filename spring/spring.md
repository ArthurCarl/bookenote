# Spring

## IOC
Spring 提供2种IOC容器：`BeanFactory` `ApplicationContext`
### BeanFactory

BeanFactory 职责：对象的注册和依赖管理

BeanDefinitionRegistry 实现Bean的注册逻辑关系如下：

![BeanFactory关系图](pic/BeanFactory.png)

BeanDefinitionRegistry 管理BeanDefinition(怎么样创建Bean)；  
BeanDefinitionReader 读取BeanDefinition(注解方式则用`classpath-scanning`)；  

BeanFactory 的XML配置

```xml
<beans default-lazy-init default-autowire default-dependency-check default-init-method default-destroy-method>
  <description/>
  <import />
  <alias name="beanName" alias="bean_name"/>
  <bean id="id" class="packag.A" name="beanName" >
    <constructor-arg index="0" ref="otherBeanName"/>
    <constructor-arg type="int">
      <value>123</value>
    </constructor-arg>
    <property name="propertyName" ref="beanName"/>
    <property name="collection">
      <list>
        <value>123</value>
        <ref bean="beanName"/>
        <bean class="beanclassName"/>
      </list>
      <set>
        <value>123</value>
        <ref bean="beanName"/>
        <bean class="beanclassName"/>
      </set>
      <map>
        <entry key="strValueKey">
          <value>something</value>
        </entry>
        <entry>
          <key>objectKey</key>
          <ref bean="someObject"/>
        </entry>
        <entry key-ref="lstKey">
          <list>
            ...
          </list>
        </entry>
      </map>
  </bean>
</beans>
```

Bean Scope : protype singleton request session globalsession
后三种为web应用特有

Scope 接口可以实现自定义Scope类型；

#### 工厂方法与 FactoryBean
1. 静态工厂方法(Static Factory Method) .
```xml
<bean id="foo" class="...Foo"> <property name="barInterface">
  <ref bean="bar"/> </property>
</bean>
<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance"/>
```
2.非静态工厂方法(Instance Factory Method)
```XML
<bean id="foo" class="...Foo">
  <property name="bar">
    <ref bean="bar"/>
  </property>
</bean>
<bean id="barFactory" class="...NonStaticFactory"/>
<bean id="bar" factory-bean="barFatory" factory-method="getInstance"/>
```

FactoryBean -扩展容器对象实例化逻辑接口，与其他容器对像一样，只是生产对象的工厂；当对象的实例化过程很复杂或第三方库不能直接注册到Spring容器的时候，实现`org.spring- framework.beans.factory.FactoryBean` 给出实例化逻辑
```java
public class NextDayDateFactoryBean implements FactoryBean{
  ......
}

public class NextDayDateDisplayer {
  private DateTime dateOfNextDay;
    // 相应的setter方法
    // ...
}
```
```XML
<bean id="nextDayDateDisplayer" class="...NextDayDateDisplayer">
  <property name="dateOfNextDay">
    <ref bean="nextDayDate"/>
  </property>
</bean>
<bean id="nextDayDate" class="...NextDayDateFactoryBean"> </bean>
```

`BeanFactoryAware`接口-Spring容器在实例化实现该接口bean时会 **自动将容器本身注入该Bean中**

`ObjectFactoryCreatingFactoryBean` Spring的FactoryBean实现，返回ObjectFactory实例，通过ObjectFactory就可以获得Spring容器管理的对象。

`MethodReplacer` 接口可以将获取Bean的方法给替换掉：
```java
public class FXNewsProviderMethodReplacer implements MethodReplacer{
  public Object reimplement(Object target,Method method , Object[] args){
    //do something
    return null;
  }
}
```
```XML
<bean id="djNewsProvider" class="...FXNewsProvider">
  <constructor-arg inde="0">
    <ref bean="djNewsProvider"/>
  </constructor-arg>
  <replaced-method name="getAndPersistNews" replacer="providerReplacer">
  </replaced-method>
</bean>
<bean id="providerReplacer" class="....FXNewsProviderMethodReplacer">
</bean>
```
#### Spring容器IoC启动分2个阶段：
1. 容器启动阶段
  - 加载配置
  - 分析配置信息
  - 装备BeanDefinition
  - 其他.....
2. Bean实例化阶段
  - 实例化对象
  - 装配依赖
  - 生命周期回调
  - 对象其他处理
  - 注册回调接口

BeanFactoryPostProcessor-Spring容器的扩展机制，允许在容器实例化对象前，对注册到容器的BeanDefinition所保存的信息作相应的修改。`PropertyPlaceholderConfigurer`和`PropertyOverrideConfigurer`是比较常用的BeanFactoryPostProcessor

BeanFactory的BeanFactoryPostProcessor
```java
//声明被后处理的BeanFactory实例
ConfigurableListableBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("..."));
//声明要使用的BeanFactoryPostProcessor
PropertyPlaceholderConfigur propertyPostProcessor = new PropertyPlaceholderConfigur();
propertyPostProcessor.setLocation(new ClassPathResource("..."));
//执行后处理操作
propertyPostProcessor.postProcessBeanFactory(beanFactory);
```

ApplicationContext 使用BeanFactoryPostProcessor
```XML
<beans>
  <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
      <list>
        <value>conf/jdbc.properties</value>
        <value>conf/mail.properties</value>
      </list>
    </property>
  </bean>
</beans>
```





























-----
