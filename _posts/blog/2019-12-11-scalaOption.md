---
layout: post
title: 讲述 Scala 中 Option
categories: Blogs
description: 讲述 Scala 中 Option
keywords: scala,option
---

之前一直在 scala 和 Java 混合编程，没时间体会 Option 的用法，今天总结一下。汇总于[知乎](https://www.zhihu.com/question/59945107)。

## Option，None，Some 的关系

![关系](/images/posts/blog/scalaOption/option.jpg)

上图说的比较清晰了，将一个变量有确切值的时候定义为Some类型，空值是定义为None类型，而Option则可看为Some和None类型的合集。

``` java
sealed abstract class Option[+A] extends Product with Serializable

final case class Some[+A](x: A) extends Option[A] {
  def isEmpty = false
  def get = x
}

case object None extends Option[Nothing] {
  def isEmpty = true
  def get = throw new NoSuchElementException("None.get")
}
```

## 好处

主要好处是解决 Java 中 null 容易报 Exception 的问题。

如：

``` java
boolean isEmptyString(String str) {
  str.length() == 0;
}
```

在 java 中可能会由于 str 为 null 报 NullPointerException。所以应该写成一下形式：

``` java
boolean isEmptyString(String str) {
  return str == null || str.length() == 0;
}
```

但这其实很容易忘的，而有了外包一层的 Option，它可以确保不出现这一问题。

``` java
def isEmptyString(maybeStr: Option[String]) = {
  maybeStr.map(_.length() == 0).getOrElse(true) // 通过 map 进行对 String 的操作，通过 getOrElse 取值。
}
```

所以总的来说就是通过包一层 Option 防止 null 带来的空指针问题。

附 Option 的方法：

![Option 方法](/images/posts/blog/scalaOption/WX20191211-214455.png)