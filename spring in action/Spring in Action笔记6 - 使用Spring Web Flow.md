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

这里逻辑视图名就是welcome, 当然也能显示声明逻辑视图名:

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
<var id="com" class="springinaction pizza domain Customer" name="customer" />
```

作为行为状态的一部分或者视图状态的入口, 可能需要使用evaluate(这里使用了SpEL):

```xml
<evaluate result="viewScope.toppingList" expression="T(com.springinaction.pizza.domain.Topping).asList()" />
```

可以使用set来设置变量的值:

```xlm
<set name="..." value="..." />
```

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
大致的流程就是先进行用户识别, 选择一个或多个披萨添加到订单中, 提供支付信息, 然后提交订单。

![08fig02alt](http://7xt2lb.com1.z0.glb.clouddn.com/08fig02_alt.jpg)

那么, xml的定义代码如下:

![ch08ex01-0.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/ch08ex01-0.jpg)
![ch08ex01-1.jpg](http://7xt2lb.com1.z0.glb.clouddn.com/ch08ex01-1.jpg)


