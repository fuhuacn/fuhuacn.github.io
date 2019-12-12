---
layout: post
title: Spark Job、Task、Stage关系
categories: Blogs
description: Spark Job、Stage、Task关系
keywords: Spark
---
Spark 中的 Job、Stage、Task 关系之前一直不是很清晰，今天研究了一下，主要参考[文档](https://www.jlpyyf.com/article/22)

这里拿一个简单的 wordcount 作为例子。代码如下：

``` scala
import org.apache.spark.{SparkConf, SparkContext}

/**
  * @Author: fuhua
  * @Date: 2019-06-19 14:45
  */
object WordCount {
  def main(args: Array[String]) {
    val conf = new SparkConf()
      .setAppName("WordCount")
      .setMaster("local[2]");
    val sc = new SparkContext(conf)

    val lines = sc.textFile("/Users/fuhua/Desktop/test.txt");
    val words = lines.flatMap { line => line.split(" ")}
    val pairs = words.map {word => (word, 1)}
    val wordCount = pairs.reduceByKey(_ + _)
    wordCount.foreach(wordCount => println(wordCount._1 + " 出现次数为： " + wordCount._2 + " times"))
    println(wordCount.top(1).toSeq)
    Thread.sleep(1000000000)
  }
}
```

## Job

spark 中的数据都是抽象为 RDD 的，它支持两种类型的算子操作：Transformation 和 Action。

Transformation 算子的代码不会真正被执行。只有当我们的程序里面遇到一个 action 算子的时候，代码才会真正的被执行。

Transformation 算子主要包括：map、mapPartitions、flatMap、filter、union、groupByKey、repartition、cache 等。

Action算子主要包括：reduce、collect、show、count、foreach、saveAsTextFile 等。

当在程序中遇到一个 action 算子的时候，就会提交一个 job，执行前面的一系列操作。因此平时要注意，如果声明了数据需要 cache 或者 persist，但在 action 操作前释放掉的话，该数据实际上并没有被缓存。

通常一个任务会有多个 job，job 之间是按照串行的方式执行的。一个 job 执行完成后，才会起下一个 job。有一段时间曾想让 job 并行执行，但没有找到解决方法。

如在代码中的 foreach 和 top 操作，就有两个 job。（即遇到 action 即生成一个新的 Job）

![Job](/images/posts/knowledge/spark-taskstagejob/WX20191104-124812.png)

## Stage

在 DAGScheduler 中，会将每个 job 划分成多个 stage，它会从触发 action 操作的那个 RDD 开始往前推，首先会为最后一个 RDD 创建一个 stage，然后往前倒推的时候，如果发现对某个 RDD 是宽依赖（执行了 shuffle 操作），那么就会将宽依赖的那个 RDD 创建一个新的 stage。（即在 job 中从后往前倒退，遇到宽依赖新建 stage）

![Stage](/images/posts/knowledge/spark-taskstagejob/stage.png)

>注：窄依赖:
>
>+ 窄依赖：一般是 transformation 操作。父 RDD 和子 RDD partition 之间的关系是一对一的。一个分区只会对应一个分区操作，不会有 shuffle 的产生。父 RDD 的一个分区去到子 RDD 的一个分区中。如：map，flatMap
>+ 宽依赖：一般是 action 操作。父 RDD 与子 RDD partition 之间的关系是一对多的。一个分区可能会被之后的多个分区所用到，会有 shuffle 的产生。父 RDD 的一个分区去到子 RDD 的不同分区里面。如：reduceByKey
> 注：join 操作即可能是宽依赖也可能是窄依赖，当要对 RDD 进行 join 操作时，如果 RDD 进行过重分区则为窄依赖，否则为宽依赖。

## Task

task 是 stage 下的一个任务执行单元，一般来说，一个 rdd 有多少个 partition，就会有多少个 task，因为每一个 task 只是处理一个 partition 上的数据。像我这次本地跑的，因为设定的是 local[2] 即两个 CPU 核心，所以是两个分区划分为两个 task。（即 stage 的执行单元）

![Task](/images/posts/knowledge/spark-taskstagejob/task.png)

## 运行结果

![Task](/images/posts/knowledge/spark-taskstagejob/jieguo1.png)
![Task](/images/posts/knowledge/spark-taskstagejob/jieguo2.png)

同时也可以在运行日志中看到 job、stage、task 的日志过程：

先通过 foreach 拿到一个 job，然后将其划分为两个 stage（ResultStage的foreach和ShuffleMapStage的map），之后每个 stage 对应提交两个 task（分区数量为 2）

``` log
19/11/04 13:47:06 INFO SparkContext: Starting job: foreach at WordCount.scala:20
19/11/04 13:47:06 INFO DAGScheduler: Registering RDD 3 (map at WordCount.scala:18)
19/11/04 13:47:06 INFO DAGScheduler: Got job 0 (foreach at WordCount.scala:20) with 2 output partitions
19/11/04 13:47:06 INFO DAGScheduler: Final stage: ResultStage 1 (foreach at WordCount.scala:20)
19/11/04 13:47:06 INFO DAGScheduler: Parents of final stage: List(ShuffleMapStage 0)
19/11/04 13:47:06 INFO DAGScheduler: Missing parents: List(ShuffleMapStage 0)
19/11/04 13:47:06 INFO DAGScheduler: Submitting ShuffleMapStage 0 (MapPartitionsRDD[3] at map at WordCount.scala:18), which has no missing parents
19/11/04 13:47:06 INFO MemoryStore: Block broadcast_1 stored as values in memory (estimated size 4.8 KB, free 2004.5 MB)
19/11/04 13:47:06 INFO MemoryStore: Block broadcast_1_piece0 stored as bytes in memory (estimated size 2.8 KB, free 2004.5 MB)
19/11/04 13:47:06 INFO BlockManagerInfo: Added broadcast_1_piece0 in memory on 10.28.197.159:58402 (size: 2.8 KB, free: 2004.6 MB)
19/11/04 13:47:06 INFO SparkContext: Created broadcast 1 from broadcast at DAGScheduler.scala:996
19/11/04 13:47:06 INFO DAGScheduler: Submitting 2 missing tasks from ShuffleMapStage 0 (MapPartitionsRDD[3] at map at WordCount.scala:18)
19/11/04 13:47:06 INFO TaskSchedulerImpl: Adding task set 0.0 with 2 tasks
19/11/04 13:47:06 INFO TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0, localhost, executor driver, partition 0, PROCESS_LOCAL, 5971 bytes)
19/11/04 13:47:06 INFO TaskSetManager: Starting task 1.0 in stage 0.0 (TID 1, localhost, executor driver, partition 1, PROCESS_LOCAL, 5971 bytes)
19/11/04 13:47:06 INFO Executor: Running task 0.0 in stage 0.0 (TID 0)
19/11/04 13:47:06 INFO Executor: Running task 1.0 in stage 0.0 (TID 1)
19/11/04 13:47:06 INFO HadoopRDD: Input split: file:/Users/fuhua/Desktop/test.txt:0+5
19/11/04 13:47:06 INFO HadoopRDD: Input split: file:/Users/fuhua/Desktop/test.txt:5+5
19/11/04 13:47:07 INFO Executor: Finished task 1.0 in stage 0.0 (TID 1). 1483 bytes result sent to driver
19/11/04 13:47:07 INFO Executor: Finished task 0.0 in stage 0.0 (TID 0). 1746 bytes result sent to driver
19/11/04 13:47:07 INFO TaskSetManager: Finished task 1.0 in stage 0.0 (TID 1) in 117 ms on localhost (executor driver) (1/2)
19/11/04 13:47:07 INFO TaskSetManager: Finished task 0.0 in stage 0.0 (TID 0) in 143 ms on localhost (executor driver) (2/2)
19/11/04 13:47:07 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool
19/11/04 13:47:07 INFO DAGScheduler: ShuffleMapStage 0 (map at WordCount.scala:18) finished in 0.168 s
19/11/04 13:47:07 INFO DAGScheduler: looking for newly runnable stages
19/11/04 13:47:07 INFO DAGScheduler: running: Set()
19/11/04 13:47:07 INFO DAGScheduler: waiting: Set(ResultStage 1)
19/11/04 13:47:07 INFO DAGScheduler: failed: Set()
19/11/04 13:47:07 INFO DAGScheduler: Submitting ResultStage 1 (ShuffledRDD[4] at reduceByKey at WordCount.scala:19), which has no missing parents
19/11/04 13:47:07 INFO MemoryStore: Block broadcast_2 stored as values in memory (estimated size 3.1 KB, free 2004.5 MB)
19/11/04 13:47:07 INFO MemoryStore: Block broadcast_2_piece0 stored as bytes in memory (estimated size 1968.0 B, free 2004.4 MB)
19/11/04 13:47:07 INFO BlockManagerInfo: Added broadcast_2_piece0 in memory on 10.28.197.159:58402 (size: 1968.0 B, free: 2004.6 MB)
19/11/04 13:47:07 INFO SparkContext: Created broadcast 2 from broadcast at DAGScheduler.scala:996
19/11/04 13:47:07 INFO DAGScheduler: Submitting 2 missing tasks from ResultStage 1 (ShuffledRDD[4] at reduceByKey at WordCount.scala:19)
19/11/04 13:47:07 INFO TaskSchedulerImpl: Adding task set 1.0 with 2 tasks
19/11/04 13:47:07 INFO TaskSetManager: Starting task 0.0 in stage 1.0 (TID 2, localhost, executor driver, partition 0, ANY, 5750 bytes)
19/11/04 13:47:07 INFO TaskSetManager: Starting task 1.0 in stage 1.0 (TID 3, localhost, executor driver, partition 1, ANY, 5750 bytes)
19/11/04 13:47:07 INFO Executor: Running task 1.0 in stage 1.0 (TID 3)
19/11/04 13:47:07 INFO Executor: Running task 0.0 in stage 1.0 (TID 2)
19/11/04 13:47:07 INFO ShuffleBlockFetcherIterator: Getting 1 non-empty blocks out of 2 blocks
19/11/04 13:47:07 INFO ShuffleBlockFetcherIterator: Getting 1 non-empty blocks out of 2 blocks
19/11/04 13:47:07 INFO ShuffleBlockFetcherIterator: Started 0 remote fetches in 5 ms
19/11/04 13:47:07 INFO ShuffleBlockFetcherIterator: Started 0 remote fetches in 5 ms
```
