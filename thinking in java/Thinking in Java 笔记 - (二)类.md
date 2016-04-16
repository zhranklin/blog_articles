# Thinking in Java 笔记 - (二)类

## 组合与继承
组合通常用于想在新类中使用现有类的功能而非其接口的情况。
继承用于使用某个现有类，同时开发一个它的特殊版本，通常需要为了某种特殊需要而将其特殊化。

组合优于继承，只有在必须使用向上转型的时候才考虑使用继承。


## 关于final方法和private方法

final用来指定一个方法无法被覆盖。而private方法本身隐含了该方法是final的故private方法实际上是不能被覆盖的，然而如果在子类中“覆盖”父类的private方法，实际上生成了一个新方法而已，并没有真正覆盖，覆盖这个概念仅对一个类的接口有效。

## 构造器内部的多态方法行为

如果在沟早期内部调用一个动态绑定的方法，就要用到那个方法被覆盖后的定义。然而这个被覆盖的方法在对象完全被构造之前就会被调用，那么其结果将难以预料，从而造成一些难以发现的隐藏错误。所以编写构造器时，
> 用尽可能简单的方法是对象进入正常状态，尽量避免调用其他方法。构造器内唯一能够安全调用的方法是基类中的final方法(private方法也是final方法)

## 包含继承的对象初始化过程
1. 将分配给对象的存储空间初始化成二进制的零。
2. 调用基类的构造器，这个步骤会递归下去。导出类的覆盖方法可能会被基类构造器调用。
3. 按照声明的顺序调用成员的初始化方法。
4. 调用导出类的构造器主体。

以下为本人写的示例：

```java
package com.zhranklin;

class A {
  {
    System.out.println("A1");
  }

  A(int num) {
    System.out.println("A2: in Constructor of A, num = " + num);
  }

  private int a = printAndGetInt("A3");

  static int printAndGetInt(String s) {
    System.out.println(s);
    return 0;
  }
}

class B extends A {
  {
    printAndGetInt("B1");
  }

  private int b = printAndGetInt("B2");

  B() {
    super(13);
    System.out.println("B3: in Constructor of B, after super(13)");
  }
}

public class Main {
  public static void main(String[] args) {
    A a = new B();
  }
}
/* output:
A1
A3
A2: Constructor of A, num = 13
B1
B2
B3: Constructor of B
 */
```

从示例中可见，在B的对象的构造过程中，虽然看起来super方法在B的构造器中才调用，但是实际上super(13)这个操作远在调用B构造器之前就已经执行完成，也就是说，必须完成父类所有的初始化操作之后，才会进行子类的初始化，所以A1 A2 A3一定在B1 B2 B3之前，这也是super()必须在构造器第一行且只能调用一次原因。