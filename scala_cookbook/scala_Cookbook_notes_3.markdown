# Scala Cookbook 笔记 1 - 字符串

## 在循环中实现break和continue
### 实现break

导入scala.util.Breaks.\_, 并将for置于breakable的代码块内:

```scala
breakable {
  for (i ← 1 to 10) {
    println(i)
    if (i > 4) break
 }
}
//1
//2
//3
//4
//5
```

### 实现continue

与前面类似，但是将breakable置于for内, for循环内代码块外:
```scala
for (x ← xs) {
  breakable{
    if (cond)
      break
  }
}
```

### break/breakable实现方式(参考)

```scala
private val breakException = BreakControl
def break(): Nothing = {thow breakException}
def breakable(op: => Unit) {
  try {
    op
  } catch {
    case ex: BreakControl =>
      if (ex ne breakException) throw ex
  }
}
```

### 多重循环与labeled breaks

使用以下语法，实现与java类似的labled breaks

```scala
import scala.util.control._
val Inner = new Breaks
val Outer = new Breaks

Outer.breakable {
  for (i ← 1 to 5) {
    Inner.breakable {
      for (j ← 'a' to 'e') {
        if (i == 1 && j == 'c') Inner.break else println(s"i:$i, j: $j")
        if (i == 2 && j == 'b') Outer.break
      }
    }
  }
}
//output:
//i: 1, j: a
//i: 1, j: b
//i: 2, j: a
```

### 用一条case语句匹配多种情况

使用|字符即可:

```scala
val i = 5
i match {
    case 1 | 3 | 5 | 7 | 9 => println("odd")
    case 2 | 4 | 6 | 8 | 10 => println("even")
}
//output: odd
```

同样可用于字符串和case对象(代码略)

### 在match语句中使用模式匹配

```scala
def echoWhatYouGaveMe(x: Any): String = x match {
  //常量
  case 0 => "zero"
  case true => "true"
  case "hello" => "you said 'hello'"
  case Nil => "an empty List"

  //序列
  case List(0, _, _) => "a three-element list with 0 ass the first element"
  case List(1, _*) => "a list beginning with 1, having any number of elements"
  case Vector(1, _*) => "a vector starting with 1, having any number of elements"

  //元组
  case (a, b) => s"got $a and $b"
  case (a, b, c) => s"got $a, $b, and $c"

  //case class的构造器
  case Person(first, "Alexander") => s"found an Alexandr, first name = $first"

  //匹配类型
  case s: String => s"you gave me this string: $s"

  //默认匹配
  case _ => "Unknown"
}
```

### 在match语句中处理List
使用List的::方法

```scala
List(1,2,3,4,5) match {
    case 1 :: 2 :: rest => println(s"1, 2, and rest($rest)")
}
//output: 1, 2, and rest(List(3, 4, 5))
```


