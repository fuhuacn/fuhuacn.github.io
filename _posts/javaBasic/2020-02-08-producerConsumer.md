---
layout: post
title: Java 生产者消费者的三种方式
categories: Java
description: Java 生产者消费者的三种方式
keywords: JAVA,生产者消费者
---

# synchronized

这里用一个 size 代替了存储的内容。

``` java
package consumerProducer;

import java.util.LinkedList;

public class SynchronizedWay {
    String LOCK = "lock";
    int full = 2;
    int size = 0;

    class Producer implements Runnable {
        public void run(){
            synchronized (LOCK){
                while(size==full){ // 一定得是 while，因为不能指定 notify 谁，所以有可能 Producer 把 Producer 也唤醒了
                    System.out.println("生产者等待！");
                    try {
                        LOCK.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                size++;
                System.out.println("生产者，当前数量："+size);
                LOCK.notifyAll();
            }
        }
    }
    class Consumer implements Runnable{

        @Override
        public void run() {
            synchronized (LOCK){
                while (size==0){
                    System.out.println("消费者等待！");
                    try {
                        LOCK.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                size--;
                System.out.println("消费者，当前数量："+size);
                LOCK.notifyAll();
            }
        }
    }

    public static void main(String[] args) {
        SynchronizedWay synchronizedWay = new SynchronizedWay();
        new Thread(synchronizedWay.new Producer()).start();
        new Thread(synchronizedWay.new Producer()).start();
        new Thread(synchronizedWay.new Producer()).start();
        new Thread(synchronizedWay.new Consumer()).start();
        new Thread(synchronizedWay.new Consumer()).start();
        new Thread(synchronizedWay.new Producer()).start();
        new Thread(synchronizedWay.new Consumer()).start();
        new Thread(synchronizedWay.new Consumer()).start();
    }
}
```

# Lock

通过两个 Condition 互相唤醒。

``` java
package consumerProducer;

import java.util.LinkedList;
import java.util.concurrent.*;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockWay {
    Lock lock = new ReentrantLock();
    Condition full = lock.newCondition();
    Condition empty = lock.newCondition();
    int FULL = 2;
    LinkedList<Integer> list = new LinkedList<>();

    public class Producer implements Runnable{

        int value;

        public Producer(Integer value){
            this.value = value;
        }

        @Override
        public void run() {
            lock.lock();
            if(list.size()==FULL){ //这块可以是 if，因为 Condition 可以指定唤醒
                System.out.println("满了！");
                try {
                    full.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            list.add(value);
            System.out.println("生产者插入值！大小为："+list.size());
            empty.signal();
            lock.unlock();
        }
    }

    public class Consumer implements Callable<Integer>{

        @Override
        public Integer call() throws Exception {
            lock.lock();
            if(list.size()==0){
                System.out.println("空的！");
                empty.await();
            }
            Integer res = list.poll();
            System.out.println("消费了一个，大小为："+list.size());
            full.signal();
            lock.unlock();
            return res;
        }
    }

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        LockWay lockWay = new LockWay();
        new Thread(lockWay.new Producer(1)).start();
        new Thread(lockWay.new Producer(2)).start();
        new Thread(lockWay.new Producer(3)).start();
        ExecutorService es = Executors.newFixedThreadPool(3);
        Future<Integer> res1 = es.submit(lockWay.new Consumer());
        Future<Integer> res2 = es.submit(lockWay.new Consumer());
        Future<Integer> res3 = es.submit(lockWay.new Consumer());
        es.shutdown();
        System.out.println(res1.get()+" "+res2.get()+" "+res3.get());
    }
}
```

# LinkedBlockingQueue

压根就是阻塞的。内部逻辑也是两个 Condition，只不过用了 Node 做链表，提前设好了头节点和尾节点，这样可以使用两把锁，提高同时插入和读取的效率。

``` java
package consumerProducer;

import java.util.concurrent.*;

public class BlockingQueueWay {
    LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<>(2);
    public class Producer implements Runnable{

        int value;

        public Producer(Integer value){
            this.value = value;
        }

        @Override
        public void run() {
            try {
                queue.put(value);
                System.out.println("Producer 当前队列大小："+queue.size());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public class Consumer implements Callable<Integer> {

        @Override
        public Integer call() {
            Integer res = null;
            try {
                res = queue.take();
                System.out.println("Consumer 当前队列大小："+queue.size());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return res;
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        BlockingQueueWay blockingQueueWay = new BlockingQueueWay();
        new Thread(blockingQueueWay.new Producer(1)).start();
        new Thread(blockingQueueWay.new Producer(2)).start();
        new Thread(blockingQueueWay.new Producer(3)).start();
        ExecutorService es = Executors.newFixedThreadPool(3);
        Future<Integer> res1 = es.submit(blockingQueueWay.new Consumer());
        Future<Integer> res2 = es.submit(blockingQueueWay.new Consumer());
        Future<Integer> res3 = es.submit(blockingQueueWay.new Consumer());
        es.shutdown();
        System.out.println(res1.get()+" "+res2.get()+" "+res3.get());
    }
}
```