---
layout: post
title: Spark分区原理
categories: Blogs
description: 学习 Spark 的分区原理
keywords: Spark,分区
---

## 前提知识

- 每个节点可以起一个或多个 Executor。
- 每个 Executor 由若干 core 组成，每个 Executor 的每个 core 一次只能执行一个 Task。
- 每个 Task 执行的结果就是生成了目标 RDD 的一个 partiton。

## Spark分区的概念

每个 RDD/Dataframe 被分成多个片，每个片被称作一个分区，每个分区的数值都在一个任务中进行，任务的个数也会由分区数决定。在我们对一个 RDD/Dataframe 时，其实是对每个分区上的数据进行操作。
![不同分区可能在不同 worker 上](/images/posts/knowledge/spark-partition/spark-partition.png)
一个生动的例子： 比如的 RDD 有 100 个分区，那么计算的时候就会生成 100 个 task，你的资源配置为 10 个计算节点，每个两 2 个核，同一时刻可以并行的 task 数目为 20，计算这个 RDD 就需要 5 个轮次。

如果计算资源不变，你有 101 个 task 的话，就需要 6 个轮次，在最后一轮中，只有一个 task 在执行，其余核都在空转。

如果资源不变，你的 RDD 只有 2 个分区，那么同一时刻只有 2 个 task 运行，其余 18 个核空转，造成资源浪费。这就是在spark调优中，增大 RDD 分区数目，增大任务并行度的做法。

## 如何设置合理的分区数

1. 默认分区数
    - 本地模式：默认为本地机器的 CPU 数目，若设置了 local[N]，则默认为 N。
    - Standalone 或 YARN：默认取集群中所有核心数目的总和，或者 2，取二者的较大值。对于 parallelize 来说，没有在方法中的指定分区数，则默认为 spark.default.parallelism，对于 textFile 来说，没有在方法中的指定分区数，则默认为 min(defaultParallelism,2)，而 defaultParallelism 对应的就是 spark.default.parallelism。如果是从 hdfs 上面读取文件，其分区数为文件分片数（128MB /片）
2. 分区设置多少合适
    - 官方推荐的并行度是 executor * cpu core 的 2-3 倍。

## 窄依赖和宽依赖

- 窄依赖：每个父 RDD 的分区都至多被一个子 RDD 使用，比如 map 操作就是典型的窄依赖。
- 宽依赖：多个子 RDD 的分区依赖一个父 RDD 的分区。比如 groupByKey 都属于宽依赖。
![宽依赖和窄依赖](/images/posts/knowledge/spark-partition/wideandnarrow1.png)
![宽依赖和窄依赖 2](/images/posts/knowledge/spark-partition/wideandnarrow2.png)
- 宽依赖的划分器：之前提到的 join 操作，如果是协同划分的话，两个父 RDD 之间，父 RDD 与子 RDD 之间能形成一致的分区安排。即同一个 Key 保证被映射到同一个分区，这样就是窄依赖。如果不是协同划分，就会形成宽依赖。Spark 提供两种划分器，HashPartitioner (哈希分区划分器)，(RangePartitioner) 范围分区划分器. 需要注意的是分区划分器只存在于 PairRDD 中，普通非（K,V）类型的 Partitioner 为 None。
![宽依赖和窄依赖](/images/posts/knowledge/spark-partition/20190108091331983.png)
5 表示 groupByKey会有 5 个分区，以HashPartitioner划分为 5 个分区
