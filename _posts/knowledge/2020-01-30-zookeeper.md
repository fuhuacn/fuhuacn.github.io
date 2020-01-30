---
layout: post
title: Zookeeper 原理解析
categories: Knowledge
description: Zookeeper 原理解析
keywords: Zookeeper,原理解析
---
[参考来源](https://juejin.im/post/5d1375b8f265da1bc75249b8)

目录

* TOC
{:toc}

ZooKeeper 是一个分布式的，开放源码的分布式应用程序协调服务，它是一个**为分布式应用提供一致性服务的软件**，提供的功能包括：数据发布、负载均衡、配置维护、域名服务、分布式同步、组服务等。Zookeeper具有以下特性：

- 最终一致性：为客户端展示同一个视图，这是 zookeeper 里面一个非常重要的功能
- 可靠性：如果消息被到一台服务器接受，那么它将被所有的服务器接受。
- 实时性：Zookeeper 不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用 sync() 接口
- 独立性：各个 Client 之间互不干预
- 原子性：更新只能成功或者失败，没有中间状态。
- 顺序性：所有 Server，同一消息发布顺序一致。

# zookeeper 设计目标

Zookeeper致力于提供一个高性能、高可用、且具有严格的顺序访问控制能力（主要是写操作的严格顺序性）的分布式协调服务，其具有如下的设计目标：

1. 简单的数据模型，Zookeeper 使得分布式程序能够通过一个**共享的树形结构的名字空间**来进行相互协调，即 Zookeeper 服务器内存中的数据模型由一系列被称为 ZNode 的数据节点组成，Zookeeper 将全量的数据存储在内存中，以此来提高服务器吞吐、减少延迟的目的。
2. 可构建集群，一个 Zookeeper 集群通常由一组机器构成，组成 Zookeeper 集群的而每台机器都会在内存中维护当前服务器状态，并且每台机器之间都相互通信。
3. 顺序访问，对于来自客户端的每个更新请求，Zookeeper 都会分配一个全局唯一的递增编号，这个编号反映了所有事务操作的先后顺序。
4. 高性能，Zookeeper **将全量数据存储在内存中**，并直接服务于客户端的所有非事务请求，因此它尤其适用于**以读操作为主**的应用场景。

# zookeeper 架构图

![架构图](/images/posts/knowledge/zookeeper/架构图.png)

## zookeeper server 角色

- leader：负责进行投票的发起和决议以及更新系统状态
- learner：
  - follower：**处理客户端读请求并向客户端返回结果。**如果是写事务，将请求转发给 leader。同步 leader 状态，会参加投票过程。
  - observer：**接受客户端读请求**，将客户端写请求转发给 leader，但 observer 不参加投票过程，只同步 leader 的状态，observer 的目的是为了扩展系统，提高读取速度。
  - follower 和 observer 区别：随着我们添加更多投票成员，写入性能也会随着下降。这是因为写操作需要（通常）需要集群中**至少一半的节点投票达成一致**，因此随着更多投票者的加入，投票的成本会显著增加。Observers 的功能与 Followers **完全相同**，但是Observer只会听取投票结果，不参与投票。由于这点，我们可以增加任意数量的Observer，同时不会影响我们集群的性能。Observers 可以失败，或者与集群断开连接，而不会损害 ZooKeeper 服务的可用性。
- client：请求发起者

## zookeeper server 状态

1. LOOKING：当前 Server **不知道 leader 是谁**，正在搜寻
2. LEADING：当前 Server 即为**选举出来的 leader**
3. FOLLOWING：leader 已经选举出来，**当前 Server 与之同步**

## zookeeper leader 选举过程

### 事务 id

事务 id：zxid，zxid 是 64 位数字，leader 分配，全局唯一且递增

### 选举规则

半数以上通过，奇数选举

- 3 台机器，挂一台 2>3/2，可以使用。
- 4 台机器，挂两台 2!>2，不可以使用。

### 选举流程

1. 每个 Server 启动以后都询问其它的 Server 它要投票给谁。
2. 对于其他 server 的询问，server 每次根据自己的状态都回复自己推荐的 leader 的 id 和上一次处理事务的 zxid（zxid 是 64 位数字，leader 分配，全局唯一且递增，系统启动时每个  server 都会推荐自己）
3. 收到所有 Server 回复以后，就计算出 zxid 最大的哪个 Server，并将这个 Server 相关信息设置成下一次要投票的 Server。
4. 计算这过程中获得票数最多的的 sever 为获胜者，如果获胜者的票数超过半数，则改 server 被选为 leader。否则，继续这个过程，直到 leader 被选举出来
5. leader 就会开始等待 server 连接
6. Follower 连接 leader，将最大的 zxid 发送给 leader
7. Leader 根据 follower 的 zxid 确定同步点
8. 完成同步后通知 follower 已经成为 uptodate 状态
9. Follower 收到 uptodate 消息后，又可以重新接受 client 的请求进行服务了

![选举](/images/posts/knowledge/zookeeper/选举.png)

# zookeeper 工作原理

Zookeeper 的核心是原子广播，这个机制保证了各个 server 之间的同步，实现这个机制的协议叫做 Zab 协议。 Zab 协议有两种模式，**它们分别是恢复模式（选主）和广播模式（同步，运行）。**当服务启动或者在领导者崩溃后，Zab 就**进入了恢复模式**，当领导者被选举出来，且大多数 server 的完成了和 leader 的状态同步以后，**恢复模式就结束了**。状态同步保证了 leader 和 server 具有相同的系统状态。

一旦 leader 已经和多数的 follower 进行了状态同步后，他就可以开始**广播消息**了，即进入广播状态。这时候当一个 server 加入 zookeeper 服务中，它会在恢复模式下启动，发现 leader，并和 leader 进行状态同步。待到同步结束，它也参与消息广播。 Zookeeper 服务一直维持在 Broadcast 状态，直到 leader 崩溃了或者 leader 失去了大部分的 followers 支持。

广播模式需要保证提议（proposal）被按顺序处理，因此 zk 采用了递增的事务 id号（zxid）来保证。所有的提议（proposal）都在被提出的时候加上了 zxid。实现中 zxid 是一个 64 位的数字，它高 32 位是 epoch 用来标识 leader 关系是否改变，每次一个 leader 被选出来，它都会有一个新的 epoch。低 32 位是个递增计数。

当 leader 崩溃或者 leader 失去大多数的 follower，这时候 zk 进入恢复模式，恢复模式需要重新选举出一个新的 leader，让所有的 server 都恢复到一个正确的状态。

# zookeeper 节点

## 目录结构

znode 是 zk 特有的数据模型，是 zk 中数据最小单元，znode上 能保存数据，通过挂载子节点形成一个树状的层次结构。根由 / 斜杠开始。

![节点结构](/images/posts/knowledge/zookeeper/节点结构.png)

## 存储数据类型

每个 znode 除了有代表自己的路径名外，还能存储一些数据，zookeeper 只支持**以字节数组的方式去存储数据。**

如果我们要在 znode 里存储一个字符串，很简单，直接调字符串的 getBytes 方法就可以。

如果我们要在 znode 里存储一个 Java 对象呢，那么就得先把这个对象在本地序列化后再存储到 zookeeper。

## 节点类型

- PERSISTENT（持久节点）：所谓持久节点，是指在节点创建后，就一直存在，直到有删除操作来主动清除这个节点——不会因为创建该节点的客户端会话失效而消失。
- EPHEMERAL（临时/挥发节点）：和持久节点不同的是，临时节点的生命周期和客户端会话绑定。也就是说，**如果客户端会话失效，那么这个节点就会自动被清除掉**。注意，这里提到的是会话失效，而非连接断开。另外，**在临时节点下面不能创建子节点**
- PERSISTENT_SEQUENTIAL（持久自增长点）：这类节点的基本特性和上面的节点类型是一致的。额外的特性是，在 ZK 中，每个父节点会为他的第一级子节点维护一份时序，会记录每个子节点创建的先后顺序。基于这个特性，在创建子节点的时候，可以设置这个属性，那么在创建节点过程中，ZK 会自动为给定节点名**加上一个数字后缀**，作为新的节点名。这个数字后缀的范围是整型的最大值。
- EPHEMERAL_SEQUENTIAL（临时自增长节点）

## ACL

Zookeeper 采用 ACL（Access Control Lists）策略来进行权限控制，其定义了如下五种权限：
- CREATE：创建子节点的权限。
- READ：获取节点数据和子节点列表的权限。
- WRITE：更新节点数据的权限。
- DELETE：删除子节点的权限。
- ADMIN：设置节点ACL的权限。

## watch 机制

znode watch 机制也是 zk 的核心功能之一，是配置管理功能的基石。client 端通过注册 watch 对象后，**只要相应的 znode 触发更改，watch 管理器就会向客户端发起回调**，可借此机制实现配置管理的分布式更新和同步队列等场景。

**值得注意的是这里，zookeeper 通知客户端的时候，只会告诉客户端发生了什么事，但并不会把发生改变的数据推送给客户端。**

## znode 版本

版本有：

1. 当前数据节点的版本号
2. 当前数据子节点版本号 
3. acl 权限变更版本号

主要用来通过版本号来实现分布式锁的一些控制，类似于 CAS。适合用多个客户端并发修改一个 znode。

# zookeeper 的保证

- 更新请求顺序进行，来自同一个 client 的更新请求按其发送顺序依次执行
- 数据更新原子性，一次数据更新要么成功，要么失败
- 全局唯一数据视图，client 无论连接到哪个 server，数据视图都是一致的
- 实时性，在一定事件范围内，client 能读到最新数据