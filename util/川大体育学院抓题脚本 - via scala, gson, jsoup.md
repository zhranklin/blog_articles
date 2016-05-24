# 川大体育学院抓题脚本 - via scala, gson, jsoup
首先, 说一句三观非常不正的话: scala是一门优秀的 **脚本** 语言。

## Github
先给出交友网站的地址, 什么都别说了, 先上车:<br>
<https://github.com/zhranklin/pe-test>

## 使用的工具
- curl 登陆网站, 可以发送POST, 保存、使用cookie等等
- iconv 网站用的编码是gb2312, 需要转码成utf-8后再处理
- gson java中用来处理json的库
- jsoup 提取html的库

## 简介
第一次写抓取的脚本, 使用linux自带的curl, iconv两个, 然后用scala.sys.process中的进程DSL(和bash有点像, 实现了管道, 重定向等功能)来执行。gson, jsoup两个框架都是临时看的, 一开始看起来很多很烦的样子, 但是开始动手之后, 简单的使用它们还是没什么压力的。

## 代码
```scala
package com.zhranklin

import java.io.{File, FileWriter}

import com.google.gson.GsonBuilder
import org.jsoup.Jsoup
import org.jsoup.nodes.{Element, Node}
import org.jsoup.select.NodeVisitor

import scala.collection.mutable
import scala.language.postfixOps
import scala.sys.process._
import scala.util.{Random, Try}

case class Question(content: String, answer: String, validity: String)
class QType() {override def toString: String = throw new NullPointerException}
case class DX() extends QType {override def toString = "dx"}
case class PD() extends QType {override def toString = "pd"}


class Petest(xh: String, pwd: String) {
  val COOKIE_FILE = xh+".txt"
  val LOGIN_UTL = "http://pead.scu.edu.cn/stu/dl.asp"
  val LX_URL = "http://pead.scu.edu.cn/stu/lllx.asp"
  val iconvCommand = Seq("iconv", "-t", "UTF-8", "-f", "GBK")
  val judgePattern = "正确|错误".r

  def login() = Seq("curl", "-d", s"xh=$xh&pwd=$pwd", "-c", COOKIE_FILE, LOGIN_UTL) !

  def getBody = Jsoup.parse(Seq("curl", "-b", COOKIE_FILE, LX_URL) #| iconvCommand !!).body

  def getQuestionElement(body: Element) = body.select("span[class=style7]").first.parent

  def getQuestionType(body: Element): QType = {
    if (body.select("input[name=dx][type=hidden]").first != null) DX()
    else if (body.select("input[name=pd][type=hidden]").first != null) PD()
    else new QType
  }

  def choose(question: Element, qType: QType): String = {
    val answers = new mutable.MutableList[String]
    question.select(s"input[name=${qType}r]").traverse(new NodeVisitor() {
      def head(node: Node, depth: Int) =
        if (depth == 0)
          answers += node.attr("value")
      def tail(node: Node, depth: Int) = {}
    })
    answers.get(Random.nextInt(answers.length)).get
  }

  def validate(id: String, ans: String, qType: QType) =
    judgePattern
      .findFirstIn(Seq("curl", "-b", COOKIE_FILE, s"$LX_URL?$qType=$id&${qType}r=$ans") #| iconvCommand !!)
      .getOrElse("无效")

  def getQuestionFromBody(body: Element, qType: QType) = {
    val id = body.select(s"input[name=$qType][type=hidden]").first.attr("value")
    val qElement = getQuestionElement(body)
    val answer = choose(qElement, qType)
    val validity = validate(id, answer, qType)
    val content =
      qElement.html
        .replaceAll("<br>", "\n")
        .replaceAll("\n", "#nl#")
        .replaceAll("<.*?>", " ")
        .replaceAll("\\s\\s+", " ")
        .replaceAll("\\s*#nl#", "\n")
    Question(content, answer, validity)
  }

  def getQuestion = {
    val body = getBody
    getQuestionFromBody(body, getQuestionType(body))
  }
}

object Petest {
  val FILE_NAME = "output.json"
  def main(args: Array[String]): Unit = {
    val gson = new GsonBuilder().create
    val petest = new Petest(args(0), args(1))
    val times = Integer.parseInt(args(2))
    val interval = if (args.length > 3) Integer.parseInt(args(3)) else 10000
    val fw = new FileWriter(FILE_NAME, true)
    petest.login()
    petest.getBody
    1 to times foreach { i =>
      print(s"第${i}次尝试...")
      try {
        val q = petest.getQuestion
        gson.toJson(q, fw)
        fw.append('\n')
        println("成功.")
      } catch {
        case _: Exception => println("失败.")
      }
      Thread.sleep(interval)
    }
    fw.close()
  }
}
```

## 写在最后
仅供娱乐之用, 不要干坏事哈...
