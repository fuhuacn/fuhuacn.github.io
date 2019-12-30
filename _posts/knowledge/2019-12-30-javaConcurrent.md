---
layout: post
title: Java 并发
categories: Knowledge
description: Java 并发
keywords: Java, Concurrent
---
[参考来源](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20并发.md)

目录

* TOC
{:toc}

# 一、使用线程

有三种使用线程的方法：

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

## 实现 Runnable 接口

需要实现 run() 方法。

通过 Thread 调用 start() 方法来启动线程。

``` java
public class MyRunnable implements Runnable {
    public void run() {
        // ...
    }
}
```

``` java
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

## 实现 Callable 接口

与 Runnable 相比，Callable 可以有返回值，实现 call 接口，返回值通过 FutureTask 进行封装。

``` java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```

``` java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
```

FutureTask 实现 RunnableFuture，RunnableFuture 继承 Runnable, Future。

``` java
public class Run {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> f = new FutureTask<String>(()->{ // lambda 表达式的 Callable
            Thread.sleep(2000);
            return "222222";
        });
        Thread t = new Thread(f);
        t.start();
        System.out.println("111111");
        System.out.println(f.get());
        System.out.println("333333");
        //输出： 111111
        // 222222
        // 333333
        //说明 f.get() 会阻塞。
    }
}
```

**FutureTask 是 Future 的唯一实现类，用于实现 Thread。通过 Executor 可以实现进程池线程池的 Future 调用。**

``` java
public class Run {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService es = Executors.newFixedThreadPool(3);
        Future<String> fu = es.submit(()->{
            Thread.sleep(3000);
            return "bbb";
        });
        Future<String> fu2 = es.submit(()->"aaa");
        System.out.println(fu2.get());
        System.out.println(fu.get());
        // 输出 aaa
        // bbb
        es.shutdown();
    }
}
```

## 继承 Thread 类

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

``` java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
```

``` java
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

## 实现接口 VS 继承 Thread

实现接口会更好一些，因为：

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

# 二、基础线程机制

## Executor

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。