---
layout: post
title: Java 中 Queue，List，Stack 的常用操作
categories: Java
description: Java 中 Queue，List，Stack 的常用操作
keywords: JAVA,List,操作
---

刷题经常用到 Queue，List，Stack，记录一下。

## List Interface 继承 Collection 接口

+ add() 插入
+ remove() 删除

## Stack Class

+ push() 插入
+ pop() 删除
+ peek() 取顶

## Queue Interface 继承 Collection 接口, Queue 的队列不允许添加 null 元素。

|     |Throws Exception|return value|
|:---:|   :----:       | :----:     |
|插入 |add|offer|
|删除 |remove|poll|
|取顶 |element|peek|

继承 Collection 接口的大部分方法自然相同。