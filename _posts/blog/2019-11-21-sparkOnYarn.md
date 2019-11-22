---
layout: post
title: Spark on Yarn 遇到的种种问题
categories: Blogs
description: Spark on Yarn 遇到的种种问题
keywords: spark,yarn
---

最近实验室由于物理机性能快满了，决定将各种 Hadoop 集群进行汇总，所以就将原有的人手一套的 Hadoop 集群最终设计只保留两个集群，一个是用于专门存储的集群，主要保留他的存储资源。另一套就是一套专门的运算集群。所以也借此机会，想把实验室内所有的运算平台进行一下汇总，不管是 MapReduce、Yarn 或是 Flink 集群都希望能统一的跑在一套 Yarn 上。

由于之前做过 Spark on yarn 的项目，本以为没有太大工作量，但没想到还是搞了一下午。

我这边的要迁移的项目是：**项目：自适应异常检测**。

## 问题一：程序中 maven 引入的 jar 包如何处理

与很多小项目不同，自适应异常检测引入的 jar 包比较多，首先是 dl4j 的机器学习包，再有 spring、dubbo 等等框架的包也着实不小。最后加着加着如果统一打在一个包中提交大小居然到了 500MB+。对于我们实验室的 1MB/s 的上传速度对这么大的包做修改实属不容易。那这里最终选择的解决方式其实与 standalone 模式下类似，通过引入 spark.executor.extraClassPath 和 spark.driver.extraClassPath 将程序单独打出的 lib 目录引入。

## 问题二：Spark 的版本不一致

由于是新装的集群，没想到出现了 Spark 版本不一致导致的错误，当时的报错如下：

``` log
19/11/21 16:27:45 ERROR ApplicationMaster: Uncaught exception: 
java.lang.IllegalArgumentException: Invalid ContainerId: container_e03_1574321597694_0007_01_000001
	at org.apache.hadoop.yarn.util.ConverterUtils.toContainerId(ConverterUtils.java:182)
	at org.apache.spark.deploy.yarn.YarnSparkHadoopUtil.getContainerId(YarnSparkHadoopUtil.scala:104)
	at org.apache.spark.deploy.yarn.YarnRMClient.getAttemptId(YarnRMClient.scala:95)
	at org.apache.spark.deploy.yarn.ApplicationMaster.run(ApplicationMaster.scala:202)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anonfun$main$1.apply$mcV$sp(ApplicationMaster.scala:768)
	at org.apache.spark.deploy.SparkHadoopUtil$$anon$1.run(SparkHadoopUtil.scala:67)
	at org.apache.spark.deploy.SparkHadoopUtil$$anon$1.run(SparkHadoopUtil.scala:66)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1491)
	at org.apache.spark.deploy.SparkHadoopUtil.runAsSparkUser(SparkHadoopUtil.scala:66)
	at org.apache.spark.deploy.yarn.ApplicationMaster$.main(ApplicationMaster.scala:766)
	at org.apache.spark.deploy.yarn.ApplicationMaster.main(ApplicationMaster.scala)
Caused by: java.lang.NumberFormatException: For input string: "e03"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Long.parseLong(Long.java:589)
	at java.lang.Long.parseLong(Long.java:631)
	at org.apache.hadoop.yarn.util.ConverterUtils.toApplicationAttemptId(ConverterUtils.java:137)
	at org.apache.hadoop.yarn.util.ConverterUtils.toContainerId(ConverterUtils.java:177)
	... 12 more
19/11/21 16:27:45 INFO ApplicationMaster: Final app status: FAILED, exitCode: 10, (reason: Uncaught exception: java.lang.IllegalArgumentException: Invalid ContainerId: container_e03_1574321597694_0007_01_000001)
19/11/21 16:27:45 INFO ShutdownHookManager: Shutdown hook called
```

大致问题就是我之前使用的 spark 2.1 不支持带字母的 container id，而获得的 am 的 container id 有字母。将 spark 升级到对应 ambari 安装的 2.2（虽然其实仍然比较老了）就好了。

## 问题三：Hadoop 包版本冲突

解决完 Spark 的版本问题，紧接着又报错了，日志如下：

``` log
Exception in thread "main" java.lang.IllegalAccessError: tried to access method org.apache.hadoop.yarn.client.ConfiguredRMFailoverProxyProvider.getProxyInternal()Ljava/lang/Object; from class org.apache.hadoop.yarn.client.RequestHedgingRMFailoverProxyProvider
	at org.apache.hadoop.yarn.client.RequestHedgingRMFailoverProxyProvider.init(RequestHedgingRMFailoverProxyProvider.java:75)
	at org.apache.hadoop.yarn.client.RMProxy.createRMFailoverProxyProvider(RMProxy.java:163)
	at org.apache.hadoop.yarn.client.RMProxy.createRMProxy(RMProxy.java:93)
	at org.apache.hadoop.yarn.client.ClientRMProxy.createRMProxy(ClientRMProxy.java:72)
	at org.apache.hadoop.yarn.client.api.impl.AMRMClientImpl.serviceStart(AMRMClientImpl.java:186)
	at org.apache.hadoop.service.AbstractService.start(AbstractService.java:193)
	at org.apache.spark.deploy.yarn.YarnRMClient.register(YarnRMClient.scala:65)
	at org.apache.spark.deploy.yarn.ApplicationMaster.registerAM(ApplicationMaster.scala:387)
	at org.apache.spark.deploy.yarn.ApplicationMaster.runDriver(ApplicationMaster.scala:430)
	at org.apache.spark.deploy.yarn.ApplicationMaster.run(ApplicationMaster.scala:282)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anonfun$main$1.apply$mcV$sp(ApplicationMaster.scala:768)
	at org.apache.spark.deploy.SparkHadoopUtil$$anon$2.run(SparkHadoopUtil.scala:67)
	at org.apache.spark.deploy.SparkHadoopUtil$$anon$2.run(SparkHadoopUtil.scala:66)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1692)
	at org.apache.spark.deploy.SparkHadoopUtil.runAsSparkUser(SparkHadoopUtil.scala:66)
	at org.apache.spark.deploy.yarn.ApplicationMaster$.main(ApplicationMaster.scala:766)
	at org.apache.spark.deploy.yarn.ApplicationMaster.main(ApplicationMaster.scala)
```

这个问题导致的结果还有就是 yarn 无法分配 node 执行。造成这个问题的原因是 hadoop yarn 版本冲突。我简单暴力的把所有引入的 hadoop 包都移出去，只使用 ambari 自己的 hadoop 包解决问题。移出的内容有：

![移出的 jar 包](/images/posts/blog/sparkonyarn/WX20191121-230011.png)

移出后还要增加 MapReduce 的包到 Yarn 的 classpath（没想通为什么）。将 /usr/hdp/current/hadoop-mapreduce-client/*,/usr/hdp/current/hadoop-mapreduce-client/lib/* 添加到 yarn.application.classpath 即可。

## 问题四：使用 hdfs 用户

这个问题解决后，报了一个很神奇的问题：

``` log
Exception in thread "streaming-job-executor-0" java.lang.Error: java.lang.InterruptedException
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1155)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:998)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)
	at scala.concurrent.impl.Promise$DefaultPromise.tryAwait(Promise.scala:202)
	at scala.concurrent.impl.Promise$DefaultPromise.ready(Promise.scala:218)
	at scala.concurrent.impl.Promise$DefaultPromise.ready(Promise.scala:153)
	at org.apache.spark.util.ThreadUtils$.awaitReady(ThreadUtils.scala:222)
	at org.apache.spark.scheduler.DAGScheduler.runJob(DAGScheduler.scala:621)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2022)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2043)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:2062)
	at org.apache.spark.rdd.RDD$$anonfun$take$1.apply(RDD.scala:1354)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:112)
	at org.apache.spark.rdd.RDD.withScope(RDD.scala:362)
	at org.apache.spark.rdd.RDD.take(RDD.scala:1327)
	at org.apache.spark.rdd.RDD$$anonfun$isEmpty$1.apply$mcZ$sp(RDD.scala:1462)
	at org.apache.spark.rdd.RDD$$anonfun$isEmpty$1.apply(RDD.scala:1462)
	at org.apache.spark.rdd.RDD$$anonfun$isEmpty$1.apply(RDD.scala:1462)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:112)
	at org.apache.spark.rdd.RDD.withScope(RDD.scala:362)
	at org.apache.spark.rdd.RDD.isEmpty(RDD.scala:1461)
	at com.free4lab.sparkml.streaming.Streaming$$anonfun$startStreaming$5.apply(Streaming.scala:181)
	at com.free4lab.sparkml.streaming.Streaming$$anonfun$startStreaming$5.apply(Streaming.scala:180)
	at org.apache.spark.streaming.dstream.DStream$$anonfun$foreachRDD$1$$anonfun$apply$mcV$sp$3.apply(DStream.scala:628)
	at org.apache.spark.streaming.dstream.DStream$$anonfun$foreachRDD$1$$anonfun$apply$mcV$sp$3.apply(DStream.scala:628)
	at org.apache.spark.streaming.dstream.ForEachDStream$$anonfun$1$$anonfun$apply$mcV$sp$1.apply$mcV$sp(ForEachDStream.scala:51)
	at org.apache.spark.streaming.dstream.ForEachDStream$$anonfun$1$$anonfun$apply$mcV$sp$1.apply(ForEachDStream.scala:51)
	at org.apache.spark.streaming.dstream.ForEachDStream$$anonfun$1$$anonfun$apply$mcV$sp$1.apply(ForEachDStream.scala:51)
	at org.apache.spark.streaming.dstream.DStream.createRDDWithLocalProperties(DStream.scala:416)
	at org.apache.spark.streaming.dstream.ForEachDStream$$anonfun$1.apply$mcV$sp(ForEachDStream.scala:50)
	at org.apache.spark.streaming.dstream.ForEachDStream$$anonfun$1.apply(ForEachDStream.scala:50)
	at org.apache.spark.streaming.dstream.ForEachDStream$$anonfun$1.apply(ForEachDStream.scala:50)
	at scala.util.Try$.apply(Try.scala:192)
	at org.apache.spark.streaming.scheduler.Job.run(Job.scala:39)
	at org.apache.spark.streaming.scheduler.JobScheduler$JobHandler$$anonfun$run$1.apply$mcV$sp(JobScheduler.scala:257)
	at org.apache.spark.streaming.scheduler.JobScheduler$JobHandler$$anonfun$run$1.apply(JobScheduler.scala:257)
	at org.apache.spark.streaming.scheduler.JobScheduler$JobHandler$$anonfun$run$1.apply(JobScheduler.scala:257)
	at scala.util.DynamicVariable.withValue(DynamicVariable.scala:58)
	at org.apache.spark.streaming.scheduler.JobScheduler$JobHandler.run(JobScheduler.scala:256)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	... 2 more
```

一直就报线程被打断了，一般来讲线程被打断都不是报这段代码的问题，而是别的地方出错误所以叫所有的代码停止。但是就是找不到出错点在哪儿，也完全没有日志。专门也去找了 spark2-history 内的记录也找不到。最后走投无路想到会不会是一直用 root 用户提交的原因（用 ambari 安装 root 用户没有什么权限，专门还把 hdfs 中创了 /user/root 文件夹并赋给了 root 用户权限）。改用 hdfs 用户提交后不再出现此问题。

## 问题五：忘了 spark streaming 的 awaitTermination()

这个说来比较有意思，上面问题解决后，一直是跑完第一次离线运算后就全部结束了，这个也比较灵异，按理说离线运算是个死循环线程，但基本上跑完就 finish。

![跑完就结束](/images/posts/blog/sparkonyarn/jieshu.png)

后来发现之前把 ssc.awaitTermination() 注释掉了。

## 运行截图

提交命令很简单：

``` shell
spark-submit --class com.free4lab.sparkonline.run.ApplicationRun  --master yarn --deploy-mode cluster --name ml --executor-memory 1g --executor-cores 1  --conf "spark.driver.extraClassPath=/hadoop/spark-ml/lib/*" --conf "spark.executor.extraClassPath=/hadoop/spark-ml/lib/*"  ml-online-1.0-SNAPSHOT.jar
```

附张正常运行截图，回来在重新配一下 spark 的 log4j 日志等级就 ok 了。

![运行](/images/posts/blog/sparkonyarn/zhencghang.png)
