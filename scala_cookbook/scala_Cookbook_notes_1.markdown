# Scala Cookbook 笔记 1 - 字符串

## 计算相等

### ==方法

直接使用==方法，可对null调用:
```scala
val s1 = "Hello"
val s2 = "H" + "ello"
val s3 = null
s1 == s2 //true
s3 == s2 //false
```

### 忽略大小写(不能对null调用):

```scala
val s1 = "Hello"
val s2 = "hello"
s1.toUpperCase == s2.toUpperCase // true
s1 equalsIgnoreCase s2 // true
```

### 关于==方法:

==为AnyRef下定义的方法，先检查null值，再调用equals方法

## 多行字符串

### 使用""":

```scala
val foo = """This is
    a multiline
    String"""
println(foo)
//output:
//This is
//    a multiline
//    String
```

### 解决行首空格:

- 顶格写
- 利用stripMargin
- 也可以向stripMargin传入参数

```scala
val s1 = """This is
a multiline
String"""
val s2 = """This is
           |a multiline
           |String""".stripMargin
val s3 = """This is
           #a multiline
           #String""".stripMargin('#')
```

## 分割

可调用split方法:
```scala
"hello world" split " " foreach println
//output:
//hello
//world
```

亦可传入正则表达式:
```scala
"hello world" split "\\s+" foreach println
//输出与前面相同
```

scala中StringLike类重载了split方法也可传入Char，如"hello world" split ' '

## 在字符串中替换变量

### string interpolator

使用s"..."和$符号即可:
```scala
val count = 15
val s1 = s"I have $count books on my shelf."
val s2 = s""
```

也可用{}限定范围:
```scala
val count = 15
val s1 = s"I have ${count + 1} books on my shelf"
```

### string formatter

使用f"..."和$可得到printf style的字符串:
```scala
println(f"$name is $age years old, and weighs $weight%.2f pounds.")
//output:
//Fred is 33 years old, and weighs 200.00 pounds.
```

### raw interpolator

raw"...", 忽略任何转义:
```scala
println(raw"I am strong \nand smart.")
//output:
//I am strong \nand smart.
```

### format方法

```scala
"%s%d".format("No.", 1)//No.1
```

## 寻找匹配

调用.r方法生成正则表达式:
```scala
"[0-9]+".r findFirstIn "123 aaabbb"//Some(123)
"[0-9]+".r findAllIn "123 456 aaabbb" foreach println
//output:
//123
//456
```

## 替换匹配

字符串的replaceAll/replaceFirst方法:
```scala
"123 Main Street".replaceAll([0-9], "x")//xxx Main Street
```

regex的replaceAllIn/replaceFirstIn方法:
```scala
"H".r.replaceFirstIn("Hello word", "J")//Jello world
```

## 提取字符串中的字符

```scala
val pattern = "([0-9]+) ([A-Za-z]+)".r 
val pattern(count, fruit) = "100 Bananas"
//count: String = 100
//fruit: String = Bananas
```

## 访问字符串中的字符

```scala
"hello".charAt(0)//'h'
"hello"(0)//'h'
```
