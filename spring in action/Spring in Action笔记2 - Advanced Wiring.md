# Spring in Action笔记2 - Advanced Wiring

## Profile
在开发过过程中, 不同的环境下可能需要不同的配置, 比如数据库设置, 加密算法, 以及扩展系统的集成等。这时我们就需要使用profile进行配置了

在3.1版本中, Spring引入了bean profile。使用profile时, 而我们需要把不同的bean放在一个或多个profile中, 以保证在某个profile被激活时, 那个bean才会被创建。

在Java配置中, 可以使用@Profile注解来说明这个bean属于哪个profile。

例如我们在开发过程中,使用h2作为数据库:

```java
package com.myapp;
import javax.activation.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType;

@Configuration
@Profile("dev")
public class DevelopmentProfileConfig {

  @Bean(destroyMethod="shutdown")
  public DataSource dataSource() {
      return new EmbeddedDatabaseBuilder()
          .setType(EmbeddedDatabaseType.H2)
          .addScript("classpath:schema.sql")
          .addScript("classpath:test-data.sql")
          .build();
  }
}
```

而作为产品时, 则可能需要使用JNDI:

```java
package com.myapp;
import javax.activation.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.jndi.JndiObjectFactoryBean;

@Configuration
@Profile("prod")
public class ProductionProfileConfig {

  @Bean
  public DataSource dataSource() {
    JndiObjectFactoryBean jndiObjectFactoryBean =
        new JndiObjectFactoryBean();
    jndiObjectFactoryBean.setJndiName("jdbc/myDS");
    jndiObjectFactoryBean.setResourceRef(true);
    jndiObjectFactoryBean.setProxyInterface(
        javax.sql.DataSource.class);
    return (DataSource) jndiObjectFactoryBean.getObject();
  }
}
```

### 使用profile
在dev这个profile被激活时, 会执行前面那个配置, 而prod被激活时, 则执行后面那个配置。

从Spring3.2开始, 可以在方法前增加@Profile注解:

```java
package com.myapp;
import javax.activation.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType;
import org.springframework.jndi.JndiObjectFactoryBean;

@Configuration
public class DataSourceConfig {
  
  @Bean(destroyMethod="shutdown")
  @Profile("def")
  public DataSource embeddedDataSource() {
    return new EmbeddedDatabaseBuilder()
      .setType(EmbeddedDatabaseType.H2)
      .addScript("classpath:schema.sql")
      .addScript("classpath:test-data.sql")
      .build();
  }

  @Bean
  @Profile("prod")
  public DataSource jndiDataSource() {
    JndiObjectFactoryBean jndiObjectFactoryBean =
        new JndiObjectFactoryBean();
    jndiObjectFactoryBean.setJndiName("jdbc/myDS");
    jndiObjectFactoryBean.setResourceRef(true);
    jndiObjectFactoryBean.setProxyInterface(
            javax.sql.DataSource.class);
    return (DataSource) jndiObjectFactoryBean.getObject();
  }
}
```

在xml文件中, 可以设置beans的profile属性:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:jdbc="http://www.springframework.org/schema/jdbc"
  xsi:schemaLocation="
    http://www.springframework.org/schema/jdbc
    http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd"
   profile="dev">

  <jdbc:embedded-database id="dataSource">
    <jdbc:script location="classpath:schema.sql" />
    <jdbc:script location="classpath:test-data.sql" />
  </jdbc:embedded-database>
</beans>
```

### 激活profile
Spring提供了两个属性用来决定那个profile被激活, 分别是spring.profiles.active和spring.profiles.default。如果active被设置, 那么就会忽略default, 否则查看default所设置的值, 如果default也没有被设置, 那么只有没有profile的bean才会被创建。

有以下几种方式来设置这两个属性:

- 作为DispatcherServlet的初始化参数
- 作为网站应用的上下文参数
- 作为JNDI条目
- 作为环境变量
- 作为JVM系统属性
- 使用集成test类的@ActiveProfiles注解

#### 在web.xml中设置

在web-app元素中添加:

```xml
<context-param>
  <param-name>spring.profiles.default</param-name>
  <param-value>dev</param-value> 
</context-param>
```

在servlet中添加:

```xml
<init-param>
  <param-name>spring.profiles.default</param-name>
  <param-value>dev</param-value> 
</init-param>
```

在真正进行部署时, 则可以直接用spring.profiles.active来覆盖default。

#### 对profile进行测试
语法如下:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes={PersistenceTestConfig.class})
@ActiveProfiles("dev")
public class PersistenceTest {
  ...
}
```

## 条件bean
如果我们需要在某些环境变量被设置, 或者某个bean被创建等等条件下, 创建某个bean, 那么我们可以使用Spring 4引入的@Confitional注解进行条件化配置(conditional configuration)。比如我们需要在系统环境中存在magic属性的情况下才生成一个bean:

```java
@Bean
@Confitional(MagicExistsCondition.class)
  public MagicBean magicBean() {
    return new MagicBean()
  }
```

MagicExistsCondition类需要实现Condition接口:

```java
public interface Condition {
  boolean matches(ConditionContext ctxt,
                  AnnotatedTypeMetadata metadata);
}
```

matches()方法返回boolean, 如果返回true, 则说明满足条件。例如:

```java
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;
import org.springframework.util.ClassUtils;

public class MagicExistsCondition implements Condition {
    public boolean matches(ConditionContext context,
                           AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        return env.containsProperty("magic");
    }
}
```

ConditionContext接口定义如下:

```java
public interface ConditionContext {
  BeanDefinitionRegistry getRegistry();
  ConfigurableListableBeanFactory getBeanFactory();
  Environment getEnvironment();
  ResourceLoader getResourceLoader();
  ClassLoader getClassLoader();
}
```

通过ConditionContext我们可以得到以下信息:

- 通过getRegistryf()返回的BeanDefinitionRegistry查看bean的定义
- 通过getBeanFactory()返回的ConfigurableListableBeanFactory查看bean的存在性以及属性
- 通过getEnvironment()返回的Environment查看环境变量
- 通过getResourceLoader()返回的ResourceLoader读取、检测资源
- 通过getClassLoader()返回的ClassLoader查看、加载某些类

AnnotatedTypeMetadata则能够得到注解相关的信息, 接口定义如下:

```java
public interface AnnotatedTypeMetadata {
  boolean isAnnotated(String annotationType);
  Map<String, Object> getAnnotationAttributes(String annotationType);
  Map<String, Object> getAnnotationAttributes(String annotationType, boolean classValuesAsString);
  MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationType);
  MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationType, boolean classValuesAsString);
}
```

在Spring 4对@Profile注解进行了重构:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {
  String[] value();
}
```

可以发现使用了@Conditional注解, 引用了ProfileCondition类:

```java
class ProfileCondition implements Condition {
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
      MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
      if (attrs != null) {
        for (Object value : attrs.get("value")) {
          if (context.getEnvironment()
                     .acceptsProfiles(((String[]) value))) {
            return true;
          }
        }
        return false;
      }
    }
    return true;
  }
}
```

ProfileCondition类会获得当前上下文的所有@Profile注解, 如果存在其value与当前的活动profile匹配(.acceptsProfiles()方法), 才会创建

## 自动装配中的歧义性
假设有这样的方法, 使用自动装配赋予dessert的值:

```java
@Autowired
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
```

并基于该接口实现了多个类:

```java
@Component
public class Cake implements Dessert { ... }
@Component
public class Cookies implements Dessert { ... }
@Component
public class IceCream implements Dessert { ... }
```

由于所有的类都加了@Component注解, 在组件扫描的过程中, 这些类都会被创建成bean, 这时, 之前的@Autowire就会出现问题, 因为产生了歧义, 运行时, 会抛出NoUniqueBeanDefinitionException。

### @Primary注解
为了解决歧义性, 同时保证自动装配能够使用, Spring有@Primary注解, 他可以与@bean或者Component同时使用:

```java
@Component
@Primary
public class IceCream implements Dessert { ... }
```

```java
@Bean
@Primary
public Dessert iceCream() {
    return new IceCream();
}
```

在xml文件中也可以设置primary属性:

```xml
<bean id="iceCream"
      class="com.desserteater.IceCream"
      primary="true" />
```

### 对自动装配进行限定
上面的问题也可以使用@Qualifier注解进行解决:

```java
@Autowired
@Qualifier("iceCream")
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
```

这样就能指定使用iceCream(IceCream bean的id)进行装配

#### 创建自己的qualifier
如果把@Qualifier注解放在bean的定义之前, 则可以为它创建一个qualifier, 比如可以和@Component注解放在一起:

```java
@Component
@Qualifier("cold")
public class IceCream implements Dessert { ... }
```

也可以和@Bean放在一起:

```java
@Bean
@Qualifier("cold")
public Dessert iceCream() {
    return new IceCream();
}
```

可以同时定义、添加多个Qualifier, 以便更灵活的限定。qualifier的最佳实践是使用描述性的术语, 而不是一个随意的名字。

使用@Qualifier还能创建自己的注解, 来表示相应的Qualifier:

```java
@Target({ElementType.CONSTRUCTOR, ElementType.FIELD,
         ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Creamy { }
```

同样我们可以创建@Cold @Crispy @Fruity注解, 实现限定的功能:

```java
@Autowired
@Cold
@Creamy
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
```

## bean的作用域
多数情况下, 使用单例是一个较好的选择, 但是如果使用的bean是可变的, 而且包含了一些状态, 重用这些对象就会导致问题。Spring为bean定义了许多作用域:

- singleton 单例
- prototype 每次被注入或者遍历到时, 都会重新创建新的实例
- session 为每个会话创建一个实例
- request 为每个请求创建一个实例

单例是默认的作用域, 如果需要选择其他作用域, 可以使用@Scope注解, 可以和@Component或者@Bean注解一起使用:

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Notepad {...}
```

也可以直接使用```@Scope("prototype")```, 但是前者是类型安全的, 不容易出错。

在xml文件中, 可以设置bean元素的scope属性:

```xml
<bean id="notepad"
      class="com.myapp.Notepad"
      scope="prototype" />
```

### 使用会话级作用域
一个典型的例子就是购物车, 我们通常会为一个会话创建一个购物列表, 这时可以将它创建在会话级作用域中:

```java
@Component
@Scope(
    value=WebApplicationContext.SCOPE_SESSION,
    proxyMode=ScopedProxyMode.INTERFACES)
public ShoppingCart cart() { ... }
```

proxyMode被设置为了ScopedProxyMode.INTERFACESi。实际上它解决了在会话或者请求作用域中进行注入时可能遇到的问题, 接下来看一下问题可能出现的场景:

假设我们需要在一个单例的StoreService的一个setter方法中注入一个ShoppingCart bean:

```java
@Component
public class StoreService {
  @Autowired
  public void setShoppingCart(ShoppingCart shoppingCart) {
    this.shoppingCart = shoppingCart;
  }
  ...
}
```

那么问题来了。

ShoppingCart是会话作用域里的单例, 而StoreService本身就是个单例, 在对StoreService的setter方法进行注入时, session还不存在呢。这时就没法注入一个会话中的ShoppingCart了。而且购物车也不是只有一个, 而StoreService只有一个, 要选哪个购物车进行注入呢?所以实际上, Spring的做法并不是真正的注入一个购物车实例, 而是注入一个代理的bean, 这个代理会暴露与ShoppingCart相同的方法, 但是在StoreService真正调用这个方法时, 代理对象会找到会话作用域中实际的购物车对象。

(卧槽这才是真正的代理模式啊..)

### 在xml文件里声明作用域代理
在XML文件里声明一个作用域的方式如下:

```xml
<bean id="cart"
      class="com.myapp.ShoppingCart"
      scope="session">
  <aop:scoped-proxy />
</bean>
```

在里面必须使用一个aop命名空间下的scoped-proxy, 它默认使用GCLib来创建目标类, 如果需要机遇接口的代理, 则需要如下配置:

```xml
<bean id="cart"
      class="com.myapp.ShoppingCart"
      scope="session">
  <aop:scoped-proxy proxy-target-class="false" />
</bean>
```

当然在beans里面是需要声明aop命名空间的:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">
...
</beans>
```

## 运行期注入
有时候我们需要在运行期决定需要注入的值, Spring提供了两种方式在运行期计算哪些值:

- 属性占位符
- Spring表达式语言(SpEcL)

### 注入外部值
最简单的方式就是创建一个属性源文件, 然后在Spring Environment中遍历他们:

#### 使用属性

```java
@Configuration
@PropertySource("classpath:/com/soundsystem/app.properties")
public class ExpressiveConfig {
  @Autowired
  Environment env;

  @Bean
  public BlankDisc disc() {
    return new BlankDisc(
      env.getProperty("disc.title"),
      env.getProperty("disc.artist"));
  }
}
```

app.properties:

```
disc.title=blabla
disc.artist=blabla
```

Environment类有以下方法, 用法不赘述:

```java
String getProperty(String key)
String getProperty(String key, String defaultValue)
T getProperty(String key, Class<T> type)
T getProperty(String key, Class<T> type, T defaultValue)
```

如果使用env.getRequiredProperty()则会在得不到相关属性时抛出IllegalStateException。

Environment类还能获得profile:

```java
String[] getActiveProfiles()
String[] getDefaultProfiles()
boolean acceptsProfiles(String... profiles)
```

#### 属性占位符
也可以使用属性占位符, 格式为${...}, 可以在xml或java文件中使用它们:

```xml
<bean id="sgtPeppers"
      class="soundsystem.BlankDisc"
      c:_title="${disc.title}"
      c:_artist="${disc.artist}" />
```

在java中, 使用@Value注解:

```java
public BlankDisc(
      @Value("${disc.title}") String title,
      @Value("${disc.artist}") String artist) {
  this.title = title;
  this.artist = artist;
}
```

在xml中使用属性占位符, 是需要事先声明命名空间的:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

  <context:property-placeholder />

</beans>
```

#### Spring表达式语言

Spring 3引入了Spring表达式语言(SpEL), 格式为#{...}, 它可以实现一些其他装配方式无法达到的功能, 为此, SpEL使用了以下技巧:

- 使用id引用bean
- 调用方法并访问对象的属性
- 各种操作符
- 正则表达式
- 集合操作

在java和xml文件中的使用与前面的属性占位符相同:

```java
public BlankDisc(
      @Value("#{systemProperties['disc.title']}") String title,
      @Value("#{systemProperties['disc.artist']}") String artist) {
  this.title = title;
  this.artist = artist;
}
```

```xml
<bean id="sgtPeppers"
      class="soundsystem.BlankDisc"
      c:_title="#{systemProperties['disc.title']}"
      c:_artist="#{systemProperties['disc.artist']}" />
```

SpEL支持以下操作符:

Operator type|Operators
---|---
Arithmetic|+, -, \*, /, %, ^
Comparison|<, lt, >, gt, ==, eq, <=, le, >=, ge
Logical|and, or, not, |
Conditional|?: (ternary), ?: (Elvis)
Regular expression|matches

以下下为示例:

```
#{sgtPeppers}                      "引用bean
#{sgtPeppers.artist}               "引用bean并获得其artist属性
#{artistSelector.selectArtist()}   "引用bean并调用其方法
#{artistSelector.selectArtist()?.toUpperCase}
                                   "使用问号避免空引用时产生的NullPointerException
#{systemProperties['disc.title']}  "使用系统属性
#{3.14159}                         "浮点数
#{9.87E4}                          "带指数的浮点数
#{false}                           "布尔值
#{T(java.lang.Math)}               "T()表示类型
#{T(java.lang.Math).PI}            "访问静态域
#{2 * T(java.lang.Math).PI * circle.radius}
                                   "使用操作符计算圆周长周长
#{(java.lang.Math).PI * circle.radius ^ 2}
                                   "还能使用^表示乘方
#{disc.title + 'by' + disc.artist} "单引号包围表示字符串常量, 可使用+连接
#{counter.total == 100}            "可使用==或者eq, 效果相同
#{counter.total eq 100}
#{admin.email matches '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.com'}
#{jukebox.songs[4].title}          "对容器使用中括号索引
#{jukebox.songs[T(java.lang.Math).random() * jukebox.songs.size()].title}
                                   "索引的复杂版
#{'This is a test'[3]}             "对字符串也能进行索引
#{jukebox.songs.?[artist eq 'Aerosmith']}
                                   ".?[]操作符用来进行过滤, 选择artist域等于"Aerosmith"的元素
                                   ".^[]和.$[]用来选择首个匹配和最后一个匹配元素
#{jukebox.songs.![title]}          "作投影, 得到一个title的集合
#{jukebox.songs.?[artist eq 'Aerosmith'].![title]}
                                   "得到所有Areosmith的歌的标题
```
