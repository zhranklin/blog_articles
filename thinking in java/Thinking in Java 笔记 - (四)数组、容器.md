# Thinking in Java 笔记 - (四)数组、容器
## 数组
### 数组与泛型
由于类型擦除，我们不能创建具有参数化类型的数组：

```java
Foo<Bar>[] ar = new Foo<Bar>[10];//Illegal
```

但是可以持有该引用：

```java
List[] la = new List[10];
List<Integer>[] li = (List<Integer>[])la;//会引发非受检警告
```

在泛型类中，不能创建泛型数组，直接创建Object数组，在进行转型更方便：

```java
T[] arr;
arr = new T[10];//Illegal
arr = (T[]) new Object[10];
```

### Arrays的实用功能
#### 判等
java.util.Arrays的静态方法Arrays.equals()可以判断两个数组是否相等。首先看元素个数是否相等，然后调用数组中元素的equals()方法，判断对应位置的元素是否都相等。

#### 排序
Array.sort()接受实现了Comparable接口类的数组，对数组内元素进行排序。如果数组内对象没有实现Comparable接口，也可以直接传入实例化的Comparator接口，用于比较数组中的元素。

#### 二分搜索
binarySearch()可以对已经排过序的数组进行二分搜索，和sort()一样，binarySearch有多个重载版本，接受基本类型和Object的数组，或者加上一个Comparator实例。

## 容器

数组除了随机存取性能较好之外，没有其他优点，可以用Java标准库中的容器。Java中的容器有四种List, Set, Queue, Map。

容器被划分为两大类，Collection和Map。Colletion用来保存独立元素的序列，其中List必须按照输入的顺序保存元素，Set不能有重复元素，而Queue则按照先进先出的方式存取。Map用来存储“键值对”，也可称为关联数组，字典。

首先看一下Java容器的类图：

![Collection类图](http://p.blog.csdn.net/images/p_blog_csdn_net/EvanLiu/map.bmp)

Abstract开头的显然是抽象类，它提供了必要方法，可以用来自己定制容器。

### Collections常用方法
Collections提供了很多使用的静态方法

- checkedCollection(Collection<T>, Class<T>)可以用来产生类型安全的容器，它能利用提供的Class对象，在执行插入操作时就进行类型检查并抛出异常，从而找出错误的根源。同样的还有checkedList, checkedMap, checkedSet, checkedSortedMap, checkedSortedSet。
- min(),max()获得容器中的最值，可以选择提供Comparator。
- indexOfSubList(List, List)获得第一个匹配子串的索引
- lastIndexOfSubList(List, List)获得最后一个匹配子串的索引
- replaceAll(List<T>, T, T)替换List中的元素
- reverse(List)将List中的对象顺序逆转
- reverseOrder() reverseOrder(Comparator<T>)获得一个与自然顺序相反的Comparator，或者与传入的Comparator顺序相反的Comparator
- rotate(List, int distance)将List向后移动distance个位置，末尾元素循环到前面来
- shuffle(List), shuffle(List, Random) 打乱List中元素的顺序
- sort,copy,swap，都是List的方法，字面意思，效率比手动实现的高
- fill(List<T>, T x) 用x填满List
- nCopies(int n, T x)返回一个大小为n的List，用x填满
- disjoint(Collection, Collection)
- frequency(Collection, Object x) 返回容器中等于x的元素的个数
- emptyList(), emptyMap(), emptySet() 返回不可变的空List、Map或Set

## Colletion
Colletion具有序列的概念，具有add方法，可以用foreach语法遍历。

### 向Colletion中添加一组元素
静态方法Arrays.asList()接受一个数组或者一个用逗号分隔的元素列表（可变参数），返回一个List对象。

静态方法Colletions.addAll()方法接受一个Colletion对象，以及一个用逗号分割的列表，将元素添加到Colletion中。

还有一个Colletion类的成员方法addAll()，不过不过它只能接受Colletion对象作为参数。

**tips**: 关于Arrays.asList()方法有一些限制，第一，他的尺寸不能改变，所以不能调用add()或delete()方法；第二，其返回的类型是传入参数类型的最小上界，可能导致一些问题，如示例所示：

```java
class Snow {}
class Powder extends Snow {}
class Light extends Powder {}
class Heavy extends Powder {}

public class AsListInference {
  public static void main(String[] args) {
    List<Snow> snow1 = Arrays.asList(
      new Light(), new Heavy());//这一行代码将导致编译错误
      //因为等号右边是List<Powder>而等号左边是List<Snow>
    List<Snow> snow2 = Arrays.<Snow>asList(
      new Light(), new Heavy());//只能指定asList方法的类型参数
  }
}
```

contains方法用来确定某个元素是否在列表中。以及remove()方法可以用来移除某个特定对象。注意这些方法以及List类中的indexOf都使用到对象的equals()方法。

addAll()，retainAll()和removeAll()类似于集合的取并集，取交集和作差



## List
List有两个常用的子类，ArrayList和LinkedList，从名字可见，他们分别用数组和链表实现。

List重载了Colletion的add()和addAll()方法，可以指定索引，从而元素插入特定位置。

还有subList()可以用来获取子列表。

### 迭代器
Colletion类实现了Iterable接口，其中iterator()方法反回一个迭代器Iterator，Java8中，它还包含了forEach()方法，接受一个Consumer对象，用来对元素做相应处理（满满的函数式编程风格...）。

Iterator有hasNext()方法和next()用来判断是否还有下一个元素以及返回下一个元素，以及remove()方法用来移除当前的元素。在Java8中remove()已经被设置为默认方法，同时添加了一个forEachRemaining()方法，接受一个Consumer，配合处理迭代器容器中剩余的所有元素。

### ListIterator

ListIterator是更强大的迭代器，能在List类中获得，它能够双向移动(于是多了hasPrevious()和previous()方法)，以及能使用set方法，更改之前调用next()或previous()返回的引用。

### LinkedLit
LinkedList添加了很多与栈，队列，双端队列相关的方法别名

双端队列：get/add/removeFirst/Last()
栈：push(),pop()peek()

### 如何选择List
可以先将ArrayList作为首选，如果需要额外功能或者出于性能需要时，选择LinkedList，如果使用的是固定数量的元素，可以选择背后用数组做支撑的List(如Array.asList()产生的List)，或者直接用真正的数组。

## Set
Set用来保存不重复的元素，它与Collection的接口完全相同，只是具有不同的行为而已。常用的具体类有HashSet和TreeSet，前者速度较快，后者可以使元素保持排序状态，而且迭代起来速度比较快。

## Map
### 使用
containsKey() containsValue() keySet() values()直接按照字面意思理解即可，而get/remove/put用来获取、移除(通过key)和添加键值对。EntrySet返回键值对的集合(Set<Map.Entry<K,V>>)

### 如何选择Map
HashMap性能较好，用作首选，而TreeMap保证有序，LinkedHashMap保证遍历顺序与插入顺序相同，所以后两者迭代速度比较快。而IdentityHashMap使用==来判等，本身用途就不同，而速度也比HashMap快很多。

## Queue Deque

LinkedList可以直接向上转型为Queue(队列)和Deque(双端队列)

以下为Deque的相关方法：

|/| 首元素 | 首元素 | 末元素 | 末元素 |
|:---:|:---:|:---:|:---:|:---:
|/| Throws exception | Special value | Throws exception | Special value
|Insert | addFirst(e) | offerFirst(e) | addLast(e) | offerLast(e)
|Remove | removeFirst() | pollFirst() | removeLast() | pollLast()
|Examine | getFirst() | peekFirst() | getLast() | peekLast()

以下为Queue的方法以及对应的Deque方法：

Queue中的方法 | Deque中的等价方法
:----:|:----:
add(e)	| addLast(e)
offer(e) | offerLast(e)
remove() | removeFirst()
poll() | pollFirst()
element() | getFirst()
peek() | peekFirst()

注：以上两个表格摘自Java API

## PriorityQueue
PriorityQueue优先队列实现了Queue接口，poll()和peek()根据自然排序或者提供的Comparator的顺序遍历

## foreach
实现了iterable接口的类都可以在foreach语法中使用。Java标准库中有大量的类实现了Iterable，例如所有的Collection类。不过要迭代Map的话，需要调用其entrySet()方法（详见前面Map的说明）。当然，数组虽然也能使用foreach语法，但是他并没有实现Iterable接口。

## 适配器方法惯用法
如果需要改变迭代器的行为（比如倒序遍历），使得foreach的效果发生改变，可以给相应的容器添加一个返回相应的Iterable(如下面的reversed()方法)：

```java
//示例来自Thinking in Java 4th Edition

import java.util.*;
class ReversibleArrayList<T> extends ArrayList<T> {
  public ReversibleArrayList(Colletion<T> c) { super(C); }
  public Iterable<T> reversed() {
    return new Iterable<T>() {
      public Iterator<T> iterator() {
        return new Iterator<T>() {
          int current = size() - 1;
          public boolean hasNext() { return current > -1; }
          public T next() { return get(current--); }
          public void remove() {throw new UnsupportedOperationException();}
        };
      }
    };
  }
}
```

