# Spring in Action笔记7 - Spring Security
## 简介
Spring Security是一个基于Spring AOP和Servlet中的Filter实现的安全框架。它的前身是Acegi Security, 它在此基础上增加了更简单的xml配置的支持, 以及java配置, 以及后来新添加的SpEL的支持。

Spring Security被分成了11个模块, 一般使用的话, 至少需要Core和Configuration(对xml和java配置的支持)这两个模块。而平时需要对web进行保护的话, 还需要加入web模块。这个反正maven上找一找就有了。

### 过滤Web请求
Spring Security借助一系列Servlet Filter来实现各种安全性功能, 不过我们并不需要在web.xml文件中添加多个Fiter, 只需要添加一个`DelegatingFilterProxy`即可, 它会自动将各种工作委托给其他的Spring应用上下文中的`javax.servlet.Filter`实现类。

如果使用web.xml来配置, 则添加下列代码:
```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
        org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
</filter>
```

也可以借助`WebApplicationInitializer`(在Spring Web里面有提到)来配置`DelegatingFilterProxy`, 直接继承`AbstractSecurityWebApplicationInitializer`:

```java
package spitter.config;
import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;
public class SecurityWebInitializer extends AbstractSecurityWebApplicationInitializer {}
```

由于`AbstractSecurityWebApplicationInitializer`实现了`WebApplicationInitializer`, Spring会发现他, 并在容器中注册`DelegatingFilterProxy`。

不管使用哪种方式, 他都会拦截发往应用中的请求, 并委托给一个id为springSecurityFilterChain(连接了一系列的Filter来实现相关的安全功能)的bean。

### 编写简单的安全性配置
下面是Java配置的minimal:

![248fig01\_alt](http://7xt2lb.com1.z0.glb.clouddn.com/248fig01_alt.jpg)

顾名思义, `@EnableWebSecurity`注解是用来开启Spring Security的, 然而Spring Security的配置必须写在实现了`WebSecurityConfigurer`的bean中。简单起见, 也可以直接扩展`WebSecurityConfigurerAdapter`(实现了`WebSecurityConfigurer`)。如果我们使用的Web应用正好用的是Spring MVC(也就是本书的例子), 则需要使用`@EnableWebMvcSecurity`注解来替代`@EnableWebSecurity`, 两者的区别在于`@EnableWebMvcSecurity`还配置了Spring MVC的参数解析器, 以及一个用来自动(在网页模板中)添加一个隐藏的`<input>`并放置一个跨站请求伪造(CSRF)token。

继承了`WebSecurityConfigurerAdapter`之后, 所有设定都使用了默认, 导致所有配置都讲应用严格锁定, 所以有必要覆盖相关方法来修改配置, `WebSecurityConfigurerAdapter`有三个方法来进行配置:

方法                                    | 描述
---                                     | ---
configure(WebSecurity)                  | 配置Filter链
configure(HttpSecurity)                 | 如何拦截请求
configure(AuthenticationManagerBuilder) | 配置user-detail服务

吐槽一下, 这本书翻译得很好, 但是这一节里他把所有的覆盖(override)都写成了重载, 让人有点汗颜...

## 选择查询用户详细信息的服务
### 基于内存的用户存储
在内存中存储信息其实最灵活也最简单, 适合开发初期使用, 如前所示, 我们需要做的就是覆盖configure(`AuthenticationManagerBuilder`)方法:

![251fig01\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/251fig01_alt.jpg)

方法中使用了builder模式进行相关设置, 添加了两个用户以及各自的用户名, 密码, 身份(.roles()方法)等信息。withUser()方法返回的是`UserDetailManagerConfigurer.UserDetailsBuilder`, 我们可以调用他的方法(如password())来设置这个用户的属性, `UserDetailManagerConfigurer.UserDetailsBuilder`的所有方法:

Module                                        | Description
---                                           | ---
accountExpired(boolean)                       | Defines if the account is expired or not
accountLocked(boolean)                        | Defines if the account is locked or not
and()                                         | Used for chaining configuration
authorities(GrantedAuthority...)              | Specifies one or more authorities to grant to the user
authorities(List<? extends GrantedAuthority>) | Specifies one or more authorities to grant to the user
authorities(String...)                        | Specifies one or more authorities to grant to the user
credentialsExpired(boolean)                   | Defines if the credentials are expired or not
disabled(boolean)                             | Defines if the account is disabled or not
password(String)                              | Specifies the user's password
roles(String...)                              | Specifies one or more roles to assign to the user

roles()方法其实是authorities()方法的简写形式, roles()方法会在给定的值前面添加一个"ROLE\_"前缀, 也就是说, 刚才那个代码等价于:

```java
auth
.inMemoryAuthentication()
  .withUser("user").password("password")
                              .authorities("ROLE_USER").and()
  .withUser("admin").password("password")
                              .authorities("ROLE_USER", "ROLE_ADMIN");
```

### 基于数据库表进行认证
使用JDBC时的minimal:
```java
@Autowired
DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
      .dataSource(dataSource);
}
```

当然了很明显能看出来这里还需要配置一个DataSource, 而这部分功能要求数据库包含某些村冲数据的表。下面的一端代码源于Spring Security内部:

```java
public static final String DEF_USERS_BY_USERNAME_QUERY =
        "select username,password,enabled " +
        "from users " +
        "where username = ?";
public static final String DEF_AUTHORITIES_BY_USERNAME_QUERY =
        "select username,authority " +
        "from authorities " +
        "where username = ?";
public static final String DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY =
        "select g.id, g.group_name, ga.authority " +
        "from groups g, group_members gm, group_authorities ga " +
        "where gm.username = ? " +
        "and g.id = ga.group_id " +
        "and g.id = gm.group_id";
```

如果原有的数据库表的定义与上面不一致, 那么可以按照下面的方式配置自己的查询:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
      .dataSource(dataSource)
      .usersByUsernameQuery(
        "select username, password, true " +
        "from Spitter where username=?")
      .authoritiesByUsernameQuery(
        "select username, 'ROLE_USER' from Spitter where username=?");
}
```

反正能够具体自己定制到什么程度, 查相关的API就能知道了。

然后看一下怎么使用转码之后的密码, 总不能用明文存储密码吧。在刚才的方法里, 需要指定一个密码转码器:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
      .dataSource(dataSource)
      .usersByUsernameQuery(
        "select username, password, true " +
        "from Spitter where username=?")
      .authoritiesByUsernameQuery(
        "select username, 'ROLE_USER' from Spitter where username=?")
      .passwordEncoder(new StandardPasswordEncoder("53cr3t"));
}
```

passwordEncoder()方法接受的是Spring Security中PasswordEncoder接口的任意实现, 可以使用Spring Security加密模块中的三个实现`BCryptPasswordEncoder`, `NoOpPasswordEncoder`, `StandardPasswordEncoder`, 也可以自己对这个接口进行实现。

### 基于LDAP进行认证
书上讲了这部分, 但实际并不知道这是什么, 所以先跳过了..

## 拦截请求
### 如何拦截请求
前面我们的`SecurityConfig`类继承了`WebSecurityConfigurerAdapter`, 但是没有覆盖configure(HttpSecurity), 那么默认就会拦截所有的请求(强制他们都进行权限验证), 然而有些公开的页面是不需要收到这么严格的保护的, 所以需要进行一些修改:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
      .antMatchers("/spitter/me").authenticated()
      .antMatchers(HttpMethod.POST, "/spittles").authenticated()
      .anyRequest().permitAll();
}
```

这里我们对`/spitter/me`要求用户验证, 对`/spittles`的POST请求也要求用户验证, 而其他的请求都是直接放行的。antMatchers()方法接受Ant风格的通配符, 而且可以接受多个路径:

```java
.antMatchers("/spitters/**", "/spittles/mine").authenticated();
```

也可以使用regexMatchers()方法, 然后指定正则表达式:

```java
.regexMatchers("/spitters/.*").authenticated();
```

选择完路径后, 还可以使用authenticated()和permitAll()方法指定如何保护路径。前者要求已经登录, 如果没有登录, Spring Security的Filter则会捕获这个请求, 并将用户重定向到达登录页面, 后者则允许没有任何安全限制, 下面是各种用来保护路径的方法:

Method                     | What it does
---                        | ---
access(String)             | Allows access if the given SpEL expression evaluates to true
anonymous()                | Allows access to anonymous users
authenticated()            | Allows access to authenticated users
denyAll()                  | Denies access unconditionally
fullyAuthenticated()       | Allows access if the user is fully authenticated (not remembered)
hasAnyAuthority(String...) | Allows access if the user has any of the given authorities
hasAnyRole(String...)      | Allows access if the user has any of the given roles
hasAuthority(String)       | Allows access if the user has the given authority
hasIpAddress(String)       | Allows access if the request comes from the given IP address
hasRole(String)            | Allows access if the user has the given role
not()                      | Negates the effect of any of the other access methods
permitAll()                | Allows access unconditionally
rememberMe()               | Allows access for users who are authenticated via remember-me

### 使用SpEL进行安全保护
如果需要对路径保护做精确的控制, 有时候必须使用Spring表达式(前面表格里的access()方法), 例如:

```java
.antMatchers("/spitter/me").access("hasRoles('ROLE_SPITTER')");
```

好吧一开始看着这个hasRoles()方法有点云里雾里, 其实是Spring Security对SpEL做了扩展, 它支持以下表达式:

Security expression       | What it evaluates to
---                       | ---
authentication            | The user��s authentication object
denyAll                   | Always evaluates to false
hasAnyRole(list of roles) | True if the user has any of the given roles
hasRole(role)             | True if the user has the given role
hasIpAddress(IP address)  | True if the request comes from the given IP address
isAnonymous()             | True if the user is anonymous
isAuthenticated()         | True if the user is authenticated
isFullyAuthenticated()    | True if the user is fully authenticated (not authenticated with remember-me)
isRememberMe()            | True if the user was authenticated via remember-me
permitAll                 | Always evaluates to true
principal                 | The user's principal object

例如我们需要同时满足有ROLE\_SPITTER和来自指定的IP:

```java
.antMatchers("/spitter/me").access("hasRole('ROLE_SPITTER') and hasIpAddress('192.168.1.2')")
```

### 强制使用https
在传入的HttpSecurity对象中, 除了authorizeRequests()方法外, 还有一个requiresChannel()方法, 调用完之后我们就能指定相应的路径所需要强制使用的通道:

![265fig01\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/265fig01_alt.jpg)

此时对"/spitter/form"的请求对会被自动重定向到https上。

而有些页面则不需要使用https来传送, 那么就应该用requiresInsecure()来代替requiresSequre():

  ```java
  .antMatchers("/").requiresInsecure();
  ```

### 防止跨站请求伪造
如果我正好已经登陆了这个网站, 然后点击了其他网站的一个表单的提交按钮:

```html
<form method="POST" action="http://www.spittr.com/spittles">
  <input type="hidden" name="message" value="I'm stupid!" />
  <input type="submit" value="Click here to win a new car!" />
</form>
```

于是我干了一件很蠢的事情: 在自己的微博上发了一条"I'm stupid!", 这就是跨站请求伪造(cross-site request forgery, CSRF), 可见这是一件很危险的事情。

Spring Security(3.2+)默认启用CSRF防护, 所以如果直接编写网页表单, 则会出现问题(无法通过CSRF防护这一关)。Spring Security使用一个同步token来实现CSRF防护, 他会拦截状态变化的请求, 并检查CSRF token, 如果请求中不包含CSRF token或者token与服务器端的不匹配, 则请求失败, 并抛出`CsrfException`。

所以在编写应用的时候, 表单中必须在一个"\_csrf"域中提交token, 且与服务器端中的一致。

Spring Security对此进行了简化, 使用Thymeleaf的时候, 只要使用th:action属性代替action属性, 那么就会自动生成一个"\_csrf"隐藏域:

```html
<form method="POST" th:action="@{/spittles}">
   ...
</form>
```

如果使用JSP, 则手动添加一个input:

```html
<input type="hidden"
       name="${_csrf.parameterName}"
       value="${_csrf.token}" />
```

当然一定要把这个功能关掉也是可以的, 调用.csrf().disable()即可。

## 认证用户
### 登录页
如果使用的是一开始那个最简单的配置, 那么就会默认得到一个登录页, 但是一旦覆盖了configure(HttpSecurity)方法, 这个默认就失效了。找回这个功能也很容易, 只要在这个方法中调用formLogin()即可:

![267fig01\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/267fig01_alt.jpg)

但是往往我们会嫌他丑, 然后换成自己的登录页, 这时候, 除了编写自己的登录页之外, 还需要将登录页的路径设置一下, 也就是在.formLogin()后面加上.loginPage("/login"), 书上好像忘记讲了..

表单部分的编写很简单, 只需要包含一个username输入域, 一个password输入域即可。

### HTTP Basic认证
很多时候, 并不仅仅是web浏览器会登录到服务器, 比如如果要编写RESTful API的话, 用表单来提示登录就不是很适合了。这时可以使用HTTP Basic认证, 在configure(HttpSecurity)方法中调用一次httpBasic()方法即可, 还可以通过realmName()方法指定域:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .formLogin()
      .loginPage("/login")
    .and()
    .httpBasic()
      .realmName("Spittr")
    .and()
  ...
}
```

### 启用Remember-me功能
多数网站都会利用cookie来实现某段时间内免登陆的功能, 这里只要调用一次rememberMe()方法即可完成配置, 然后通过相关方法来设置有效时间以及指定相应的私钥:

```java
@Override
  protected void configure(HttpSecurity http) throws Exception {
    http
      .formLogin()
        .loginPage("/login")
      .and()
      .rememberMe()
        .tokenValiditySeconds(2419200)
        .key("spittrKey")
  ...
}
```

默认的私钥是SpringSecured, 这里将它设置为spitterKey, 是他专门用于Spittr应用。然后在表单中添加一个复选框就能让用户选择是否"记住我":

```java
<input id="remember_me" name="remember-me" type="checkbox"/>
<label for="remember_me" class="inline">Remember me</label>
```

### 退出
Spring Security默认实现了退出登录的功能, 路径为"/logout", 所以只要有这样的链接即可实现该功能, 默认情况下登出之后, 会重定向到"/login?logout", 从而允许用户再次登录。而如果需要对这些设置进行更改, 同样是在configure(HttpSecurity)方法里:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .formLogin()
      .loginPage("/login")
    .and()
    .logout()
      .logoutSuccessUrl("/")
      .logoutUrl("/signout")
  ...
}
```

## 对视图进行保护
在网页中, 我们希望显示与安全限制相关的信息, 比如用户的个人信息之类的, Spring Security本身提供了一个JSP标签库, 而Thymeleaf则通过特定的方言提供了与Spring Security的集成。

### JSP中使用Spring Security
Spring Security的JSP标签库很小, 只有三个标签(汗...):

JSP tag                        | What it does
---                            | ---
`<security:accesscontrollist>` | Conditionally renders its body content if the user is granted authorities by an access control list
`<security:authentication>`    | Renders details about the current authentication
`<security:authorize>`         | Conditionally renders its body content if the user is granted certain authorities or if a SpEL expression evaluates to true

好吧其实功能还是齐全的, 可以指定各种属性来规定一些细节, 为了使用这个标签库, 需要先声明它:

```html
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```

最先想到的就应该是显示用户的认证信息了, 这也是能做到的最简单的事情:

```html
Hello <security:authentication property="principal.username" />!
```

这种"你好某某某!"的格式我们常常在论坛或者各种应用的网页上能看到, property用来说明要显示用户的哪个属性, 可用的书性取决于用户认证的方式, 一些通用的属性如下:

Authentication property | Description
---                     | ---
authorities             | A collection of GrantedAuthority objects that represent the privileges granted to the user
credentials             | The credentials that were used to verify the principal (commonly, this is the user's password)
details                 | Additional information about the authentication (IP address, certificate serial number, session ID, and so on)
principal               | The user's principal

如果想要把得到的东西复制给一个变量, 增加一个var属性, 指明为变量的名字即可, 这个变量的作用域默认是在这个页面内的, 如果要在其他作用域内创建它, 如请求或者会话作用域, 则可以通过scope属性声明:

```html
<security:authentication property="principal.username"
        var="loginId" scope="request" />
```

其次就是条件性渲染某些内容了(比如回复可见之类的), 刚才那个表格里又说到, `<security:authorize>`标签能够根据用户被授予的权限有条件的渲染内容, 例如, 如果规定只对有ROLE\_SPITTER角色的用户, 才显示添加新Spitter记录的表单:

![273fig01\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/273fig01_alt.jpg)

和前面HttpSecurity对象的路径设置中的access方法是一样的, 接受某个SpEL表达式, 其为真的时候才会将整个标签内的内容显示出来。SpEL能使用的内容参照前面的表格即可。

`<security:authorize>`还可以指定一个url属性, 只有当前的url满足一定的模式的时候, 整个内容才会被渲染出来。

### Thymeleaf的Spring Security方言
在Thymeleaf中, 是使用相关的属性来完成权限控制的功能的, 为了达到这个框架本身的初衷嘛, 保证能直接在网页中显示出来。

Attribute          | What it does
---                | ---
sec:authentication | Renders properties of the authentication object. Similar to Spring Security��s <sec:authentication/> JSP tag.
sec:authorize      | Conditionally renders content based on evaluation of an expression. Similar to Spring Security's <sec:authorize/> JSP tag.
sec:authorize-acl  | Conditionally renders content based on evaluation of an expression. Similar to Spring Security's <sec:accesscontrollist/> JSP tag.
sec:authorize-expr | An alias for the sec:authorize attribute.
sec:authorize-url  | Conditionally renders content based on evaluation of security rules associated with a given URL path. Similar to Spring Security's <sec:authorize/> JSP tag when using the url attribute.

好吧虽然多了几个标签, 但实际功能和JSP一模一样..

为了使用这个Spring Security方言, 我们需要保证Thymeleaf Extras Spring Security在应用的classpath下, 然后在Spring中使用`SpringTemplateEngine`来注册`SpringSecurityDialect`:

![276fig01\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/276fig01_alt.jpg)

然后在Thymeleaf中声明他的命名空间:

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec=
          "http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
  ...
</html>
```

东西和JSP的那一部分是一模一样的, 所以也没什么好说的了。(终于写完了累死我了....)
