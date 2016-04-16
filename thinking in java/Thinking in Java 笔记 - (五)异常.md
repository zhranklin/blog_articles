# Thinking in Java 笔记 - (五)异常
Java中异常Exception的父类是Throw(它还有个子类是Error，表示编译时和系统错误)。

## 创建自定义异常
从平时的编程经验来看，一般来说，我们只需要用到某个异常的类型，所以只需要继承一个意思相近的Exception，或者直接继承Exception即可:

```java
class SimpleExption extends Exception {}
```

或者加一个接受字符串的构造器：

```java
class MyException() {
  public MyExption() {}
  public MyExption(String msg) {
    super(msg);
  }
}
```

## Exception的方法
- String getMessage()
- String getLocalizedMessage()<br>获取详细信息，或用本地语言表示的详细信息
- String toString() 返回对Throwable的简单描述，如果有详细信息的话，也会包括在内
- void printStackTrace()
- void printStackTrace(PrintStream)
- void printStackTrace(java.io.PrintWriter)<br>将栈轨迹输出到标准输出流或者指定的输出流
- Throwable fillInStackTrace()<br>记录当前栈帧的状态，用于程序重新抛出错误或异常

## 栈轨迹
getStackTrace()方法可以返回由栈轨迹组成的数组(StackTraceElement[])。StackTraceElement具有以下方法（取自Java8 API）：

|Modifier and Type |  Method | Description|
|:---:|:---:|:---:|
|boolean |   equals(Object obj) | Returns true if the specified object is another StackTraceElement instance representing the same execution point as this instance.|
|String  |   getClassName()     | Returns the fully qualified name of the class containing the execution point represented by this stack trace element.|
|String  |   getFileName()      | Returns the name of the source file containing the execution point represented by this stack trace element.|
|int     |   getLineNumber()    | Returns the line number of the source line containing the execution point represented by this stack trace element.|
|String  |   getMethodName()    | Returns the name of the method containing the execution point represented by this stack trace element.|
|int     |   hashCode()         | Returns a hash code value for this stack trace element.|
|boolean |   isNativeMethod()   | Returns true if the method containing the execution point represented by this stack trace element is a native method.|
|String  |   toString()         | Returns a string representation of this stack trace element.|

由此可见，可以利用getStackTrace获得各个栈帧的各种信息诸如方法名，所处对象的类名，行号等等，可以很方便的用于日志输出。

## 重新抛出异常
有时候，尤其是捕获所有异常后，我们可能需要重新抛出异常，而新的异常是不包含之前异常的任何信息的，这时可以调用原异常的fillInStackTrace()方法，得到一个Throwable对象，再将其抛出。

然而如果需要重新抛出不同类型的异常，这是则需要重新创建一个Throwable对象，并保存旧异常的信息。有些异常自带接受cause参数的构造器，但是只有三个最基本的Throwable子类具有这样的构造器：Error, Exception和RuntimeException。如果重新抛出的异常是其他类型的，则需要调用initCause()方法：

```java
class MyException extends Exception {}
try {
  ...
} catch (Exception: e) {
  MyException me = new MyException();
  me.initCause(e);
  throw me;
}
```

## RuntimeException
RuntimeExceptio是一种非受检异常，我们不需要抛出该类异常，也不用在方法声明中说明该方法会抛出某个RuntimeException，也不需要捕获该异常，JVM会在发生该类错误时自动抛出异常。我们需要做的就是尽量避免这类错误，比如NullPointerException, ArrayIndexOutOfBoundsException等等。处理这些异常就显得太多余了。

## finally
一般用来处理一些收尾清理工作，比如文件IO等等,无论try语句块里发生了什么样的跳转，比如break，continue，return，以及抛出异常，finally内的语句块都会在跳转之前执行。