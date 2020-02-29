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

[关于线程池非常详细的介绍](https://juejin.im/post/5aeec0106fb9a07ab379574f)

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

### 3. 同步中的类

``` java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

这时的 Monitor 是 SynchronizedExample 这个类（类加载器加载出来的同一个 class），锁住的也就是这个对象。锁住同一个对象时两个同步方法/块不可同时运行。

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

### 7. 异常释放

synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而 Lock 在发生异常时，如果没有主动通过 unLock() 去释放锁，则很可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁；

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

如果不加 synchronized 可能会出现以下情况：

``` java
class BlockingQueue {
    Queue<String> buffer = new LinkedList<String>();
 
    public void give(String data) {
        buffer.add(data);
        notify();                   // Since someone may be waiting in take!
    }
 
    public String take() throws InterruptedException {
        while (buffer.isEmpty())    // 不能用if，因为为了防止虚假唤醒
            wait();
        return buffer.remove();
    }
}
```

这段代码可能会导致如下问题：

1. 一个消费者调用 take，发现 buffer.isEmpty
2. 在消费者调用 wait 之前，由于 cpu 的调度，消费者线程被挂起，生产者调用 give，然后 notify
3. 然后消费者调用 wait（注意，由于错误的条件判断，导致 wait 调用在 notify 之后，这是关键）
4. 如果很不幸的话，生产者产生了一条消息后就不再生产消息了，那么消费者就会一直挂起，无法消费，造成死锁。

解决这个问题的方法就是：总是让give/notify和take/wait为原子操作。

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

### 等待队列

调用 obj 的 wait(), notify() 方法前，必须获得 obj 锁，也就是必须写在 synchronized(obj) 代码段内，等待队列相关的步骤和图：

![等待队列](/images/posts/knowledge/javaConcurrent/等待队列.jpeg)

*线程一的 wait 被 notify 之后会被重新加入同步队列，不是直接唤醒，等待被调用。*

``` java
package thread;

public class SynchronizedNotify {
    public String lock = "";
    public class Notify implements Runnable{

        @Override
        public void run() {
            synchronized (lock){
                System.out.println("notify start");
                lock.notifyAll();
                Thread.yield();
                System.out.println("Notify");
            }
        }
    }
    public class Wait implements Runnable{

        @Override
        public void run() {
            synchronized (lock){
                System.out.println("wait start");
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("wait");
            }
        }
    }

    public static void main(String[] args) {
        SynchronizedNotify synchronizedNotify = new SynchronizedNotify();
        new Thread(synchronizedNotify.new Notify()).start();
        new Thread(synchronizedNotify.new Notify()).start();
        new Thread(synchronizedNotify.new Notify()).start();
        new Thread(synchronizedNotify.new Notify()).start();
        new Thread(synchronizedNotify.new Notify()).start();
        new Thread(synchronizedNotify.new Wait()).start();
    }
}
```

结果是：

``` text
notify start
Notify
wait start
notify start
Notify
notify start
Notify
notify start
Notify
notify start
Notify
wait
```

### 同步队列的状态

- 当前线程想调用对象 A 的同步方法时，发现对象 A 的锁被别的线程占有，此时当前线程进入同步队列。简言之，同步队列里面放的都是想争夺对象锁的线程。
- 当一个线程 1 被另外一个线程 2 唤醒时，**1 线程进入同步队列，去争夺对象锁。**
- 同步队列是在同步的环境下才有的概念，一个对象对应一个同步队列。
- 线程等待时间到了或被 notify/notifyAll 唤醒后，会进入同步队列竞争锁，如果获得锁，进入 RUNNABLE（包含 running 和 ready）状态，否则进入 BLOCKED 状态等待获取锁。

### 几个方法的比较

1. Thread.sleep(long millis)，一定是当前线程调用此方法，当前线程进入 TIMED_WAITING 状态，但不释放对象锁，millis 后线程自动苏醒进入就绪状态。作用：给其它线程执行机会的最佳方式。
2. Thread.yield()，一定是当前线程调用此方法，当前线程放弃获取的CPU时间片，但不释放锁资源，由运行状态变为就绪状态，让OS再次选择线程。作用：让相同优先级的线程轮流执行，但并不保证一定会轮流执行。实际中无法保证 yield() 达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。Thread.yield() 不会导致阻塞。该方法与 sleep() 类似，只是不能由用户指定暂停多长时间。
3. thread.join()/thread.join(long millis)，当前线程里调用其它线程 t 的 join 方法，当前线程进入 WAITING/TIMED_WAITING 状态，当前线程不会释放已经持有的对象锁。线程 t 执行完毕或者 millis 时间到，当前线程一般情况下进入 RUNNABLE 状态，也有可能进入 BLOCKED 状态（因为 join 是基于 wait 实现的）。
4. obj.wait()，当前线程调用对象的 wait() 方法，当前线程释放对象锁，进入等待队列。依靠 notify()/notifyAll() 唤醒或者 wait(long timeout) timeout 时间到自动唤醒。
5. obj.notify() 唤醒在此对象监视器上等待的单个线程，选择是任意性的。notifyAll() 唤醒在此对象监视器上等待的所有线程。
6. LockSupport.park()/LockSupport.parkNanos(long nanos),LockSupport.parkUntil(long deadlines), 当前线程进入 WAITING/TIMED_WAITING 状态。对比 wait 方法，不需要获得锁就可以让线程进入 WAITING/TIMED_WAITING 状态，需要通过 LockSupport.unpark(Thread thread) 唤醒。

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

### 公平锁和非公平锁实现

[链接](https://www.jianshu.com/p/f584799f1c77)

# 六、线程状态

![线程状态](/images/posts/knowledge/javaConcurrent/线程状态.jpeg)

一个线程只能处于一种状态，并且这里的线程状态特指 Java 虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

## 1. 初始状态

实现 Runnable 接口和继承 Thread 可以得到一个线程类，new 一个实例出来，线程就进入了初始状态。

**只有在此状态下可以 start() 运行。所以当 start() 之后，这个对象不能再运行线程。**

## 2.1. 就绪状态

1. 就绪状态只是说你资格运行，调度程序没有挑选到你，你就永远是就绪状态。
2. 调用线程的 start() 方法，此线程进入就绪状态。
3. 当前线程 sleep() 方法结束，其他线程 join() 结束，等待用户输入完毕，某个线程拿到对象锁，这些线程也将进入就绪状态。
4. 当前线程时间片用完了，调用当前线程的 yield() 方法，当前线程进入就绪状态。
5. 锁池里的线程拿到对象锁后，进入就绪状态。

## 2.2. 运行中状态

线程调度程序从可运行池中选择一个线程作为当前线程时线程所处的状态。这也是线程进入运行状态的唯一一种方式。

## 3. 阻塞状态

阻塞状态是线程阻塞在进入 synchronized 关键字修饰的方法或代码块（获取锁）时的状态。

## 4. 等待

处于这种状态的线程不会被分配 CPU 执行时间，它们要等待被**显式地唤醒**，否则会处于无限期等待的状态。

| 进入方法 | 退出方法 |
| --- | --- |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕 |
| LockSupport.park() 方法 | LockSupport.unpark(Thread) |

> 注：LockSupport 是 JDK 中比较底层的类，用来创建锁和其他同步工具类的基本线程阻塞原语。java 锁和同步器框架的核心 AQS:AbstractQueuedSynchronizer（也就是 lock），就是通过调用 LockSupport.park() 和 LockSupport.unpark() 实现线程的阻塞和唤醒的。LockSupport 很类似于二元信号量（只有 1 个许可证可供使用），如果这个许可还没有被占用，当前线程获取许可并继续执行；如果许可已经被占用，当前线程阻塞，等待获取许可。

## 限期等待/超时等待

处于这种状态的线程不会被分配 CPU 执行时间，不过无须无限期等待被其他线程显示地唤醒，**在达到一定时间后它们会自动唤醒。**

| 进入方法 | 退出方法 |
| --- | --- |
| Thread.sleep() 方法 | 时间结束 |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll()  |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕 |
| LockSupport.parkNanos() 方法 | LockSupport.unpark(Thread) |
| LockSupport.parkUntil() 方法 | LockSupport.unpark(Thread) |

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，同时释放锁，常常用“挂起一个线程”进行描述。**睡眠和挂起**是用来描述行为，而**阻塞和等待**用来描述状态。

## 死亡（TERMINATED）

1. 当线程的 run() 方法完成时，或者主线程的 main() 方法完成时，我们就认为它终止了。这个线程对象也许是活的，但是，它已经不是一个单独执行的线程。线程一旦终止了，就不能复生。
2. 在一个终止的线程上调用 start() 方法，会抛出 java.lang.IllegalThreadStateException异常。

# 七、J.U.C - AQS

java.util.concurrent（J.U.C）大大提高了并发性能，AQS 被认为是 J.U.C 的核心。

[AQS 基础知识](http://fuhuacn.top/2020/01/01/aqs/)

**AQS主要提供了如下一些方法：**

## CountDownLatch

用来控制一个或者多个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，**减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。**

![CountDownLatch](/images/posts/knowledge/javaConcurrent/coundownlatch.png)

``` java
public class CountdownLatchExample {

    public static void main(String[] args) throws InterruptedException {
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("end");
        executorService.shutdown();
    }
}
```

``` text
run..run..run..run..run..run..run..run..run..run..end
```

## CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。在这类方法中再考虑同步问题是没有意义的，因为需要每一个都 await 后才触发。而如果同步了，await 就会一直等待。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。

``` java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

![CyclicBarrier](/images/posts/knowledge/javaConcurrent/CyclicBarrier.png)

``` java
public class CyclicBarrierExample {

    public static void main(String[] args) {
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before..");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.print("after..");
            });
        }
        executorService.shutdown();
    }
}
```

``` text
before..before..before..before..before..before..before..before..before..before..after..after..after..after..after..after..after..after..after..after..
```

## Semaphore

Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

以下代码模拟了对某个服务的并发请求，每次只能有 3 个客户端同时访问，请求总数为 10。

``` java
public class SemaphoreExample {

    public static void main(String[] args) {
        final int clientCount = 3;
        final int totalRequestCount = 10;
        Semaphore semaphore = new Semaphore(clientCount);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalRequestCount; i++) {
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    System.out.print(semaphore.availablePermits() + " ");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}
```

``` text
1 0 1 1 1 1 2 2 2 2
```

所以最多同时运行三个线程。

# 八、J.U.C - 其它组件

## FutureTask

在介绍 Callable 时我们知道它可以有返回值，返回值通过 Future 进行封装。FutureTask 实现了 RunnableFuture 接口，该接口继承自 Runnable 和 Future 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值。

``` java
public class FutureTask<V> implements RunnableFuture<V>
```

``` java
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask 可用于异步获取执行结果或取消执行任务的场景。当一个计算任务需要执行很长时间，那么就可以用 FutureTask 来封装这个任务，主线程在完成自己的任务之后再去获取结果。

``` java
public class FutureTaskExample {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for (int i = 0; i < 100; i++) {
                    Thread.sleep(10);
                    result += i;
                }
                return result;
            }
        });

        Thread computeThread = new Thread(futureTask);
        computeThread.start();

        Thread otherThread = new Thread(() -> {
            System.out.println("other task is running...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        otherThread.start();
        System.out.println(futureTask.get()); // 这个方法在没执行完前会阻塞
    }
}
```

``` text
other task is running...
4950
```

## BlockingQueue

java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

- FIFO 队列 ：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
- 优先级队列 ：PriorityBlockingQueue

提供了阻塞的 **take()** 和 **put()** 方法：如果队列为空 take() 将阻塞，直到队列中有内容；如果队列为满 put() 将阻塞，直到队列有空闲位置。

**使用 BlockingQueue 实现生产者消费者问题**

``` java
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
}
```

``` java
public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
        Producer producer = new Producer();
        producer.start();
    }
    for (int i = 0; i < 5; i++) {
        Consumer consumer = new Consumer();
        consumer.start();
    }
    for (int i = 0; i < 3; i++) {
        Producer producer = new Producer();
        producer.start();
    }
}
```

``` text
produce..produce..consume..consume..produce..consume..produce..consume..produce..consume..
```

## ForkJoin

主要用于并行计算中，和 MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。类似于归并排序。

``` java
public class ForkJoinExample extends RecursiveTask<Integer> {

    private final int threshold = 5;
    private int first;
    private int last;

    public ForkJoinExample(int first, int last) {
        this.first = first;
        this.last = last;
    }

    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            // 任务足够小则直接计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            // 拆分成小任务
            int middle = first + (last - first) / 2;
            ForkJoinExample leftTask = new ForkJoinExample(first, middle);
            ForkJoinExample rightTask = new ForkJoinExample(middle + 1, last);
            leftTask.fork();
            rightTask.fork();
            result = leftTask.join() + rightTask.join();
        }
        return result;
    }
}
```

``` java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ForkJoinExample example = new ForkJoinExample(1, 10000);
    ForkJoinPool forkJoinPool = new ForkJoinPool();
    Future result = forkJoinPool.submit(example);
    System.out.println(result.get());
}
```

ForkJoin 使用 ForkJoinPool 来启动，它是一个特殊的线程池，线程数量取决于 CPU 核数。

``` java
public class ForkJoinPool extends AbstractExecutorService
```

ForkJoinPool 实现了工作窃取算法来提高 CPU 的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。例如下图中，Thread2 从 Thread1 的队列中拿出最晚的 Task1 任务，Thread1 会拿出 Task2 来执行，这样就避免发生竞争。但是如果队列中只有一个任务时还是会发生竞争。

![ForkJoin](/images/posts/knowledge/javaConcurrent/forkjoin.png)

# 九、线程不安全示例

如果多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的。

以下代码演示了 1000 个线程同时对 cnt 执行自增操作，操作结束之后它的值有可能小于 1000。

``` java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    ThreadUnsafeExample example = new ThreadUnsafeExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

输出并不是 1000。

# 十、Java 内存模型

Java 内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

## 主内存与工作内存

Java内存模型规定了所有的变量都存储在主内存中，此处的主内存仅仅是虚拟机内存的一部分，而虚拟机内存也仅仅是计算机物理内存的一部分（为虚拟机进程分配的那一部分）。

Java内存模型分为主内存，和工作内存。主内存是所有的线程所共享的，工作内存是每个线程自己有一个，不是共享的。

每条线程还有自己的工作内存，线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝。线程对变量的所有操作（读取、赋值），都必须在工作内存中进行，而不能直接读写主内存中的变量。不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成，线程、主内存、工作内存三者之间的交互关系如下图：

![主内存与工作内存](/images/posts/knowledge/javaConcurrent/工作内存.png)

## 内存间交互操作

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

![内存间交互操作](/images/posts/knowledge/javaConcurrent/内存间交互操作.png)

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：内存屏障（memory barrier）是一个CPU指令。基本上，它是这样一条指令： a) 确保一些特定操作执行的顺序； b) 影响一些数据的可见性(可能是某些指令执行后的结果)。一个写屏障会把这个屏障前写入的数据刷新到缓存，这样任何试图读取该数据的线程将得到最新值，而不用考虑到底是被哪个cpu核心或者哪颗CPU执行的。

## 内存模型三大特性

### 1. 原子性

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

有一个错误认识就是，int 等原子性的类型在多线程环境中不会出现线程安全问题。前面的线程不安全示例代码中，cnt 属于 int 类型变量，1000 个线程对它进行自增操作之后，得到的值为 997 而不是 1000。

为了方便讨论，将内存间的交互操作简化为 3 个：load、assign、store。

下图演示了两个线程同时对 cnt 进行操作，load、assign、store 这一系列操作整体上看不具备原子性，那么在 T1 修改 cnt 并且还没有将修改后的值写入主内存，T2 依然可以读入旧值。可以看出，这两个线程虽然执行了两次自增运算，但是主内存中 cnt 的值最后为 1 而不是 2。因此对 int 类型读写操作满足原子性只是说明 load、assign、store 这些单个操作具备原子性。

![主内存读到线程](/images/posts/knowledge/javaConcurrent/主内存读出.jpeg)

AtomicInteger 能保证多个线程修改的原子性。

``` java
public class AtomicExample {
    private AtomicInteger cnt = new AtomicInteger();

    public void add() {
        cnt.incrementAndGet();
    }

    public int get() {
        return cnt.get();
    }
}
```

``` java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicExample example = new AtomicExample(); // 只修改这条语句
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

AtomicIntger 用 CAS 保证线程安全，输出 1000。

除了使用原子类之外，也可以使用 synchronized 互斥锁来保证操作的原子性。它对应的内存间交互操作为：lock 和 unlock，在虚拟机实现上对应的字节码指令为 monitorenter 和 monitorexit。

``` java
public class AtomicSynchronizedExample {
    private int cnt = 0;

    public synchronized void add() {
        cnt++;
    }

    public synchronized int get() {
        return cnt;
    }
}
```

``` java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicSynchronizedExample example = new AtomicSynchronizedExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

结果同样是 1000。

### 2. 可见性

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有三种实现可见性的方式：

- volatile，1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，会立刻刷新回主内存，其他线程的工作内存失效，使得这新值对其他线程来说是立即可见的。2）禁止进行指令重排序。
- synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

对前面的线程不安全示例中的 cnt 变量使用 volatile 修饰，不能解决线程不安全问题，因为 volatile 并不能保证操作的原子性。从 Load 到 store 到 lock（内存屏障），一共 4 步，其中最后一步jvm让这个最新的变量的值在所有线程可见，**也就是最后一步让所有的 CPU 内核都获得了最新的值，但中间的几步（从Load到Store）是不安全的，中间如果其他的CPU修改了值将会丢失。**

### 3. 有序性

#### JVM 之指令重排分析:

**什么是指令重排：**

在计算机执行指令的顺序在经过程序编译器编译之后形成的指令序列，一般而言，这个指令序列是会输出确定的结果；以确保每一次的执行都有确定的结果。但是，一般情况下，CPU和编译器为了提升程序执行的效率，会按照一定的规则允许进行指令优化，在某些情况下，这种优化会带来一些执行的逻辑问题，主要的原因是代码逻辑之间是存在一定的先后顺序，在并发执行情况下，会发生二义性，即按照不同的执行逻辑，会得到不同的结果信息。

**数据依赖性：**

主要指不同的程序指令之间的顺序是不允许进行交互的，即可称这些程序指令之间存在数据依赖性。 

主要的例子如下：

``` text
名称 	代码示例 	说明
写后读 	a = 1;b = a; 	写一个变量之后，再读这个位置。
写后写 	a = 1;a = 2; 	写一个变量之后，再写这个变量。
读后写 	a = b;b = 1; 	读一个变量之后，再写这个变量。
```

进过分析，发现这里每组指令中都有写操作，这个写操作的位置是不允许变化的，否则将带来不一样的执行结果。

编译器将不会对存在数据依赖性的程序指令进行重排，这里的依赖性仅仅指单线程情况下的数据依赖性；多线程并发情况下，此规则将失效。

**as-if-serial语义：**

不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器，runtime 和处理器都必须遵守as-if-serial语义。

分析：关键词是单线程情况下，必须遵守；其余的不遵守。

``` java
double pi  = 3.14;    //A
double r   = 1.0;     //B
double area = pi * r * r; //C
```

分析代码：A->C  B->C；A,B之间不存在依赖关系； 故在单线程情况下， A与B的指令顺序是可以重排的，C不允许重排，必须在A和B之后。

结论性的总结：

as-if-serial语义把单线程程序保护了起来，遵守as-if-serial语义的编译器，runtime 和处理器共同为编写单线程程序的程序员创建了一个幻觉：单线程程序是按程序的顺序来执行的。as-if-serial语义使单线程程序员无需担心重排序会干扰他们，也无需担心内存可见性问题。

核心点还是单线程，多线程情况下不遵守此原则。

#### 解决有序性方法

有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

## 先行发生原则 happens-before

上面提到了可以用 volatile 和 synchronized 来保证有序性。除此之外，JVM 还规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。通过这个原则帮助解决有序性问题。

### 1. 单一线程原则 Single Thread rule

在一个线程内，在程序前面的操作先行发生于后面的操作。

这个应该是程序看起来执行的顺序是按照代码顺序执行的，因为虚拟机可能会对程序代码进行指令重排序。虽然进行重排序，但是最终执行的结果是与程序顺序执行的结果一致的，它只会对不存在数据依赖性的指令进行重排序。因此，在单个线程中，程序执行看起来是有序执行的，这一点要注意理解。事实上，这个规则是用来保证程序在单线程中执行结果的正确性，但无法保证程序在多线程中执行的正确性。

### 2. 管程锁定规则 Monitor Lock Rule

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。

### 3. volatile 变量规则 Volatile Variable Rule

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

### 4. 线程启动规则 Thread Start Rule

Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

### 5. 线程加入规则 Thread Join Rule

Thread 对象的结束先行发生于 join() 方法返回。

### 6. 线程中断规则 Thread Interruption Rule

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

### 7. 对象终结规则 Finalizer Rule

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

### 8. 传递性 Transitivity

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

# 十一、线程安全

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

## 不可变

不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

- final 关键字修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long，Double 和 Integer 等数值包装类型，- BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

对于集合类型，可以使用 Collections.unmodifiableXXX() 方法来获取一个不可变的集合。

``` java
public class ImmutableExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);
        unmodifiableMap.put("a", 1);
    }
}
```

会抛出异常：

``` text
Exception in thread "main" java.lang.UnsupportedOperationException
    at java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
    at ImmutableExample.main(ImmutableExample.java:9)
```

Collections.unmodifiableXXX() 先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。

``` java
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}
```

## 互斥同步

synchronized 和 ReentrantLock。

Synchronized 来说，它是 java 语言的关键字，是原生语法层面的互斥，需要 jvm 实现。而 ReentrantLock 它是 JDK 1.5 之后提供的 API 层面的互斥锁，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成。

使用 synchronized。如果 Thread1 不释放，Thread2 将一直等待，不能被中断。synchronized 也可以说是 Java 提供的原子性内置锁机制。内部锁扮演了互斥锁（mutual exclusion lock ，mutex）的角色，一个线程引用锁的时候，别的线程阻塞等待。使用 ReentrantLock。如果 Thread1 不释放，Thread2 等待了很长时间以后，可以中断等待，转而去做别的事情。

synchronized 的锁是非公平锁，ReentrantLock 默认情况下也是非公平锁，但可以通过带布尔值的构造函数要求使用公平锁。

ReentrantLock 可以同时绑定多个 Condition 对象，只需多次调用 newCondition 方法即可。synchronized中，锁对象的 wait() 和 notify() 或 notifyAll() 方法可以实现一个隐含的条件。但如果要和多于一个的条件关联的时候，就不得不额外添加一个锁。

## 非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

### 1. CAS

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

### 2. AtomicInteger

J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

以下代码使用了 AtomicInteger 执行了自增的操作。

``` java
private AtomicInteger cnt = new AtomicInteger();

public void add() {
    cnt.incrementAndGet();
}
```

以下代码是 incrementAndGet() 的源码，它调用了 Unsafe 的 getAndAddInt() 。

``` java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。

``` java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

### 3. ABA

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。

``` text
ABA问题带来的危害：
小明在提款机，提取了50元，因为提款机问题，有两个线程，同时把余额从100变为50
线程1（提款机）：获取当前值100，期望更新为50，
线程2（提款机）：获取当前值100，期望更新为50，
线程1成功执行，线程2某种原因block了，这时，某人给小明汇款50
线程3（默认）：获取当前值50，期望更新为100，
这时候线程3成功执行，余额变为100，
线程2从Block中恢复，获取到的也是100，compare之后，继续更新余额为50！！！
此时可以看到，实际余额应该为100（100-50+50），但是实际上变为了50（100-50+50-50）这就是ABA问题带来的成功提交。
```

## 无同步方案

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

### 1. 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

``` java
public class StackClosedExample {
    public void add100() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt++;
        }
        System.out.println(cnt);
    }
}
```

``` java
public static void main(String[] args) {
    StackClosedExample example = new StackClosedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> example.add100());
    executorService.execute(() -> example.add100());
    executorService.shutdown();
}
```

``` text
100
100
```

### 2. 线程本地存储（Thread Local Storage）

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。

对于以下代码，thread1 中设置 threadLocal 为 1，而 thread2 设置 threadLocal 为 2。过了一段时间之后，thread1 读取 threadLocal 依然是 1，不受 thread2 的影响。

``` java
public class ThreadLocalExample {
    public static void main(String[] args) {
        ThreadLocal threadLocal = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal.set(1);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
            threadLocal.remove();
        });
        Thread thread2 = new Thread(() -> {
            threadLocal.set(2);
            threadLocal.remove();
        });
        thread1.start();
        thread2.start();
    }
}
```

``` java
1
```

为了理解 ThreadLocal，先看以下代码：

``` java
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
            try {
                Thread.sleep(10);
            } catch (Exception e) {
                //TODO: handle exception
            }
            threadLocal2.set(3);
            System.out.println("thread1"+threadLocal2.get()); // 输出仍为 2，thread1 设置并不会生效
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
            try{
                Thread.sleep(100);
            }catch(Exception e){

            }
            System.out.println("thread2"+threadLocal2.get()); // 输出仍为 2，thread1 设置并不会生效
        });
        thread1.start();
        thread2.start();
    }
}
```

输出仍为 2。

``` text
2
```

可以看出每个 ThreadLocal 的赋值只对给他赋值的线程生效。

它所对应的底层结构图为：

![ThreadLocal](/images/posts/knowledge/javaConcurrent/threadLocal.png)

*所以说没线程设置的 ThreadLocal 都是自己线程的。*

每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。

``` java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中。

``` java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

get() 方法类似。

``` java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。

在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。

#### threadLocal 的内存泄漏

每个 thread 中都存在一个 map, map 的类型是 ThreadLocal.ThreadLocalMap。Map 中的 key 为一个 threadlocal 实例。这个 Map 的确使用了弱引用,不过弱引用只是针对 key（threadLocal）。每个 key 都弱引用指向 threadlocal。当把 threadlocal 实例置为 null 以后,没有任何强引用指向 threadlocal 实例（因为那个 map 中的 key 是弱引用），所以 threadlocal 将会被 gc 回收。但是，我们的 value 却不能回收，因为存在一条从 current thread 连接过来的强引用（在 ThreadLocal.ThreadLocalMap 中）。只有当前 thread 结束以后，current thread 就不会存在栈中，强引用断开，Current Thread、Map、value 将全部被 GC 回收。**所以得出一个结论就是只要这个线程对象被 gc 回收，就不会出现内存泄露，但在 threadLocal 设为 null 和线程结束这段时间不会被回收的，就发生了我们认为的内存泄露。**其实这是一个对概念理解的不一致，也没什么好争论的。最要命的是线程对象不被回收的情况，这就发生了真正意义上的内存泄露。比如使用线程池的时候，线程结束是不会销毁的，会再次使用的就可能出现内存泄露 。（在 web 应用中，每次 http 请求都是一个线程，tomcat 容器配置使用线程池时会出现内存泄漏问题）

下面代码在一个线程内建立了两个 ThreadLocal。

``` java
ThreadLocal<Integer> tl1 = new ThreadLocal<>();
ThreadLocal<Integer> tl1 = new ThreadLocal<>();
```

对应到线程中（Thread 类）的 Map：

``` java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

看一下 ThreadLocalMap 类，可以看到 ThreadLocal 是弱引用，也就是 Entry 的 key 可能会被回收掉。

``` java
static class ThreadLocalMap {

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
...
```

所以可以看见在 ThreadLocalMap 中有两个 get 方法，为了应对有可能 key 被回收（如果回收了 value 也会被扫描到一并回收），防止内存泄漏。

``` java
/**
    * Get the entry associated with key.  This method
    * itself handles only the fast path: a direct hit of existing
    * key. It otherwise relays to getEntryAfterMiss.  This is
    * designed to maximize performance for direct hits, in part
    * by making this method readily inlinable.
    *
    * @param  key the thread local object
    * @return the entry associated with key, or null if no such
    */
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

/**
    * Version of getEntry method for use when key is not found in
    * its direct hash slot.
    *
    * @param  key the thread local object
    * @param  i the table index for key's hash code
    * @param  e the entry at table[i]
    * @return the entry associated with key, or null if no such
    */
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);// 这个方法会清楚所有 key 为 null，即 threadLocal 被回收了的。
        else // 这里是典型的线性地址探测法
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

注意这里的 key 是 ThreadLocal，他被回收只有可能是在调用代码手动调用了 threadLocal = null。也就是这里的弱引用的作用是当 threadLocal 为 null 时，ThreadLocalMap 中可以被清除掉防止内存泄漏。

**之前好奇 WeakReference 不是弱引用吗，key 有可能随时被回收啊。但要想到在你声明的 threadLocal 可是强引用。所以只有当声明的 threadLocal 设为 null 的时候，弱引用才能被回收。**

可以看到线程中所有的 threadLocal 对应的值都在 Thread 类的 ThreadLocalMap 中。所以如果这里不是弱引用，有可能出现 threadLocal 设为 null 但 ThreadLocalMap 中还被引用所以不能回收导致内存泄漏的问题。所以这里要设成弱引用。

![threadLocal内存](/images/posts/knowledge/javaConcurrent/threadLocal内存泄漏.png)

### 3. 可重入代码（Reentrant Code）

这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

# 十二、锁优化

这里的锁优化主要是指 JVM 对 synchronized 的优化。

## 自旋锁

互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

## 锁消除

锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

对于一些看起来没有加锁的代码，其实隐式的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：

``` java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作：

``` java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

每个 append() 方法中都有一个同步块（方法被 synchronized 修饰）。虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString() 方法内部。也就是说，sb 的所有引用永远不会逃逸到 concatString() 方法之外，其他线程无法访问到它，因此可以进行消除。

## 锁粗化

即将某一部分的锁（比如就一行）扩展到一块（几行，因为这几行一直都被加锁）。这样从多次的加锁解锁变成了一次加锁解锁。

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上一节的示例代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

## JAVA synchronized 的改变

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态（unlocked）-> 偏向锁状态（biasble）-> 轻量级锁状态（lightweight locked）-> 重量级锁状态（inflated）。

以下是 HotSpot 虚拟机对象头的内存布局，这些数据被称为 Mark Word。其中 tag bits 对应了五个状态，这些状态在右侧的 state 表格中给出。除了 marked for gc 状态，其它四个状态已经在前面介绍过了。

![轻量级锁](/images/posts/knowledge/javaConcurrent/轻量级锁.png)

下图左侧是一个线程的虚拟机栈，其中有一部分称为 Lock Record 的区域，这是在轻量级锁运行过程创建的，用于存放锁对象的 Mark Word。而右侧就是一个锁对象，包含了 Mark Word 和其它信息。

![轻量级锁2](/images/posts/knowledge/javaConcurrent/轻量级锁2.png)

### 偏向锁

偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为 1 01。同时使用 CAS 操作将线程 ID 记录到 Mark Word 中，如果 CAS 操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向（Revoke Bias）后恢复到未锁定状态或者轻量级锁状态。

![偏向锁](/images/posts/knowledge/javaConcurrent/偏向锁.jpeg)

### 轻量级锁

轻量级锁是相对于传统的重量级锁而言，它使用 CAS 操作来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用 CAS 操作进行同步，如果 CAS 失败了再改用互斥量进行同步。

当尝试获取一个锁对象时，如果锁对象标记为 0 01，说明锁对象的锁未锁定（unlocked）状态。此时虚拟机在当前线程的虚拟机栈中创建 Lock Record，然后使用 CAS 操作将对象的 Mark Word 更新为 Lock Record 指针。如果 CAS 操作成功了，那么线程就获取了该对象上的锁，并且对象的 Mark Word 的锁标记变为 00，表示该对象处于轻量级锁状态。

![轻量锁检查](/images/posts/knowledge/javaConcurrent/轻量锁检查.png)

如果 CAS 操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的虚拟机栈，如果是的话说明当前线程已经拥有了这个锁对象，那就可以直接进入同步块继续执行，否则说明这个锁对象已经被其他线程线程抢占了。如果有两条以上的线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁。

![优化汇总](/images/posts/knowledge/javaConcurrent/synchronized优化汇总.jpeg)

# 十三、多线程开发良好的实践

- 给线程起个有意义的名字，这样可以方便找 Bug。
- 缩小同步范围，从而减少锁争用。例如对于 synchronized，应该尽量使用同步块而不是同步方法。
- 多用同步工具少用 wait() 和 notify()。首先，CountDownLatch, CyclicBarrier, Semaphore 和 Exchanger 这些同步类简化了编码操作，而用 wait() 和 notify() 很难实现复杂控制流；其次，这些同步类是由最好的企业编写和维护，在后续的 JDK 中还会不断优化和完善。
- 使用 BlockingQueue 实现生产者消费者问题。
- 多用并发集合少用同步集合，例如应该使用 ConcurrentHashMap 而不是 Hashtable。
- 使用本地变量和不可变类来保证线程安全。
- 使用线程池而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限的线程来启动任务。