# Spring in Action 笔记4 - Web
## Spring Web的大致架构
下图为一个请求在Spring MVC中途径的各个部件:

![](http://7xt2lb.com2.z0.glb.clouddn.com/05fig01.jpg)

所有的请求都会首先经过DispatcherServlet, DispatcherServlet做的工作主要是接受请求并进行合适的分派工作。

DispatcherServlet首先会根据Handler mapping来决定将请求提交给哪个控制器进行处理, 提交请求给相应控制器之后, 得到一个模型和一个逻辑的视图名称(view name), 然后DispatcherServlet会把view name提交给ViewResolver来决定加载哪个模板, 最后模板会根据model生成最终的视图, 作为响应的内容。

## 开始使用Spring MVC
### 配置DispatcherServlet
Dispatcher是Spring MVC的中心, 也是网络请求在这个框架中第一次到达的位置。按照传统的Servlet开发方式, DispatcherServlet会被放在web.xml文件中, 但是Servlet 3开始支持注解了, 所以可以利用注解在java文件中配置。

```java
package spittr.config;
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class SpittrWebApInitializer extends AbstractAnnotationConfigDispatcherServletInitializer{
  
  @Override
  protected String[] getServletMappings() {
    return new String[] { "/" };
  }
  
  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class<?>[] { RootConfig.class };
  }
  
  @Override
  protected Class<?>[] getServletConfigClasses() {
    return new Class<?>[] { WebConfig.class };
  }
}
```

首先它继承了一个名字长的反人类的抽象类AbstractAnnotationConfigDispatcherServletInitializer, 这个AbstractAnnotationConfigDispatcherServletInitializer似乎还有点历史..在Servlet 3.0的环境中, 容器会先寻找javax.servlet.ServletContainerInitializer接口的实现, 如果找到了, 那么就会使用它们来配置容器。Spring提供了它的实现, SpringServletContainerInitializer, 用来查找所有实现了WebApplicationInitializer, 并将他们进行装饰, 用来配置。在3.2中, Spring引入了WebApplicationInitializer的一个抽象实现, AbstractAnnotationConfigDispatcherServletInitializer, 我们只需要继承这个抽象类, 实现三个必要的方法即可创建一个WebApplicationInitializer, 然后被SpringServletContainerInitializer发现并部署于Servlet中。

第一个方法getServletMappings()指定了需要处理的路径, 后面两个方法则指定了两个上下文的配置: ContextLoaderListener和DispatcherServlet下的上下文。后者一般用来加载控制器, 渲染器, 路由控制等组件, 而前者一般用于加载中间层和数据层(data-tier)的组件用来驱动后端工作。

如果要在DispatcherServlet中启用Spring MVC, 则只需在WebConfig上加入两个注解:

```
package spittr.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableWebMvc
public class WebConfig {
}
```

当然做这些还是不够的, 还需要:

- 启用组件扫描
- 配置页面渲染器
- 处理静态资源的请求

所以WebConfig的代码修改如下:

```java
package spittr.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
@Component
public class WebConfig extends WebMvcConfigurerAdapter {
  @Bean
  public ViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    resolver.setExposeContextBeansAsAttributes(true);
    return resolver;
  }
  
  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }
}
```

WebConfig继承了WebMvcConfigurerAdapter并覆盖了configureDefaultServletHandling方法, 开启了默认的请求处理方式, 那么就能直接找到静态资源。

下面是RootConfig:

```java
package spittr.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@ComponentScan(basePackages = {"spitter"},
  excludeFilters = {
    @Filter(type = FilterType.ANNOTATION, value = EnableWebMvc.class)
  })
public class RootConfig {
}
```

可以看出, RootConfig对spitter包下都开启了组件扫描, 但是把带有@EnableWebMvc注解的类排除在外。

### 一个最简单的控制器

```java
package spittr.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import static org.springframework.web.bind.annotation.RequestMethod.*;

@Controller
public class HomeController {

  @RequestMapping(value = "/", method = GET)
  public String home() {
    return "home";
  }
}
```

@Controller是一个基于@Component的符合注解, 所以在ComponentScan开启的情况下, HomeController能被实例化成一个bean。@RequestMapping注解很容易看出, 方法直接返回了字符串"home", 根据之前ViewResolver的配置, 将使用/WEB-INF/views/home.jsp作为网页模板。

直接把@RequestMapping放在类前面也是可以的:

```java
package spittr.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import static org.springframework.web.bind.annotation.RequestMethod.*;

@Controller
@RequestMapping("/")
public class HomeController {

  @RequestMapping(method = GET)
  public String home() {
    return "home";
  }
}
```

如果需要同时处理多个路径, 也可以传入多个字符串:

```java
@Controller
@RequestMapping({"/", "/homepage"})
public class HomeController {
  ...
}
```

这样的话, 请求/, /homepage都会得到home.jsp对应的网页。

src/main/rsources/views/home.jsp(我是用的是maven):

```xml
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
<head>
    <title>Spittr</title>
    <link rel="stylesheet"
          type="text/css"
          href="<c:url value="/resources/style.css" />" >
</head>
<body>
<h1>Welcome to Spittr</h1>
<a href="<c:url value="/spittles" />">Spittles</a> |
<a href="<c:url value="/spitter/register" />">Register</a>
</body>
</html>
```

### 向视图传递数据

书中使用的例子为类似微博的应用, 每条微博被称为Spittle:

```java
package spittr;
  import org.apache.commons.lang3.builder.EqualsBuilder;
  import org.apache.commons.lang3.builder.HashCodeBuilder;

  import java.util.Date;

public class Spittle {
  private final Long id;
  private final String message;
  private final Date time;
  private Double latitude;
  private Double longitude;

  public Spittle(String message, Date time) {
    this(message, time, null, null);
  }

  public Spittle(
    String message, Date time, Double longitude, Double latitude) {
    this.id = null;
    this.message = message;
    this.time = time;
    this.longitude = longitude;
    this.latitude = latitude;
  }

  public long getId() {
    return id;
  }

  public String getMessage() {
    return message;
  }

  public Date getTime() {
    return time;
  }

  public Double getLongitude() {
    return longitude;
  }

  public Double getLatitude() {
    return latitude;
  }

  @Override
  public boolean equals(Object that) {
    return EqualsBuilder.reflectionEquals(this, that, "id", "time");
  }

  @Override
  public int hashCode() {
    return HashCodeBuilder.reflectionHashCode(this, "id", "time");
  }
}
```

这里的hashCode和equals方法使用了Apache Commons Lang来实现, (还有这么好用的东西...), 书上居然没有导入就能直接用, 是因为它的代码运行在Apache的JVM上么?

为此我们创建一个Spittle仓库接口:

```java
package spittr.data;

import spittr.Spittle;

import java.util.List;

public interface SpittleRepository {
  List<Spittle> findSpittles(long max, int count);
}
```

出于解耦的目的, 需要写这样的接口, 而不是与具体数据库相关的代码, 他的实现, 会在后面的章节提到(汗...)

可以用类似这样的语法调用SpittleRepository的findSpittles()方法:

```java
List<Spittle> recent = SpittleRepository.findSpittles(Long.Max_VALUE, 20);
```

```java
package spittr.web;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import spittr.data.SpittleRepository;

/**
 * Created by Zhranklin on 16/4/24.
 */
@Controller
@RequestMapping("/spittles")
public class SpittleController {
  
  private SpittleRepository spittleRepository;
  
  @Autowired
  public SpittleController(SpittleRepository spittleRepository) {
    this.spittleRepository = spittleRepository;
  }
  
  @RequestMapping(method = RequestMethod.GET)
  public String spittles(Model model) {
    model.addAttribute(spittleRepository.findSpittles(Long.MAX_VALUE, 20));
    return "spittles";
  }
  
  
}
```

spittles()方法接受一个Model, 可以往里面添加属性, 在模板实例化的时候就可以使用到里面的值了。Model的addAttribute方法可以只接受一个对象(如上所示), 那么他的key(根据对象的类型)就是spitleList, 也可以显式的指定key:

```java
model.addAttribute("spittleList", spittleRepository.findSpittles(Long.MAX_VALUE, 20));
```

将传入的参数改成Map也是可以的, 这时候直接使用Map的put()方法, 可以达到相同的效果:

```java
RequestMapping(method = RequestMethod.GET)
public String spittles(Map model) {
  model.put("spittleList", spittleRepository.findSpittles(Long.MAX_VALUE, 20));
}
```

甚至不使用Model作为参数, 直接返回List<Spittle>也是可以的:

```java
RequestMapping(method = RequestMethod.GET)
public String spittles() {
  return spittleRepository.findSpittles(Long.MAX_VALUE, 20);
}
```

返回的对象会自动放进Model里, 在传递给view, view的逻辑名字则隐含于请求的路径中(/spittles)。

## 接受请求的输入
请求中的数据有三类:

- 查询参数(?a=x&b=y...)
- 表单参数
- 路径中的变量

### 请求参数
前面两类是一样的, 都是请求参数, 使用@RequestParam可以进行提取:

```java
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(
    @RequestParam(value="max",
                  defaultValue=MAX_LONG_AS_STRING) long max,
    @RequestParam(value="count", defaultValue="20") int count) {
  return spittleRepository.findSpittles(max, count);
}
```

defaultValue用来设置默认值。

### 路径中的变量
```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
    @PathVariable("spittleId") long spittleId,
    Model model) {
  model.addAttribute(spittleRepository.findOne(spittleId));
  return "spittle";
}
```

路径中由一个{spittleId}作为占位符, 提取出来的long会作为函数的参数(spittleId), 而@PathVariable则说明了这是路径中的变量spittleId。

## 处理表单
### 处理post请求
该书的例子使用Spitter代表用户, 但是非常尴尬的是, 书上连Spitter的代码都没有, 找了半天找不到...大概是包含了firstName, lastName, username, password四个域以及相关的getter, setter方法。

```java
package spittr.web;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import spittr.Spitter;

public class SpitterController {
  private SpitterRepository spitterRepository;
  @RequestMapping(value = "/register", method = RequestMethod.GET)
  public String processRegistraction() {
    return "registerForm";
  }

  @RequestMapping(value = "/register", method = RequestMethod.POST)
  public String processRegistraction(Spitter spitter) {
    spitterRepository.save(spitter);
    return "redirect:/spitter/" + spitter.getUsername();
  }
}
```

照例, SpitterRepository(不是SpittleRepository)的代码是没有的, 后面章节会有。processRegistraction()这个方法神奇的地方在于, 他直接使用Spitter作为传入参数。Spring会将spitter中的域与名字相同的POST请求参数关联, 再进行传递, 也就是说, POST请求会包含firstName, lastName, username, password四个参数。

关于返回的字符串, InternalResourceViewResolver会根据"rediredt:"这个前缀, 执行重定向的操作。Spring MVC同样支持"forward:"前缀, 后面指定view name来加载相应的视图。


### 验证表单
Spring 3.0开始, Spring MVC实现了Java Validation API, 只要类路径里有表单验证的库, 如Hibernate Validation, 就可以直接使用。Java Validation API支持以下注解:

注解|功能
---|---
@AssertFalse|The annotated element must be a Boolean type and be false.
@AssertTrue|The annotated element must be a Boolean type and be true.
@DecimalMax|The annotated element must be a number whose value is less than or equal to a given BigDecimalString value.
@DecimalMin|The annotated element must be a number whose value is greater than or equal to a given BigDecimalString value.
@Digits|The annotated element must be a number whose value has a specified number of digits.
@Future|The value of the annotated element must be a date in the future.
@Max|The annotated element must be a number whose value is less than or equal to a given value.
@Min|The annotated element must be a number whose value is greater than or equal to a given value.
@NotNull|The value of the annotated element must not be null.
@Null|The value of the annotated element must be null.
@Past|The value of the annotated element must be a date in the past.
@Pattern|The value of the annotated element must match a given regular expression.
@Size|The value of the annotated element must be either a String, a collection, or an array whose length fits within the given range.

![](http://7xt2lb.com2.z0.glb.clouddn.com/161fig01.jpg)

```java
package spittr.web;

import org.springframework.validation.Errors;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import spittr.Spitter;

public class SpitterController {
  private SpitterRepository spitterRepository;
  @RequestMapping(value = "/register", method = RequestMethod.GET)
  public String processRegistraction() {
    return "registerForm";
  }

  @RequestMapping(value = "/register", method = RequestMethod.POST)
  public String processRegistraction(
    @Validated Spitter spitter,
    Errors errors) {
    if (errors.hasErrors()) {
      return "registerForm";
    }
    spitterRepository.save(spitter);
    return "redirect:/spitter/" + spitter.getUsername();
  }
}
```
