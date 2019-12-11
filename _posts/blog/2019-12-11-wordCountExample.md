---
layout: post
title: 记录 Spark RDD，Spark dataset 和 Flink 的 WordCount
categories: Blogs
description: 记录 Spark RDD，Spark dataset 和 Flink 的 WordCount
keywords: spark,flink,wordcount
---

记录一下 Spark RDD，Spark dataset 和 Flink 的 WordCount，毕竟这是基础。

## Spark RDD

RDD 的比较简单，用 flatMap 把每行分开，用 Map 拿出对应 key，用 reduceBy 把 key 相同的加起来就好了。

``` java
package com.free4lab.sparktest

import org.apache.spark.{SparkConf, SparkContext}

/**
  * @Author: fuhua
  * @Date: 2019-06-19 14:45
  */
object WordCount {
  def main(args: Array[String]) {
    val conf = new SparkConf()
      .setAppName("WordCount")
      .setMaster("local[2]")
    val sc = new SparkContext(conf)
    val lines = sc.textFile("/Users/fuhua/Desktop/test.txt");
    val words = lines.flatMap { line => line.split(" ")}
    val pairs = words.map {word => (word, 1)}
    val wordCount = pairs.reduceByKey(_ + _)
    // 在 Executor 分布式执行，所以注意不能在 Driver 用个 Map 之类的收集结果。
    wordCount.foreach(wordCount => println(wordCount._1 + " 出现次数为： " + wordCount._2 + " times"))
    // 如果想在 Driver 端使用可以用 collect() 方法转成数组。
    println(wordCount.collect().toSeq)
  }
}
```

结果：

``` text
fuhua 出现次数为： 1 times
ssss 出现次数为： 1 times
lalalal 出现次数为： 1 times
hello 出现次数为： 3 times
hahaha 出现次数为： 2 times
world 出现次数为： 1 times

WrappedArray((ssss,1), (hello,3), (world,1), (fuhua,1), (lalalal,1), (hahaha,2))
```

注意：

+ foreach 方法会把数据集分到 Executor 上执行，具体分配数量由并行度决定。所以如果在 Driver 端设置一个可变 List 想在 foreach 收集是无法得到结果的，因为各个 Executor 上是往复制的 List 的副本添加元素。
+ 使用 collect() 方法可以把 RDD 转化为数组。

## Spark DataSet

要麻烦一些，第一步是要把读出来的词分开，如果是 dataset 可以做 map 用 split 方式打开。也可以使用 spark 的内置 sql function split 打开成数组。第二步是要把数组拆成一行一行的值。需要用到 explode 函数，把数组拆开。然后用 groupBy 汇总，count 计数就可以了。

``` java
package com.free4lab.sparktest

import org.apache.spark.sql.{DataFrame, Dataset, SparkSession}

/**
  * @Author: fuhua
  * @Date: 2019-06-19 14:45
  */
object WordCount {
  def main(args: Array[String]) {
    val spark = SparkSession.builder
      .master("local[2]")
      .appName("example")
      .getOrCreate()
    import spark.implicits._
//    val ds:Dataset[String] = spark.read.text("/Users/fuhua/Desktop/test.txt").as[String]
//    val words = ds.map(v=>v.split(" ")) // 空格分隔转化成数组的 DataSet
    val ds:DataFrame = spark.read.text("/Users/fuhua/Desktop/test.txt")
    import org.apache.spark.sql.functions._
    val words = ds.select(split(ds("value")," ").alias("value")) // 注意这是 dataframe，同样转化为数组，转化后依然命名为 value（默认读出来时就是 value）

    val wordDF = words.select(explode($"value").alias("value")) //explode 函数可以将单列扩展成多行。
	val res = wordDF.groupBy("value").count() // 最后用 groupBy 就可以了
	res.show()//也可以用 collect 转数组
  }
}
```

结果：

``` text
+-------+-----+
|  value|count|
+-------+-----+
|  hello|    3|
|  fuhua|    1|
| hahaha|    2|
|   ssss|    1|
|  world|    1|
|lalalal|    1|
+-------+-----+
```

## Flink

Flink 的批处理就是把文件想象成了有界的流。由于 Flink 用 java 写的而不是 scala，所以这部分代码也换成 java。

``` java
package org.flinkETL;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;
import org.apache.flink.api.java.tuple.Tuple2;//一定用 Flink 的，不要用 scala 的


/**
 * @Author: fuhua
 * @Date: 2019/12/11 10:23 下午
 */
public class WordCount {

    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);//设置并行度，这个并行度与 Spark 类似，只不过 Flink 中是分配到各个 TaskManager（每个 TaskManager 有固定的 slot 最小运行单位的个数） 的 Slot 的个数。
        DataStream<String> dataStream = env.readTextFile("/Users/fuhua/Desktop/test.txt");
        DataStream<Tuple2<String, Integer>> counts = dataStream.flatMap(new FlatMapFunction<String, Tuple2<String,Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                String[] array = s.split(" ");
                for (String w:array){
                    collector.collect(new Tuple2<>(w,1));
                }
            }
        }).keyBy(0).sum(1);
        counts.print();
        env.execute("word count");
    }
}
```

基本步骤都是一致的。只不过可以看到打印结果，是以流的形式显示的，分别打印两个分区的内容：

``` text
2> (world,1)
2> (fuhua,1)
1> (hello,1)
1> (lalalal,1)
1> (hello,2)
1> (hello,3)
```