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
PropertyOverrideConfigurer -覆盖之前的属性配置值

CustomEditorConfigurer-属性值转换为对象的规则信息；
```java
public class DatePropertyEditor extends PropertyEditorSupport{
  private String dataPatern;

  @override
  public void setAsText(String text)throws IllegalArgumentException{
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.forPattern(getDatePattern());
    Date dateValue = dateTimeFormatter.parseDateTime(text).toDate();
    setValue(dateValue);
  }
  public String getDatePattern(){
    return datePattern;
  }
  public void setDatePattern(String datePattern){
    this.datePattern = datePattern;
  }
}

public class DateFoo{
  private Date date;

  public Date getDate(){
    return date;
  }

  public void setDate(Date date){
    this.date = date;
  }
}
```
```XML
<bean id="dateFoo" class="....DateFoo">
  <property name="date">
    <value>2007/10/18</value>
  </property>
</bean>
```

BeanFactory 应用CustomerEditorConfigurer到容器：
```java
XmlBeanFactory  beanFactory = new XmlBeanFactory(new ClassPathResource("..."));
CustomEditorConfigurer ceConfig = new CustomEditorConfigurer();
Map customeeEditor = new HashMap();
customeeEditor.put(java.util.Date.class,new DatePropertyEditor());
ceConfig.setCustomerEditors(customeeEditor);
ceConfig.postProcessBeanFactory(beanFactory);
```

ApplicationContext 的使用：
```XML
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
  <property name="customEditors">
    <map>
      <entry key="java.util.Date">
        <ref bean ="datePropertyEditor"/>
      </entry>
    </map>
  </property>
</bean>

<bean id="datePropertyEditor" class="...DatePropertyEditor">
  <property name="datePattern">
    <value>yyyy/mm/dd</value>
  </property>
</bean>
```

Spring 2.0之后将采用`org.springframework.beans. PropertyEditorRegistrar`实现注册新的自定义

Bean的实例化：
1. BeanFactory容器的实例化可以采用延迟初始化策略，`getBean()`才去实例化bean
2. ApplicationContext容器的在启动后会紧接着调用`getBean()`(见`org.springframework.context.support. AbstractApplicationContext.refresh()`);

Bean的实例化过程图如下：
![Ban的实例化过程](pic/Bean的实例化过程.png)

注：`org.springframework.beans.factory.support.AbstractBeanFactory.getBean()`  
`org.springframework.beans. factory.support.AbstractAutowireCapableBeanFactory.createBean()`

1. Bean的实例化与BeanWrapper  
  - 根据`BeanDefinition`返回一个`BeanWrapperImp`实例
  - 对`BeanWrapperImp`包裹着的Bean进行操作：设置属性值等；
2. 各色`Aware`接口  
检查Bean是否实现了Aware结尾的接口  
  - `org.springframework.beans.factory.BeanNameAware`
  - `org.springframework.beans.factory.BeanClassLoaderAware`
  - `org.springframework.beans.factory.BeanFactoryAware`
  - `org.springframework.context.ResourceLoaderAware`
  - `org.springframework.context.ApplicationEventPublisherAware`
  - `org.springframework.context.MessageSourceAware`
  - `org.springframework.context.ApplicationContextAware`
3. BeanPostProcessor  
BeanPostProcessor 存在于对象实例化阶段，而BeanFactoryPostProcessor存在在于容器启动阶段；  
`ApplicationContextAwareProcessor.postProcessBeforeInitialization()`
4. InitializingBean和init-method
5. DisposableBean与destroy-method

## ApplicationContext































-----
