# Thinking in Java 笔记 - (八)泛型
## 类型擦除
我们都知道，早期的Java为了“迁移兼容性”，其实现的泛型不得不做“类型擦除”的处理。也就是说，我们在编写泛型代码时，与类型参数相关的东西(<>内的东西)只有在编译器是已知的，所以它的作用只有为我们做编译期的检查，而实际上所有相关的引用都是Object(或泛型参数列表里面指定的基类)引用，在运行期，所有的类型信息都会被擦除。

所以，任何在运行期需要知道确切类型信息的操作都将无法操作。

```java
public class Erased<T> {
  private final int SIZE = 100;
  public static void f(Object arg) {
    if (arg instanceof T) {}//Error
    T vat = new T();//Error
    T[] array = enw T[SIZE];//Error
    T[] aray = (T)new Object[SIZE];//Unchecked warning
  }
}
```

由上例可知，运行期我们无法从类型参数T中获得任何类型信息，它只是在编译期保证类型安全的一种方式而已。

## 使用Class对象
如需在泛型类中创建相应类型的实例或数组(数组不是泛型)，我们可以传递一个Class对象。

```java
public class SomeClass<T> {
  private Class cc;
  public SomeClass(Class<T> kind) {
    cc = kind;
  } 
}
```

kind参数限定了传入Class对象的类型参数T，保证了相应操作创建的一定是T的对象，然后我们可以通过cc来创建T的对象或者数组(也可以干脆直接使用Object数组)：

```java
  T tobj = cc.newInstance();
  T[] arr = (T[])Array.newInstance(cc, 10);
```

当然直接像上面那样写是不行的，如果该类没有无参构造器或者无法创建该类的实例（抽象类或私有内部类等），那么第一行代码会分别抛出InstantiationException和IllegalAccessException，所以第一行代码需要放在try catch块内。

第二行代码由于没有检查就进行了向下转型，会导致编译器发出警告，但事实上我们能保证它是类型安全的，所以需要添加@SuppressWarnings("unchecked")注解。

## 通配符

如果我们需要指定某个引用可以指向所有Foo子类的List(协变)我们可以使用如下语法:

```java
class Foo {}
class Bar extends Foo {}
...
    List <? extends Foo> flist = new ArrayList<Bar>();
...
```

而如果我们需要指定某个引用可以指向所有Bar父类的List(逆变):

```java
List <? super Bar> flist = new ArrayList<Foo>();
```

逆变机制适用于类似于函数类型的构造，比如Function<A,B,T>，A，B表示传入参数的类型，当覆盖父类方法时，如果参数里有Function，那么A，B就应该设置成逆变，而T则应该为协变。因为覆盖方法时，传入的函数应该能接受更多类型的参数，而返回类型的范围应该能缩小。当然，最好是都不变了。

如果需要匹配任何类型的类型参数，直接使用List<?>这样的语法即可，它表示可以指向任何List的引用。

## Java泛型的缺陷

### 多接口继承
首先由于类型擦除，如果我们有Foo<T>接口，那么Foo<Integer>和Foo<Double>是一样的，所以无法同时实现Foo<Integer>和Foo<Double>两个接口

### Mixin编程
在C++中可以有如下的声明：

```c++
template<class T> class Foo1 : public T {...};
template<class T> class Foo2 : public T {...};
template<class T> class Foo3 : public T {...};
class Basic{};
```

在实际使用中，可以有这样的声明:

```c++
Foo1<Foo2<Foo3<Basic>>> mixin;
```

这样我们可以轻松地将Foo1，Foo2，Foo3的行为混合在一起。这里的混合，有两种方式，一个是同时拥有混合类的多种方法，另一个是达到装饰器的效果，比如：

```c++
#include <iostream>

using namespace std;

struct Logger {
  virtual void log(string s) {
    cout << s << endl;
  }
};

template<class T> struct DoubleLogger: public T {
  virtual void log(string s) {
    T::log(s+s);
  }
};

template<class T> struct PrefixLogger: public T {
  virtual void log(string s) {
    T::log("log: " + s); 
  }
};

int main(void) {
    DoubleLogger<PrefixLogger<Logger> > mix;
    mix.log("hello");
    return 0;
}

```

如果我们能想前面c++那样定义模板类，就通过调用父类的相同方法实现一层一层的装饰作用，因为它们使用的类型参数恰好就是自己的父类(Foo1<Foo2<Foo3<Basic>>>的父类是Foo2<Foo3<Basic>>，等等)，Java中实现装饰器模式，则需要像Java IO一样使用嵌套的构造器来实现，而如果Java要混合各种类的方法，就需要使用动态代理：

```java
import java.lang.reflect.*;
import java.util.*;

class Pair<A,B> {
  private A a;
  private B b;
  Pair(A a, B b) {this.a = a;this.b = b;}
  public A getA() {return a;}
  public B getB() {return b;}
}

class MixinProxy implements InvocationHandler {
  Map<String, Object> delegatesByMethod;
  
  public MixinProxy(Pair<Object, Class<?>>... pairs) {
    delegatesByMethod = new HashMap<String, Object>();
    for(Pair<Object, Class<?>> pair: pairs) {
      for(Method method: pair.getB().getMethods()) {
        String methodName = method.getName();
        if (!delegateByMethod.containsKey(methodName))
          delegateByMethod.put(methodName, pair.getA())
      }
    }
  }
  
  public Object invoke(Object proxy, Method method, Object[] args)
    throws Throwable {
    String methodName = method.getName();
    Object delegate = delegatesByMethod.get(methodName);
    return method.invoke(delegate, args);
  }
  
  @SuppressWarnings("unchecked")
  public static Object newInstance(Pair... pairs) {
    Class[] interfaces = new Class[pairs.length];
    for (int i = 0; i < pairs.length; i++) {
      interfaces[i] = (Class)pairs[i].getB();
    }
    ClassLoader cl = pairs[0].getA().getClass().getClassLoader();
    return Proxy.newProxyInstance(
      cl, interfaces, new MixinProxy(pairs));
  }
} 
```

使用时直接调用静态方法newInstance()来获取相关对象即可，不过如果需要调用属于具体某接口的方法，必须要先强制转换成那个接口。

不过如果真的这么需要使用Mixin风格进行编程的话，也许换一个语言会更合适一些，比如scala中的trait（特质），它是接口的高级版，其接口可以在创建实例的时候再进行绑定：

```scala
abstract class Logger {
  def log(msg: String)
}

class MyLogger extends Logger{
  def log(msg: String) = println(msg)
}

trait PrefixLogger extends Logger {
  abstract override log(msg: String) = super.log("log: " + msg)
}

trait UpperCaseLogger extends Logger {
  abstract override log(msg: String) = super.log(msg.toUpperCase)
}

trait LastMsgLogger extends Logger {
  private var m = ""
  abstract override log(msg: String) = {
    m = msg
    super.log(msg)
  }
  def lastMsg() = m
}
```

override关键字等同于Java的override注解，但是是强制的，abstract是因为该方法取决于super的行为，所以实际上是还没有被实现的

当我们使用一个具体的Logger，MyLogger来向控制台打印日志时，可以创建MyLogger的实例：

```scala
val logger = new MyLogger
logger.log("hello")
//output:
//hello
```

当我们需要向这个logger加入一些“特质”时，可以在创建时进行绑定：

```scala
val logger = new MyLogger with PrefixLogger with UpperCaseLogger with LastMsgLogger
logger.log("This is a Message.")
//output: 
//log: THIS IS A MESSAGE.
```
可以注意到，logger还绑定了LastMsgLogger特质，所以可以调用lasgMsg()获得之前记录过的信息：

```scala
println(logger.lastMsg)
//output:
//This is a Message.
```

关键在于super引用的是那个类。scala使用的是线性化继承的方式，先把涉及到的类和特质(以及其所有父类)按with的顺序列出来

```
Logger, MyLogger,
Logger, PrefixLogger,
Logger, UpperCaseLogger,
Logger,  LastMsgLogger
```

重复的部分，取继承树中的最上端，得到

```
Logger <- MyLogger <- PrefixLogger <- UpperCaseLogger <- LastMsgLogger
```

所以LastMsgLogger.log()中，super.log()就是UpperCaseLogger.log()以此类推，和前面c++的方式基本相同，真正实现了Mixin，同时解决了c++多重继承带来的麻烦，因为super是唯一的，实际上是单继承。

(我只是顺道重温一下scala而已。。)

写完文章突然想起来，Java8的接口已经实现了默认方法，获得Mixin的特性将会容易很多，不过要实现c++，scala那种Mixin还是有点麻烦(没试过)

### 潜在类型机制
这本书还提到了"潜在类型机制"，即使用泛型时，通过判断该对象是否具有某个成员来判断是否能使用该泛型。c++和python支持这种机制，其中c++在编译期进行检查(因为是"模板"，类型可以直接替换嘛)，而python就更不用说了，本身就是动态类型语言，什么东西都是运行期才知道的。仅仅凭这一点，可以片面的说c++是弱类型的。Java的泛型怎么实现呢，hhhhh当然是没法直接做到了。当然了，用反射总能解决一切问题，但是反射总有种把问题复杂化的感觉。

再提一提scala。。似乎可以用结构化类来限制一个引用必须包含某个方法：

```scala
class Foo[T <: { def someMethod(a: SomeClass1): SomeClass2 }] {...}
```

限定类型T必须是一个包含方法someMethod(a: Bar)的类，当然实际调用T的someMethod方法时，其底层还是必须用反射来实现。
