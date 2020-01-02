---
layout: post
title: Java 同一个线程启动两次会发生什么
categories: Java
description: Java 同一个线程启动两次会发生什么
keywords: Java,线程
---

## 代码

``` java
class Solution extends Thread{
    
    public static void main(String[] args) throws Exception{
        Solution s = new Solution();
        s.start();
        s.start();
    }

    public void aa() throws Exception{
        System.out.println("11");
        synchronized(this){
            Thread.sleep(1111);
            System.out.println("22");
        }
    }

    public void run(){
        try{
            aa();
        }catch(Exception e){

        }
    }
}
```

上面代码输出会是什么？

## 输出

``` text
Exception in thread "main" 11
java.lang.IllegalThreadStateException
        at java.lang.Thread.start(Thread.java:708)
        at Solution.main(Solution.java:16)
22
```

## [原因分析](https://blog.csdn.net/zl1zl2zl3/article/details/80776112)

Java 的线程是不允许启动两次的，第二次调用必然会抛出 IllegalThreadStateException，这是一种运行时异常，多次调用 start 被认为是编程错误。

**线程必须在新建状态才能运行，即使上一个线程完成了也是不能再运行的。而上面的代码都是同一个线程对象，第二次不可能是在新建状态。**

线程周期：

- 新建（NEW），表示线程被创建出来还没真正启动的状态，可以认为它是个 Java 内部状态。
- 就绪（RUNNABLE），表示该线程已经在 JVM 中执行，当然由于执行需要计算资源，它可能是正在运行，也可能还在等待系统分配给它 CPU 片段，在就绪队列里面排队。
- 在其他一些分析中，会额外区分一种状态 RUNNING，但是从 Java API 的角度，并不能表示出来。
- 阻塞（BLOCKED），这个状态和我们前面两讲介绍的同步非常相关，阻塞表示线程在等待 Monitor lock。比如，线程试图通过 synchronized 去获取某个锁，但是其他线程已经独占了，那么当前线程就会处于阻塞状态。
- 等待（WAITING），表示正在等待其他线程采取某些操作。一个常见的场景是类似生产者消费者模式，发现任务条件尚未满足，就让当前消费者线程等待（wait），另外的生产者线程去准备任务数据，然后通过类似 notify 等动作，通知消费线程可以继续工作了。Thread.join() 也会令线程进入等待状态。
- 计时等待（TIMED_WAIT），其进入条件和等待状态类似，但是调用的是存在超时条件的方法，比如 wait 或 join 等方法的指定超时版本，如下面示例：public final native void wait(long timeout) throws InterruptedException;
- 终止（TERMINATED），不管是意外退出还是正常执行结束，线程已经完成使命，终止运行，也有人把这个状态叫作死亡。

在第二次调用 start() 方法的时候，线程可能处于终止或者其他（非 NEW）状态，但是不论如何，都是不可以再次启动的。

## 解决办法

建多个线程对象。这样不同线程对象就都处于新建状态。

``` java
class Solution{
    
    public static void main(String[] args) throws Exception{ // 这个 throws Exception 可以不要
        Solution s = new Solution();
        Runnable r = ()->{
            try{
                s.aa();
            }catch(Exception e){ // 必须处理，因为线程归 JVM 管不归 main 方法管，他的上层是 JVM 需要单独异常处理。

            }
        };
        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);
        t1.start();
        t2.start();
    }

    public void aa() throws Exception{
        System.out.println("11");
        synchronized(this){
            Thread.yield();
            System.out.println("22");
        }
    }
}
```

上面是用 Runnbale 的事例，其实直接下面这样也是可以的，只要不是一个线程对象：

``` java
Solution s1 = new Solution();
Solution s2 = new Solution();
s1.start();
s2.start();
```