---
layout: post
title: 记 Spark Streaming 消费 Kafka 多 topic 遇到的问题
categories: Blogs
description: 记 Spark Streaming 消费 Kafka 多 topic 遇到的问题
keywords: spark
---

## 问题背景

最近在改[自适应异常检测](http://www.fuhuacn.top/projects/5anomalyDetction/)的代码，之前写的是一个 DStream 同时接受多个 topic，并对 RDD 遍历 topic 处理。如下所示：

``` java
stream.map(p => {
	val topic=p.topic()
	val unitKafka = UnitKafka(p.value().asInstanceOf[GenericRecord],appFormatMapBroad.value(topic))
	(unitKafka.attrs, topic)
}).foreachRDD(rdd => {
	if (!rdd.isEmpty()) {
	val df_Or = rdd.toDF("value", "topic")
	df_Or.show()
	for (i <- 0 until apps.size()) {
		val app = apps.get(i)
		val df_this = df_Or.filter($"topic" === app.getTopic).select($"value")
		...	
	}
})
```

这样做由于在我的项目中，涉及一些需要 window 窗口的特定 topic，并不好做，所以想改成如下方式，创建多个 DStream，代码如下：

``` java
stream.map(p => {
	val unitKafka = UnitKafka(p.value().asInstanceOf[GenericRecord], appFormatMapBroad.value(topics(i)))
	unitKafka.attrs
	}).foreachRDD(rdd => {
	if (!rdd.isEmpty()) {
		logger.info(s"${topics(i)}收到新数据！")
		val df_this = rdd.toDF("value")
		val app = apps.get(i)
		...
	}
})
```

结果遇上了不能接受 Kafka 消息的情况。

## 问题描述

在本地和 Yarn 上可以正常跑，但是一到 Spark 集群上时，就接受不到 Kafka 的消息，排查了很久也没查出什么原因，后来没办法觉得会不会是性能的原因，提升了核心数，如下：

``` text
spark-submit --class com.free4lab.sparkonline.run.ApplicationRun  --master spark://ambari-namenode.com:7077 --deploy-mode cluster --executor-memory 1G --total-executor-cores 5 --executor-cores 1 --driver-cores 3 --driver-memory 3G  --conf "spark.driver.extraClassPath=/opt/spark-ml/lib/*" --conf "spark.executor.extraClassPath=/opt/spark-ml/lib/*" hdfs://ambari-namenode.com:8020/sintest/ml-online-1.0-SNAPSHOT.jar
```

发现可以接受消息了，但是当 topic 数量（之前是四个）再进一步增长时，又不可以了，便去查阅相关资料。

## 原因

在 Spark Streaming 官网中有这么一段话：

> Input DStreams are DStreams representing the stream of input data received from streaming sources. In the quick example, lines was an input DStream as it represented the stream of data received from the netcat server. Every input DStream (except file stream, discussed later in this section) is associated with a Receiver (Scala doc, Java doc) object which receives the data from a source and stores it in Spark’s memory for processing.  
>*这里的 Receiver 不是指旧的 Spark Streaming Kafka Receiver。*

可以看到每一个 DStream 都被关联了一个 Receiver。在接下来还有这么一段话：

> + When running a Spark Streaming program locally, do not use “local” or “local[1]” as the master URL. Either of these means that only one thread will be used for running tasks locally. If you are using an input DStream based on a receiver (e.g. sockets, Kafka, Flume, etc.), then the single thread will be used to run the receiver, leaving no thread for processing the received data. Hence, when running locally, always use “local[n]” as the master URL, where n > number of receivers to run (see Spark Properties for information on how to set the master).
> + Extending the logic to running on a cluster, the number of cores allocated to the Spark Streaming application must be more than the number of receivers. Otherwise the system will receive data, but not be able to process it.

也就是 Receiver 的个数必须小于核心数（本地是线程数）。

所以当我是 4 个 topic 但之前核心数一直是 1，自然有可能跑不了了。但也证明这句话并不是当不满足时一定会出问题。

## 另一个小问题

其实在测试中还出现了个小问题，最开始时发现总是消费一个 topic，不消费其他 topic。后来发现是 batch time 设定的太小了（居然只设定了 2 秒）。这样处理时间追不上 batch time 导致造成了积压（我们有一个很短频率持续在跑的天气探测组建），看不到消费别的 topic 了。

## 截图

最后附一个 4040 的 Streaming 检测截图：

![截图](/images/posts/blog/sparktopics/WX20191203-225634.png)