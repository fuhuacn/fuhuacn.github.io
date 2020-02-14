---
layout: post
title: 计算机操作系统
categories: Knowledge
description: 计算机操作系统
keywords: 操作系统
---
[参考来源](https://github.com/CyC2018/CS-Notes/blob/master/notes/计算机操作系统%20-%20目录.md)

目录

* TOC
{:toc}

# 一、概述

## 基本特征

### 1.并发

并发是指宏观上在一段时间内能同时运行多个程序，而并行则指同一时刻能运行多个指令。

并行需要硬件支持，如多流水线、多核处理器或者分布式计算系统。

操作系统通过引入进程和线程，使得程序能够并发运行。

### 2. 共享

共享是指系统中的资源可以被多个并发进程共同使用。

有两种共享方式：互斥共享和同时共享。

互斥共享的资源称为临界资源，例如打印机等，在同一时刻只允许一个进程访问，需要用同步机制来实现互斥访问。

### 3. 虚拟

虚拟技术把一个物理实体转换为多个逻辑实体。

主要有两种虚拟技术：时（时间）分复用技术和空（空间）分复用技术。

多个进程能在同一个处理器上并发执行使用了时分复用技术，让每个进程轮流占用处理器，每次只执行一小个时间片并快速切换。

虚拟内存使用了空分复用技术，它将物理内存抽象为地址空间，每个进程都有各自的地址空间。地址空间的页被映射到物理内存，地址空间的页并不需要全部在物理内存中，当使用到一个没有在物理内存的页时，执行页面置换算法，将该页置换到内存中。

### 4. 异步

同步，就是实时处理，异步，就是分时处理（如收发短信）。

对于写程序，同步往往会阻塞，没有数据过来，我就等着，异步则不会阻塞，没数据来我干别的事，有数据来去处理这些数据。

异步指进程不是一次性执行完毕，而是走走停停，以不可知的速度向前推进。

## 操作系统的基本功能

### 1. 进程管理

进程控制、进程同步、进程通信、死锁处理、处理机调度等。

### 2. 内存管理

内存分配、地址映射、内存保护与共享、虚拟内存等。

### 3. 文件管理

文件存储空间的管理、目录管理、文件读写管理和保护等。

### 4. 设备管理

完成用户的 I/O 请求，方便用户使用各种设备，并提高设备的利用率。

主要包括缓冲管理、设备分配、设备处理、虛拟设备等。

## 系统调用

操作系统本质上是一个系统程序，即为别的程序提供服务的程序。操作系统是以系统调用（system call）的方式提供服务的。

系统调用就是操作系统提供的应用程序接口（Application Programming Interface，API），用户程序即可通过调用这些 API 获得操作系统的服务；

![系统调用](/images/posts/knowledge/operationSystem/系统调用.png)

例如，如果用户程序需要进行读磁盘内容的操作，在 C 程序代码中可使用如下的语句：

``` c
result = read(fd, buffer, nbytes);
```

> fd：相信你对"On Unix, Everything is a file"已经耳熟能详了，比如 /dev目录、 /proc目录等，你几乎可以通过操作文件的方式获取系统所有能够获取的 data。而和文件进行交互，就需要用到文件描述符（file descriptors，简称 fd，就是一个数字，在某个进程内唯一）。

该 read 函数是 C 语言提供的库函数，而这个库函数本身则是调用的操作系统的 read 系统调用。这里有两个 read：

+ 一个是 C 语言提供的 read 库函数；
+ 另一个是 read 系统调用，由操作系统提供。

编译器在看到上述语句后，将 read 库函数扩展为 read 系统调用。

Linux 的系统调用主要有以下这些：

| Task | Commands |
| :---: | --- |
| 进程控制 | fork(); exec(); exit(); wait(); sleep();|
| 进程通信 | pipe(); shmget(); mmap(); |
| 文件操作 | open(); read(); write(); |
| 设备操作 | ioctl(); read(); write(); |
| 信息维护 | getpid(); alarm(); sleep(); |
| 安全 | chmod(); umask(); chown(); |

### 进程创建

一个进程都由另一个称之为父进程的进程启动，被父进程启动的进程叫做子进程。Linux 系统启动时候，它将运行一个名为 init 的进程，该进程是系统运行的第一个进程，它的进程号为 1，它负责管理其它进程，可以把它看做是操作系统进程管理器，它是其它所有进程的祖先进程。系统中的进程要么是由 init 进程启动，要么是由 init 进程启动的其他进程启动。

“fork”用来产生一个新进程，这个进程默认会复制自身。

“exec”则是“启动参数指定的程序，代替自身进程”。配合fork使用，就成了“当前进程启动另一个进程”。

## 用户态和内核态

为了限制不同程序的访问能力，防止一些程序访问其它程序的内存数据，CPU 划分了用户态和内核态两个权限等级。

- 用户态只能受限地访问内存，且不允许访问外围设备，没有占用 CPU 的能力，CPU 资源可以被其它程序获取；
- 内核态可以访问内存所有数据以及外围设备，也可以进行程序的切换。

所有用户程序都运行在用户态，但有时需要进行一些内核态的操作，比如从硬盘或者键盘读数据，这时就需要进行系统调用，使用陷阱指令，CPU 切换到内核态，执行相应的服务，再切换为用户态并返回系统调用的结果。

### 为什么要分用户态和内核态？

- 安全性：防止用户程序恶意或者不小心破坏系统/内存/硬件资源；
- 封装性：用户程序不需要实现更加底层的代码；
- 利于调度：如果多个用户程序都在等待键盘输入，这时就需要进行调度；统一交给操作系统调度更加方便。

### 如何从用户态切换到内核态？

- 系统调用：比如读取命令行输入。本质上还是通过中断实现
- 用户程序发生异常时：比如缺页异常
- 外围设备的中断：外围设备完成用户请求的操作之后，会向 CPU 发出中断信号，这时 CPU 会转去处理对应的中断处理程序

## 大内核和微内核

+ 大内核

    大内核是将操作系统功能作为一个紧密结合的整体放到内核。

    由于各模块共享信息，因此有很高的性能。

+ 微内核

    由于操作系统不断复杂，因此将一部分操作系统功能移出内核，从而降低内核的复杂性。移出的部分根据分层的原则划分成若干服务，相互独立。

    在微内核结构下，操作系统被划分成小的、定义良好的模块，只有微内核这一个模块运行在内核态，其余模块运行在用户态。

    因为需要频繁地在用户态和核心态之间进行切换，所以会有一定的性能损失。

    ![微内核](/images/posts/knowledge/operationSystem/微内核.jpeg)

## 中断分类

### 中断

Linux 内核需要对连接到计算机上的所有硬件设备进行管理，毫无疑问这是它的份内事。如果要管理这些设备，首先得和它们互相通信才行，一般有两种方案可实现这种功能：

+ 轮询（polling） 让内核定期对设备的状态进行查询，然后做出相应的处理；

+ 中断（interrupt） 让硬件在需要的时候向内核发出信号（变内核主动为硬件主动）。

第一种方案会让内核做不少的无用功，因为轮询总会周期性的重复执行，大量地耗用 CPU 时间，因此效率及其低下，所以一般都是采用第二种方案 。

### [中断、异常和陷入的对比](https://www.cnblogs.com/zhangyunhao/p/4409410.html)

中断/异常/陷入机制是操作系统由用户态转为内核态的唯一途径，是操作系统的驱动力。

中断、异常机制有以下特征：

+ 随机发生
+ 自动处理（硬件完成）
+ 可恢复

中断、异常的区别：

+ 中断属外部事件，是正在运行的程序所不期望的
+ 异常由正在执行的指令引发

在中断、异常过程中，软件和硬件分别担任什么角色：

+ 硬件--中断/异常响应
+ 软件--中断/异常处理程序
　　
中断/异常的引入目的：

+ 中断的引入是为了 CPU 与设备之间的并行操作
+ 异常的引入是为了表示 CPU 执行指令时本身出现的问题
　　
举例：一个故事：小明在看书，突然来了个电话，接完电话继续看书，这是中断；小明在看书，感觉口渴了，喝了水接着看书，这是异常。

| |类别|原因|同步/异步|返回行为|
|:---|:---:|:---:|:---:|:---:|
|中断|中断|来自I/O设备或其他硬件部件|异步|总是返回到下一条指令|
|异常|陷入|有意识安排的|同步|返回到下一条指令|
|异常|故障|可恢复的错误|同步|返回到当前指令|
|异常|终止|不可恢复的错误|同步|不会返回|

# 2. 进程、线程

## 进程与线程

### 进程

进程是资源分配的基本单位。

进程控制块 (Process Control Block, PCB) 描述进程的基本信息和运行状态，所谓的创建进程和撤销进程，都是指对 PCB 的操作。

### 线程

线程是独立调度的基本单位。

一个进程中可以有多个线程，它们共享进程资源。

QQ 和浏览器是两个进程，浏览器进程里面有很多线程，例如 HTTP 请求线程、事件响应线程、渲染线程等等，线程的并发执行使得在浏览器中点击一个新链接从而发起 HTTP 请求时，浏览器还可以响应用户的其它事件。

### 进程与线程的区别

+ 拥有资源

    进程是资源分配的基本单位，但是线程不拥有资源，线程可以访问隶属进程的资源。

+ 调度

    线程是独立调度的基本单位，在同一进程中，线程的切换不会引起进程切换，从一个进程中的线程切换到另一个进程中的线程时，会引起进程切换。

+ 系统开销

    由于创建或撤销进程时，系统都要为之分配或回收资源，如内存空间、I/O 设备等，所付出的开销远大于创建或撤销线程时的开销。类似地，在进行进程切换时，涉及当前执行进程 CPU 环境的保存及新调度进程 CPU 环境的设置，而线程切换时只需保存和设置少量寄存器内容，开销很小。

+ 通信方面

    线程间可以通过直接读写同一进程中的数据进行通信，但是进程通信需要借助 IPC。

## 进程状态的切换

![进程状态的切换](/images/posts/knowledge/operationSystem/进程切换.png)

+ 新建状态（created）：进程刚刚被创建的状态。
+ 就绪状态（ready）：备运行条件，等待系统分配处理器以便运行。
+ 运行状态（running）：占有处理器正在运行。
+ 阻塞状态（waiting）：不具备运行条件，正在等待某个事件的完成。
+ 终止状态（terminated）：当一个进程到达了自然结束点，或是出现了无法克服的错误，或是被操作系统所终结，或是被其他有终止权的进程所终结，它将进入终止态。进入终止态的进程以后不再执行，但依然临时保留在操作系统中等待善后。

引起进程状态转换的具体原因如下：

+ 运行态 -> 阻塞态：等待使用资源；如等待外设传输；等待人工干预。
+ 阻塞态 -> 就绪态：资源得到满足；如外设传输结束；人工干预完成。
+ 运行态 -> 就绪态：运行时间片到；出现有更高优先权进程。
+ 就绪态 -> 运行态：CPU 空闲时选择一个就绪进程。
+ NULL -> 新建态：执行一个程序，创建一个子进程。
+ 新建态 -> 就绪态：当操作系统完成了进程创建的必要操作，并且当前系统的性能和虚拟内存的容量均允许。
+ 运行态 -> 终止态：当一个进程到达了自然结束点，或是出现了无法克服的错误，或是被操作系统所终结，或是被其他有终止权的进程所终结。
+ 终止态 -> NULL：完成善后操作。

注意：

+ 只有就绪态和运行态可以相互转换，其它的都是单向转换。就绪状态的进程通过调度算法从而获得 CPU 时间，转为运行状态；而运行状态的进程，在分配给它的 CPU 时间片用完之后就会转为就绪状态，等待下一次调度。
+ 阻塞状态是缺少需要的资源从而由运行状态转换而来，但是该资源不包括 CPU 时间，缺少 CPU 时间会从运行态转换为就绪态。

## 进程调度算法

不同环境的调度算法目标不同，因此需要针对不同环境来讨论调度算法。

### 0. 操作系统分类

+ 批处理阶段

    早期的一种大型机用操作系统。可对用户作业成批处理，期间勿需用户干预，分为单道批处理系统（系统对作业的处理是成批进行的，但内存中始终保持一道作业。）和多道批处理系统（多道程序设计技术允许多个程序同时进入内存并运行）。如：DOS

+ 分时操作系统

    利用分时技术的一种联机的多用户交互式操作系统，每个用户可以通过自己的终端向系统发出各种操作控制命令，完成作业的运行。分时是指把处理机的运行时间分成很短的时间片，按时间片轮流把处理机分配给各联机作业使用。如：Windows，Unix，Linux

+ 实时操作系统

    能够在指定或者确定的时间内完成系统功能以及对外部或内部事件在同步或异步时间内做出响应的系统,实时意思就是对响应时间有严格要求,要以足够快的速度进行处理。分为硬实时和软实时两种。如：UCOSII

### 1. 批处理系统调度算法

批处理系统没有太多的用户操作，在该系统中，调度算法目标是保证吞吐量和周转时间（从提交到终止的时间）。

+ 先来先服务 first-come first-serverd（FCFS）

    非抢占式的调度算法，按照请求的顺序进行调度。

    有利于长作业，但不利于短作业，因为短作业必须一直等待前面的长作业执行完毕才能执行，而长作业又需要执行很长时间，造成了短作业等待时间过长。

+ 短作业优先 shortest job first（SJF）

    非抢占式的调度算法，按估计运行时间最短的顺序进行调度。

    长作业有可能会饿死，处于一直等待短作业执行完毕的状态。因为如果一直有短作业到来，那么长作业永远得不到调度。

+ 最短剩余时间优先 shortest remaining time next（SRTN）

    最短作业优先的抢占式版本，按剩余运行时间的顺序进行调度。 当一个新的作业到达时，其整个运行时间与当前进程的剩余时间作比较。如果新的进程需要的时间更少，则挂起当前进程，运行新的进程。否则新的进程等待。

### 2. 交互式系统（分时操作系统）

交互式系统有大量的用户交互操作，在该系统中调度算法的目标是快速地进行响应。

+ 时间片轮转

    将所有就绪进程按 FCFS（先来先服务）的原则排成一个队列，每次调度时，把 CPU 时间分配给队首进程，该进程可以执行一个时间片。当时间片用完时，由计时器发出时钟中断，调度程序便停止该进程的执行，并将它送往就绪队列的末尾，同时继续把 CPU 时间分配给队首的进程。

    时间片轮转算法的效率和时间片的大小有很大关系：
    + 因为进程切换都要保存进程的信息并且载入新进程的信息，如果时间片太小，会导致进程切换得太频繁，在进程切换上就会花过多时间。
    + 而如果时间片过长，那么实时性就不能得到保证。

+ 优先级调度

    为每个进程分配一个优先级，按优先级进行调度。

    为了防止低优先级的进程永远等不到调度，可以随着时间的推移增加等待进程的优先级。

+ 多级反馈队列

    + UNIX 的一个分支 BSD5.3 版所采用的调度算法
    + 一个综合调度算法（折中权衡）
    + 设置多个就绪队列，第一级队列优先级最高
    + 给不同就绪队列的进程分配长度不同的时间片，第一级队列时间片最小；随着队列优先级别的降低，时间片增大。
    + 当第一级队列为空时，就在第二级队列调度，以此类推
    + 各级队列按照时间片轮转方式进行调度
    + 当一个新创建进程就绪后，进入第一级队列
    + 进程用完时间片而放弃cpu，进入下一级就绪队列
    + 由于阻塞而放弃cpu的进程进入相应的等待队列，一旦等待的事件发生，该进程回到原来一级就绪队列

    以上所说都是属于非抢占式的，如果允许抢占，则当有一个优先级更高的进程就绪时，可以抢占cpu，被抢占的进程回到原来一级就绪队列的末尾。

    ![多级反馈队列](/images/posts/knowledge/operationSystem/多级反馈队列.png)

### 3. 实时系统

实时系统要求一个请求在一个确定时间内得到响应。

分为硬实时和软实时，前者必须满足绝对的截止时间，后者可以容忍一定的超时。

## 并发中的同步问题

同步可以作为进程问题也可以作为线程问题。在 JAVA 中一般都只能在线程中实现，所以看到的 JAVA 代码都是在线程中实现。但其实本以上是讲进程问题。

本章代码[来源](https://my.oschina.net/hosee/blog/485121)。

### 1. 临界资源

在操作系统中，进程是占有资源的最小单位（线程可以访问其所在进程内的所有资源，但线程本身并不占有资源或仅仅占有一点必须资源）。但对于某些资源来说，其在同一时间只能被一个进程所占用。这些一次只能被一个进程所占用的资源就是所谓的临界资源。典型的临界资源比如物理上的打印机，或是存在硬盘或内存中被多个进程所共享的一些变量和数据等（如果这类资源不被看成临界资源加以保护，那么很有可能造成丢数据的问题）。

对于临界资源的访问，必须是互诉进行。也就是**当临界资源被占用时，另一个申请临界资源的进程会被阻塞，直到其所申请的临界资源被释放。**而进程内访问临界资源的代码被成为临界区。

对于临界区的访问过程分为四个部分：

1. 进入区:查看临界区是否可访问，如果可以访问，则转到步骤二，否则进程会被阻塞。

2. 临界区:在临界区做操作。

3. 退出区:清除临界区被占用的标志。

4. 剩余区：进程与临界区不相关部分的代码。

### 2. 同步与互斥

+ 同步：**多个进程因为合作产生的直接制约关系，使得进程有一定的先后执行关系。**
  
    比如说进程 B 需要从缓冲区读取进程 A 产生的信息，当缓冲区为空时，进程 B 因为读取不到信息而被阻塞。而当进程 A 产生信息放入缓冲区时，进程 B 才会被唤醒。
+ 互斥：**多个进程在同一时刻只有一个进程能进入临界区。**
  
    比如进程B需要访问打印机，但此时进程A占有了打印机，进程B会被阻塞，直到进程A释放了打印机资源,进程B才可以继续执行。

### 3. 信号量

信号量（Semaphore）是一个整型变量，可以对其执行 down 和 up 操作，也就是常见的 P 和 V 操作（P 和 V 操作分别来自荷兰语 Passeren 和 Vrijgeven，分别表示占有和释放）。

+ down（P）：表示有一个进程将占用或等待资源，如果信号量大于 0 ，执行 -1 操作；如果信号量等于 0，进程睡眠，等待信号量大于 0；
+ up（V）：表示占用或等待资源的进程减少了1个。对信号量执行 +1 操作，唤醒睡眠的进程让其完成 down 操作。

down 和 up 操作需要被设计成原语，不可分割，通常的做法是在执行这些操作的时候屏蔽中断。

如果信号量的取值只能为 0 或者 1，那么就成为了 互斥量（Mutex） ，0 表示临界区已经加锁，1 表示临界区解锁。

**java 中的信号量：**
``` java
Semaphore semaphore = new Semaphore(10); //设定为 10 的信号量，注意这个 10 并不是最大值，可以设第二个参数是否公平也就是是否先请求先拿到
try{
    semaphore.acquire(); //使用一个信号量，如果不够会阻塞，所以需要 InterruptedException
    semaphore.acquire(8); //使用 8 个信号量，如果不够会阻塞，所以需要 InterruptedException
}catch(InterruptedException e){
    e.printStackTrace();
}
semaphore.release(); //释放一个信号量
semaphore.release(20); // 释放 20 个信号量
System.out.println(semaphore.availablePermits()); // 输出当前还剩的信号量
```

**使用信号量实现生产者-消费者问题：**

本作业要求设计在同一个进程地址空间内执行的两个线程。生产者线程生产物品，然后将物品放置在一个空缓冲区中供消费者线程消费。消费者线程从缓冲区中获得物品，然后释放缓冲区。当生产者线程生产物品时，如果没有空缓冲区可用，那么生产者线程必须等待消费者线程释放出一个空缓冲区。当消费者线程消费物品时，如果没有满的缓冲区，那么消费者线程将被阻塞，直到新的物品被生产出来。

这里生产者和消费者是既同步又互斥的关系，首先只有生产者生产了，消费着才能消费，这里是同步的关系。但他们对于临界区的访问又是互斥的关系。因此需要三个信号量 empty 和 full 用于同步缓冲区（对缓冲区剩余或满加减），而 mut 变量用于在访问缓冲区时是互斥的（0、1 控制消费者或生产者是否可以访问）。

``` java
import java.util.concurrent.Semaphore;

public class Hosee{
	int count = 0;
	final Semaphore notFull = new Semaphore(10);
	final Semaphore notEmpty = new Semaphore(0);
	final Semaphore mutex = new Semaphore(1);

	class Producer implements Runnable{
		@Override
		public void run(){
			for (int i = 0; i < 10; i++){
				try{
					Thread.sleep(3000);
				}
				catch (Exception e){
					e.printStackTrace();
				}
				try{
					notFull.acquire();//顺序不能颠倒，否则会造成死锁。
					mutex.acquire();
					count++;
					System.out.println(Thread.currentThread().getName()
							+ "生产者生产，目前总共有" + count);
				}
				catch (Exception e){
					e.printStackTrace();
				}
				finally{
					mutex.release();
					notEmpty.release();
				}
			}
		}
	}

	class Consumer implements Runnable{
		@Override
		public void run(){
			for (int i = 0; i < 10; i++)
			{
				try
				{
					Thread.sleep(3000);
				}
				catch (InterruptedException e1)
				{
					e1.printStackTrace();
				}
				try
				{
					notEmpty.acquire();//顺序不能颠倒，否则会造成死锁。
					mutex.acquire();
					count--;
					System.out.println(Thread.currentThread().getName()
							+ "消费者消费，目前总共有" + count);
				}
				catch (Exception e)
				{
					e.printStackTrace();
				}
				finally
				{
					mutex.release();
					notFull.release();
				}
			}
		}
	}

	public static void main(String[] args) throws Exception{
		Hosee hosee = new Hosee();
		new Thread(hosee.new Producer()).start();
		new Thread(hosee.new Consumer()).start();
		new Thread(hosee.new Producer()).start();
		new Thread(hosee.new Consumer()).start();

		new Thread(hosee.new Producer()).start();
		new Thread(hosee.new Consumer()).start();
		new Thread(hosee.new Producer()).start();
		new Thread(hosee.new Consumer()).start();
	}
}
```

### 4. 管程（Monitor）

管程（monitor）只是保证了同一时刻只有一个进程在管程内活动，即管程内定义的操作在同一时刻只被一个进程调用（由编译器实现）。但是这样并不能保证进程以设计的顺序执行，因此需要设置 condition 变量，让进入管程而无法继续执行的进程阻塞自己。

使用信号量机制实现的生产者消费者问题需要客户端代码做很多控制，而管程把控制的代码独立出来，不仅不容易出错，也使得客户端代码调用更容易。

管程有一个重要特性：在一个时刻只能有一个进程使用管程。进程在无法继续执行的时候不能一直占用管程，否则其它进程永远不能使用管程。

在并发编程领域，有两大核心问题：
一个是互斥，即同一时刻只允许一个线程访问共享资源；
另一个是同步，即线程之间如何通信、协作。这两大问题，管程都是能够解决的。

+ 我们先来看看管程是如何解决互斥问题的：

    管程解决互斥问题的思路很简单，就是将共享变量及其对共享变量的操作统一封装起来。在下图中，管程 X 将共享变量 queue 这个队列和相关的操作入队 enq()、出队 deq() 都封装起来了；线程 A 和线程 B 如果想访问共享变量 queue，只能通过调用管程提供的 enq()、deq() 方法来实现；enq()、deq() 保证互斥性，只允许一个线程进入管程。

+ 管程如何解决线程间的同步问题:

    + enq的时候while判断队列是否满了，如果满了，notFull.await()阻塞当前线程;
    + enq如果没满，添加对象，并且用notEmpty.single()通知deque停止阻塞；
    + deq可以顺利执行出队列的操作；
    + deq的时候while判断队列是否为空，如果为空，notEmpty.await()阻塞当前线程;
    + deq如果不为空，poll对象，并且用notFull.single()通知enq停止阻塞；
    + enq可以顺利执行队列

管程引入了 条件变量（也有可能是 Condition） 以及相关的操作：wait() 和 signal()/notify() 来实现同步操作。对条件变量执行 wait() 操作会导致调用进程阻塞，把管程让出来给另一个进程持有。signal() 操作用于唤醒被阻塞的进程。

下面是 BlockingQueue 的 [JAVA 代码](https://liuhao163.github.io/JAVA中的管程/)：

*真实的 LinkedBlockingQueue 用的是 Node 节点做链表，用了 putLock 和 takeLock 两把锁，一把从 tail 插入，一把从 head 取出，计数时用的 AtomicInteger，这样两把锁可以放取一体。*

>关于 Condition 接口：  
Condition是个接口，基本的方法就是await()和signal()方法；  
Condition依赖于Lock接口，生成一个Condition的基本代码是lock.newCondition() 
调用Condition的await()和signal() 方法，都必须在lock保护之内，就是说必须在lock.lock()和lock.unlock之间才可以使用  
Conditon中的await()对应Object的wait()；  
Condition中的signal()对应Object的notify()；  
Condition中的signalAll()对应Object的notifyAll()。  
*使用 Condition 相比于 notify 的优势是，可以对任何一个 lock 生成一个对应的 Condition，分解业务。而传统的 wait 和 notify 都是对 Object 的，一旦 notify 就全唤醒了。*

``` java
public class BlockQueue {
    ReentrantLock lock = new ReentrantLock();

    // 使用 condition 的优势就在这里，他可以对每个具体情况进行 await 和 signal
    Condition notFull = lock.newCondition();
    Condition notEmpty = lock.newCondition();

    private Queue queue = new LinkedList();
    private int queSize = 10;

    public BlockQueue(int queSize) {
        this.queSize = queSize;
    }

    public void enq(Object o) {
        lock.lock(); // 如果获取不到锁，会阻塞
        try {
            //如果为慢noFull阻塞线程
            while (queue.size() == queSize) {
                notFull.await();
            }

            queue.add(o);
            //添加成功通知deq停止阻塞
            notEmpty.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public Object deque() {
        lock.lock();
        Object ret = null;
        try {
           //如果为空notEmpty阻塞线程
            while (queue.size() == 0) {
                notEmpty.await();
            }
            return queue.poll();
        } catch (InterruptedException e) {
            e.printStackTrace();
            return null;
        } finally {
            //出队列成功通知队列未满可以入队列
            notFull.signal();
            lock.unlock();
        }
    }
}
```

## 经典同步问题

### 1. 生产者与消费者问题

+ synchronized 实现：生产和消费的过程都上上 synchronized 块，生产和消费的过程前判断一次当前数量是否为空/满即可。

    ``` java
    public class Hosee{
        private static Integer count = 0;
        private final Integer FULL = 10;
        private static String LOCK = "LOCK";
        class Producer implements Runnable{
            @Override
            public void run()
            {
                for (int i = 0; i < 10; i++)
                {
                    try
                    {
                        Thread.sleep(3000);
                    }
                    catch (Exception e)
                    {
                        e.printStackTrace();
                    }
                    synchronized (LOCK)
                    {
                        while (count == FULL)
                        {
                            try
                            {
                                LOCK.wait();
                            }
                            catch (Exception e)
                            {
                                e.printStackTrace();
                            }
                        }
                        count++;
                        System.out.println(Thread.currentThread().getName()
                                + "生产者生产，目前总共有" + count);
                        LOCK.notifyAll();
                    }
                }
            }
        }
        class Consumer implements Runnable{
            @Override
            public void run()
            {
                for (int i = 0; i < 10; i++)
                {
                    try
                    {
                        Thread.sleep(3000);
                    }
                    catch (InterruptedException e1)
                    {
                        e1.printStackTrace();
                    }
                    synchronized (LOCK)
                    {
                        while (count == 0)
                        {
                            try
                            {
                                LOCK.wait();
                            }
                            catch (Exception e)
                            {
                                // TODO: handle exception
                                e.printStackTrace();
                            }
                        }
                        count--;
                        System.out.println(Thread.currentThread().getName()
                                + "消费者消费，目前总共有" + count);
                        LOCK.notifyAll();
                    }
                }
            }
        }
        public static void main(String[] args) throws Exception{
            Hosee hosee = new Hosee();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();

            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
        }
    }
    ```
+ await() / signal()方法实现：使用 Condition，如果当生产/消费为空或满时，上对应的空/满 Condition。当有对应的写入/消费后，唤醒对应的 Condition。这样通过指定唤醒无需 synchronized。

    ``` java
    import java.util.concurrent.locks.Condition;
    import java.util.concurrent.locks.Lock;
    import java.util.concurrent.locks.ReentrantLock;

    public class Hosee {
        private static Integer count = 0;
        private final Integer FULL = 10;
        final Lock lock = new ReentrantLock();
        final Condition NotFull = lock.newCondition();
        final Condition NotEmpty = lock.newCondition();

        class Producer implements Runnable {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(3000);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    lock.lock();
                    try {
                        while (count == FULL) {
                            try {
                                NotFull.await();
                            } catch (InterruptedException e) {
                                // TODO Auto-generated catch block
                                e.printStackTrace();
                            }
                        }
                        count++;
                        System.out.println(Thread.currentThread().getName()
                                + "生产者生产，目前总共有" + count);
                        NotEmpty.signal();
                    } finally {
                        lock.unlock();
                    }

                }
            }
        }

        class Consumer implements Runnable {

            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e1) {
                        e1.printStackTrace();
                    }
                    lock.lock();
                    try {
                        while (count == 0) {
                            try {
                                NotEmpty.await();
                            } catch (Exception e) {
                                // TODO: handle exception
                                e.printStackTrace();
                            }
                        }
                        count--;
                        System.out.println(Thread.currentThread().getName()
                                + "消费者消费，目前总共有" + count);
                        NotFull.signal();
                    } finally {
                        lock.unlock();
                    }

                }

            }

        }

        public static void main(String[] args) throws Exception {
            Hosee hosee = new Hosee();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();

            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
        }

    }
    ```

+ BlockingQueue阻塞队列方法：BlockingQueue 是 JDK5.0 的新增内容，它是一个已经在内部实现了同步的队列，实现方式采用的是我们第 2 种 await()/signal() 方法。它可以在生成对象时指定容量大小。它用于阻塞操作的是 put() 和 take() 方法。

    ``` java
    import java.util.concurrent.ArrayBlockingQueue;
    import java.util.concurrent.BlockingQueue;

    public class Hosee {
        private static Integer count = 0;
        final BlockingQueue<Integer> bq = new ArrayBlockingQueue<Integer>(10);
        class Producer implements Runnable {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(3000);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    try {
                        bq.put(1);
                        count++;
                        System.out.println(Thread.currentThread().getName()
                                + "生产者生产，目前总共有" + count);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
            }
        }
        class Consumer implements Runnable {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e1) {
                        e1.printStackTrace();
                    }
                    try {
                        bq.take();
                        count--;
                        System.out.println(Thread.currentThread().getName()
                                + "消费者消费，目前总共有" + count);
                    } catch (Exception e) {
                        // TODO: handle exception
                        e.printStackTrace();
                    }
                }
            }

        }
        public static void main(String[] args) throws Exception {
            Hosee hosee = new Hosee();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();

            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
            new Thread(hosee.new Producer()).start();
            new Thread(hosee.new Consumer()).start();
        }
    }
    ```

### 2. 读者 - 写者问题


允许多个进程同时对数据进行读操作，但是不允许读和写以及写和写操作同时发生。

+ 使用 Java 的读写锁：

    ``` java
    class Queue3 {
        private Object data = null;// 共享数据，只能有一个线程能写该数据，但可以有多个线程同时读该数据。
        private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();// 该类继承 ReadWriteLock 接口

        public void get() {
            rwl.readLock().lock();// 上读锁，其他线程只能读不能写
            System.out.println(Thread.currentThread().getName()
                    + " be ready to read data!");
            try {
                Thread.sleep((long) (Math.random() * 1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()
                    + "have read data :" + data);
            rwl.readLock().unlock(); // 释放读锁，最好放在finnaly里面
        }

        public void put(Object data) {
            rwl.writeLock().lock();// 上写锁，不允许其他线程读也不允许写
            System.out.println(Thread.currentThread().getName()
                    + " be ready to write data!");
            try {
                Thread.sleep((long) (Math.random() * 1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.data = data;
            System.out.println(Thread.currentThread().getName()
                    + " have write data: " + data);
            rwl.writeLock().unlock();// 释放写锁
        }
    }
    ```

+ Semaphore信号量：当读的时候判断一次有没有在读的，如果有在读的（证明没有写的）无需获取信号量，否则获取一次信号量。写入的时候必须获取信号量。

    ``` java
    package readWriteLock;

    import java.util.concurrent.Semaphore;
    import java.util.concurrent.atomic.AtomicInteger;

    public class ReadWriteLockDemo {
        Semaphore wlock = new Semaphore(1);
        AtomicInteger readCount = new AtomicInteger(0);

        public class ReadLock{
            public void lock(){
                if(readCount.get()!=0){
                    readCount.getAndIncrement();
                    return;
                }
                try {
                    wlock.acquire(1);
                    readCount.getAndIncrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            public void unlock(){
                readCount.getAndDecrement();
                if(readCount.get()==0){
                    wlock.release();
                }
            }
        }

        public class WriteLock{
            public void lock(){
                try {
                    wlock.acquire(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            public void unlock(){
                wlock.release();
            }
        }

    }
    ```

### 3. 哲学家就餐问题

五个哲学家围着一张圆桌，每个哲学家面前放着食物。哲学家的生活有两种交替活动：吃饭以及思考。当一个哲学家吃饭时，需要先拿起自己左右两边的两根筷子，并且一次只能拿起一根筷子。

考虑到如果所有哲学家同时拿起左手边的筷子，那么就没有人拿起右手边的筷子，造成死锁。

![哲学家就餐](/images/posts/knowledge/operationSystem/哲学家就餐.jpeg)

如上图，只有 5 只筷子，首先每只筷子都是临界资源，当一个筷子被拿后，另一个人不能拿。并且是一个同步问题，当一个哲学家拿了左边的筷子后必须拿他右边的筷子。

[代码参考](https://blog.csdn.net/gao23191879/article/details/75168867)

先给出一个可能为死锁的例子，在这个例子中有可能所有的哲学家都只拿到一只筷子，就会死锁了。

``` java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;
//哲学家吃饭问题
public class ETTest {
    //创建大小为5的信号量数组，模拟5根筷子
    static Semaphore[] arry=new Semaphore[5];
    public static void main(String[] args) {
        //创建一个5个线程的线程池
        ExecutorService es=Executors.newFixedThreadPool(5);
        //初始化信号量
        for(int i=0;i<5;i++){
            arry[i]=new Semaphore(1,true);
        }
        //创建5个哲学家 但这样有可能会产生死锁问题
        for(int i=0;i<5;i++){
            es.execute(new ActionRunnable(i));
        }
    }
    //第i+1号哲学家的活动过程
    static class ActionRunnable implements Runnable{
        private int i=0;
        ActionRunnable(int i){
            this.i=i;
        }
        @Override
        public void run() {
            while(!Thread.interrupted()){
                try {
                    arry[i].acquire();
                    //请求右边的筷子
                    arry[(i+1)%5].acquire();
                    //吃饭
                    System.out.println("我是哲学家"+(i+1)+"号我在吃饭");
                    //释放左手的筷子
                    arry[i].release();
                    //释放右手的筷子
                    arry[(i+1)%5].release();
                     //哲学家开始思考
                    System.out.println("我是哲学家"+(i+1)+"号我吃饱了我要开始思考了");
                    //通知cpu 将调度权让给其他哲学家线程
                    Thread.yield(); //让步一下
                    //思考1秒
                    //把休眠关闭，造成死锁的概率就会增加 
                    //Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }
    }
}
```

+ 解法一：每次最多四个人拿筷子，这样至少有一个人能吃上

    ``` java
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    import java.util.concurrent.Semaphore;
    //哲学家吃饭问题
    public class ETTest2 {
        //创建大小为5的信号量数组，模拟5根筷子
        static Semaphore[] arry=new Semaphore[5];
        //定义一个值为4的信号量，代表最多只能有四个哲学家拿起左边的筷子
        static  Semaphore leftCount=new Semaphore(4,true);
        public static void main(String[] args) {
            //创建一个5个线程的线程池
            ExecutorService es=Executors.newFixedThreadPool(5);
            //初始化信号量
            for(int i=0;i<5;i++){
                arry[i]=new Semaphore(1,true);
            }
            //创建5个哲学家 但这样有可能会产生死锁问题
            for(int i=0;i<5;i++){
                es.execute(new ActionRunnable(i));
            }
        }
        //第i+1号哲学家的活动过程
        static class ActionRunnable implements Runnable{
            private int i=0;
            ActionRunnable(int i){
                this.i=i;
            }

            @Override
            public void run() {
                while(!Thread.interrupted()){
                    try {
                        //看拿起左边筷子的线程数是否已满,可以，则能拿起左边筷子的线程数减一，不能则等待
                        leftCount.acquire();
                        arry[i].acquire();
                        //请求右边的筷子
                        arry[(i+1)%5].acquire();
                        //吃饭
                        System.out.println("我是哲学家"+(i+1)+"号我在吃饭");
                        //释放左手的筷子
                        arry[i].release();
                        //能拿起左边筷子的线程数量加一
                        leftCount.release();
                        //释放右手的筷子
                        arry[(i+1)%5].release();
                        //哲学家开始思考
                        System.out.println("我是哲学家"+(i+1)+"号我吃饱了我要开始思考了");
                        //通知cpu 将调度权让给其他哲学家线程
                        Thread.yield();
                        //思考1秒
                        //把休眠关闭，造成死锁的概率就会增加 
                        //Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    ```

+ 解法二：奇数号的哲学家先拿起左边的筷子，在拿起右边的筷子。偶数号的哲学家先拿起右边的筷子，再拿起左边的筷子，则以就变成，只有1号和2号哲学家会同时竞争1号的筷子，3号和4四号的哲学家会同时竞争3号的筷子，即5位哲学家会先竞争奇数号的筷子，再去竞争偶数号的筷子，最后总会有一个哲学家可以进餐成功。

    ``` java
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    import java.util.concurrent.Semaphore;
    //哲学家进餐问题
    public class ETTest {
        //创建大小为5的信号量数组，模拟5根筷子
        static Semaphore[] arry=new Semaphore[5];
        public static void main(String[] args) {
            //创建一个5个线程的线程池
            ExecutorService es=Executors.newFixedThreadPool(5);

            //初始化信号量
            for(int i=0;i<5;i++){
                arry[i]=new Semaphore(1,true);
            }
            //创建5个哲学家 但这样有可能会产生死锁问题
            for(int i=0;i<5;i++){
                es.execute(new ActionRunnable(i));
            }

        }
        //第i+1号哲学家的活动过程
        static class ActionRunnable implements Runnable{
            private int i=0;
            ActionRunnable(int i){
                this.i=i;
            }

            @Override
            public void run() {
                while(!Thread.interrupted()){
                    try {
                        if((i+1)%2!=0){
                        //奇数号哲学家
                        //请求左边的筷子
                        arry[i].acquire();
                        //请求右边的筷子
                        arry[(i+1)%5].acquire();
                        }else{
                        //偶数号哲学家
                            //请求右边的筷子
                            arry[(i+1)%5].acquire();
                            //再请求左边的筷子
                            arry[i].acquire();
                        }
                        //吃饭
                        System.out.println("我是哲学家"+(i+1)+"号我在吃饭");
                        if((i+1)%2!=0){
                        //奇数号哲学家
                        //释放左手的筷子
                        arry[i].release();
                        //释放右手的筷子
                        arry[(i+1)%5].release();
                        }else{
                        //偶数号的哲学家
                            arry[(i+1)%5].release();
                            arry[i].release();
                        }
                        //哲学家开始思考
                        System.out.println("我是哲学家"+(i+1)+"号我吃饱了我要开始思考了");
                        //通知cpu 将调度权让给其他哲学家线程
                        Thread.yield();
                        //思考1秒
                        //把休眠关闭，造成死锁的概率就会增加 
                        //Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    ```

*TODO:// 哲学家问题需要重新检查是否正确。*

## 进程通信

进程同步与进程通信很容易混淆，它们的区别在于：

+ 进程同步：控制多个进程按一定顺序执行；
+ 进程通信：进程间传输信息。

进程通信是一种手段，而进程同步是一种目的。也可以说，为了能够达到进程同步的目的，需要让进程进行通信，传输一些进程同步所需要的信息。

### 1. 管道

管道是单向的、先进先出的、无结构的字节流，它把一个进程的输出和另一个进程的输入连接在一起。

+ 写进程在管道的尾端写入数据，读进程在管道的首端读出数据。数据读出后将从管道中移走，其它读进程都不能再读到这些数据。
+ 管道提供了简单的流控制机制。进程试图读一个空管道时，在数据写入管道前，进程将一直阻塞。同样，管道已经满时，进程再试图写管道，在其它进程从管道中读走数据之前，写进程将一直阻塞。

匿名管道具有的特点：

+ 只能用于具有亲缘关系的进程之间的通信（也就是父子进程或者兄弟进程之间）。

+ 一种半双工的通信模式，具有固定的读端和写端。

+ LINUX 把管道看作是一种文件，采用文件管理的方法对管道进行管理，对于它的读写也可以使用普通的 read() 和 write() 等函数。但是它不是普通的文件，并不属于其他任何文件系统，只存在于内核的内存空间中。

#### 管道创建与关闭说明

+ 管道是基于文件描述符的通信方式，当一个管道建立时，它会创建两个文件描述符fds[0]和fds[1]：


+ fd[0]固定用于读管道；（0 一般是标准输入）

+ fd[1]固定用于写管道；（1 一般是标准输出，2 是错误输出）

+ 管道关闭时只需将这两个文件描述符关闭即可，可使用普通的close()函数逐个关闭各个文件描述符。

#### 管道创建函数（C 语言中，在 JAVA 中管道只能用于线程通信，并不常用）

创建管道可以通过调用pipe()来实现：

![管道 pipe](/images/posts/knowledge/operationSystem/管道.png)

它具有以下限制（即匿名管道）：

+ 只支持半双工通信（单向交替传输）；
+ 只能在父子进程或者兄弟进程中使用。

![管道 C 示意图](/images/posts/knowledge/operationSystem/管道2.png)

``` c++
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
 
int pipe_default[2];  
 
int main()
{
    pid_t pid;
    char buffer[32];
 
    memset(buffer, 0, 32);
    if(pipe(pipe_default) < 0)
    {
        printf("Failed to create pipe!\n");
        return 0;
    }
 
    if(0 == (pid = fork()))
    {
        close(pipe_default[1]);
        sleep(5);
        if(read(pipe_default[0], buffer, 32) > 0)
        {
            printf("Receive data from server, %s!\n", buffer);
        }
        close(pipe_default[0]);
    }
    else
    {
        close(pipe_default[0]);
        if(-1 != write(pipe_default[1], "hello", strlen("hello")))
        {
            printf("Send data to client, hello!\n");
        }
        close(pipe_default[1]);
        waitpid(pid, NULL, 0);
    }
 
    return 1;
}
```

### 2. FIFO

也称为命名管道，去除了管道只能在父子进程中使用的限制。命名管道也被称为FIFO文件，它是一种特殊类型的文件，它在文件系统中以文件名的形式存在，但是它的行为却和之前所讲的没有名字的管道（匿名管道）类似。

由于Linux中所有的事物都可被视为文件，所以对命名管道的使用也就变得与文件操作非常的统一，也使它的使用非常方便，同时我们也可以像平常的文件名一样在命令中使用。

我们可以使用两下函数之一来创建一个命名管道，他们的原型如下：

``` c++
#include <sys/stat.h>
int mkfifo(const char *path, mode_t mode);
int mkfifoat(int fd, const char *path, mode_t mode);
```

FIFO 常用于客户-服务器应用程序中，FIFO 用作汇聚点，在客户进程和服务器进程之间传递数据。

![FIFO](/images/posts/knowledge/operationSystem/fifo.png)

### 3. 消息队列

相比于 FIFO，消息队列具有以下优点：

+ 消息队列可以独立于读写进程存在，从而避免了 FIFO 中同步管道的打开和关闭时可能产生的困难；
+ 避免了 FIFO 的同步阻塞问题，不需要进程自己提供同步方法；
+ 读进程可以根据消息类型有选择地接收消息，而不像 FIFO 那样只能默认地接收。

### 4. 信号量

它是一个计数器，用于为多个进程提供对共享数据对象的访问。

### 5. 共享内存

允许多个进程共享一个给定的存储区。因为数据不需要在进程之间复制，所以这是最快的一种 IPC。

需要使用信号量用来同步对共享存储的访问。

多个进程可以将同一个文件映射到它们的地址空间从而实现共享内存。另外 XSI 共享内存不是使用文件，而是使用内存的匿名段。

Java 中共享内存可以用 MappedByteBuffer（java.nio 中）实现。[参考](https://zhuanlan.zhihu.com/p/27698585)具体如下：

+ 写到内存：

  ``` java
  public class Main {
      public static void main(String args[]){
          RandomAccessFile f = null;
          try {
              f = new RandomAccessFile("C:/hinusDocs/hello.txt", "rw");
              FileChannel fc = f.getChannel();
              MappedByteBuffer buf = fc.map(FileChannel.MapMode.READ_WRITE, 0, 20);
              // 只是在内存中写入
              buf.put("how are you?".getBytes());

              Thread.sleep(10000);

              fc.close();
              f.close();

          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  ```

+ 从内存中读出

  ``` java
  public class MapMemoryBuffer {
      public static void main(String[] args) throws Exception {
          RandomAccessFile f = new RandomAccessFile("C:/hinusDocs/hello.txt", "rw");
          FileChannel fc = f.getChannel();
          MappedByteBuffer buf = fc.map(FileChannel.MapMode.READ_WRITE, 0, fc.size());

          while (buf.hasRemaining()) {
              System.out.print((char)buf.get());
          }
          System.out.println();
      }
  }
  ```

原理是使用 mmap 系统调用（linux系统调封装）。

### 6. 套接字（Socket）

与其它通信机制不同的是，它可用于不同机器间的进程通信。

## 死锁

### 必要条件

![死锁](/images/posts/knowledge/operationSystem/死锁.png)

+ 互斥：每个资源要么已经分配给了一个进程，要么就是可用的。
+ 占有和等待：已经得到了某个资源的进程可以再请求新的资源。
+ 不可抢占：已经分配给一个进程的资源不能强制性地被抢占，它只能被占有它的进程显式地释放。
+ 环路等待：有两个或者两个以上的进程组成一条环路，该环路中的每个进程都在等待下一个进程所占有的资源。

### 处理办法

主要有以下四种方法：

+ 鸵鸟策略：
+ 死锁检测与死锁恢复
+ 死锁预防
+ 死锁避免

#### 鸵鸟策略

把头埋在沙子里，假装根本没发生问题。

因为解决死锁问题的代价很高，因此鸵鸟策略这种不采取任务措施的方案会获得更高的性能。

当发生死锁时不会对用户造成多大影响，或发生死锁的概率很低，可以采用鸵鸟策略。

大多数操作系统，包括 Unix，Linux 和 Windows，处理死锁问题的办法仅仅是忽略它。

#### 死锁检查与死锁恢复

![死锁检查与死锁恢复](/images/posts/knowledge/operationSystem/死锁检测与恢复.png)

**不试图阻止死锁**，而是当检测到死锁发生时，采取措施进行恢复。

1. 每种类型一个资源的死锁检测

    当可能出现死锁时，进程和资源请求会产生了环路等待条件，因此会发生死锁。

    每种类型一个资源的死锁检测算法是通过检测有向图是否存在环来实现，从一个节点出发进行深度优先搜索，对访问过的节点进行标记，如果访问了已经标记的节点，就表示有向图存在环，也就是检测到死锁的发生。

2. 每种类型多个资源的死锁恢复

    ![多个资源死锁](/images/posts/knowledge/operationSystem/多个资源死锁.png)

    上图中，有三个进程四个资源，每个数据代表的含义如下：

    + E 向量：资源总量
    + A 向量：资源剩余量
    + C 矩阵：每个进程所拥有的资源数量，每一行都代表一个进程拥有资源的数量
    + R 矩阵：每个进程请求的资源数量

    进程 P1 和 P2 所请求的资源都得不到满足，只有进程 P3 可以，让 P3 执行，之后释放 P3 拥有的资源，此时 A = (2 2 2 0)。P2 可以执行，执行后释放 P2 拥有的资源，A = (4 2 2 1) 。P1 也可以执行。所有进程都可以顺利执行，没有死锁。

    算法总结如下：
  
    每个进程最开始时都不被标记，执行过程有可能被标  记。当算法结束时，任何没有被标记的进程都是死锁进  程。
  
    1. 寻找一个没有标记的进程 Pi，它所请求的资源小于等于 A。
    2. 如果找到了这样一个进程，那么将 C 矩阵的第 i 行向量加到 A 中，标记该进程，并转回 1。
    3. 如果没有这样一个进程，算法终止。

3. 死锁恢复

    + 利用抢占恢复
    + 利用回滚恢复
    + 最简单，最常用的方法就是进行系统的重新启动，不过这种方法代价很大，它意味着在这之前所有的进程已经完成的计算工作都将付之东流，包括参与死锁的那些进程，以及未参与死锁的进程。
    + 撤消进程，剥夺资源。终止参与死锁的进程，收回它们占有的资源，从而解除死锁。这时又分两种情况：一次性撤消参与死锁的全部进程，剥夺全部资源；或者逐步撤消参与死锁的进程，逐步收回死锁进程占有的资源。一般来说，选择逐步撤消的进程时要按照一定的原则进行，目的是撤消那些代价最小的进程，比如按进程的优先级确定进程的代价；考虑进程运行时的代价和与此进程相关的外部作业的代价等因素。

#### 死锁预防

**在程序运行之前预防发生死锁。通过破坏死锁的必要条件。**

1. 破坏互斥条件
  
    如果允许系统资源都能共享使用，则系统不会进入死锁状态。但有些资源根本不能同时访问，如打印机等临界资源只能互斥使用。

    例如假脱机打印机技术允许若干个进程同时输出，唯一真正请求物理打印机的进程是打印机守护进程。

2. 破坏占有和等待条件

    釆用预先静态分配方法，即进程在运行前一次申请完它所需要的全部资源，在它的资源未满足前，不把它投入运行。一旦投入运行后，这些资源就一直归它所有，也不再提出其他资源请求，这样就可以保证系统不会发生死锁。

    这种方式实现简单，但缺点也显而易见，系统资源被严重浪费，其中有些资源可能仅在运行初期或运行快结束时才使用，甚至根本不使用。而且还会导致“饥饿”现象，当由于个别资源长期被其他进程占用时，将致使等待该资源的进程迟迟不能开始运行。

    一种实现方式是规定所有进程在开始执行前请求所需要的全部资源。

3. 破坏不可抢占条件

    当一个已保持了某些不可剥夺资源的进程，请求新的资源而得不到满足时，它必须释放已经保持的所有资源，待以后需要时再重新申请。这意味着，一个进程已占有的资源会被暂时释放，或者说是被剥夺了，或从而破坏了不可剥夺条件。

    该策略实现起来比较复杂，释放已获得的资源可能造成前一阶段工作的失效，反复地申请和释放资源会增加系统开销，降低系统吞吐量。这种方法常用于状态易于保存和恢复的资源，如CPU的寄存器及内存资源，一般不能用于打印机之类的资源。

4. 破坏环路等待

    为了破坏循环等待条件，可釆用顺序资源分配法。首先给系统中的资源编号，规定每个进程，必须按编号递增的顺序请求资源，同类资源一次申请完。也就是说，只要进程提出申请分配资源 Ri，则该进程在以后的资源申请中，只能申请编号大于 Ri 的资源。

    这种方法存在的问题是，编号必须相对稳定，这就限制了新类型设备的增加；尽管在为资源编号时已考虑到大多数作业实际使用这些资源的顺序，但也经常会发生作业使甩资源的顺序与系统规定顺序不同的情况，造成资源的浪费；此外，这种按规定次序申请资源的方法，也必然会给用户的编程带来麻烦。

#### 死锁避免

**在程序运行时避免发生死锁。通过计算是否可能出现死锁。**

1. 安全状态

    ![安全状态](/images/posts/knowledge/operationSystem/安全状态.png)

    图 a 的第二列 Has 表示已拥有的资源数，第三列 Max 表示总共需要的资源数，Free 表示还有可以使用的资源数。从图 a 开始出发，先让 B 拥有所需的所有资源（图 b），运行结束后释放 B，此时 Free 变为 5（图 c）；接着以同样的方式运行 C 和 A，使得所有进程都能成功运行，因此可以称图 a 所示的状态时安全的。

    定义：如果没有死锁发生，并且即使所有进程突然请求对资源的最大需求，也仍然存在某种调度次序能够使得每一个进程运行完毕，则称该状态是安全的。

    安全状态的检测与死锁的检测类似，因为安全状态必须要求不能发生死锁。下面的银行家算法与死锁检测算法非常类似，可以结合着做参考对比。

2. 银行家算法

    ![银行家算法](/images/posts/knowledge/operationSystem/银行家算法.png)

    假设资源P1申请资源，银行家算法先试探的分配给它（当然先要看看当前资源池中的资源数量够不够），若申请的资源数量小于等于 Available，然后接着判断分配给P1后剩余的资源，能不能使进程队列的某个进程执行完毕，若没有进程可执行完毕，则系统处于不安全状态（即此时没有一个进程能够完成并释放资源，随时间推移，系统终将处于死锁状态）。

    若有进程可执行完毕，则假设回收已分配给它的资源（剩余资源数量增加），把这个进程标记为可完成，并继续判断队列中的其它进程，若所有进程都可执行完毕，则系统处于安全状态，并根据可完成进程的分配顺序生成安全序列（如 {P0，P3，P2，P1} 表示将申请后的剩余资源Work先分配给P0–>回收（Work+已分配给P0的A0=Work）–> 分配给P3 –> 回收（Work+A3=Work）–> 分配给P2 –> ······ 满足所有进程）。

    + 单个资源

        一个小城镇的银行家，他向一群客户分别承诺了一定的贷款额度，算法要做的是判断对请求的满足是否会进入不安全状态，如果是，就拒绝请求；否则予以分配。

        ![单个资源银行家](/images/posts/knowledge/operationSystem/单个资源银行家.png)

        对于上图，(a)，(b) 状态是安全的，但 (c) 是不安全的。因此算法会拒绝之前的请求，从而避免进入图 c 中的状态。
  
    + 多个资源

        ![多个资源银行家](/images/posts/knowledge/operationSystem/多个资源.png)

        上图中有五个进程，四个资源。左边的图表示已经分配的资源，右边的图表示还需要分配的资源。最右边的 E、P 以及 A 分别表示：总资源、已分配资源以及可用资源，注意这三个为向量，而不是具体数值，例如 A=(1020)，表示 4 个资源分别还剩下 1/0/2/0。

        检查一个状态是否安全的算法如下：

        查找右边的矩阵是否存在一行小于等于向量 A。如果不存在这样的行，那么系统将会发生死锁，状态是不安全的。

        假若找到这样一行，将该进程标记为终止，并将其已分配资源加到 A 中。

        重复以上两步，直到所有进程都标记为终止，则状态时安全的。

        如果一个状态不是安全的，需要拒绝进入这个状态。

#### 死锁检测、死锁预防和死锁避免区别

死锁检测是不怕出现死锁，出现死锁后检测到了死锁并对死锁恢复。

死锁的防止是系统预先确定一些资源分配策略，进程按规定申请资源，系统按预先规定的策略进行分配从而防止死锁的发生。而死锁的避免是当进程提出资源申请时系统测试资源分配仅当能确保系统安全时才把资源分配给进程，使系统一直处于安全状态之中，从而避免死锁。

# 3. 内存管理

## 内存管理的方法

内存管理的方法有许多种，固定加载地址的、固定分区的、非固定分区的和交换内存管理，其中只有第一种固定加载地址的内存管理适用于单道编程，其余三种则适合多道编程。与此同时，它们的共同实现机制是——基址和极限。

交换内存管理是这些方法中最灵活的。它使用的基址和极限机制实际上就是“将程序发出的虚拟地址加上基址得到物理地址”。

但这样也就会带来两大问题：

+ 空间浪费：程序不断的执行并释放的过程中，造成了内存空间中的可用空间不连续就，难以加以应用，这种现象也称为“外部碎片化”。 
  ![空间浪费](/images/posts/knowledge/operationSystem/20171104145803969.png)

+ 程序大小受限：当程序需要更多的内存空间时，需要将其全部从物理内存中“倒出”到磁盘上，再在内存中找到更大的一片区域去存放增长了的程序，这样使得程序的增长效率低下。同时，一个程序的大小还不能超过物理内存空间的大小。

## 虚拟内存

传统存储管理有的一个缺点：很多用不到的数据也会长期地占用内存，导致内存利用率不高。

传统存储管理还有以下几个特征：

1. 一次性：作业必须一次性全部装入内存后才能开始运行。这会造成两个问题：

    + 作业很大时，不能全部装入内存，导致大作业无法运行：(比如大型的游戏)

    + 当大量作业要求运行时，由于内存无法容纳所有作业，因此只有少量作业能运行，导致多道程序并发度下降。

2. 驻留性：一旦作业被装入内存，就会一直驻留在内存中，直至作业运行结束。事实上，在一个时间段内，只需访问作业的一小部分数据即可正常运行，这就导致了内存中会驻留大量的、暂时用不到的数据，浪费了宝贵的内存资源。

虚拟存储技术是基于局部性原理提出来的:

+ 时间局部性：如果执行了程序中的某条指令，那么不久后这条指令很有可能再次被执行；如果某个数据被访问过，不就之后该数据很有可能再次被访问。（因为程序中存在大量的循环）
+ 空间局部性：一旦程序访问了某个存户单元，在不久之后，其附近的存储单元也很有可能被访问。（因为很多数据在内存中都是连续存放的）。

虚拟内存的目的是为了让物理内存扩充成更大的逻辑内存，从而让程序获得更多的可用内存。

为了更好的管理内存，操作系统将内存抽象成地址空间。每个程序拥有自己的地址空间，这个地址空间被分割成多个块，每一块称为一页。这些页被映射到物理内存，但不需要映射到连续的物理内存，也不需要所有页都必须在物理内存中。当程序引用到不在物理内存中的页时，由硬件执行必要的映射，将缺失的部分装入物理内存并重新执行失败的指令。

从上面的描述中可以看出，虚拟内存允许程序不用将地址空间中的每一页都映射到物理内存，也就是说一个程序不需要全部调入内存就可以运行，这使得有限的内存运行大程序成为可能。例如有一台计算机可以产生 16 位地址，那么一个程序的地址空间范围是 0~64K。该计算机只有 32KB 的物理内存，虚拟内存技术允许该计算机运行一个 64K 大小的程序。

![虚拟内存](/images/posts/knowledge/operationSystem/虚拟内存.png)

## 分页（系统地址映射）

用户程序的地址空间被划分成若干固定大小的区域，称为“页”，相应地，内存空间分成若干个物理块，页和块的大小相等。可将用户程序的任一页放在内存的任一块中，实现了离散分配。

**内存管理单元（MMU）**管理着地址空间和物理内存的转换，其中的页表（Page table）存储着页帧（程序地址空间）和页框（物理内存空间）的映射表。

- 页帧：被分成一页大小的逻辑地址
- 页框：被分成一页大小的物理地址
- 页表：页帧映射到页框的表格
- 标志位：标志页帧是否已成功映射到页框，0否 1是

一个虚拟地址分成两个部分，一部分存储页面号，一部分存储偏移量。

下图的页表存放着 16 个页，这 16 个页需要用 4 个比特位来进行索引定位。例如对于虚拟地址（0010 000000000100），前 4 位是存储页面号 2，读取表项内容为（110 1），页表项最后一位表示是否存在于内存中，1 表示存在。后 12 位存储偏移量。这个页对应的页框的地址为 （110 000000000100）。

![分页地址映射](/images/posts/knowledge/operationSystem/分页地址映射.png)

### 页面置换算法

在程序运行过程中，如果要访问的页面不在内存中，就发生缺页中断从而将该页调入内存中。此时如果内存已无空闲空间，系统必须从内存中调出一个页面到磁盘对换区中来腾出空间。

页面置换算法和缓存淘汰策略类似，可以将内存看成磁盘的缓存。在缓存系统中，缓存的大小有限，当有新的缓存到达时，需要淘汰一部分已经存在的缓存，这样才有空间存放新的缓存数据。

页面置换算法的主要目标是使页面置换频率最低（也可以说缺页率最低）。

1. 最佳（理论上）

    > OPT, Optimal replacement algorithm

    所选择的被换出的页面将是最长时间内不再被访问，通常可以保证获得最低的缺页率。

    是一种理论上的算法，因为无法知道一个页面多长时间不再被访问。

    举例：一个系统为某进程分配了三个物理块，并有如下页面引用序列：

    7，0，1，2，0，3，0，4，2，3，0，3，2，1，2，0，1，7，0，1

    开始运行时，先将 7, 0, 1 三个页面装入内存。当进程要访问页面 2 时，产生缺页中断，会将页面 7 换出，因为页面 7 再次被访问的时间最长。

2. 最近最久未使用换出

    > LRU, Least Recently Used

    **把最近用的放在前面，不常用的放在后面，淘汰后面的。是最常用的页面置换算法。**

    虽然无法知道将来要使用的页面情况，但是可以知道过去使用页面的情况。LRU 将最近最久未使用的页面换出。

    为了实现 LRU，需要在内存中维护一个所有页面的链表/计数器。
  
    使用链表时，当一个页面被访问时，将这个页面移到链表表头。这样就能保证链表表尾的页面是最近最久未访问的。因为每次访问都需要更新链表，因此这种方式实现的 LRU 代价很高。
  
    使用计数器时，每个页表项对应一个使用时间字段，并给CPU增加一个逻辑时钟或计数器。每次存储访问，该时钟都加1。每当访问一个页面时，时钟寄存器的内容就被复制到相应页表项的使用时间字段中。这样我们就可以始终保留着每个页面最后访问的“时间”。在置换页面时，选择该时间值最小的页面。这样做，不仅要查页表，而且当页表改变时（因CPU调度）要维护这个页表中的时间，还要考虑到时钟值溢出的问题。

    举例：4，7，0，7，1，0，1，2，1，2，6

    ![LRU](/images/posts/knowledge/operationSystem/LRU.png)

3. 最近未使用

    > NRU, Not Recently Used

    考虑到LRU实现困难，Clock 页面置换算法（NRU）应运而生。

    记录谁最早被使用很难，那么换一种思路，把时间分成一个个周期，如果最近一个周期都没有被使用，那就干脆当做一直没有被使用。

    本质上这可以说是一种逆向思维：不一定要最早被使用的被淘汰，只要不是最近被使用的被淘汰就好了。而统计谁最近被使用则成本低廉，只需要在某个时间段清空访问状态，是否新被访问就很容易判断。

    在最近的一个时钟周期内，淘汰一个没有被访问的已修改页面，近似 LRU 算法，NRU 只是更粗略些。

    每个页面都有两个状态位：R 与 M，当页面被访问时设置页面的 R=1，当页面被修改时设置 M=1。其中 R 位会定时被清零。可以将页面分成以下四类：

    + R=0，M=0
    + R=0，M=1
    + R=1，M=0
    + R=1，M=1

    当发生缺页中断时，NRU 算法随机地从类编号最小的非空类中挑选一个页面将它换出。

    NRU 优先换出已经被修改的脏页面（R=0，M=1），而不是被频繁使用的干净页面（R=1，M=0）。

4. 先进先出

    > FIFO, First In First Out

    与队列相同，选择换出的页面是最先进入的页面。

    该算法会将那些经常被访问的页面也被换出，从而使缺页率升高。

5. 第二次机会算法

    FIFO 算法可能会把经常使用的页面置换出去，为了避免这一问题，对该算法做一个简单的修改：

    当页面被访问 (读或写) 时设置该页面的 R 位为 1。需要替换的时候，检查最老页面的 R 位。如果 R 位是 0，那么这个页面既老又没有被使用，可以立刻置换掉；如果是 1，就将 R 位清 0，并把该页面放到链表的尾端（最后，即再轮一轮才是淘汰他），修改它的装入时间使它就像刚装入的一样，然后继续从链表的头部开始搜索。

    第二次机会算法所做的是寻找一个从上一次对它检查以来没有引用过的页面，如果所有的页都被引用了，它就降级为纯粹的FIFO算法。特别地，让我们设想假如所有的页的R位都被设置了，操作系统将一个接一个地把各个页移到链表的尾部并清除被移动的页的R位。最后算法又将回到页A，这时它的R位已经被清除了，因此A将被淘汰，所以这个算法总是可以结束的。

6. 时钟

    第二次机会算法需要在链表中移动页面，降低了效率。时钟算法使用环形链表将页面连接起来，再使用一个指针指向最老的页面。

    ![时钟](/images/posts/knowledge/operationSystem/时钟.png)

## 分段

分段的做法是将用户程序地址空间分成若干个大小不等的段，每段可以定义一组相对完整的逻辑信息。存储分配时，以段为单位，段与段在内存中可以不相邻接，也实现了离散分配。

![分段](/images/posts/knowledge/operationSystem/0943350.jpg)

## 段页式

程序的地址空间划分成多个拥有独立地址空间的段，每个段上的地址空间划分成大小相同的页。这样既拥有分段系统的共享和保护，又拥有分页系统的虚拟内存功能。

## 分页与分段的比较

+ 对程序员的透明性：分页透明，但是分段需要程序员显式划分每个段。

+ 地址空间的维度：分页是一维地址空间，第 0 页的最后一个地址和第 1 页的第一个地址在数值上是连续的。因此分页机制的逻辑地址空间是一维的。。分段是二维的，分段机制，这段程序就会被编译程序编译成多个段，比如数据段、代码段、附加段等，每个段的段号是编译器自动分配的，每个段的长度不定（一般最长64k）。由于每个段的长度不一，因此虽然数据段、代码段的段号是连续的，但是数据段的最后一个地址和代码段的第一个地址是不连续，因此分段机制中的地址不是一维的，而是二维的。

+ 大小是否可以改变：页的大小不可变，段的大小可以动态改变。

+ 出现的原因：分页主要用于实现虚拟内存，从而获得更大的地址空间；分段主要是为了使程序和数据可以被划分为逻辑上独立的地址空间并且有助于共享和保护。

## 分页和分段的区别

- 目的不同：分页的目的是管理内存，用于虚拟内存以获得更大的地址空间；分段的目的是满足用户的需要，使程序和数据可以被划分为逻辑上独立的地址空间；
- 大小不同：段的大小不固定，由其所完成的功能决定；页的大小固定，由系统决定；
- 地址空间维度不同：分段是二维地址空间（段号+段内偏移），分页是一维地址空间（每个进程一个页表/多级页表，通过一个逻辑地址就能找到对应的物理地址）；
- 分段便于信息的保护和共享；分页的共享受到限制；
- 碎片：分段没有内碎片，但会产生外碎片；分页没有外碎片，但会产生内碎片（一个页填不满）

# 4. 磁盘管理

## 磁盘结构

+ 盘面（Platter）：一个磁盘有多个盘面；
+ 磁道（Track）：盘面上的圆形带状区域，一个盘面可以有多个磁道；
+ 扇区（Track Sector）：磁道上的一个弧段，一个磁道可以有多个扇区，它是最小的物理储存单位，目前主要有 512 bytes 与 4 K 两种大小；
+ 磁头（Head）：与盘面非常接近，能够将盘面上的磁场转换为电信号（读），或者将电信号转换为盘面的磁场（写）；
+ 制动手臂（Actuator arm）：用于在磁道之间移动磁头；
+ 主轴（Spindle）：使整个盘面转动。

![磁盘](/images/posts/knowledge/operationSystem/磁盘.jpeg)

## 磁盘调度算法

读写一个磁盘块的时间的影响因素有：

+ 旋转时间（主轴转动盘面，使得磁头移动到适当的扇区上）
+ 寻道时间（制动手臂移动，使得磁头移动到适当的磁道上）
+ 实际的数据传输时间

其中，寻道时间最长，因此磁盘调度的主要目标是使磁盘的平均寻道时间最短。

## 1. 先来先服务（FCFS, First Come First Served）

按照磁盘请求的顺序进行调度。

优点是公平和简单。缺点也很明显，因为未对寻道做任何优化，使平均寻道时间可能较长。

## 2. 最短寻道时间优先（SSTF, Shortest Seek Time First）

优先调度与当前磁头所在磁道距离最近的磁道。

虽然平均寻道时间比较低，但是不够公平。如果新到达的磁道请求总是比一个在等待的磁道请求近，那么在等待的磁道请求会一直等待下去，也就是出现饥饿现象。具体来说，两端的磁道请求更容易出现饥饿现象。

## 3. 电梯算法（SCAN）

电梯总是保持一个方向运行，直到该方向没有请求为止，然后改变运行方向。

电梯算法（扫描算法）和电梯的运行过程类似，总是按一个方向来进行磁盘调度，直到该方向上没有未完成的磁盘请求，然后改变方向。

因为考虑了移动方向，因此所有的磁盘请求都会被满足，解决了 SSTF 的饥饿问题。

## RAID [参考](https://github.com/jayesslin/Backend_Interview/blob/master/操作系统部分/操作系统部分.md)

磁盘阵列（Redundant Arrays of Independent Disks，RAID），独立冗余磁盘阵列之。原理是利用数组方式来作磁盘组，配合数据分散排列的设计，提升数据的安全性。

+ RAID 0
    + RAID 0是最早出现的RAID模式，需要2块以上的硬盘，可以提高整个磁盘的性能和吞吐量。
    + RAID 0没有提供冗余或错误修复能力，其中一块硬盘损坏，所有数据将遗失。

+ RAID 1
    + RAID 1就是镜像，其原理为在主硬盘上存放数据的同时也在镜像硬盘上写一样的数据。
    + 当主硬盘（物理）损坏时，镜像硬盘则代替主硬盘的工作。因为有镜像硬盘做数据备份，所以RAID 1的数据安全性在所有的RAID级别上来说是最好的。
    + 但无论用多少磁盘做RAID 1，仅算一个磁盘的容量，是所有RAID中磁盘利用率最低的。

**RAID 2/3/4 很少用**

+ RAID 2
    这是RAID 0的改良版，以汉明码（Hamming Code）的方式将数据进行编码后分区为独立的比特，并将数据分别写入硬盘中。因为在数据中加入了错误修正码（ECC，Error Correction Code），所以数据整体的容量会比原始数据大一些，RAID2最少要三台磁盘驱动器方能运作。
+ RAID 3
    + 采用Bit－interleaving（数据交错存储）技术，它需要通过编码再将数据比特分割后分别存在硬盘中，而将同比特检查后单独存在一个硬盘中，但由于数据内的比特分散在不同的硬盘上，因此就算要读取一小段数据资料都可能需要所有的硬盘进行工作，所以这种规格比较适于读取大量数据时使用。
+ RAID 4
    + 它与RAID 3不同的是它在分区时是以区块为单位分别存在硬盘中，但每次的数据访问都必须从同比特检查的那个硬盘中取出对应的同比特数据进行核对，由于过于频繁的使用，所以对硬盘的损耗可能会提高。（块交织技术，Block interleaving）

+ RAID 5
    + RAID Level 5是一种储存性能、数据安全和存储成本兼顾的存储解决方案。它使用的是Disk Striping（硬盘分区）技术。
    + RAID 5至少需要三块硬盘，RAID 5不是对存储的数据进行备份，而是把数据和相对应的奇偶校验信息存储到组成RAID5的各个磁盘上，并且奇偶校验信息和相对应的数据分别存储于不同的磁盘上。
    + RAID 5 允许一块硬盘损坏。
    + 实际容量 Size = (N-1) * min(S1, S2, S3 ... SN)

+ RAID 6
    + 与RAID 5 相比，RAID 6 增加第二个独立的奇偶校验信息块。两个独立的奇偶系统使用不同的算法，数据的可靠性非常高，即使两块磁盘同时失效也不会影响数据的使用。
    + RAID 6 至少需要4块硬盘。
    + 实际容量 Size = (N-2) * min(S1, S2, S3 ... SN)

+ RAID 10/01（RAID 1+0，RAID 0+1）
    + RAID 10 是先镜射再分区数据，再将所有硬盘分为两组，视为是RAID 0的最低组合，然后将这两组各自视为RAID 1运作。
    + RAID 01 则是跟 RAID 10 的程序相反，是先分区再将数据镜射到两组硬盘。它将所有的硬盘分为两组，变成 RAID 1 的最低组合，而将两组硬盘各自视为 RAID 0 运作。
    + 当 RAID 10 有一个硬盘受损，其余硬盘会继续运作。 RAID 01 只要有一个硬盘受损，同组 RAID 0 的所有硬盘都会停止运作，只剩下其他组的硬盘运作，可靠性较低。如果以六个硬盘建 RAID 01，镜射再用三个建 RAID 0，那么坏一个硬盘便会有三个硬盘脱机。因此，RAID 10 远较 RAID 01 常用，零售主板绝大部份支持 RAID 0/1/5/10，但不支持 RAID 01。
    + RAID 10 至少需要4块硬盘，且硬盘数量必须为偶数。

# 5. [链接](https://github.com/CyC2018/CS-Notes/blob/master/notes/计算机操作系统%20-%20链接.md)
