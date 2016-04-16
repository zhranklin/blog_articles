# Spring in Action笔记1 - Wiring Beans
## wiring装配
在spring中,对象不需要自行寻找、创建所需的其他对象, 容器(container)会给予相应的对象引用供其使用。比如一个订单管理组件, 需要一个账户验证其(authorizer)用来验证, 而我们什么都不用做就能得到相应的验证器。

创建这种应用间联系, 就是依赖注入(DI, dependency injection)的关键, 也常被称为装配(wiring)。DI是Spring中最基本的东西, 在开发给予Spring的应用时, 会非常频繁的使用。bean装配有一下三种基本方式: 

- 在XML中显式地配置
- 在Java中显式地配置
- 在bean搜寻和自动装配中隐含

## 自动装配(automatically wiring beans)
自动装配通常是最好的选择, 他可以省去一些工作。Spring在两方面完成自动装配:
- 组件扫描(Component scanning) -- Spring自动发现bean, 并在应用上下文中创建
- 自动装配(Autowiring) -- Spring会自动满足bean的依赖

结合两者, Spring可以最大程度的避免显式配置

### 创建可被发现(discoverable)的bean

按照书上以CD和CD播放器为例, 我们认为CD播放器的工作依赖CD。先定义一个CD的接口:

```java
package soundsystem;
public interface CompactDisc {
  void play();
}
```

首先我们定义一个CompactDisc的实现:

```java
package soundsystem;
import org.springframwork.stereotype.Component;

@Component
public class SgtPeppers impolements CompatDisc {
  private String title = "Sgt. Pepper's Lonely Hearts Club Band";
  private String artist = "The Beatles";

  public void play() {
    System.out.println("Playing " + title + " by " + artist);
  }
}
```

这个类前面加了一个@Component注解, 告诉Spring这个类作为组件, 应该被创建成一个bean。

组件扫描(Component Scanning)默认是不开启的, 所以需要显式说明寻找带有@Component注解的类并且创建bean:(一下为最简单的配置方式)

```java
package soundsystem;
import org.springframwork.context.annotation.ComponentScan;
import org.springframwork.context.annotation.Configuration;

@Configuration
@ComponentScan
public class CDPlayerConfig{}
```

写完这两个类之后, 可以直接利用JUnit测试SgtPeppers是否能被发现:

```java
package soundsystem;
import static org.junit.Assert.*;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframwork.beans.factory.annotation.Autowired;
import org.springframwork.test.context.ContextConfiguration;
import org.springframwork.test.context.junit4.SprintJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=CDPlayerConfig.class)
  public class CDPlayerTest {
    @Autowire
    private CompactDisc cd;

    @Test
    public void cdShouldNotBeNull() {
      assertNutNull(cd);
    }
  }
```

@ContextConfiguration注解告诉spring需要在CDPlayerConfig类中加载配置, 而CDPlayerConfig包含了@ComponentScan注解,所以相应的上下文中就应该包含CompactDisc这个bean。
于是这个测试为了证明这一点,声明了一个CompactDisc私有变量, 并用@Autowire注解, 而这个变量在测试时不是null, 那么说明上下文中存在CompactDisc的bean, 被自动装配到了该测试类中的cd变量中。

由于CDPlayerConfig加了@ComponentScan注解, soundsystem包以及其子包中包含@Comonent注解的类都会被自动创建成bean。

### component的命名
在刚才的例子中,SgtPepper会被命名为类名首字母改成小写的形式:sgtPepper, 如果需要对这个组件命名, 则需要在@Component中加入参数:

```java
@Component("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc {
    ...
}
```

使用java依赖注入规范的Named注解也能达到相同的效果, Spring支持这个注解, 其效果与前者大致相同:

```java
import javax.inject.Named;
@Named("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc {
    ...
}
```

@ComponentScan注解可以使用参数指定扫描的基包:

```java
@Configuration
@ComponentScan("soundsystem")
public calss CDPlayerConfig {}
```

或者使用basePackages属性使其更加清晰:

```java
@Configuration
@ComponentScan(basePackages="soundsystem")
public calss CDPlayerConfig {}
```

也能同时定义多个包:

```java
@Configuration
@ComponentScan(basePackages={"soundsystem", "video"})
public calss CDPlayerConfig {}
```

还能使用对应包中所包含的类确定需要扫描的包:


```java
@Configuration
@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})
public calss CDPlayerConfig {}
```

### 用注解使bean能被自动装配

```java
package soundsystem;
import org.springframwork.beans.factory.annotation.Autowired;
import org.springframwork.stereotype.Component;

@Component
public class CDPlayer implements MediaPlayer {
  private CompactDisc cd;

  @Autowired
  public CDPlayer(CompactDisc cd) {
    this.cd = cd;
  }

  public void play() {
    cd.play();
  }
}
```

setter方法也能被自动装配:

```java
@Autowired
public void setCompactDisc(CompactDisc cd) {
  this.cd = cd;
}
```

如果不存在匹配的bean, Spring会抛出异常, 如果要避免这个异常,可以将required属性设置为false:

```java
@Autowired(required=false)
public CDPlayer(CompactDisc cd) {
    this.cd = cd;
}
```

不过由于这种方式会导致相应的bean无法创建, 所以其引用有可能是null, 从而导致NullPointerException, 所以使用时需要小心。

也可以使用java依赖注入规范中的@Inject注解代替@Autowired, 一般情况下, 两者可以替代:

```java
package soundsystem;
import javax.injection.Named;
import javax.injection.Inject;

@Named
public class CDPlayer {
  ...

  @Inject
  public CDPlayer(CompactDisc cd) {
    this.cd = cd;
  }
}

```

### 验证自动配置

下面用单元测试进行验证

```java
package soundsystem;
import static org.junit.Assert.*;
import org.junit.Rule;
import org.junit.Test;
import org.junit.contrib.java.lang.system.StandardOutputStreamLog;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=CDPlayerConfig.class)

public class CDPlayerTest {

  @Rule
  public final StandardOutputStreamLog log = new StandardOutputStreamLog();

  @Autowired
  private MediaPlayer player;

  @Autowired
  private CompactDisc cd;

  @Test
  public void cdShouldNotBeNull() {
    assertNotNull(cd);
  }

  @Test
  public void play() {
    player.play();
    assertEquals(
        "Playing Sgt. Pepper's Lonely Hearts Club Band" +
        " by The Beatles\n",
        log.getLog());
  }

}

```

播放器和cd都得到了自动装配, 所以调用player.play()会得到播放cd时的输出: Playing Sgt. Pepper's Lonely Hearts Club Band by The Beatles

## 使用xml进行配置
由于类型安全、易于重构等各种优势, 使用Java进行配置, 但是由于事先已经存在了很多用xml写的配置, 了解如何xml进行配置也是有必要的。

最简单的xml配置文件如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context">

  <!-- 在这里编写细节 -->

</beans>
```

### 声明一个简单的bean

声明一个CompactDisc bean的方式如下:

```xml
<bean class="soundsystem.SgtPeppers" />
```

由于没有显式声明其id, bean会根据完整类名被赋予一个id。在这里, 这个bean的id是soundsystem.SgtPeppers#0。 #0代表这个bean的编号, 如果重新声明了另外的SgtPeppers, 且没有显式的赋予id, 那么他的id就是soundsystem.SgtPeppers#1。

也可以指定id:

```xml
<bean id="compactDisc" class="soundsystem.SgtPeppers" />
```

### 指定构造器参数

如果构造器需要参数, 如CDplayer类, 有两种选择指定参数:

- <constructor-arg>元素
- 使用Spring3.0引入的c-namespace

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer">
 <constructor-arg ref="compactDisc" />
</bean>
```

如果使用c-namespace, 需要声明它的schema:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:c="http://www.springframework.org/schema/c"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context">

  ...

</beans>
```

语法如下:

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer"
  c:cd-ref="compactDisc" />
```

c:cd-ref="compactDisc":

- c: c-namespace前缀
- cd: 构造器参数名
- -ref: 表示引用
- compactDisc: 注入的bean的id

也可以指定参数在参数表中的位置:

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer"
  c:_0-ref="compactDisc" />
```

由于只有一个参数, 所以不需要指定数字:

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer"
  c:_-ref="compactDisc" />
```

### 使用字面常量
语法如下:

```xml
<constructor-arg value="aaaa" />
<constructor-arg><null /></constructor-arg>
```

以上分别为一个字符串和一个null

list:

```xml
<constructor-arg>
  <list>
    <value>aaaa</value>
    ...
  </list>
<constructor-arg>
```

也可以声明引用

```xml
<constructor-arg>
  <list>
    <ref bean="..."/>
    ...
  </list>
<constructor-arg>
```

也可以用通用的方式声明set

### 设置属性

假设CDPlayer是这样的:

```java
public class CDPlayer impolements MediaPlayer {
    private CompactDisc compactDisc;
    @Autowired
    public void setCompactDisc(CompactDisc compactDisc) {
        this.compactDisc = compactDisc;
    }
    public void play() {
        compactDisc.play();
    }
}
```

spring可以直接创建它:

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer" />
```

我们可以在xml中设置compactDisc属性:

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer">
  <property name="compactDisc" ref="compactDisc" />
</bean>
```

与c-namespace对应, spring也提供了p-namespace用来设置属性, 其xml文件如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context">


</beans>
```

假设有个BlankDisc类:

```java
package soundsystem;
import java.util.List;
import soundsystem.CompactDisc;

public class BlankDisc implements CompactDisc {
    
    private String title;
    private String artist;
    private List<String> tracks;

    public void setTitle(String title) {
        this.title = title;
    }

    public void set Artist(String artist) {
        this.title = title;
    }

    public setTracks(List<String> tracks) {
        this.tracks = tracks;
    }

    public void play() {...}
}
```

那么可以按照这种方式设置属性:

```xml
<bean id="compactDisc" class="soundsystem.BlankDisc"
  p:title="Sgt. Pepper's Lonely Hearts Club Band"
  p:artist="The Beatles">
  <property name="tracks">
    <list>
      <value>track1</value>
      ...
    </list>
  </property>
</bean>
```

由于p-namespace不能声明list,所以还是需要使用property来指定list

spring提供了util-namespace, 用来直接创建一个list

当然首先还是要先声明一个schema, 添加一行xmlns:util="http://www.springframework.org/schema/util", 在schemaLocation中添加两行,http://www.springframework.org/schema/util和http://www.springframework.org/schema/spring-util.xsd

```xml
<util:list id="trackList">
  <value>track1</value>
  ...
</util:list>
```

那么前面的bean就可以这么写:

```xml
<bean id="compactDisc" class="soundsystem.BlankDisc"
  p:title="Sgt. Pepper's Lonely Hearts Club Band"
  p:artist="The Beatles"
  p:tracks-ref="trackList"/>
```

### 导入

可以使用@Import和@ImportResource注解进行导入class和xml:

```java
@Configuration
@Import(CDPlayerConfig.class)
@Import("classpath:cd-config.xml")
public class SoundSystemConfig {
}
```

也可以在xml文件里进行导入:

```xml
<beans
  ...>
  <bean calss="soundsystem.CDConfig" />
  <import resource="cdplayer-config.xml" />
</beans>
```

以上展示了xml导入的两种方式, 导入java配置和xml配置
