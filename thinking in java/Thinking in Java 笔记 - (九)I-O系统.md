# Thinking in Java 笔记 - (九)I/O系统
## File
java.io.File类是java对文件的抽象，它提供了一系列方法用来对文件进行操作，其中canExecute, canRead(), canWrite(), createNewFile(), delete(), deleteOnExit(), exits(), mkdir()以及一系列get\*,set\*,is\*方法，从字面意思可以理解。

list()方法用来列出当前文件夹下包含的所有文件名，listFiles()作用相同，但是返回的是File数组，list()还能接受FileNameFilter来过滤文件，而listFiles()接受FileNameFilter或FileFilter。

## 输入输出流简介
所有输入流都继承自InputStream，输出流都继承自OutputStream。Java的输入输出流采用装饰器的模式对输入输出流进行连接，达到定制的目的。

### InputStream类型
输入流的作用是用来表示不同数据源产生输入的类，包括：

- 字节数组
- String对象
- 文件
- "管道"，概念与unix的管道类似，一端输入，一端输出
- 一个由其他种类的流组成的序列
- 其他数据源，Internet连接等

FilterInputStream类作为InputStream的装饰器的基类，其他的则为输入源。InputStream的子类有：

- ByteArrayInputStream 数组缓冲区
- StringBufferInputStream 字符串
- FileInputStream 文件
- PipedInputStream 管道
- SequenceInputStream 多个InputStream转换成单一的InputStream
- FilterInputStream 作为装饰器的接口

### OutputStream类型
与输入流类似，其子类有：

- ByteArrayOutputStream
- FileOutputStream
- PipedOutputStream
- FilterOutputStream

## FilterInputStream和FilterOutputStream
这两个类其实有点类似于linux的过滤器，用来实现一些输入输出流的特性，比如是否缓冲，是否保留读过的行等等，各种FilterInputStream的子类如下：

- DataInputStream 与DataOutputStream配合使用，用以读取基本数据类型
- BufferedInputStream 使用缓冲区
- LineNumberInputStream 用来跟踪输入流中的行号，可以调用getLineNumber()和setLineNumber(int)
- PushBackInputStream 可以将最后一个字符回退

FilterOutputStream的子类如下：

- DataOutputStream
- PrintStream 用于产生格式化输出，DataOutputStream用于数据存储，PrintStream用于处理显示
- BufferedOutputStream

## Reader和Writer
多数InputStream和OutputStream都有对应的Reader或Writer，主要用来进行Unicode的读写，而InputStream/OutputSteam则是面向字节的
比如：

- FilterInput/OutputStream -- FilterReader/Writer
- BufferedInput/OutputStream -- BufferedReader/Writer
- PrintStream -- PrintWriter
- LineNumberInputStream(弃用) -- LineNumberReader
- PushbackInputStream -- PushbackReader

为了更好的兼容性，PrintWriter的构造器可以接受OutputStream对象

## RandomAccessFile
RandomAccessFile用于对文件进行底层操作，与C中的文件指针等概念相似

## IO的惯用法
