# Spring in Action 笔记3 - AOP
## 面向切面编程
AOP全称为aspect oriented programming, 一般译作面向切面编程。下面摘录了部分百度百科的词条:

> AOP、OOP在字面上虽然非常类似，但却是面向不同领域的两种设计思想。OOP（面向对象编程）针对业务处理过程的实体及其属性和行为进行抽象封装，以获得更加清晰高效的逻辑单元划分。

> 而AOP则是针对业务处理过程中的切面进行提取，它所面对的是处理过程中的某个步骤或阶段，以获得逻辑过程中各部分之间低耦合性的隔离效果。这两种设计思想在目标上有着本质的差异。

> 上面的陈述可能过于理论化，举个简单的例子，对于“雇员”这样一个业务实体进行封装，自然是OOP/OOD的任务，我们可以为其建立一个“Employee”类，并将“雇员”相关的属性和行为封装其中。而用AOP设计思想对“雇员”进行封装将无从谈起。

> 同样，对于“权限检查”这一动作片断进行划分，则是AOP的目标领域。而通过OOD/OOP对一个动作进行封装，则有点不伦不类。

> 换而言之，OOD/OOP面向名词领域，AOP面向动词领域。

一般我们使用aop用来分离业务逻辑和系统级服务(如事务管理、日志等), 进行有针对性的开发, 避免了在实现业务逻辑的代码中充满了权限检查、日之记录的代码, 也就是交叉关注点(cross-cutting):

![](http://7xt2lb.com2.z0.glb.clouddn.com/image/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%882.25.56.png)

横向为各种业务, 而竖向则为执行各种业务是需要附加的工作。在AOP中每一个交叉关注点, 则被称为一个切面(aspect)。

## 相关术语
### Advice 通知
Advice定义的是需要做的是什么工作, 也就是所有需要的功能, Spring支持五种通知:

- Before —The advice functionality takes place before the advised method is invoked.
- After —The advice functionality takes place after the advised method completes, regardless of the outcome.
- After-returning —The advice functionality takes place after the advised method successfully completes.
- After-throwing —The advice functionality takes place after the advised method throws an exception.
- Around —The advice wraps the advised method, providing some functionality before and after the advised method is invoked.

### Join Point 连接点
Join Point就是可以插入切面代码执行的点, 可以是一个方法的调用, 一个属性的更改, 一个异常的抛出。


### PointCut 切入点
指定应该在哪个方法方法执行时相关的通知, 因为我们并不想让某个通知在所有的动作执行时都出发, 而只是在某些特定的对象, 调用某些特定的方法时才触发。是连接点的具体化概念, 也就是以某种方式选择出的join point的集合。

### Aspect 切面
即通知和切入点的结合

### Introduction 引入
向现有的类添加方法

### Weaving 织入
将切面应用到对象来创建代理对象的过程。可以在编译期、类加载期、运行期执行织入的工作, Spring使用的是运行期使用动态代理进行织入的。

## Spring AOP
由于Spring是使用动态代理实现AOP的, 所以只支持与方法调用相关的连接点。在Spring中, advice是用标准的Java代码写的, 可以得到IDE的良好支持。切面将以动态代理封装的方式被入到由spring管理的bean中, 当bean的方法被调用时, 代理就会触发相应的切面的逻辑:

![](http://7xt2lb.com2.z0.glb.clouddn.com/image/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%883.12.13.png)

## 使用pointcut选择join point
### 编写pointcut
Spring使用了AspectJ的pointcut语法:

AspectJ designator|Description
---|---
args()|Limits join-point matches to the execution of methods whose arguments are instances of the given types
@args()|Limits join-point matches to the execution of methods whose arguments are annotated with the given annotation types
execution()|Matches join points that are method executions
this()|Limits join-point matches to those where the bean reference of the AOP proxy is of a given type
target()|Limits join-point matches to those where the target object is of a given type
@target()|Limits matching to join points where the class of the executing object has an annotation of the given type
within()|Limits matching to join points within certain types
@within()|Limits matching to join points within types that have the given annotation (the execution of methods declared in types with the given annotation when using Spring AOP)
@annotation|Limits join-point matches to those where the subject of the join point has the given annotation

假设我们有一个Performance接口, 可以用戏剧、电影、演唱会等方式来实现:

```java
package concert;

public interface Performance {
  public void perform();
}
```

如果我们要写一个在所有Performance.perform()方法调用时触发的切面, 可以写这样一个pointcut:

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%883.52.45.png)

如果需要把调用perform方法的类限定在某个包下, 可以使用within():

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%883.51.05.png)

在xml中使用and, or, not来替代&&, ||, !会更方便。

### 使用pointcut选择bean
如果需要限定特定的bean, 可以使用bean():

```
execution(* concert.Performance.perform()) and bean('woodstock')
```

这时只有woodstock这个bean调用perform时, 才会执行相应动作。

也可以使用!:


```
execution(* concert.Performance.perform()) and !bean('woodstock')
```

那么在id不是woodstock的情况下才会触发

### 使用注解创建aspect
#### 定义一个aspect
我们定义了一个观众类, 会在表演前后执行如找座位、关手机、鼓掌等动作, 如果出了问题, 还会退款(汗...):

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%884.02.27.png)

AspectJ使用以下五种注解来声明advice:

Annotation|Advice
---|---
@After|The advice method is called after the advised method returns or throws an exception.
@AfterReturning|The advice method is called after the advised method returns.
@AfterThrowing|The advice method is called after the advised method throws an exception.
@Around|The advice method wraps the advised method.
@Before|The advice method is called before the advised method is called

我们会发现四个advice使用的pointcut完全相同, 在spring中, 可以先定义一个pointcut, 再对其引用:

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%884.08.16.png)

我们使用@Aspect注解将Audience类中的一个空方法performance()定义成了一个pointcut, 可以直接在advice的定义中使用。

在Spring中一个aspect也可成为一个bean:

```java
@Bean
public Audience audience() {
    return new Audience();
}
```

如果仅做到这里而已, 这个aspect是不会生效的, 我们需要相应的机制对其进行处理, 是它起作用, 如果使用JavaConfig:

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%884.17.22.png)

xml:

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%884.17.38.png)

### Around Advice
使用@Around注解可以定义Around Advice, 用来定义触发一个方法前后需要做的事情:

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%884.21.17.png)

这个方法接受一个ProceddingJoinPoint对象, 在方法内部, 使用jp.proceed()方法表示pointcut方法的执行, proceed()方法也可以调用多次。注意, 这个方法中最好要包含对proceed()的调用, 否则advice就会将相应的域屏蔽了, 这不是一个好的设计。


### 对方法参数进行处理
有时候, 我们的Aspect需要使用到pointcut捕获的方法使用的参数, 比如如果需要对cd中每一首歌的播放次数进行统计:

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%884.21.46.png)

args(trackNumber)表示传给playTrack的参数也必须传给这个advice, 参数名trackNumber与pointcut的方法签名中的参数名相同, 同时与advice的方法中参数名相同

### 引入 Introduction
许多动态语言支持在运行时给类动态添加方法, 而如果需要在java中实现这个功能, 则需要使用动态代理, 反观Spring的通知(Advice), 实际上就是使用代理实现的, 即使用动态代理重新实现原有接口。在这一点上讲, 引入和通知是一样的, 只不过将其执行的功能放在了一个新的方法里而已。spring对原有类进行了装饰:


![](http://7xt2lb.com2.z0.glb.clouddn.com/image/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%884.50.09.png)

例如, 我们引入了一个Encoreable接口:

```java
package concert;

public interface Encoreable {
    void performEncore();
}
```

如果我们要为所有的Performance类实现Encoreable接口, 使用AOP的Introduction就可以在不侵入源代码的情况下进行扩展。先创建一个Aspect:

```java
package concert;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.DeclareParents;

@Aspect
public class EncoreableIntroducer {
  @DeclareParents(value="concert.Performance+",
                  defaultImpl=DefaultEncoreable.class)
  public static Encoreable encoreable;
}
```

使用@DeclareParents注解即可实现Introduction, 我们需要三样东西:

- value属性, 规定了需要引入新方法的类, +代表Performance的所有子类
- defaultImpl属性, 指定了由哪个类来实现这个接口
- 一个Encorable静态域, 说明需要注入的是Encorable接口

因为是Aspect, 所以需要将EncorableIntroducer声明为一个bean:

```xml
<bean class="concert.EncoreableIntroducer" />
```

## 使用xml声明aspect
如果我们没有源代码, 或者不希望对源代码加上各种注解, 则需要使用xml声明aspect, 以下为xml中的aop元素:

AOP configuration element|Purpose
---|---
&lt;aop:advisor&gt;|Defines an AOP advisor.
&lt;aop:after&gt;|Defines an AOP after advice (regardless of whether the advised method returns successfully).
&lt;aop:after-returning&gt;|Defines an AOP after-returning advice.
&lt;aop:after-throwing&gt;|Defines an AOP after-throwing advice.
&lt;aop:around&gt;|Defines an AOP around advice.
&lt;aop:aspect&gt;|Defines an aspect.
&lt;aop:aspectj-autoproxy&gt;|Enables annotation-driven aspects using @AspectJ.
&lt;aop:before&gt;|Defines an AOP before advice.
&lt;aop:config&gt;|The top-level AOP element. Most &lt;aop:\*&gt; elements must be contained within &lt;aop:config&gt;.
&lt;aop:declare-parents&gt;|Introduces additional interfaces to advised objects that are transparently implemented.
&lt;aop:pointcut&gt;|Defines a pointcut.

如果我们已有一个Audience类:

```java
package concert;

public class Audience {

  public void silenceCellPhones() {
    System.out.println("Silencing cell phones");
  }

  public void takeSeats() {
    System.out.println("Taking seats");
  }

  public void applause() {
    System.out.println("CLAP CLAP CLAP!!!");
  }

  public void demandRefund() {
    System.out.println("Demanding a refund");
  }

}
```

使用xml则可以在不改动代码的情况下将Audience声明为一个Aspet:

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%886.21.36.png)

同样也可以单独声明一个pointcut的引用:

![](http://7xt2lb.com2.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%20%E4%B8%8B%E5%8D%886.21.36.png)

### 声明Around Aspect
在前面使用JavaConfig声明Around Aspect时, 定义了一个叫watchPerformance的方法, 接受一个ProceedingJoinPoint参数, 这里假设Audience由一个一模一样的方法, 只是没有前面的@Around注解, 此时就可以在xml中进行配置:

![](http://7xt2lb.com2.z0.glb.clouddn.com/aroud%20aspect.png)

### 向advice传递参数
实际上与使用JavaConfig基本类似:

![](http://7xt2lb.com2.z0.glb.clouddn.com/advi_para.png)

![](http://7xt2lb.com2.z0.glb.clouddn.com/CC87743E-91FB-4651-BA31-44D20B3773AF.png)

### 引入
```xml
<aop:aspect>
  <aop:declare-parents
    types-matching="concert.Performance+"
    implement-interface="concert.Encoreable"
    default-impl="concert.DefaultEncoreable"
    />
</aop:aspect>
```

可以将default-impl换成bean的引用: delegate-ref="encoreableDelegate", 当然, 需要有这个bean的声明:

```xml
<bean id="encoreableDelegate"
      class="concert.DefaultEncoreable" />
```

## 注入AspectJ的切面
虽然Spring AOP已经满足许多应用的需求, 单毕竟这是一个比较弱的AOP解决方案, AspectJ提供了许多在Spring AOP中不可能实现的PointCut。
下面来实现对应Performance的CriticAspect, 表示评论家:

![](http://7xt2lb.com2.z0.glb.clouddn.com/aspectj_java.png)

CriticAspect中的CriticismEngine需要spring进行DI:

```java
package com.springinaction.springidol;
public class CriticismEngineImpl implements CriticismEngine {
  public CriticismEngineImpl() {}

  public String getCriticism() {
    int i = (int) (Math.random() * criticismPool.length);
    return criticismPool[i];
  }

  // injected
  private String[] criticismPool;
  public void setCriticismPool(String[] criticismPool) {
    this.criticismPool = criticismPool;
  }
}
```

```xml
<bean id="criticismEngine"
    class="com.springinaction.springidol.CriticismEngineImpl">
  <property name="criticisms">
    <list>
      <value>Worst performance ever!</value>
      <value>I laughed, I cried, then I realized I was at the
             wrong show.</value>
      <value>A must see show!</value>
    </list>
  </property>
</bean>
<bean class="com.springinaction.springidol.CriticAspect"
    factory-method="aspectOf">
  <property name="criticismEngine" ref="criticismEngine" />
</bean>
```

由于CriticAspect是AspectJ的Aspect, 所以其bean并不是由Spring管理, 所以对其进行依赖注入的方式会有点小小的不同。AspectJ定义的每一个aspect(这里指CriticAspect)都包含了一个静态方法aspectOf()来获取相应aspect实例的引用所以这里给相应的bean添加了factory-method属性, 设为"aspectOf"。
