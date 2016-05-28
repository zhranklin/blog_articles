# Spring in Action¿¿7 - Spring Security
## ¿¿
Spring Security¿¿¿¿¿Spring AOP¿Servlet¿¿Filter¿¿¿¿¿¿¿¿¿¿¿¿¿Acegi Security, ¿¿¿¿¿¿¿¿¿¿¿¿¿xml¿¿¿¿¿, ¿¿java¿¿, ¿¿¿¿¿¿¿¿SpEL¿¿¿¿

Spring Security¿¿¿¿11¿¿¿, ¿¿¿¿¿¿, ¿¿¿¿Core¿Configuration(¿xml¿java¿¿¿¿¿)¿¿¿¿¿¿¿¿¿¿¿¿web¿¿¿¿¿¿, ¿¿¿¿¿web¿¿¿¿¿¿¿maven¿¿¿¿¿¿¿¿

### ¿¿Web¿¿
Spring Security¿¿¿¿¿Servlet Filter¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿web.xml¿¿¿¿¿¿¿Fiter, ¿¿¿¿¿¿¿`DelegatingFilterProxy`¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿Spring¿¿¿¿¿¿¿`javax.servlet.Filter`¿¿¿¿

¿¿¿¿web.xml¿¿¿, ¿¿¿¿¿¿¿:
```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
        org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
</filter>
```

¿¿¿¿¿`WebApplicationInitializer`(¿Spring Web¿¿¿¿¿)¿¿¿`DelegatingFilterProxy`, ¿¿¿¿`AbstractSecurityWebApplicationInitializer`:

```java
package spitter.config;
import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;
public class SecurityWebInitializer extends AbstractSecurityWebApplicationInitializer {}
```

¿¿`AbstractSecurityWebApplicationInitializer`¿¿¿`WebApplicationInitializer`, Spring¿¿¿¿, ¿¿¿¿¿¿¿`DelegatingFilterProxy`¿

¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿id¿springSecurityFilterChain(¿¿¿¿¿¿¿Filter¿¿¿¿¿¿¿¿¿¿)¿bean¿

### ¿¿¿¿¿¿¿¿¿¿
¿¿¿Java¿¿¿minimal:

![248fig01\_alt](http://7xt2lb.com1.z0.glb.clouddn.com/248fig01_alt.jpg)

¿¿¿¿, `@EnableWebSecurity`¿¿¿¿¿¿¿Spring Security¿, ¿¿Spring Security¿¿¿¿¿¿¿¿¿¿`WebSecurityConfigurer`¿bean¿¿¿¿¿¿, ¿¿¿¿¿¿¿`WebSecurityConfigurerAdapter`(¿¿¿`WebSecurityConfigurer`)¿¿¿¿¿¿¿¿Web¿¿¿¿¿¿¿Spring MVC(¿¿¿¿¿¿¿¿), ¿¿¿¿¿`@EnableWebMvcSecurity`¿¿¿¿¿`@EnableWebSecurity`, ¿¿¿¿¿¿¿`@EnableWebMvcSecurity`¿¿¿¿Spring MVC¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿(¿¿¿¿¿¿)¿¿¿¿¿¿¿`<input>`¿¿¿¿¿¿¿¿¿¿¿(CSRF)token¿

¿¿¿`WebSecurityConfigurerAdapter`¿¿, ¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, `WebSecurityConfigurerAdapter`¿¿¿¿¿¿¿¿¿¿:

¿¿                                    | ¿¿
---                                     | ---
configure(WebSecurity)                  | ¿¿Filter¿
configure(HttpSecurity)                 | ¿¿¿¿¿¿
configure(AuthenticationManagerBuilder) | ¿¿user-detail¿¿

¿¿¿¿, ¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿(override)¿¿¿¿¿¿, ¿¿¿¿¿¿...

## ¿¿¿¿¿¿¿¿¿¿¿¿¿
### ¿¿¿¿¿¿¿¿¿
¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿, ¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿configure(`AuthenticationManagerBuilder`)¿¿:

![251fig01\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/251fig01_alt.jpg)

¿¿¿¿¿¿builder¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿, ¿¿(.roles()¿¿)¿¿¿¿withUser()¿¿¿¿¿¿`UserDetailManagerConfigurer.UserDetailsBuilder`, ¿¿¿¿¿¿¿¿¿¿(¿password())¿¿¿¿¿¿¿¿¿¿, `UserDetailManagerConfigurer.UserDetailsBuilder`¿¿¿¿¿:

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

roles()¿¿¿¿¿authorities()¿¿¿¿¿¿¿, roles()¿¿¿¿¿¿¿¿¿¿¿¿¿¿"ROLE\_"¿¿, ¿¿¿¿, ¿¿¿¿¿¿¿¿¿:

```java
auth
.inMemoryAuthentication()
  .withUser("user").password("password")
                              .authorities("ROLE_USER").and()
  .withUser("admin").password("password")
                              .authorities("ROLE_USER", "ROLE_ADMIN");
```

### ¿¿¿¿¿¿¿¿¿¿
¿¿JDBC¿¿minimal:
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

¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿DataSource, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿Spring Security¿¿:

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

¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿:

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

¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿API¿¿¿¿¿¿

¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿:

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

passwordEncoder()¿¿¿¿¿¿Spring Security¿PasswordEncoder¿¿¿¿¿¿¿, ¿¿¿¿Spring Security¿¿¿¿¿¿¿¿¿¿`BCryptPasswordEncoder`, `NoOpPasswordEncoder`, `StandardPasswordEncoder`, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿

### ¿¿LDAP¿¿¿¿
¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿..

## ¿¿¿¿
### ¿¿¿¿¿¿
¿¿¿¿¿`SecurityConfig`¿¿¿¿`WebSecurityConfigurerAdapter`, ¿¿¿¿¿¿configure(HttpSecurity), ¿¿¿¿¿¿¿¿¿¿¿¿¿(¿¿¿¿¿¿¿¿¿¿¿), ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿:

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

¿¿¿¿¿`/spitter/me`¿¿¿¿¿¿, ¿`/spittles`¿POST¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿antMatchers()¿¿¿¿Ant¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿:

```java
.antMatchers("/spitters/**", "/spittles/mine").authenticated();
```

¿¿¿¿¿regexMatchers()¿¿, ¿¿¿¿¿¿¿¿¿:

```java
.regexMatchers("/spitters/.*").authenticated();
```

¿¿¿¿¿¿, ¿¿¿¿¿authenticated()¿permitAll()¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿, Spring Security¿Filter¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿¿¿¿¿¿¿¿:

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

### ¿¿SpEL¿¿¿¿¿¿
¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿, ¿¿¿¿¿¿¿Spring¿¿¿(¿¿¿¿¿¿access()¿¿), ¿¿:
