# Thinking in Java 笔记 - (七)RTTI
## RTTI
RTTI RunTime Type Infomation, 运行期类型信息，分为两种，一种是编译时就已经知道的类型信息，另一种则是通过反射获得的。

## Class对象

### Class对象的获取
通过Class对象是与类相关的对象，可以通过调用对象的getClass()方法获得其所属类的Class对象:

```java
import java.util.Date;
Date d = new Date();
Class c = d.getClass();
```

如果不想为了获得Class对象创建实例，可以使用静态方法Class.forName()方法。它接受String作为参数，作为类名(必须带完整的包名):

```java
Class c = Class.forName("java.util.Date");
```

也可以直接使用Class类的字面常量，可以得到编译器检查，同时提高效率，因为不需要调用forName()方法:

```java
Class c = java.util.Date.class;
```

### Class对象的相关方法
Class对象可以用来获得类的相关信息，以下列举其中几个：

方法名|作用
---|---
getName()|获得类名(含包名)
getSimpelName()|获得不含包名的类名
getCanonicalName()|获得含包名的类名
isInterface()|是否为接口
getInterfaces()|获得其实现的所有接口的Class对象
getSuperclass()|获得其基类

Class的newInstance()方法获得该类的一个实例，但只能通过默认构造器生成。

### 泛型Class
为了保证编译器时的类型安全，Class可以使用泛型语法指定Class的类型参数。

```java
Class incClass = int.class;
Class<Integer> genericIntClass = int.class;
Class<Number> genericNumberClass = int.class;//无法编译
Class<? extends Number> genericNumberClass = int.class;
```

### 利用Class对象向下转型
使用Class的cast()方法可以进行向下转型：

```java
class building {}
class House extends Building {}
public class ClassCasts {
  public static void main(String[] args) {
    Building b = new House();
    Class<House> houseType = House.class;
    House h = houseType.cast(b);
    h = (House)b;//这种做法与上一行等价
  }
}
```

### 类型检查
在进行向下转型前，我们需要对其类型进行检查，这时就可以使用instanceof关键字:

```java
class Dog {}
public class Test {
  public static void main(String[] args) {
    Dog d = new Dog();
    if (d instanceof Dog) {
      System.out.println("d is instance of Dog");
    }
  }
}
```

但是instanceof关键字后面只能跟类名，而不能使用Class对象，如果需要利用Class对象进行动态的检查，则需要Class类的isInstance()方法。

```java
Dog d = new Dog();
Class dt = Dog.class;
if (dt.isInstance(d)) {
  System.out.println("d is instance of Dog");
}
```

## 反射
Class对象有getMethods()和getConstructors()方法，返回Method对象的数组和Constructor对象的数组，这两个类都提供了相应方法获取名字，输入参数，返回值等信息。

### 动态代理
Java的反射为代理提供了高级的支持，它可以动态的创建代理并动态的处理对所代理方法的调用。在动态代理商做的所有方法调用都会被重定向到一个调用处理器上(InvocationHandler)。

```java
//From Thingking in Java 4th edition
import java.lang.reflect.*;
class DynamicProxyHandler implements InvocationHandler {
  private Objected proxied;
  public DynamicProxyHandler(Object proxied) {
    this.proxied = proxied;
  }
  public Object invoke(Object proxy, Method method, Object[] args) 
    throws Throwable {
    Syetem.out.println("proxy: " + proxy.getClass() +
                       ", method: " + method + ", args" + args);
    return method.invoke(proxied, args);
  }
}

interface Interface {
  void doSometing();
  void somethingElse(String arg);
}

class RealObject implements Interface {
  public void doSomething() { System.out.println("doSomething"); }
  public void somethingElse(String arg) {
    System.out.println("somethingElse " + arg);
  }
}

public class SimpleDynamicProxy {
  public static void consumer(Interface iface) {
    iface.doSomething();
    iface.someThingElse();
  }
  public static void main(String[] args) {
    RealObject real = new RealObject();
    consumer(real);//原本的方式
    Interface proxy = (Interface)Proxy.newProxyInstance(
      Interface.class.getClassLoader(),
      new Class[]{ Interface.class },
      new DynamicProxyHandler(real));
    consumer(proxy);//经过代理
  }
}
```