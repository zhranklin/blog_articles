# Spring in Action笔记6 - 使用Spring Web Flow
五一放假玩了几天之后, 状态就再也没有转换回来, 不过最近突然看到京东上开始卖Spring in Action的中文版了, 就又重拾了认真看书做笔记的兴趣。(好了不说废话了...)

## Spring Web Flow
有时候, web应用程序需要控制网页浏览的流程, 引导用户一步一步地访问应用, 比如淘宝/亚马逊等购物网站的下单流程。这时候需要使用到Spring Web Flow了, 他是Spring MVC的扩展, 其机制在我个人看来, 和状态机的模型比较相像。

## 在Spring中配置Web Flow
### 命名空间
Spring Web Flow 基于Spring MVC, 所有请求会先经过DispatcherServlet, 所以需要在应用上下文配置一些bean来处理流程请求。由于目前还不支持Java配置(刚查了官方文档, 2.4.x已经支持Java Config了hhh), 我们需要在xml文件中进行配置。国际惯例, 需要一个命名空间:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:webflow="http://www.springframework.org/schema/webflow-config"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
           http://www.springframework.org/schema/webflow-config
           http://www.springframework.org/schema/webflow-config/spring-webflow-config-2.4.xsd">

    <!-- Setup Web Flow here -->

</beans>
```

### 流程执行器
flow executor负责创建和执行流程, 定义方式如下:

```xml
<flow:flow-executor id="flowExecutor" />
```

### 配置流程注册表
flow registry用来加载流程定义, 使得流程执行器能够使用它们。定义如下:

```xml
<flow:flow-registry id="flowRegistry" base-path="/WEB-INF/flows">
  <flow:flow-location-pattern value="*-flow.xml" />
</flow:flow-registry>
```

不难看出, 这个流程注册表会查找/WEB-INF/flows目录下以"-flow.xml"结尾的文件作为流程的定义。得到的流程的id是相对于base-path的路径, 例如/WEB-INF/flows/**order**/order-flow.xml的id就是order。也可以去除base-path, 使用flow:flow-location显式声明流程定义文件的位置:

```xml
<flow:flow-registry id="flowRegistry">
  <flow:flow-location value="/WEB-INF/flows/springpizza.xml" />
</flow:flow-registry>
```

也可以指定id:

```xml
<flow:flow-registry id="flowRegistry">
  <flow:flow-location id="pizza"
    value="/WEB-INF/flows/springpizza.xml" />
</flow:flow-registry>
```

### 处理流程的请求
要使用Spring Web Flow处理请求, 就要帮助DispatcherServlet将请求发送给Spring Web Flow, 需要使用的是FlowHandlerMapping:

```xml
<bean class="org.springframework.webflow.mvc.servlet.FlowHandlerMapping">
  <property name="flowRegistry" ref="flowRegistry" />
</bean>
```

FlowHandlerMapping装配了流程注册表, 于是相应的流程定义就能生效。然而还需要FlowHandlerAdapter来响应请求(等同于Spring MVC的控制器):

```xml
<bean class="org.springframework.webflow.mvc.FlowHandlerAdapter">
  <property name="flowExecutor" ref="flowExecutor">
</bean>
```

然后FlowHandlerAdapter就会利用前面定义的flowExecutor来响应请求了。

## 流程的组件
在Spring Web Flow中, 流程有三个要素: 状态、转移和流程数据, 基本上能看做带消息传递的状态机。

### 状态
状态有五种, 分别是行为(Action), 决策(Decision), 结束(End), 子流程(Subflow), 视图(View)。(神tm字母顺序...)

#### 视图状态
视图状态用来想用户展示信息:

```xml
<view-state id="welcome" />
```

这里逻辑视图名就是welcome, 当然也能显式声明逻辑视图名:

```xml
<view-state id="welcome" view="greeting"/>
```

逻辑视图名就成了greeting。

#### 行为状态
行为状态是程序自身执行的任务, 一般会触发bean的一些方法并根据其调用结果转移到另一个状态。

```xml
<action-state id="saveOrder">
  <evaluate expression="pizzaFlowActions.saveOrder(order)" />
  <transition to="thankYou">
</action-state>
```

expression中使用的是SpEL表达式, 表示调用id为pizzaFlowActions的bean并调用他的saveOrder()方法。

#### 决策状态
相当于if语句, 接受一个Boolean类型的表达式, 然后根据结果执行不同的转换:

```xml
<decision-state id="checkDeliveryArea">
  <if test="pizzaFlowActions.checkDeliveryArea(customer.zipCode)"
      then="addCustomer"
      else="deliveryWarning" />
</decision-state>
```

#### 子流程状态
我们可以定义子流程, 供其他流程调用, 类似于一个方法调用另一个方法:

```xml
<subflow-state id="order" subflow="pizza/order">
  <input name="order" value="order" />
  <transaction on="orderCreation" to="payment"/>
</subflow>
```

这里`<input>`元素用于传递订单对象作为子流程的输入。如果子流程结束时的`<end-state>`状态id为orderCreated, 那么流程将会转移到payment的状态。

#### 结束状态
所有的流程都要结束, `<end-state>`元素制定了流程的结束, 一般声明为:

```xml
<end-state id="customerReady" />
```

流程结束之后会发生的事情:
- 如果是子流程的`<end-state>`, 那么会从调用它的`<subflow-state>`继续执行, `<end-state>`的id将会作为一个事件, 来触发`<subflow-state>`开始的转移。
- 如果`<end-state>`设定了view属性, 相应的视图将会被渲染, 如果让添加`externalRedirect:`前缀, 则重定向到流程外部的页面, 如果添加`flowRedirect:`前缀, 则重定向到另一个流程中。
- 如果不满足上面两个条件, 流程只是结束而已, 而浏览器会加载流程的基本URL地址。

### 转移
有了转移, 状态才会变化, 转移用`<transition>`表示, 作为`<action-state> <view-state> <subflow-state>`的子元素。它的写法如下:

```xml
<transition to="customerReady" />
```

也可以用on指定触发转移的事件:


```xml
<transition on="phoneEntered" to="lookupCustomer" />
```

也可以用抛出的异常来触发转移, 只需使用on-exception来制定相应的Exception类即可。

#### 全局转移
可以定义全局转移, 来全局地监听事件:

```xml
<global-transitions>
  <transition on="cancel" to="endState" />
</global-transitions>
```

这样定义之后, 只要触发了cancel事件, 就会跳转到endState

### 流程数据
最简单的方式就是声明变量

```xml
<var id="com" class="springinaction.pizza.domain.Customer" name="customer" />
```

作为行为状态的一部分或者视图状态的入口, 可能需要使用evaluate(这里使用了SpEL):

```xml
<evaluate result="viewScope.toppingList" expression="T(com.springinaction.pizza.domain.Topping).asList()" />
```

可以使用set来设置变量的值:

```xml
<set name="..." value="..." />
```

evaluate与set之间的区别基本上就在于set直接使用类的默认构造器(指定class属性)或使用java中的表达式(指定value属性), 而evaluate则使用SpEL。evaluate中, 也可以不指定result, 也就是直接把expression属性中的内容执行一遍

#### 流程数据的作用域
Spring Web Flow的作用域有五个:

范围        | 生命作用域和可见性
---         | ---
Convesation | 最高层级的流程开始时创建, 被最高层级的流程以及其所有子流程共享
Flow        | 只有在创建它的流程中可见
Request     | 请求进入流程是创建, 流程返回时销毁
Flash       | 流程开始时创建, 流程结束时销毁, 视图状态渲染后, 他也会被清除
View        | 只在视图状态内可见

## 实例: 披萨流程
### 大致结构
大致的流程就是先进行用户识别, 选择一个或多个披萨添加到订单中, 提供支付信息, 然后提交订单。

![08fig02alt](http://7xt2lb.com1.z0.glb.clouddn.com/08fig02_alt.jpg)

那么, xml的定义代码如下:

![ch08ex01-0.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/ch08ex01-0.jpg)
![ch08ex01-1.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/ch08ex01-1.jpg)

在这个基本流程中, 第一件事就是定义了一个com.springinaction.pizza.domain.Order, Order类的定义:

Order是一个比较简单的类, 定义了一个订单的各种属性以及相应的getter, setter方法。

```java
package com.springinaction.pizza.domain;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;
public class Order implements Serializable {
   private static final long serialVersionUID = 1L;
   private Customer customer;
   private List<Pizza> pizzas;
   private Payment payment;
      public Order() {
      pizzas = new ArrayList<Pizza>();
      customer = new Customer();
   }
   public Customer getCustomer() {
      return customer;
   }
   public void setCustomer(Customer customer) {
      this.customer = customer;
   }
   public List<Pizza> getPizzas() {
      return pizzas;
   }
   public void setPizzas(List<Pizza> pizzas) {
      this.pizzas = pizzas;
   }
   public void addPizza(Pizza pizza) {
      pizzas.add(pizza);
   }
   public float getTotal() {
      return 0.0f;
   }
   public Payment getPayment() {
      return payment;
   }
   public void setPayment(Payment payment) {
      this.payment = payment;
   }
}
```

通常情况下, flow里面定义的第一个状态就是流程访问的第一个状态, 也可以将flow元素的start-state属性设置为某个状态的id。

流程变量order在前三个状态中进行了填充, 第一个流程(identifyCustomer)使用output元素填充order的customer属性, 而后面两个(buildOrder和takePayment)则将order同时作为输入和输出, 于是能够直接对order进行填充操作。

到了第四个流程, 就可以对订单进行保存, 这里直接使用`<action-state>`, 利用里面的`<evaluate>`元素调用pizzaFlowActions.saveOrder()方法。最后转移到thankCustomer显示相应的页面。

thankCustomer是一个`<view-state>`根据前面的说明, 没有显式声明逻辑视图名称的时候, 直接使用id(thankCustomer)作为逻辑视图名称, 实际使用的是`/WEB-INF/flows/pizza/thankCustomer.jsp`这个JSP文件。

![232fig01alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/232fig01_alt.jpg)

Spring Web Flow 为视图提供了一个flowExecutionUrl变量, 包含了流程的URL, 并指定\_eventId参数以便触发finished事件(不过我好像没看到代码里有这个, 我认为应该在包含的那个`<transition>`元素里添加一个on="finished"属性), 是的流程达到结束状态。

流程结束后, 由于没有定义下一步做什么, 流程将会重新从identifuCustomer状态开始准备接受另一个披萨订单。

### 收集顾客信息
先来定义一下identifyCustomer这个子流程, 如图:

![08fig03\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/08fig03_alt.jpg)

逻辑比较简单 不解释, 下面是这个子流程的xml文件代码:

  
![ch08ex04-0.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/ch08ex04-0.jpg)
![ch08ex04-1.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/ch08ex04-1.jpg)

#### 各个部分的JSP网页代码

![235fig01\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/235fig01_alt.jpg)

关注一个hidden输入和一个提交按钮。那个hidden输入保存了一个flowExecutionKey, 当进入视图状态时, 服务器端的流程执行就会暂停并等待用户, 这个key用来指定哪个执行(等待)中的流程需要被恢复。而按钮的名字的"\_eventId"用来告诉Spring Web Flow接下来要触发的事件, 在这里, 会触发phoneEntered事件(进而转移到lookupCustomer状态)。

注册部分的网页也差不多, 关注一下一个hidden input和两个submit input的作用即可:

```html
<html xmlns:c="http://java.sun.com/jsp/jstl/core"
     xmlns:jsp="http://java.sun.com/JSP/Page"
     xmlns:spring="http://www.springframework.org/tags"
     xmlns:form="http://www.springframework.org/tags/form">
  <jsp:output omit-xml-declaration="yes"/>
  <jsp:directive.page contentType="text/html;charset=UTF-8" />
  <head><title>Spizza</title></head>
  <body>
    <h2>Customer Registration</h2>
    <form:form commandName="customer">
      <input type="hidden" name="_flowExecutionKey"
             value="${flowExecutionKey}"/>
      <b>Phone number: </b><form:input path="phoneNumber"/><br/>
      <b>Name: </b><form:input path="name"/><br/>
      <b>Address: </b><form:input path="address"/><br/>
      <b>City: </b><form:input path="city"/><br/>
      <b>State: </b><form:input path="state"/><br/>
      <b>Zip Code: </b><form:input path="zipCode"/><br/>
      <input type="submit" name="_eventId_submit"
             value="Submit" />
      <input type="submit" name="_eventId_cancel"
             value="Cancel" />
    </form:form>
    </body>
</html>
```

还有个显示区域无法送达的JSP, 不贴了。

### 构建订单
![08fig04.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/08fig04.jpg)

![238fig01\_alt.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/238fig01_alt.jpg)

这个xml代码的其他部分原理和前面都差不多, 不同点就在于有一个`<input>`元素, 而且require属性被设置成了true, 这会从接受主流程中的对象(在主流程中定义这个子流程的时候有说明), 有点类似于Java中方法的签名。

后面的其他代码基本类似, 在解释起来就没意思了(懒), 所以不贴了。

## 保护Web流程
并不是所有人都可以有权限访问某个流程(隐私嘛), 所以Web流程应该受到保护, 下一章关于Spring Security的介绍后, 我们就可以用它来结合Web流程对用户的行为进行限制了。
