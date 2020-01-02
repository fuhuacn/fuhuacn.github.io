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

FutureTask 实现 RunnableFuture，RunnableFuture 继承 Runnable, Future。也就是实际上 Thread 内还是 Runnable。

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

主要有三种 Executor：

- CachedThreadPool：CachedThreadPool 是通过 java.util.concurrent.Executors 创建的 ThreadPoolExecutor 实例。这个实例会根据需要，在线程可用时，重用之前构造好的池中线程。这个线程池在执行 大量短生命周期的异步任务时（many short-lived asynchronous task），可以显著提高程序性能。调用 execute时，可以重用之前已构造的可用线程，如果不存在可用线程，那么会重新创建一个新的线程并将其加入到线程池中。如果线程超过 60 秒还未被使用，就会被中止并从缓存中移除。因此，线程池在长时间空闲后不会消耗任何资源。一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

``` java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

**[execute() 和 submit() 的区别：](https://blog.csdn.net/guhong5153/article/details/71247266)**

- execute提交的方式只能提交一个 Runnable 的对象，且该方法的返回值是void，也即是提交后如果线程运行后，和主线程就脱离了关系了，当然可以设置一些变量来获取到线程的运行结果。并且当线程的执行过程中抛出了异常通常来说主线程也无法获取到异常的信息的，只有通过 ThreadFactory 主动设置线程的异常处理类才能感知到提交的线程中的异常信息。

- submit提交的方式有如下三种情况
  - <T> Future<T> submit(Callable<T> task);
    这种提交的方式是提交一个实现了 Callable 接口的对象，这种提交的方式会返回一个 Future 对象，这个 Future 对象代表这线程的执行结果。当主线程调用 Future 的 get 方法的时候会获取到从线程中返回的结果数据。如果在线程的执行过程中发生了异常，get会获取到异常的信息。
  - Future<?> submit(Runnable task);
    也可以提交一个 Runable 接口的对象，这样当调用 get 法的时候，如果线程执行成功会直接返回 null，如果线程执行异常会返回异常的信息。
  - <T> Future<T> submit(Runnable task, T result);
    除了 task 之外还有一个 result 对象，当线程正常结束的时候调用 Future 的 get 方法会返回 result 对象，当线程抛出异常的时候会获取到对应的异常的信息。

## Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于**非**守护线程。

在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。

``` java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```

## sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，**因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。**线程中抛出的其它异常也同样需要在本地进行处理。

``` java
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

## yield()

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

``` java
public void run() {
    Thread.yield();
}
```

# 三、中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

## InterruptedException

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

``` java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

``` java
public static void main(String[] args) throws Exception{
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    // interrupt 中断后会自动复位
    System.out.println(thread1.isInterrupted());
    System.out.println("Main run");
}
```

``` text
false
Main run
java.lang.InterruptedException: sleep interrupted
        at java.lang.Thread.sleep(Native Method)
        at Solution$MyThread1.run(Solution.java:26)
```

## interrupted()

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

> 1. 调用线程的interrupt方法，并不能真正中断线程，只是给线程做了中断状态的标志  
> 2. Thread.interrupted()：测试当前线程是否处于中断状态。执行后将中断状态标志为false  
> 3. Thread.isInterrupte()： 测试线程Thread对象是否已经处于中断状态。但不具有清除功能

``` java
public class InterruptExample {

    private static class MyThread2 extends Thread {
        @Override
        public void run() {
            // Thread.interrupted()：测试当前线程是否处于中断状态。执行后将中断状态标志为false
            while (!interrupted()) {
                // ..
            }
            System.out.println("Thread end");
        }
    }
}
```

``` java
public static void main(String[] args) throws Exception{
    Thread thread2 = new MyThread2();
    thread2.start();
    thread2.interrupt();
    Thread.yield();
    System.out.println(thread2.isInterrupted()); // interrupted() 方法判断为 true 后又复原回了 flase
}
```

``` text
Thread end
false
```

## Executor 的中断操作

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

``` java
public static void main(String[] args) throws Exception{
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.submit(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.submit(() -> {
        while(true){
            System.out.println("x");
            Thread.sleep(1);// 如果没有这个 sleep() 则无法接收到 interrupt() 就中端掉，所以 shutdownNow() 也就不能停止。
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}
```

``` text
x
Main run
java.lang.InterruptedException: sleep interrupted
        at java.lang.Thread.sleep(Native Method)
        at Solution.lambda$0(Solution.java:18)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
```

如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。

> cancel() 中传入 false 参数只能取消还没有开始的任务，若任务已经开始了，就任由其运行下去。

``` java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```

# 四、互斥同步

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

## synchronized

### 1. 同步一个代码块

``` java
public void func() {
    // ...
    synchronized (this) {
        // ... 只同步这个块内的
    }
}
```

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

``` java
public class SynchronizedExample {
    public void func1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

``` java
public static void main(String[] args) throws Exception{ // 这个 throws Exception 可以不要
    Solution s = new Solution();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> s.func1());
    executorService.execute(() -> s.func1());
    executorService.shutdown();
}
```

输出：

``` text
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

可以看到都是按顺序输出的。

对于以下代码，两个线程调用了**不同对象**的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。

``` java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e2.func1());
}
```

输出：

``` java
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

### 2. 同步一个方法

``` java
public synchronized void func () {
    // ...
}
```

它和同步代码块一样。作用于 this 对象。

### 同步中的类

``` java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

这时的 Monitor 是 SynchronizedExample 这个类，锁住的也就是这个对象。锁住同一个对象时两个同步方法/块不可同时运行。

作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

``` java
public class SynchronizedExample {

    public void func2() {
        // 这个地方不是 this 了
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

``` java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    // 如果是 this 此时就锁住的不同对象，会乱序
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}
```

``` text
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

### 4. 同步一个静态方法

``` java
public synchronized static void fun() {
    // ...
}
```

此时锁住的是 .class 即类文件而不是对象。同一个类的都会同步。

## ReentrantLock

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

``` java
public class LockExample {

    private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock(); // 确保释放锁，从而避免发生死锁。
        }
    }
}
```

``` java
public static void main(String[] args) {
    LockExample lockExample = new LockExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> lockExample.func());
    executorService.execute(() -> lockExample.func());
}
```

输出：

``` text
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

## 比较

### 1. 锁的实现

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。都是可重入锁即如果是锁同一个对象，可重复可递归调用的锁。

### 2. 性能

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

### 3. 等待可中断

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断（有 tryLock() 和 tryLock(long timeout,TimeUnit unit) 方法），而 synchronized 不行。

### 4. 公平锁

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

``` java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

### 5. 锁绑定多个条件

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

``` java
Lock lock = new ReentrantLock();
Condition emptyCondition = lock.newCondition();

lock.lock();
try {
    while (!b){
        emptyCondition.await();
    }
} finally {
    lock.unlock();
}

lock.lock();
try {
    b = true;
    emptyCondition.signalAll();
} finally {
    lock.unlock();
}
```

而 synchronized 只有 wait() 和 notify()。

### 6. 实现

ReetrantLock 基于 AQS，synchronized 基于 JVM 虚拟机的 Monitor 实现。

### 选择

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

# 五、线程之间的协作

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

## join()

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

``` java
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();// 所以需要先让 A 跑完
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}
```

``` java
public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
```

输出：

``` text
A
B
```

## wait() notify() notifyAll()

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会**释放锁**。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

``` java
public class WaitNotifyExample {

    public synchronized void before() {
        System.out.println("before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("after");
    }
}
```

``` java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    WaitNotifyExample example = new WaitNotifyExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```

输出：

``` text
before
after
```

**wait() 和 sleep() 的区别：**

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

## await() signal() signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件即指定 await 和 signal 的 Condition，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。

``` java
public class AwaitSignalExample {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    // private Condition condition2 = lock.newCondition(); 如果底下一个 condition 一个 condition2 after 就无法被唤醒


    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

``` java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```

``` text
before
after
```

# 六、线程状态

一个线程只能处于一种状态，并且这里的线程状态特指 Java 虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

## 新建（NEW）

创建后尚未启动。**只有在此状态下可以 start() 运行。所以当 start() 之后，这个对象不能再运行线程。**

## 可运行（RUNABLE）

正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。**所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。**

## 阻塞（BLOCKED）

请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 RUNABLE 需要其他线程释放 monitor lock。

## 无限期等待（WAITING）

等待其它线程显式地唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取 monitor lock。而等待是主动的，通过调用 Object.wait() 等方法进入。

| 进入方法 | 退出方法 |
| --- | --- |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕 |
| LockSupport.park() 方法 | LockSupport.unpark(Thread) |

> 注：LockSupport 是 JDK 中比较底层的类，用来创建锁和其他同步工具类的基本线程阻塞原语。java 锁和同步器框架的核心 AQS:AbstractQueuedSynchronizer（也就是 lock），就是通过调用 LockSupport.park() 和 LockSupport.unpark() 实现线程的阻塞和唤醒的。LockSupport 很类似于二元信号量（只有 1 个许可证可供使用），如果这个许可还没有被占用，当前线程获取许可并继续执行；如果许可已经被占用，当前线程阻塞，等待获取许可。

## 限期等待（TIMED_WAITING）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法 | 退出方法 |
| --- | --- |
| Thread.sleep() 方法 | 时间结束 |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll()  |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕 |
| LockSupport.parkNanos() 方法 | LockSupport.unpark(Thread) |
| LockSupport.parkUntil() 方法 | LockSupport.unpark(Thread) |

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。**睡眠和挂起**是用来描述行为，而**阻塞和等待**用来描述状态。

## 死亡（TERMINATED）

可以是线程结束任务之后自己结束，或者产生了异常而结束。

# 七、J.U.C - AQS

java.util.concurrent（J.U.C）大大提高了并发性能，AQS 被认为是 J.U.C 的核心。



**AQS主要提供了如下一些方法：**

## CountDownLatch

用来控制一个或者多个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。