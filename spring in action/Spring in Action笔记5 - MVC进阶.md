# Spring in Action 笔记5 - MVC进阶
中间还有一章关于网页渲染的, Thymeleaf的官方教程中有关于在Spring中使用的方法, 个人觉得直接看那个就好了, 所以暂时放入todolist。

## 修改Spring MVC配置
前面的章节中, 我们通过继承Spring MVC自带的抽象类AbstractAnnotationConfigDispatcherServletInitializer方便的进行配置(还是很想吐槽这么长的名字..), 但是这种情况只适用于对DispatcherServlet和ContextLoaderListener进行简单配置, 而且只是用java而不是xml配置的情况。如果需要对DispatcherServlet进行额外的配置, 或者说使用的容器在Servlet 3.0之前, 那么就不得不使用传统的web.xml进行配置了。

### 定制DispatcherServlet配置
之前的SpittrWebApInitializer继承了AbstractAnnotationConfigDispatcherServletInitializer, 只需要实现三个方法, 实际上还是有很多方法可以覆盖的, 比如customizeRegistration()。当AbstractAnnotationConfigDispatcherServletInitializer在Servlet中注册了DispatcherServlet后, 它会调用customizeRegistration()方法, 并将ServletRegistration.Dynamic作为参数传入。故可以在这里对DispatcherServlet进行额外的配置, 如需要启用multipart请求, 则需要像这样覆盖customizeRegistration方法:

```java
@Override
protected void customizeRegistration(Dynamic registration) {
  registration.setMultipartConfig(new MultipartConfigElement("/tmp/spittr/uploads"));
}
```

利用ServletRegistration.Dynamic还可以做很多事情, 如设定初始化参数等等(其实不懂..)。

### 添加额外的Servlet和过滤器
下面就是通过继承WebApplicationInitializer向容器注册一个Servlet(名为MyServlet)的方法:

![](http://7xt2lb.com2.z0.glb.clouddn.com/196fig01_alt.jpg)

类似的, 也可以注册监听器、过滤器:

![](http://7xt2lb.com2.z0.glb.clouddn.com/197fig01_alt.jpg)

如果需要将过滤器映射到DispatcherServlet, 只需覆盖AbstractAnnotationConfigDispatcherServletInitializer的getServletFilters()方法即可:

```java
@Override
protected Filter[] getServletFilters() {
  return new Filter[] {new MyFilter()};
}
```

也可以编写web.xml(这个好烦..不写了)

## 处理multipart表单
### 配置multipart resolver
Dispatcher并没有提供逻辑上的支持, 故需要使用resolver的方式提供专门的支持。有一种已经实现了的方案, StandardServletMultipartResolver, 直接声明成Bean即可:

```java
@Bean
public MultipartResolver multipartResolver() throws IOException {
  return StandardServletMultipartResolver();
}
```

不过这仅仅是提供了一个resolver而已, 真正相关的配置, 比如临时文件路径, 文件大小限制等, 还得写在servlet的配置里(这样才能使不同的servlet可以有不同的配置嘛)。实际的配置需要写在DispatcherServlet的配置里或者web.xml文件中。

如果DispatcherServlet的配置是在WebApplicationInitializer的实现里:

```java
DispatcherServlet ds = new DispatcherServlet();
Dynamic registration = context.addServlet("appServlet", ds);
registration.addMapping("/");
registration.setMultipartConfig(
    new MultipartConfigElement("/tmp/spittr/uploads"));
```

如果继承了AbstractAnnotationConfigDispatcherServletInitializer或者AbstractDispatcherServletInitializer, 则覆盖customizeRegistration方法:

```java
@Override
protected void customizeRegistration(Dynamic registration) {
  registration.setMultipartConfig(new MultipartConfigElement("/tmp/spittr/uploads"));
}
```

MultipartConfigElement的构造器还可以接受另外的参数, 分别表示:

- 每个文件的最大大小
- 整个multipart的最大总大小
- 不将文件写入磁盘的最大大小(超过这个限制, 就会把文件写入临时目录中)

如果用web.xml配置:

```xml
<servlet>
  <servlet-name>appServlet</servlet-name>
  <servlet-class>
    org.springframework.web.servlet.DispatcherServlet
  </servlet-class>
  <load-on-startup>1</load-on-startup>
  <multipart-config>
    <location>/tmp/spittr/uploads</location>
    <max-file-size>2097152</max-file-size>
    <max-request-size>4194304</max-request-size>
  </multipart-config>
</servlet>
```

### 处理multipart请求
multipart表单一般如下:

```html
<form method="POST" th:object="${spitter}"
      enctype="multipart/form-data">

...
  <label>Profile Picture</label>:
    <input type="file"
           name="profilePicture"
           accept="image/jpeg,image/png,image/gif" /><br/>

...

</form>
```

有了之前的配置, 处理表单就比较简单了:

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    @RequestPart("profilePicture") byte[] profilePicture,
    @Valid Spitter spitter,
    Errors errors) {
  ...
}
```

#### 接收MultipartFile
直接处理上传的文件简单, 但是有限制, Spring提供了一种另外的方式: MultipartFile来获得更丰富的对象以处理multipart数据。MultipartFile是一个接口:

```java
package org.springframework.web.multipart;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;

public interface MultipartFile {
  String getName();
  String getOriginalFilename();
  String getContentType();
  boolean isEmpty();
  long getSize();
  byte[] getBytes() throws IOException;
  InputStream getInputStream() throws IOException;
  void transferTo(File dest) throws IOException;
}
```

MultipartFile提供了更多的方法, 甚至还有输入流, 文件移动等。

直接使用javax.servlet.http.Part也是可以的, Part接口定义如下:

```java
package javax.servlet.http;
import java.io.*;
import java.util.*;

public interface Part {
  public InputStream getInputStream() throws IOException;
  public String getContentType();
  public String getName();
  public String getSubmittedFileName();
  public long getSize();
  public void write(String fileName) throws IOException;
  public void delete() throws IOException;
  public String getHeader(String name);
  public Collection<String> getHeaders(String name);
  public Collection<String> getHeaderNames();
}
```

和MultipartFile接口定义很接近

processRegistration则可以这么写:

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    @RequestPart("profilePicture") Part profilePicture,
    @Valid Spitter spitter,
    Errors errors) {
  ...
}
```

只有当需要使用MultipartFile的时候才需要配置StandardServletMultipartResolver, 所以如果使用Part, 是不需要StandardServletMultipartResolver的。

## 处理异常
Spring提供了很多方式将异常转换成响应:

- 某个Spring exception自动映射到标准的HTTP状态码
- 一个使用了@ResponseStatus注解的异常会被映射到相应的状态码
- 一个方法可以使用@ExceptionHandler来处理异常

### 使用SpringException进行映射
这是最简单的方式:

Spring exception|HTTP status code
---|---
BindException|400 - Bad Request
ConversionNotSupportedException|500 - Internal Server Error
HttpMediaTypeNotAcceptableException|406 - Not Acceptable
HttpMediaTypeNotSupportedException|415 - Unsupported Media Type
HttpMessageNotReadableException|400 - Bad Request
HttpMessageNotWritableException|500 - Internal Server Error
HttpRequestMethodNotSupportedException|405 - Method Not Allowed
MethodArgumentNotValidException|400 - Bad Request
MissingServletRequestParameterException|400 - Bad Request
MissingServletRequestPartException|400 - Bad Request
NoSuchRequestHandlingMethodException|404 - Not Found
TypeMismatchException|400 - Bad Request

在需要的时候抛出这些异常即可, 当然自己继承这些异常也是很好的方式

### 使用@ExceptionHandler注解
例如:

```java
@ExceptionHandler(DuplicateSpittleException.class)
  public String handleDuplicateSpittle() {
    return "error/duplicate";
  }
```

这样就能自行处理, 返回相应的逻辑视图名称, 可以用它来作用于同一个Controller内的方法。

## 通知(advise) controller
有些场景下需要对控制器进行切面编程, 比如刚才的@ExceptionHandler, 如果需要对所有的Controller起作用, 则需要AOP的帮助。在Spring 3.2, 引入了controller advice, 只需加一个@ControllerAdvice即可表示一个controller advice, 它可以有一个或几个下面的方法:

- @ExceptionHandler注解的方法
- @InitBinder注解的方法
- @ModelAttribute注解的方法

在@ControllerAdvice注解的类中, 上面三种方法会应用于整个项目中所有控制器中的@RequestMapping注解的方法。Controller本身附带了@Component注解, 故其会在组件扫描中被发现。

用法与前面相同, 不赘述。

## 在重定向中携带数据
最简单的方式就是按照路由的规则插入数据, 比如起来请求路径中包含数据"redirect:/spitter/{spitterId}"。

在重定向中携带数据有两种方式:

- 使用URL模板传递路径变量或查询参数
- 在flash attribute中发送数据

### 使用URL模板进行重定向
直接使用+进行字符串连接是一件很危险的事情, 因为其中包含了转义的过程, 一不小心很可能会被攻击。所以一种比较好的方式是使用URL模板:

`return "redirect:/spitter/{username}";`

username是Model里面的属性:

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    Spitter spitter, Model model) {
  spitterRepository.save(spitter);
  model.addAttribute("username", spitter.getUsername());
  return "redirect:/spitter/{username}";
}
```

如果添加的属性在重定向指令中没有相应的占位符, 则会自动转换成查询参数:

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    Spitter spitter, Model model) {
  spitterRepository.save(spitter);
  model.addAttribute("username", spitter.getUsername());
  model.addAttribute("spitterId", spitter.getId());
  return "redirect:/spitter/{username}";
```

如果username是zhranklin, id是35, 那么重定向地址就是/spitter/zhranklin?spitterId=35

### 使用flash属性
如果需要传递 比如一个Spitter, 用上面的方法就行不通了。将对象放进会话是一个不错的选择, 但是传递完毕之后, 就需要自己清理这个Spitter对象了, Spring并不认为这是你的责任, 所以提供了flash attribute的机制(就是一闪而过的意思), 顾名思义, flash attribute 会保持数据, 直到下一个请求到达。

Spring 3.1引入了一个Model的扩展接口, RedirectAttributes:

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    Spitter spitter, RedirectAttributes model) {
  spitterRepository.save(spitter);
  model.addAttribute("username", spitter.getUsername());
  model.addFlashAttribute("spitter", spitter);
  return "redirect:/spitter/{username}";
}
```

在同一个会话中, 当下一个请求到达时, 就能够获得spitter属性。

处理响应请求的方法则可以改写:

```java
@RequestMapping(value="/{username}", method=GET)
public String showSpitterProfile(
        @PathVariable String username, Model model) {
  if (!model.containsAttribute("spitter")) {
    model.addAttribute(
        spitterRepository.findByUsername(username));
  }
  return "profile";
}
```

flash attributes的原理图:

![](http://7xt2lb.com2.z0.glb.clouddn.com/07fig02.jpg)

