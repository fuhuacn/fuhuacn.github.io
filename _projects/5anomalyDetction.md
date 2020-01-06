---
layout: wiki
title: 自适应异常检测平台
categories: 大数据
description: 自适应异常检测平台
---

目录

* TOC
{:toc}

自适应异常检测平台不出意外将是我在研究生期间的最后一个项目，他的项目初衷是为中石化塔河硫化塔做异常预警，进而为其他的化工用塔做异常检测预警。该平台的目的是打造简便、易用、易配置的自适应异常检测平台。

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

# 3. 技术架构

## 概括架构

![架构图](/images/projects/anomaly/自适应异常检测.png)

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

## 训练方式

+ 基准预警：用于比较稳定的数据类型，仅训练一次之后不再离线训练，对于所有的实时异常检测都使用同样的训练模型。
+ 累计预警：训练全部的已有数据，随着数据的增多训练量一直增大。
+ 窗口预警：仅训练最近的配置数量的数据。这样当数据超过配置数量数据时，仅训练最新一段时间的数据作为模型。用于数据形态可能会变化的场景。

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

为了方便管理，与实时分析在同一项目中，均适用 SpringBoot 管理，这样也能读取到内存中的应用配置信息。

在读取应用时先要注意是否在 HDFS 中有离线训练数据，如果没有数据不进行训练。

离线分析组件会定时扫描配置信息，并检查有无到达重新训练时间的应用。离线训练中传统模型使用 Spark MLlib 完成训练，对于机器学习的部分，采用 [Deeplearning4j](http://deeplearning4j.org) 完成。

逻辑结构代码如下：

``` java
for (i <- 0 until apps.size()) {
  val app = apps.get(i)
  if (app.getNexttrain==null||app.getNexttrain.compareTo(Timestamp.valueOf(LocalDateTime.now())) < 0) {
    logger.info(app + " 到达离线计算运行时间，开始离线计算")
    var mse:Double = -1
    try {
      val datasetReadCsv = spark.read.format("csv").option("header", "false").load(Constants.FILE_PATH + app.getId + "data.csv")
      val realCount = datasetReadCsv.columns.length
      if (realCount == app.getColumnnumber) {
        val runningApp = RunningTimeData.readAndWriteMap.get(app.getTopic)
        if(runningApp!=null){
          mse = RunningTimeData.readAndWriteMap.get(app.getTopic).batchPredict.predict(datasetReadCsv, app, spark)
        }else{
          logger.warn("应用：" + app + "，预测方法是" + app.getTrainModel + "，暂时无该离线预测模型。")
        }
        var next = Timestamp.valueOf(LocalDateTime.now().plusMinutes(app.getPeriod.longValue()))
        if(app.getLasttrain!=null){
          next = Timestamp.valueOf(app.getLasttrain.toLocalDateTime.plusMinutes(app.getPeriod.longValue()))
        }
        app.setMse(mse)
        app.setNexttrain(next)
        app.setLasttrain(Timestamp.valueOf(LocalDateTime.now()))
        appOperate.save(app)
      }else{
        logger.error(app+"在离线训练中，列数量与设定数量不符，真实数量："+realCount)
      }
    } catch {
      case e:AnalysisException => {
        logger.warn(app + " 没有文件！路径为："+Constants.FILE_PATH + app.getId + "data.csv")
      }
      case e: Exception => {
        e.printStackTrace()
        logger.error(app + " 离线计算出错！")
      }
    }
  }
}
logger.info("此次全部训练完成，app数量为："+apps.size())
```

注意：实时在线分析和离线分析给出的架构代码都运行在 Driver 端。

## 启动类管理

启动类利用了 CommandLineRunner，这样当 SpringBoot 启动后，自动开始运行 MySQL 配置加载，离线分析和在线分析部分代码。

``` java
@Controller
public class Run implements CommandLineRunner {

    @Autowired
    Streaming streaming;
    @Autowired
    Batching batching;
    @Autowired
    AppUpdate appUpdate;

    @Override
    public void run(String... args) {
        Runnable r = () -> {
            batching.startBatching();
        };
        ScheduledExecutorService service=Executors.newScheduledThreadPool(2);
        service.scheduleAtFixedRate(r,0,10,TimeUnit.MINUTES);
        appUpdate.setStreaming(streaming);
        service.scheduleAtFixedRate(appUpdate,0,1,TimeUnit.MINUTES);
        streaming.startStreaming();
    }
}
```

## 线程并发部分

由于离线分析和在线实时分析都需要写入/读取 HDFS 中的训练的模型，所以如果离线分析写入时，实时分析直接读取模型可能造成程序崩溃。

在程序中为每个程序在内存中设计了 RunningApp 类，在类中包含了读取和加载模型的方法，并上锁以保证并发时不冲突。

``` java
public class RunningApp implements Serializable {
    public Predict streamingPredict;
    public com.free4lab.sparkml.batch.predict.Predict batchPredict;
    private FileMethods fileMethods;
    public LinkedList<CountEntity> count;

    public RunningApp(Predict streamingPredict, com.free4lab.sparkml.batch.predict.Predict batchPredict, FileMethods fileMethods) {
        this.streamingPredict = streamingPredict;
        this.batchPredict = batchPredict;
        this.fileMethods = fileMethods;
        this.count = new LinkedList<>();
    }

    public synchronized Object loadModel(App app){
        return fileMethods.loadModel(app);
    }

    public synchronized void saveModel(App app, MultiLayerNetwork m){
        fileMethods.saveModel(app,m);
    }

    public synchronized void saveModel(App app, KMeansModel m){
        fileMethods.saveModel(app,m);
    }
}
```

## 人工确认预警结果

由于预警可能存在错误，例如：

+ 正常数据标为异常；
+ 异常数据没有完成预警。

需要认为给系统干预，以增强系统的准确性。

对于标注后正常/正常的数据，存储进 HDFS 等待离线分析模块再次训练。

对于标注后异常/异常数据，留为记录。

## 告警模块

接收 Kafka 中实时训练平台的告警信息，读取应用的告警用户发送邮件。注意这里需要可以配置用户接受的告警频率，因为当出现告警的情况时，基本是不停的发送预警信息，可能发送邮件会过于频繁。

## 添加预测方法

扩展以下两个类就可以了。


实时处理部分：

``` java
trait Predict {
  protected val logger = LoggerFactory.getLogger(classOf[Predict])

  def predict(df: DataFrame, app: App, spark: SparkSession, kafkaProducerBroad: Broadcast[KafkaSink[String, GenericRecord]]): Unit

  def generateNoTrainSet(app: App): mutable.Set[Int] = {
    CommonMethods.generateNoTrainSet(app)
  }

  def checkAlarm(errorData: Array[String], errorValue: Double, app: App, kafkaProducerBroad: Broadcast[KafkaSink[String, GenericRecord]]) = {
    if (errorValue > app.getMse * app.getAlarmtimes) {
      logger.error(app + "出现告警！训练规则是：" + app.getTrainModel + errorData.mkString(Constants.PATTERN))
      val message = new GenericData.Record(new Schema.Parser().parse(asvc))
      message.put("topic", Constants.ALARM_TOPIC)
      message.put("createdTime", System.currentTimeMillis)
      val map = new util.HashMap[String, String]
      map.put("index", errorData(app.getOrderindex))
      map.put("time", System.currentTimeMillis.toString)
      map.put("errorValue", errorValue + "")
      map.put("appName", app.getAppname)
      map.put("errorData", errorData.mkString(","))
      message.put("props", map)
      kafkaProducerBroad.value.send(Constants.ALARM_TOPIC, message)
    }
  }
}
```

离线分析部分：

``` java
trait Predict {
  protected val logger = LoggerFactory.getLogger(classOf[Predict])

  // 返回mse
  def predict(df: DataFrame, app: App, spark: SparkSession):Double = {
    try{
      logger.info(app+" 开始训练！")
      var datasetOr:DataFrame = null
      app.getTrainmethod match {
        case Constants.BASE => datasetOr = dealBase(df,app,spark)
        case Constants.GROWTH => datasetOr= dealGrowth(df, app,spark)
        case Constants.WINDOWS => datasetOr = dealWindows(df,app,spark)
        case _ => {
          logger.warn("应用：" + app + "，训练模式错误" + app.getTrainmethod)
          return -1
        }
      }
      datasetOr.show()
      val dataSize = datasetOr.count()
      if(dataSize<app.getMintrainnumber){
        logger.error(app+"训练数量不足！dataSize:"+dataSize+",最小数量："+app.getMintrainnumber)
        return -1
      }
      predicting(datasetOr,app,spark)
    }catch {
      case e:Exception => {
        logger.error(app+"离线计算错误！")
        e.printStackTrace()
        -1
      }
    }
  }
  def predicting(df: DataFrame, app: App, spark: SparkSession):Double

  def generateNoTrainSet(app: App): mutable.Set[Int] = {
    CommonMethods.generateNoTrainSet(app)
  }

  def updateWsse(app: App,value:Double):Unit={
    MysqlOperation.writeValue("app","mse","id = "+app.getId,value)
    logger.info(app+"更新mse完成，更新的值是："+value+" id:"+app.getId)
  }
  def dealBase(df:DataFrame, app:App, spark:SparkSession):DataFrame
  def dealGrowth(df:DataFrame,app:App, spark:SparkSession):DataFrame
  def dealWindows(df:DataFrame, app:App, spark:SparkSession):DataFrame

  def getActivation(activationString:String):Activation={
    var act:Activation = null
    activationString match {
      case "relu" => act = Activation.RELU
      case "sigmod" => act = Activation.SIGMOID
      case "softmax" => act = Activation.SOFTMAX
      case _ => act = Activation.IDENTITY
    }
    act
  }

  def getOptimization(optimizationString:String):OptimizationAlgorithm={
    var opt:OptimizationAlgorithm = null
    optimizationString match {
      case "line_gradient_descent" => opt = OptimizationAlgorithm.LINE_GRADIENT_DESCENT
      case _ => opt = OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT
    }
    opt
  }

  def getLossFunctions(lossFunctionsString:String):LossFunctions.LossFunction={
    var loss:LossFunctions.LossFunction = null
    lossFunctionsString match {
      case "mcxent" => loss = LossFunctions.LossFunction.MCXENT
      case "squared_loss" => loss = LossFunctions.LossFunction.SQUARED_LOSS
      case _ => loss = LossFunctions.LossFunction.MSE
    }
    loss
  }
}
```

# 4. 预测结果

对实际应用的中石化塔河项目中，24 小时运行，并成功完成一次预警。

在实际应用时使用了 LSTM 和 KMeans 两种模型分别测试了预警效果，结果都可以完成实时预测（因为其实异常蛮明显的）。

*项目还在不断改进。。。*
