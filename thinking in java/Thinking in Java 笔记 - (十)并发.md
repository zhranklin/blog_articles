# Thinking in Java 笔记 - (十)并发.md
*Thinking in Java*这本书最后几章还没有记过笔记, 搁置了很久, 正好最近操作系统试验要做多进程的实验, 就顺便把最后的并发(Swing已经被我忽略了)看一看。

## 基本的线程机制
### 定义任务
定义一个任务, 只需实现Runnable接口即可:

```java
public class LiftOff implements Runnable {
  protected int countDown = 10;
  private static int taskCount = 0;
  private final int id = taskCount++;
  public LiftOff() {}
  public LiftOff(int countDown) {
    this.countDown = countDown;
  }
  public String status() {
    return "#" + id + "(" +
      (countDown > 0 ? countDown : "Listoff!") + "), ";
  }
  public void run() {
    while (countDown-- > 0) {
      System.out.println(status());
      Thread.yield();
    }
  }
}
```

Thread.yield()用来告诉JVM可以在这里切换线程, 这里使用yield的目的只是让代码在实际运行的时候真正产生多线程的效果。

```
public class MainThread {
  public static void main(String[] args) {
    LiftOff launch = new LiftOff();
    launch.run();
  }
} //output:
// #0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(Liftoff!).
```

### Thread类
Runnable仅仅定义了一段任务是如何工作的, 如果需要将它作为线程来处理, 则可以将Runnable对象提交给一个Thread构造器:
      
```java
public class BasicThreads {
  public static void main(String[] args) {
    Thread t = new Thread(new LiftOff());
    t.start();
    System.out.println("Waiting for LiftOff");
  }
} // output:
Waiting for LiftOff
#0(9) #0(8) .....
```

可见t.start()调用之后立刻返回了(然后马上输出了"Waiting for LiftOff")

如果同时开启5个线程:

```java
public calss MoreBasicThreads {
  public static void main(String[] args) {
    for (int i = 0; i < 5; i++)
      new Thread(new LiftOff()).start();
    System.out.println("Waiting for LiftOff")
  }
}
```


