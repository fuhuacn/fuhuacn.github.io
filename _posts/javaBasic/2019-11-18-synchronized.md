---
layout: post
title: Java Synchronized 原理
categories: Java
description: Java Synchronized 原理
keywords: JAVA,Synchronized
---

[参考](https://blog.csdn.net/javazejian/article/details/72828483)

# 一、JAVA 对象布局

## [Unsafe 操作 JAVA 内存码](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)

Unsafe是位于sun.misc包下的一个类，主要提供一些用于执行低级别、不安全操作的方法，如直接访问系统内存资源、自主管理内存资源等，这些方法在提升Java运行效率、增强Java语言底层资源操作能力方面起到了很大的作用。但由于Unsafe类使Java语言拥有了类似C语言指针一样操作内存空间的能力，这无疑也增加了程序发生相关指针问题的风险。在程序中过度、不正确使用Unsafe类会使得程序出错的概率变大，使得Java这种安全的语言变得不再“安全”，因此对Unsafe的使用一定要慎重。

### 引用方式

+ 从getUnsafe方法的使用限制条件出发，通过Java命令行命令-Xbootclasspath/a把调用Unsafe相关方法的类A所在jar包路径追加到默认的bootstrap路径中，使得A被引导类加载器加载，从而通过Unsafe.getUnsafe方法安全的获取Unsafe实例。

+ 通过反射获取单例对象theUnsafe。

### 操作

这部分主要包含堆外内存的分配、拷贝、释放、给定地址值操作等方法。

``` java
//分配内存, 相当于C++的malloc函数
public native long allocateMemory(long bytes);
//扩充内存
public native long reallocateMemory(long address, long bytes);
//释放内存
public native void freeMemory(long address);
//在给定的内存块中设置值
public native void setMemory(Object o, long offset, long bytes, byte value);
//内存拷贝
public native void copyMemory(Object srcBase, long srcOffset, Object destBase, long destOffset, long bytes);
//获取给定地址值，忽略修饰限定符的访问限制。与此类似操作还有: getInt，getDouble，getLong，getChar等
public native Object getObject(Object o, long offset);
//为给定地址设置值，忽略修饰限定符的访问限制，与此类似操作还有: putInt,putDouble，putLong，putChar等
public native void putObject(Object o, long offset, Object x);
//获取给定地址的byte类型的值（当且仅当该内存地址为allocateMemory分配时，此方法结果为确定的）
public native byte getByte(long address);
//为给定地址设置byte类型的值（当且仅当该内存地址为allocateMemory分配时，此方法结果才是确定的）
public native void putByte(long address, byte x);
```

通常，我们在Java中创建的对象都处于堆内内存（heap）中，堆内内存是由JVM所管控的Java进程内存，并且它们遵循JVM的内存管理机制，JVM会采用垃圾回收机制统一管理堆内存。与之相对的是堆外内存，存在于JVM管控之外的内存区域，Java中对堆外内存的操作，依赖于Unsafe提供的操作堆外内存的native方法。

## JOL 查看 JAVA 内存

Unsafe 比较复杂，这里使用比较简单的 JOL。

可以使用 Maven 引入相关包。

``` xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.9</version>
</dependency>
```

可以很方便打印类的内存信息如下：

``` java
package test.synchronizedCode;

/**
 * Lock
 */
public class Lock {
    public int lock = 0;
    public boolean isLock = false;
}

System.out.println(ClassLayout.parseInstance(l).toPrintable());
```

结果为：

![jol](/images/javaBasic/synchronized/WX20191118-194404.png)

## 对象在内存中的布局

在JVM中，对象在内存中的布局分为三块区域：对象头、实例数据和对齐填充。

![堆内存](/images/javaBasic/synchronized/20170603163237166.png)

可以看见之前使用 JOL 打印出来的对象，含有对象头和实例变量，最下面一行 7 个字节即为对齐填充。

+ 实例变量：存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐。

+ 填充数据：由于虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐。不一定一定有。**数据对齐是为了方便 CPU 运算和 GC。**

+ 对象头：每个 GC 管理的堆对象开头的通用结构。固定格式。每一个 OOP 的指针只想一个对象头。包括堆对象的布局、对象的模版信息（方法区后改为元空间中）、GC状态、同步状态（Synchronized）和表示哈希码。一般由两个属性组成（Mark Word, Klass Pointer），数组由三个属性组成，会多一个长度属性。

## 对象头

### 对象头结构

Java 头对象其主要结构是由Mark Word 和 Klass Pointer（Class Metadata Address）组成，其结构说明如下：

![对象头结构](/images/javaBasic/synchronized/WX20191118-210525.png)

其中 Mark Word 在默认情况下存储着对象的 HashCode、分代年龄、锁标记位等以下是 32 位 JVM 的 Mark Word 默认存储结构

![32 位 Mark Word](/images/javaBasic/synchronized/WX20191118-210718.png)

### 对象头的状态

+ 无锁
+ 偏向锁
+ 轻量锁
+ 重量锁
+ GC 标记

*锁在对象头中总共两个 bit 位，如何代表 5 个状态？*

拥有一个记录是否是偏向锁的标志位（1 bit）。

![对象状态](/images/javaBasic/synchronized/Object.png)

注：上图仅适用于 64 位机器，每个状态为一行。

*刚刚跑的代码中，可以看到对象头总共 96 个 bit，而在上图中写的对象头总共位 128 个 bit，是因为一般情况下虚拟机默认开启指针压缩。一般情况下，intel 处理器用的是小端存储，即在小地址存储数据（倒着的），所以可以看见图中前 8 位是 1。*

# 2. Java 中的锁

## CAS

CAS，compare and swap 的缩写，中文翻译成比较并交换。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。 如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值 。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前 值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。”

通常将 CAS 用于同步的方式是从地址 V 读取值 A，执行多步计算来获得新值 B，然后使用 CAS 将 V 的值从 A 改为 B。如果 V 处的值尚未同时更改，则 CAS 操作成功。

类似于 CAS 的指令允许算法执行读-修改-写操作，而无需害怕其他线程同时修改变量，因为如果其他线程修改变量，那么 CAS 会检测它（并失败），算法可以对该操作重新计算。

## 重量级锁

我们主要分析一下重量级锁也就是通常说 synchronized 的**对象锁**，锁标识位为 10 ，其中指针指向的是 monitor 对象（也称为管程或监视器锁）的起始地址。每个对象都存在着一个 monitor 与之关联，对象与其 monitor 之间的关系有存在多种实现方式，如 monitor 可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，但当一个 monitor 被某个线程持有后，它便处于锁定状态。在Java虚拟机（HotSpot）中，monitor 是由 ObjectMonitor 实现的，其主要数据结构如下（位于 HotSpot 虚拟机源码 ObjectMonitor.hpp 文件，C++ 实现的）：

``` c++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; //记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

ObjectMonitor 中有两个队列，_WaitSet  和 _EntryList，用来保存 ObjectWaiter 对象列表(每个等待锁的线程都会被封装成ObjectWaiter对象)，_owner 指向持有 ObjectMonitor 对象的线程，当多个线程同时访问一段同步代码时，首先会进入 _EntryList 集合，当线程获取到对象的 monitor 后进入 _Owner 区域并把 monitor 中的 owner 变量设置为当前线程同时 monitor 中的计数器 count 加 1，若线程调用 wait() 方法，将释放当前持有的 monitor，owner 变量恢复为 null ，count 自减 1，同时该线程进入 WaitSet 集合中等待被唤醒。若当前线程执行完毕也将释放 monitor（锁）并复位变量的值，以便其他线程进入获取 monitor（锁）。如下图所示：

![锁](/images/javaBasic/synchronized/20170604114223462.png)

由此看来，monitor 对象存在于每个 Java 对象的对象头中（存储的指针的指向），synchronized 锁便是通过这种方式获取锁的，也是为什么 Java 中任意对象可以作为锁的原因，同时也是 notify/notifyAll/wait 等方法存在于顶级对象 Object 中的原因（关于这点稍后还会进行分析），有了上述知识基础后，下面我们将进一步分析 synchronized 在字节码层面的具体语义实现。

![两个线程竞争](/images/javaBasic/synchronized/4417484-0f740594de7ae935.jpg)

### Sychronized 代码块底层原理

现在我们重新定义一个 synchronized 修饰的同步代码块，在代码块中操作共享变量 i，如下：

``` java
public class SyncCodeBlock {

   public int i;

   public void syncTask(){
       //同步代码库
       synchronized (this){
           i++;
       }
   }
}
```

编译上述代码并使用javap反编译后得到字节码如下(这里我们省略一部分没有必要的信息)：

``` java
Classfile /Users/zejian/Downloads/Java8_Action/src/main/java/com/zejian/concurrencys/SyncCodeBlock.class
  Last modified 2017-6-2; size 426 bytes
  MD5 checksum c80bc322c87b312de760942820b4fed5
  Compiled from "SyncCodeBlock.java"
public class com.zejian.concurrencys.SyncCodeBlock
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
  //........省略常量池中数据
  //构造函数
  public com.zejian.concurrencys.SyncCodeBlock();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
  //===========主要看看syncTask方法实现================
  public void syncTask();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter  //注意此处，进入同步方法
         4: aload_0
         5: dup
         6: getfield      #2             // Field i:I
         9: iconst_1
        10: iadd
        11: putfield      #2            // Field i:I
        14: aload_1
        15: monitorexit   //注意此处，退出同步方法
        16: goto 24
        19: astore_2
        20: aload_1
        21: monitorexit //注意此处，退出同步方法
        22: aload_2
        23: athrow
        24: return
      Exception table:
      //省略其他字节码.......
}
SourceFile: "SyncCodeBlock.java"
```

其中以下代码是重点部分：

``` java
3: monitorenter  //进入同步方法
//..........省略其他  
15: monitorexit   //退出同步方法
16: goto 24
//省略其他.......
21: monitorexit //退出同步方法
```

从字节码中可知同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置，当执行 monitorenter 指令时，当前线程将试图获取  objectref（即对象锁) 所对应的 monitor 的持有权，当 objectref 的 monitor 的进入计数器为 0，那线程可以成功取得 monitor，并将计数器值设置为 1，取锁成功。如果当前线程已经拥有 objectref 的 monitor 的持有权，那它可以重入这个 monitor（关于重入性稍后会分析），重入时计数器的值也会加 1。倘若其他线程已经拥有 objectref 的 monitor 的所有权，那当前线程将被阻塞，直到正在执行线程执行完毕，即 monitorexit 指令被执行，执行线程将释放 monitor（锁）并设置计数器值为0 ，其他线程将有机会持有 monitor。值得注意的是编译器将会确保无论方法通过何种方式完成，方法中调用过的每条 monitorenter 指令都有执行其对应 monitorexit 指令，而无论这个方法是正常结束还是异常结束。为了保证在方法异常完成时 monitorenter 和 monitorexit 指令依然可以正确配对执行，编译器会自动产生一个异常处理器，这个异常处理器声明可处理所有的异常，它的目的就是用来执行 monitorexit 指令。从字节码中也可以看出多了一个 monitorexit 指令，它就是异常结束时被执行的释放 monitor 的指令。

### Sychronized 方法底层原理

现在我们重新定义一个 synchronized 修饰的同步代码块，在代码块中操作共享变量 i，如下：

``` java
public class SyncMethod {

   public int i;

   public synchronized void syncTask(){
           i++;
   }
}
```

同样使用 javap 反编译：

``` java
Classfile /Users/zejian/Downloads/Java8_Action/src/main/java/com/zejian/concurrencys/SyncMethod.class
  Last modified 2017-6-2; size 308 bytes
  MD5 checksum f34075a8c059ea65e4cc2fa610e0cd94
  Compiled from "SyncMethod.java"
public class com.zejian.concurrencys.SyncMethod
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool;

   //省略没必要的字节码
  //==================syncTask方法======================
  public synchronized void syncTask();
    descriptor: ()V
    //方法标识ACC_PUBLIC代表public修饰，ACC_SYNCHRONIZED指明该方法为同步方法
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: dup
         2: getfield      #2                  // Field i:I
         5: iconst_1
         6: iadd
         7: putfield      #2                  // Field i:I
        10: return
      LineNumberTable:
        line 12: 0
        line 13: 10
}
SourceFile: "SyncMethod.java"
```

从字节码中可以看出，synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。这便是 synchronized 锁在同步代码块和同步方法上实现的基本原理。同时我们还必须注意到的是在 Java 早期版本中，synchronized 属于重量级锁，效率低下。因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，而操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。庆幸的是在Java 6 之后 Java 官方对从 JVM 层面对 synchronized 较大优化，所以现在的 synchronized 锁效率也优化得很不错了，Java 6 之后，为了减少获得锁和释放锁所带来的性能消耗，引入了轻量级锁和偏向锁，接下来我们将简单了解一下 Java 官方在 JVM 层面对 synchronized 锁的优化。

## synchronized 的优化

锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁，但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级，关于重量级锁，前面我们已详细分析过，下面我们将介绍偏向锁和轻量级锁以及 JVM 的其他优化手段，这里并不打算深入到每个锁的实现和转换过程更多地是阐述 Java 虚拟机所提供的每个锁的核心优化思想，毕竟涉及到具体过程比较繁琐，如需了解详细过程可以查阅《深入理解Java虚拟机原理》。

### 偏向锁

偏向锁是 Java 6 之后加入的新锁，它是一种针对加锁操作的优化手段，经过研究发现，在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁（会涉及到一些 CAS 操作,耗时）的代价而引入偏向锁。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时 Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁。但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。下面我们接着了解轻量级锁。

### 轻量级锁

倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段（1.6之后加入的），此时 Mark Word 的结构也变为轻量级锁的结构。轻量级锁能够提升程序性能的依据是“对绝大部分的锁，在整个同步周期内都不存在竞争”，注意这是经验数据。需要了解的是，轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁。

+ 加锁

    线程在执行同步块之前，JVM 会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的 Mark Word 复制到锁记录中，官方称为 Displaced Mark Word。然后线程尝试使用 CAS 将对象头中的 Mark Word 替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，则自旋获取锁，当自旋获取锁仍然失败时，表示存在其他线程竞争锁（两条或两条以上的线程竞争同一个锁），则轻量级锁会膨胀成重量级锁。

+ 解锁

    轻量级解锁时，会使用原子的 CAS 操作来将 Displaced Mark Word 替换回到对象头，如果成功，则表示同步过程已完成。如果失败，表示有其他线程尝试过获取该锁，则要在释放锁的同时唤醒被挂起的线程。

### 自旋锁

轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。这是基于在大多数情况下，线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，因此自旋锁会假设在不久将来，当前的线程可以获得锁，因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因)，一般不会太久，可能是 50 个循环或 100 个循环，在经过若干次循环后，如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式，这种方式确实也是可以提升效率的。最后没办法也就只能升级为重量级锁了。

### 锁消除

消除锁是虚拟机另外一种锁的优化，这种优化更彻底，Java 虚拟机在 JIT 编译时（可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译），通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间，如下 StringBuffer 的 append 是一个同步方法，但是在 add 方法中的 StringBuffer 属于一个局部变量，并且不会被其他线程所使用，因此 StringBuffer 不可能存在共享资源竞争的情景，JVM 会自动将其锁消除。

``` java
public class StringBufferRemoveSync {

    public void add(String str1, String str2) {
        //StringBuffer是线程安全,由于sb只会在append方法中使用,不可能被其他线程引用
        //因此sb属于不可能共享的资源,JVM会自动消除内部的锁
        StringBuffer sb = new StringBuffer();
        sb.append(str1).append(str2);
    }

    public static void main(String[] args) {
        StringBufferRemoveSync rmsync = new StringBufferRemoveSync();
        for (int i = 0; i < 10000000; i++) {
            rmsync.add("abc", "123");
        }
    }

}
```

# 3. 其他重点

## synchronized 的可重用性

从互斥锁的设计上来说，当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入锁，请求将会成功，在 java 中 synchronized 是基于原子性的内部锁机制，是可重入的，因此在一个线程调用 synchronized 方法的同时在其方法体内部调用该对象另一个 synchronized 方法，也就是说一个线程得到一个对象锁后再次请求该对象锁，是允许的，这就是 synchronized 的可重入性。如下：

``` java
public class AccountingSync implements Runnable{
    static AccountingSync instance=new AccountingSync();
    static int i=0;
    static int j=0;
    @Override
    public void run() {
        for(int j=0;j<1000000;j++){

            //this,当前实例对象锁
            synchronized(this){
                i++;
                increase();//synchronized的可重入性
            }
        }
    }

    public synchronized void increase(){
        j++;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(instance);
        Thread t2=new Thread(instance);
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}
```

正如代码所演示的，在获取当前实例对象锁后进入 synchronized 代码块执行同步代码，并在代码块中调用了当前实例对象的另外一个 synchronized 方法，再次请求当前实例锁时，将被允许，进而执行方法体代码，这就是重入锁最直接的体现，需要特别注意另外一种情况，当子类继承父类时，子类也是可以通过可重入锁调用父类的同步方法。注意由于 synchronized 是基于 monitor 实现的，因此每次重入，monitor 中的计数器仍会加1。

## 中断与 synchronized

### 中断

正如中断二字所表达的意义，在线程运行（run方法）中间打断它，Java 中，提供了以下 3 个有关线程中断的方法：

``` java
//中断线程（实例方法）
public void Thread.interrupt();

//判断线程是否被中断（实例方法）
public boolean Thread.isInterrupted();

//判断是否被中断并清除当前中断状态（静态方法）
public static boolean Thread.interrupted();
```

当一个线程处于被***阻塞状态或者试图执行一个阻塞操作***时，使用 Thread.interrupt() 方式中断该线程，注意此时将会抛出一个 InterruptedException 的异常，同时中断状态将会被复位（由中断状态改为非中断状态），如下代码将演示该过程：

``` java
public class InterruputSleepThread3 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                //while在try中，通过异常中断就可以退出run循环
                try {
                    while (true) {
                        //当前线程处于阻塞状态，异常必须捕捉处理，无法往外抛出
                        TimeUnit.SECONDS.sleep(2);
                    }
                } catch (InterruptedException e) {
                    System.out.println("Interruted When Sleep");
                    boolean interrupt = this.isInterrupted();
                    //中断状态被复位
                    System.out.println("interrupt:"+interrupt);
                }
            }
        };
        t1.start();
        TimeUnit.SECONDS.sleep(2);
        //中断处于阻塞状态的线程
        t1.interrupt();

        /**
         * 输出结果:
           Interruted When Sleep
           interrupt:false
         */
    }
}
```

如上述代码所示，我们创建一个线程，并在线程中调用了 sleep 方法从而使用线程进入阻塞状态，启动线程后，调用线程实例对象的 interrupt 方法中断阻塞异常，并抛出 InterruptedException 异常，此时中断状态也将被复位。除了阻塞中断的情景，我们还可能会遇到处于运行期且非阻塞的状态的线程，这种情况下，直接调用 Thread.interrupt() 中断线程是不会得到任响应的，如下代码，将无法中断***非阻塞状态***下的线程：

``` java
public class InterruputThread {
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(){
            @Override
            public void run(){
                while(true){
                    System.out.println("未被中断");
                }
            }
        };
        t1.start();
        TimeUnit.SECONDS.sleep(2);
        t1.interrupt();

        /**
         * 输出结果(无限执行):
             未被中断
             未被中断
             未被中断
             ......
         */
    }
}
```

虽然我们调用了interrupt方法，但线程t1并未被中断，因为处于***非阻塞状态***的线程需要我们手动进行中断检测并结束程序，改进后代码如下：

``` java
public class InterruputThread {
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(){
            @Override
            public void run(){
                while(true){
                    //判断当前线程是否被中断
                    if (this.isInterrupted()){
                        System.out.println("线程中断");
                        break;
                    }
                }

                System.out.println("已跳出循环,线程中断!");
            }
        };
        t1.start();
        TimeUnit.SECONDS.sleep(2);
        t1.interrupt();

        /**
         * 输出结果:
            线程中断
            已跳出循环,线程中断!
         */
    }
}
```

是的，我们在代码中使用了实例方法 isInterrupted 判断线程是否已被中断，如果被中断将跳出循环以此结束线程，注意非阻塞状态调用 interrupt() 并不会导致中断状态重置。综合所述，可以简单总结一下中断两种情况，一种是当线程处于阻塞状态或者试图执行一个**阻塞操作**时，我们可以使用实例方法 interrupt() 进行线程中断，执行中断操作后将会抛出 interruptException 异常（该异常必须捕捉无法向外抛出）并将中断状态复位，另外一种是当线程处于**运行状态**时，我们也可调用实例方法 interrupt() 进行线程中断，但同时必须手动判断中断状态，并编写中断线程的代码（其实就是结束run方法体的代码）。有时我们在编码时可能需要兼顾以上两种情况，那么就可以如下编写：

``` java
public void run(){
    try {
    //判断当前线程是否已中断,注意interrupted方法是静态的,执行后会对中断状态进行复位
    while (!Thread.interrupted()) {
        TimeUnit.SECONDS.sleep(2);
    }
    } catch (InterruptedException e) {

    }
}
```

### synchronized 中的中断

事实上线程的中断操作对于正在等待获取的锁对象的synchronized方法或者代码块并不起作用，也就是对于 synchronized 来说，如果一个线程在等待锁，那么结果只有两种，要么它获得这把锁继续执行，要么它就保存等待，即使调用中断线程的方法，也不会生效。演示代码如下：

``` java
public class SynchronizedBlocked implements Runnable{

    public synchronized void f() {
        System.out.println("Trying to call f()");
        while(true) // Never releases lock
            Thread.yield();
    }

    /**
     * 在构造器中创建新线程并启动获取对象锁
     */
    public SynchronizedBlocked() {
        //该线程已持有当前实例锁
        new Thread() {
            public void run() {
                f(); // Lock acquired by this thread
            }
        }.start();
    }
    public void run() {
        //中断判断
        while (true) {
            if (Thread.interrupted()) {
                System.out.println("中断线程!!");
                break;
            } else {
                f();
            }
        }
    }


    public static void main(String[] args) throws InterruptedException {
        SynchronizedBlocked sync = new SynchronizedBlocked();
        Thread t = new Thread(sync);
        //启动后调用f()方法,无法获取当前实例锁处于等待状态
        t.start();
        TimeUnit.SECONDS.sleep(1);
        //中断线程,无法生效
        t.interrupt();
    }
}
```

我们在 SynchronizedBlocked 构造函数中创建一个新线程并启动获取调用 f() 获取到当前实例锁，由于 SynchronizedBlocked 自身也是线程，启动后在其 run 方法中也调用了 f()，但由于对象锁被其他线程占用，导致 t 线程只能等到锁，此时我们调用了 t.interrupt(); 但并不能中断线程。

## 等待唤醒机制与synchronized

所谓等待唤醒机制本篇主要指的是 notify/notifyAll 和 wait 方法，在使用这3个方法时，必须处于 synchronized 代码块或者 synchronized 方法中，否则就会抛出 IllegalMonitorStateException 异常，这是因为调用这几个方法前必须拿到当前对象的监视器 monitor 对象，也就是说 notify/notifyAll 和 wait 方法依赖于 monitor 对象，在前面的分析中，我们知道 monitor 存在于对象头的 Mark Word 中（存储 monitor 引用指针），而 synchronized 关键字可以获取 monitor ，这也就是为什么 notify/notifyAll 和 wait 方法必须在 synchronized 代码块或者 synchronized 方法调用的原因。

需要特别理解的一点是，与 sleep 方法不同的是 wait 方法调用完成后，线程将被暂停，但 wait 方法将会释放当前持有的监视器锁（monitor），直到有线程调用 notify/notifyAll 方法后方能继续执行，而 sleep 方法只让线程休眠并不释放锁。同时 notify/notifyAll 方法调用后，并不会马上释放监视器锁，而是在相应的 synchronized(){}/synchronized 方法执行结束后才自动释放锁。