---
layout: post
title: 二分法求中间数除以 2 时的向上取整和向下取整
categories: Blogs
description: 二分法求中间数除以 2 时的向上取整和向下取整
keywords: 二分法,除法
---

由于二分法的位置都是正数，所以可以采用以下两种方式向上/下取整。

与 java 中 3/2 => 1，-3/2 => -1 保持一致。

## 绝对值向下取整

即 1 2 取 1。

``` java
int start = 1;
int end = 2;
int mid = start + (end-start)/2;
// mid = 1
```

``` java
int start = -1;
int end = -2;
int mid = start + (end-start)/2;
// mid = -1
```

## 绝对值向上取整

即 1 2 取 2。

``` java
int start = 1;
int end = 2;
int mid = end + (start-end)/2;
// mid = 2
```

``` java
int start = -1;
int end = -2;
int mid = end + (start-end)/2;
// mid = -2
```