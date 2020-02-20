---
layout: post
title: Stack toArray 的坑
categories: Blogs
description: Stack toArray 的坑
keywords: java,stack
---

## 背景

LinkedList 在初始化的时候有这么个构造器：

``` java
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

他可以方便的把别的集合添加进来，调用的也是 addAll 方法。

但是最近写了这么个代码发现不对：

``` java
// stack 从底到上 1 2 3
List<Integer> list = new LinkedList<>(stack);
// list 元素也是 1 2 3
```

其中 stack 是 Stack 类，按理说栈加进去的元素应该是从栈顶输出到栈底的，但实际上还是以栈底在前顺序加进了 list。

## 解释

Stack 在 java 是简单的 Vector 类的子类，所以他也是线程安全的，但是不适宜使用的。所以官方也在头上建议使用 Deque 的实现类，如 LinkedList。

而 Vector 内部是和 ArrayList 一样基于数组实现的。

看一下 public LinkedList(Collection<? extends E> c) 中的 addAll 方法。对应调用的 Vector 中的 addAll 方法。

``` java
public synchronized boolean addAll(Collection<? extends E> c) {
    modCount++;
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityHelper(elementCount + numNew);
    System.arraycopy(a, 0, elementData, elementCount, numNew);
    elementCount += numNew;
    return numNew != 0;
}
```

所以他就是简单的将数组从前往后输出，自然也就是倒序了。