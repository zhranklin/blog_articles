# Thinking in Java 笔记 - (六)字符串

## StringBuilder
由于String是不可变对象，出于效率考虑，应该使用StringBuilder来代替String的+操作符

```java
StringBuilder sb = new StringBuilder("...");
sb.append("...");
sb.append...;
...
String s = sb.toString();
```
不过很多时候编译器会对代码进行优化，将+操作符编译成StringBuilder的相应操作。

## formatter

### System.out.format()
可以使用System.out对象的format()方法或printf()方法，与C语言的prtinf的方式相同。

```java
public class SimpleFormat {
  public static void main(String[] args) {
    int x = 5;
    double y = 5.332542
    System.out.format("Row 1: [%d %f]\n", x, y);
    System.out.printf("Row 1: [%d %f]\n", x, y);
  }
//output:
//Row 1: [5 5.332542]
//Row 1: [5 5.332542]
}
```

### Formatter类
Formatter是用来格式化字符串的类，它能用来直接将字符串格式化并输出到相应的位置。其构造器接受一个OutputStream, PrintStream, File等等对象作为输入。

```java
public class FormatterTest {
  public static void main(String[] args) {
    Formatter f = new Formatter(System.out);
    f.format("It's a %s.", "Formatter");
  }
  //output:
  //It's a Formatter.
}
```

### String.format()
String.format()方法与C语言的sprintf相同，可得到格式化后的字符串。

```java
String s = String.format("(%d, %d)", 1, 1); //s == "(1, 1)"
```

## 正则表达式
### String中用到正则表达式的方法
方法|参数|作用
---|---|---
matches()|代表正则表达式的字符串|返回boolean表示字符串是否与正则表达式匹配
split()|代表正则表达式的字符串|用正则表达式代表的模式(如"\\w+")来分割字符串，并返回相应的数组。
replaceFirst()/replaceAll()|一个代表正则表达式的字符串，一个用来替换的字符串|将字符串中第一个/所有匹配的部分替换成给定字符串

### Pattern类
Pattern编译之后的正则表达式，可以通过静态方法Pattern.compile()获得。可以通过matches()判断是否匹配，matcher()返回一个Matcher对象，以及split()用来分割字符串。Java8中还能通过AsPredicate得到一个Predicate<String>对象，可作为Stream类的filter()方法的谓词等等。

### Matcher类
通过Matcher对象我们可以判断正则表达式是否与字符串匹配，以及从字符串中提取相应信息等等。
matches(),lookingAt(),find()均返回boolean，matches()判断输入字符串是否匹配正则表达式，lookingAt判断该字符串的开始部分是否匹配，find()则用来查找字符串中的多个匹配。

每次进行匹配操作之后，都可以通过group()方法得到前一次匹配的字符串。如果传入int，就可以得到相应的组(0代表匹配的整个字符串)。用法大致如示例所示

```java
Matcher mat = ...
while(mat.find()){
  System.out.println(group());
  for (int i = 0; i <= mat.groupCount(); i++)
    System.out.println(group(i));
}
```

find()类似于迭代器的next，每次查找下一个匹配，再通过group()得到匹配字符串。如果需要获得上次匹配字符串在院字符串中的索引，可以使用start()和end()方法获得匹配字符的起始和结束(末位+1)索引。