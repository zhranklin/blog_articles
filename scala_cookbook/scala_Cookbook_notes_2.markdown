# Scala Cookbook 笔记 2 - 数字

## scala中的数字类型
在scala中, 所有数字类型都是对象，包括Byte, Char, Double, Float, Int, Long, Short, 它们继承了AnyVal trait, 同样的还有Unit和Boolean类

## 数据范围
与java中相同

要查看特定类型的最大值、最小值，可以用伴生对象的MinValue/MaxValue方法:

```scala
Short.MinValue // -32768
Float.MaxValue // -3.4028235E38
```

## 日期
有个Joda Time的scala封装——nscala-time:
```scala
DateTime.now // 返回一个org.joda.time.DateTime
DateTime.now + 2.months
DateTime.nextMonth < DateTime.now + 2.months
(2.hours + 45.minutes + 10.seconds).millis
```

## 从字符串中提取数字
### String的to\*方法
使用String的to\*方法即可:

```scala
"100".toInt
"100".toDouble
```

*注意:* 这些方法可能抛出NumberFormatException

### BigInt和BigDecimal的构造函数
```scala
BigIng("1") // res0: scala.math.BigInt = 1
BigDecimal("3.14159") // res1: scala.math.BigDecimal = 3.14159
```

### 处理进制转换
使用java.lang.Integer类的方法:

```scala
Integer.parseInt("100", 2) // 4
```

## 数字类型间的转换
### 各种数字类型的to\*方法
调用toInt/toDouble等等方法:

```scala
19.45.toInt // res0: Int = 19
19.toDouble // res1: Double = 19.0
```

从"大数"转换成"小数"时, 可用isValid\*测试是否会溢出:
```scala
val a = 1000L // a: Long = 1000
a.isValidByte // false
a.isValidShort // true
```

## 覆盖默认类型
```scala
val a = 1 // a: Int = 1
val a = 1d // a: Double = 1.0
val a = 1000L // a: Long = 1000
val a = 0: Byte // a: Byte = 0
val a = 0: Int // a: Int = 0
val a: Int = 0 // a: Int = 0
val a = 0x20 //16进制
```

## ++ -- 的替代方式

用var值的+=、-=方法:
```scala
var a = 1
a += 1
println(a) // 2
```

同时也有\*=、/=

## 处理大数

使用scala的BigInt和BigDecimal即可:

```scala
var b = BigIng(1234567890)
var b = BigDecimal
```

两者都实现了相应的操作符:

```scala
b + b // res0: scala.math.BigInt = 2469135780
b += 1; println(b) // 1234567891
b.toInt//res1: Int = 1234567891
```

以及isValid\*方法:

```scala
b.isValidByte // false
b.isFalidInt // true
```

## 生成随机数

```scala
val r = scala.util.Random
r.nextInt// res0: Int = -132477914
r.nextInt(100) // 限定范围:[0, 100)
// res0: Int = 58
r.nextFloat // 0.5031704
r.nextDouble // 0.6946000981900997
```
nextFloat和nextDouble方法的范围都是0.0~1.0

也可在随机数种子种指定范围
```scala
val r = new scala.util.Random(100) // 生成的范围是[0,100)
r.setSeed(1000L) // [0, 1000)
```

## 生成数列
调用to方法:

```scala
val r = 1 to 10
```
可以用by制定步长:

```scala
val r = 1 to 10 by 2 // Range(1,3,5,7,9)
```

### 将Range转换成Array和List

```scala
val x = 1 to 6 toArray
// x: Array[Int] = Array(1,2,3,4,5,6)

val x = 1 to 6 toList
// x: Array[Int] = List(1,2,3,4,5,6)
```

## 对数字和货币进行格式化

f"...":
```scala
val pi = scala.math.Pi
println(f"$pi%1.5f") // 3.14159
f"$pi%1.2f" // 3.14
f"$pi%06.2f" // 003.14
```

### java.text.NumberFormat

NumberFormat的getIntegerInstance方法可用来给数字添加逗号:
```scala
val formatter = java.text.Number.getIntegerInstance
formatter.format(10000)//10,000
val java.text.NumberFormat.getIntegerInstance(java.util.Locale("de", "DE")) // 设定地区
formatter.format(1000000) // 1.000.000
```

NumberFormat的getCurrencyInstance返回的formatter可以以货币的方式生成字符串


