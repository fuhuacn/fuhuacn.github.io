---
layout: wiki
title: 自适应异常检测平台
categories: 大数据
description: 自适应异常检测平台
---

目录

* TOC
{:toc}

自适应异常检测平台不出意外将是我在研究生期间的最后一个项目，他的项目初衷是为中国石油塔河硫化塔做异常预警，进而为其他的化工用塔做异常检测预警。该平台的目的是打造简便、易用、易配置的自适应异常检测平台。

# 1. 功能点

## 在线异常检测预警

对配置的应用接收发送来的实时数据，利用历史训练中的正常模型实时预测是否处于异常。

## 离线大数据异常检测算法分析

定时可以完成对平台中配置的应用的历史数据进行离线分析，生成正常数据模型。根据数据的时间变化，更改模型的特征。

## 便捷的算法扩展

有基础标准的算法规范，可以自行便捷的根据规范对算法库扩展。

# 2. 异常检测调研

其实总的来讲都是根据**异常数据与正常数据特点不同**这一思想做的。

+ 基于统计方法：如传统的数据是否符合正态分布。
+ 基于距离方法：如 KNN 距离最远的 K 个点为异常点。
+ 基于密度方法：如 LOF，异常点附近点的密度小。
+ 基于分类方法：对于一个有标签数据集，任何一个点都可以用分类方法确定一个大概率的分类。但如果是异常点，很难确定异常点的分类。
+ 基于集成学习：
  + Baggaing：对数据集特征和数据量随机抽取，应用于相同或不同的弱机器学习算法，最终结合成一个强机器学习算法。
  + Boosting：对数据集经过不同的弱学习器，根绝训练结果不断调整调整算法的权重使得学习器是在不断的增强。
+ 机器学习方法：这里举两个例子。
  + LSTM：利用 LSTM 根据历史数据预测下一条数据。根据预测的准不准确定是否是异常。
  + AutoEncoder：一种压缩算法，检测新的数据是否可以根据历史数据的压缩解压算法压缩复原数据，检测是否是异常。

# 技术架构

## 概括架构

![架构图](/images/projects/anomaly/自适应异常检测架构图.png)

首先搭建了一套 Hadoop 集群，其实在我们实验室框架下是将 Hadoop 集群分成了两套，一套专门的运算（YARN）集群，一套专门的存储集群（HDFS）。

实时和离线处理部分都是用 Spark 作为计算引擎，部署时采用了 Spark on YARN 方式部署。这里关于使用什么计算引擎其实进行了一番讨论，主要有以下三个方向：

+ Storm 作为实时计算引擎，后台使用 MapReduce 做离线计算。
+ Spark Streaming 实时计算，Spark SQL 做为离线计算。
+ Flink 完成实时计算和离线计算。

最终选择了 Spark 原因主要有以下两点：

+ 相比于 Flink 技术更为成熟，有现成的 Spark MLlib 库，比 Flink 的机器学习库内容更多。同时方便集成 Dl4j。缺点是运算效率不如 Flink。
+ 相比于 Storm + MapReduce 框架更加统一，离线计算内存计算速度更快。

其中各组件与 MySQL 连接均采用 Dubbo 完成 RPC 连接。

>Spark 版本：2.2.0  
Kafka 版本：2.1.0  
scala 版本：2.11
Dubbo 版本：2.7.1

## 管理架构模型

![管理架构模型](/images/projects/anomaly/app管理模型.png)

## Spark Streaming 实时预警

项目通过 SpringBoot 启动，方便于 Dubbo 连接。

当第一次启动后，会将 MySQL 中的应用配置加载进内存中，并每个 1 分钟更新一次。

实时预警从 Kafka 实时读取最新的传感器发来的数据。采用的是直连 Kafka 的方式。

当一条新的数据读进实时预警系统后，首先会检查该数据的格式是否和内存中的应用配置信息是否一致，如果一致则开始预警处理。

预警过程中首先检测 HDFS 中是否有离线预警的模型，如果没有模型则跳过预警步骤，如有离线预警模型，贼将实时数据通过离线预警模型，检查数据是否偏离正常模型。如果超过偏离值，发送预警信息给预警组件。

当预警完成后，存储实时的数据至 MySQL 中。

``` java
// markdown 代码块不知道为什么没有 scala 格式
/*
  * 实时预警
  */
stream.map(p => {
  val topic=p.topic()
  val unitKafka = UnitKafka(p.value().asInstanceOf[GenericRecord],appFormatMapBroad.value(topic))
  (unitKafka.attrs, topic)
}).foreachRDD(rdd => {
  if (!rdd.isEmpty()) {
    logger.info("读取到新数据。")
    val df_Or = rdd.toDF("value", "topic")
    for (i <- 0 until apps.size()) {
      val app = apps.get(i)
      val df_this = df_Or.filter($"topic" === app.getTopic).select($"value")
      val df_this_count = df_this.count()
      if (df_this_count > 0) {
        logger.info("应用：" + app + "实时预测方法是" + app.getTrainModel + "，预测条数：" + df_this_count+" 告警阈值："+app.getMse)
        val realCount = df_this.first().getString(0).split(Constants.PATTERN).length
        if (realCount == app.getColumnnumber) {
          try {
            val runningApp = RunningTimeData.readAndWriteMap.get(app.getTopic)
            if(runningApp!=null){
              val countEntity = new CountEntity(df_this.count().toInt,System.currentTimeMillis())
              RunningTimeData.readAndWriteMap.get(app.getTopic).count.add(countEntity)
              RunningTimeData.readAndWriteMap.get(app.getTopic).streamingPredict.predict(df_this, app, spark,kafkaProducerBroad)
            }else{
              logger.warn("应用：" + app + "，预测方法是" + app.getTrainModel + "，暂时无该预测模型。")
            }
          } catch {
            case ex: Exception => {
              logger.error(app + "时时流处理失败!!!")
              ex.printStackTrace()
            }
          }
        } else {
          logger.error(app + "发送数量与接受数量不等！" + " 真实数量：" + realCount + "，设置数量：" + app.getColumnnumber)
        }
      }
    }
  }
})

/*
  * 存储至MySQL
  */
stream.map(p => {
  val topic = p.topic()
  val unitKafka: UnitKafka = UnitKafka(p.value().asInstanceOf[GenericRecord],appFormatMapBroad.value(topic))
  (unitKafka.attrs, Constants.data_no_sign, topic, unitKafka.createdTime)
}).foreachRDD(rdd => {
  if (!rdd.isEmpty()) {
    val df = rdd.toDF("data", "tag", "topic", "time")
    MysqlOperation.writeDF(df, "data", SaveMode.Append)
    logger.info("已保存新数据！")
  }
})
```

## Spark SQL 离线分析

