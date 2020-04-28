---
layout: post
title: ScheduledExecutorService 遇到异常后续任务不继续执行的问题
categories: Blogs
description: ScheduledExecutorService 遇到异常后续任务不继续执行的问题
keywords: java,定时线程池
---

## 背景
项目中有部分监控任务用了 java 中的 ScheduledExecutorService 对线程定时运行。但发现当程序运行时间长了后，线程不在定时运行。

## 原因
当线程报错，后面的任务不会再定时执行。代码中原文：

``` text
* If any execution of the task
* encounters（遇到） an exception, subsequent executions are suppressed（抑制）.
* Otherwise, the task will only terminate via cancellation or
* termination of the executor.  If any execution of this task
* takes longer than its period, then subsequent executions
* may start late, but will not concurrently execute.
```

## 验证
采用了[博客](https://blog.csdn.net/zmx729618/article/details/51436274)中的测试方法，可以证明当异常出现后，后面任务被取消。所以用到这里记着加 try catch 或 throws 掉。

## 延伸问题-当定时线程池内任务时间超长怎么办
这里用两断代码做验证：

``` java
package test;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class Test {

    private final static ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

    public static void main(String[] args) {
        scheduler.scheduleAtFixedRate(() -> {
            try {
                System.out.println(System.currentTimeMillis());
                //这里只暂停一秒
                Thread.sleep(1000);
            } catch (Throwable t) {
                System.out.println("Error");
            }

        }, 0, 2, TimeUnit.SECONDS);
    }
}
```

当暂停一秒时，可以看到输出如下，线程池仍然时间以 2 秒为间隔运行：

``` text
1588047076485
1588047078486
1588047080483
```

*这说明在定时线程池内，上一个任务可以及时运行结束时，是按照固定时间定时执行的。*

但将代码改为下面样式：

``` java
package test;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class Test {

    private final static ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

    public static void main(String[] args) {
        scheduler.scheduleAtFixedRate(() -> {
            try {
                System.out.println(System.currentTimeMillis());
                //暂停 3 秒，任务超时了
                Thread.sleep(3000);
            } catch (Throwable t) {
                System.out.println("Error");
            }

        }, 0, 2, TimeUnit.SECONDS);
    }
}
```

输出则是按照 3 秒为间隔输出，超时了：

```
1588047209003
1588047212007
1588047215010
```

## 原因分析
在新建一个定时线程池时，内部的队列使用的是：

``` java
static class DelayedWorkQueue extends AbstractQueue<Runnable>
        implements BlockingQueue<Runnable>
```

这是一个延迟队列，也就是线程池的 workingQueue 是一个延迟队列。

scheduleAtFixedRate 方法代码是：

``` java
ScheduledFutureTask<Void> sft = new ScheduledFutureTask<Void>(command,
                                null,
                                triggerTime(initialDelay, unit),
                                unit.toNanos(period));
RunnableScheduledFuture<Void> t = decorateTask(command, sft);
sft.outerTask = t;
delayedExecute(t);
```

把每一个 command 都封装了一层，告诉他们延迟的时间，triggerTime 会生成他的执行时间。同时 ScheduledFutureTask 中的 run 方法是这样的：

``` java
public void run() {
    boolean periodic = isPeriodic();
    if (!canRunInCurrentRunState(periodic))
        cancel(false);
    else if (!periodic)
        ScheduledFutureTask.super.run();
    else if (ScheduledFutureTask.super.runAndReset()) {
        setNextRunTime();
        //再次 ensurePrestart
        reExecutePeriodic(outerTask);
    }
}
```

也就是执行完后会再次设定下次执行时间再次执行。

之后对于首次启动时核心的方法就是 delayedExecute，传递的任务也都是封装后的了：

``` java
private void delayedExecute(RunnableScheduledFuture<?> task) {
    if (isShutdown())
        reject(task);
    else {
        super.getQueue().add(task);
        if (isShutdown() &&
            !canRunInCurrentRunState(task.isPeriodic()) &&
            remove(task))
            task.cancel(false);
        else
            ensurePrestart();
    }
}
```

其实就是将一个封装好的 task 加入到了延迟队列中。之后看 ensurePrestart（这个是 ThreadExecutorPoll 的方法）：

``` java
void ensurePrestart() {
    int wc = workerCountOf(ctl.get());
    if (wc < corePoolSize)
        addWorker(null, true);
    else if (wc == 0)
        addWorker(null, false);
}
```

核心就是 addWorker 了，即把每个线程封装成 worker 启动。addWorker 中执行 woker 的线程 run 方法。run 方法其实就是 runWorker，他会不停（在循环中取 task）的从之前的 delayedQueue 中拿要执行的任务，这些任务都是之前封装好的 ScheduledFutureTask，所以也就可以一直不停的执行了。当前面任务长的时候，后面 ScheduledFutureTask 放进任务自然会受到拖延。