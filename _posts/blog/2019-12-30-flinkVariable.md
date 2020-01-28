---
layout: post
title: Flink、Spark 代码中的全局变量的分配方式
categories: Blogs
description: Flink 代码中的全局变量的分配方式
keywords: Flink,Spark,全局变量
---

# 前提

这个全局变量必须是 serializable 的，否则也无法传递到各 worker 中。

# Flink

Flink 在代码中的全局变量会自动分发到各个 Worker 中并且赋有初始值。全局变量在各 Worker 上统计**各 Worker 内的操作**。

如下类中：

``` java
package org.flinkETL;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.*;

//做规则判断用的
public class StreamingJobSocket {
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(5);

        List<Integer> list = new LinkedList<>();
        list.add(5);
//        People people = new People(); // people 没有序列化，代码运行会直接报错
//        people.setId(100);

        DataStream<String> socketStockStream = env
                .socketTextStream("localhost", 9999)
                .map(new MapFunction<String, String>() {

                    @Override
                    public String map(String s) throws Exception {
                        System.out.println(list.size());
                        list.add(100);
//                        System.out.println(people.getId());
                        return "null";
                    }
                });

        env.execute("Flink Streaming Java API Skeleton");
    }
}
```

如果 People 没有注释掉会报错：

``` log
Exception in thread "main" org.apache.flink.api.common.InvalidProgramException: null
 is not serializable. The object probably contains or references non serializable fields.
	at org.apache.flink.api.java.ClosureCleaner.clean(ClosureCleaner.java:151)
	at org.apache.flink.api.java.ClosureCleaner.clean(ClosureCleaner.java:126)
	at org.apache.flink.api.java.ClosureCleaner.clean(ClosureCleaner.java:71)
	at org.apache.flink.streaming.api.environment.StreamExecutionEnvironment.clean(StreamExecutionEnvironment.java:1574)
	at org.apache.flink.streaming.api.datastream.DataStream.clean(DataStream.java:185)
	at org.apache.flink.streaming.api.datastream.DataStream.map(DataStream.java:587)
	at org.flinkETL.StreamingJobSocket.main(StreamingJobSocket.java:31)
Caused by: java.io.NotSerializableException: org.flinkETL.bean.People
	at java.base/java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1185)
	at java.base/java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:349)
	at org.apache.flink.util.InstantiationUtil.serializeObject(InstantiationUtil.java:586)
	at org.apache.flink.api.java.ClosureCleaner.clean(ClosureCleaner.java:133)
	... 6 more
```

不看 people 对于这个 list，用 nc -lk 9999 随便输入字符串，在所有 Worker 都会有第一个初始值 5。但之后的 add 方法只会对各自 Worker 内的 list 生效。

输出所以是每 5 个增加一次（其实 5 个打印的日志地点可能都不相同）。

``` text
1
1
1
1
1
2
2
2
2
2
3
```

# Spark

Spark 在代码中的全局变量会自动分发到各个 Executor 中并且赋有初始值。全局变量在各 Executor 上**值不变**。

``` scala
package com.free4lab.sparkml.streaming

import java.util

import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.{SparkConf, SparkContext}
import java.util.LinkedList

object Test {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("socketSparkStreaming").setMaster("local[2]")
    val sc = new SparkContext(conf)
    val ssc = new StreamingContext(sc,Seconds(5))

    val lines = ssc.socketTextStream("localhost",9999,StorageLevel.MEMORY_ONLY)

    val list:LinkedList[Int] = new util.LinkedList[Int]()
    list.add(1)

    lines.foreachRDD(rdd=>{
      rdd.foreach(line=>{
        println(list.size())
        list.add(100)
      })
    })

    ssc.start()
    ssc.awaitTermination()
  }
}
```

同样用 nc -lk 9999 输入：

``` text
1
1
1
1
1
1
1
1
1
1
1
1
1
1
```

# 解决

## Spark 中可以使用广播变量方式

``` scala
val kafkaProducerBroad: Broadcast[KafkaSink[String, GenericRecord]] = {
    val kafkaProducerConfig = KafkaConf.loadKafkaConf("kafka-avro-producer.properties")
    logger.info("kafka已经完成初始化！")
    spark.sparkContext.broadcast(KafkaSink[String, GenericRecord](kafkaProducerConfig))
}

val kafkaSink:KafkaSink[String, GenericRecord] = kafkaProducerBroad.value
```

## Flink 除了广播外还有广播状态方式可选

``` java
MapStateDescriptor<String, String> broadcastStateDesc = new MapStateDescriptor<>( "broadcast-state-desc", String.class, String.class );

BroadcastStream<String> broadcastStream = controlStream.setParallelism(1) .broadcast(broadcastStateDesc);

BroadcastConnectedStream<String, String> connectedStream = sourceStream.connect(broadcastStream);
```

最后就要调用 process() 方法对连接起来的流进行处理了。如果 DataStream 是一个普通的流，
需要定义 BroadcastProcessFunction，反之，如果该 DataStream 是一个 KeyedStream，
就需要定义 KeyedBroadcastProcessFunction。
并且与之前我们常见的 ProcessFunction 不同的是，它们都多了一个专门处理广播数据的方法 
processBroadcastElement()。

``` java
connectedStream.process(new BroadcastProcessFunction<String, String, String>() {
    private static final long serialVersionUID = 1L;

    @Override
    public void processElement(String value, ReadOnlyContext ctx, Collector<String> out) throws Exception {
        ReadOnlyBroadcastState<String, String> state = ctx.getBroadcastState(broadcastStateDesc);
        for (Entry<String, String> entry : state.immutableEntries()) {
            String bKey = entry.getKey();
            String bValue = entry.getValue();
            // 根据广播数据进行原数据流的各种处理
        }
        out.collect(value);
    }

    @Override
    public void processBroadcastElement(String value, Context ctx, Collector<String> out) throws Exception {
        BroadcastState<String, String> state = ctx.getBroadcastState(broadcastStateDesc);
        // 如果需要的话，对广播数据进行转换，最后写入状态
        state.put("some_key", value);
    }
 });
```