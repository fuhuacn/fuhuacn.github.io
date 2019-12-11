---
layout: wiki
title: 公安部第一研究所身份信息大数据迁移项目
categories: 大数据
description: 公安部第一研究所身份信息大数据迁移项目
---

公安部第一研究所大数据迁移评估项目是在邮储项目开发接近尾声时接到的，与之前的迁移项目不同的是，这个项目时间短，强度大。从接到任务到任务的研发完成再到安装，前前后后也就不到两周的时间。

# 1. 项目需求

+ 搭建大数据平台。
+ 从 Oracle 中批量抽取人口身份信息（约 14 亿条），并将数据存储至 HIVE 中。
+ 抽取时需要重新判断身份证信息是否符合规则。
+ HIVE 中的简单离线数据分析。
+ 离线数据分析结果展示。

# 2. 系统架构设计

## 设计总览

![概览](/images/projects/gonganbu/公安部框架图.png)

有了之前的迁移设计经验呢，对之前的进行了一些变动：

+ 由于这次确定都是属于技术人员用，并且短期内只会迁移固定的一张表，所以并没有管理员界面。
+ 迁移不再用 Sqoop 工具迁移，而是采用多个线程批量读取 Oracle 后将数据发送至 Kafka。使用 Spark 做数据校验并将校验通过的数据返回 Kafka 后存储至 HIVE。
+ 接收校验后的数据存储至文件中，使用 HIVE 的 LOAD 命令加载文件至 HIVE 中。
+ 增加了离线分析及数据展示模块。

由于该项目只是一个验证性质的项目，所以将大数据相关的组件（Spark，Kafka，HIVE）均装在同一集群中。

接下来具体解释一下系统的每一模块。

## Hadoop 安装

主要难题还是没外网，需要配置 yum 源等，可以参考[邮储项目](http://www.fuhuacn.top/projects/youchu/)。

## Oracle 采集组件

必须使用特定物理机连接 Oracle 数据库。

采用多线程同时读取 Oracle 数据库。由于原数据表中没有主键，只能采用 ROWNUM 形式作为行数表示符。

由于涉及了多个线程同时数据库，必须保证线程间不出现重复读取，使用了一个加锁类专门存储目前读取到的最新 ROWNUM，当一个线程完成一批数据的读取后，再次读取时加锁读取并写入最新行数，防止出现重复读取。

在读取完成后，所有线程同时写入一个 LinkedBlockingQueue 中，注意使用的是 offer() 方法添加元素，这样在 Queue 空间不足时可以等待。之后一个线程会一直对这个队列 poll() 元素，并将数据以 JSON 的 list 格式批量发送至 Kafka 的 topic 1。

**注意：** topic 1 和 topic 2 的分区个数均设定为 5。与物理机数量（每台物理机上均安装 Kafka 和 Spark）相同。

## Spark 实时分析组件（后以更改为 Flink）

*Spark 部分后以优化为 Flink，下面是更改前的部分。*

Spark 作为 topic1 的消费者，实时消费发送来的身份证列表进行验证。主要验证就是身份证信息是否合规。这也是为什么放弃了 Sqoop 的原因。

这部分逻辑比较简单，主要是使用 Spark Streaming 配置消费者，判断消费就可以了，但这里有两个小点需要注意：

+ 对于身份证信息验证错误的数据，则需要保存至 MySQL。如果每条数据都建立一次 jdbc 连接，则开销过大，速度很很慢。所以采用为每个分区建立一个 jdbc 连接解决这一问题。其实很简单，就是在调用 foreach 前调用一次 foreachParation 就可以了。

    ``` scala
    out.foreachPartition(p=>{
                var conn: Connection = null
                var ps: PreparedStatement = null
                p.foreach(row=>{
                })
                MysqlOperation.closeConnect(conn,ps,null)
            })
    ```

+ 同理对于身份证信息验证正常数据发送回 Kafka，不可能每次使用都建立一个 KafkaProducer 送回，而如果像上面 jdbc 每个分区建立一个 KafkaProducer 可能会出现两个分区在一台机器时，KafkaProducer 报错：KafkaConsumer is not safe for multi-threaded access。这时候就要使用一个巧方法。可以[参考](https://www.jianshu.com/p/7cada5beb199)）：
    
    将 KafkaProducer 包装成一个可序列化的 class，对 KafkaProducer 采用 lazy 处理。这样可以先将 KafkaProducer.class 分发至各 executor 中，当使用时创建 KafkaProducer 对象，每台机器一个 KafkaProducer。

    同样，每个 executor 内部也会维护一个自有的队列，以文本格式组合校验完成后的数据发送给 kafka。

校验信息完成的身份证号会以 topic 2 发送回 kafka 等待处理。

Spark 程序使用 yarn 集群方式提交。

*后使用 Flink 代替 Spark Streaming。*

## HIVE 存储组件

这部分就是一个简单的 JAVA 程序，作为 topic 2 的消费者消费数据。这个程序的核心由三部分：

+ 带有并发控制的写文件方法。该类会将字符串写入文件。

+ 读方法，会将该文件提交到 hdfs 并删除文件。之后执行 HIVE LOAD 命令将文件 LOAD 进 HIVE 数据表中。在写入文件和删除文件时，使用 synchronized 关键字做并发控制。

+ 定时插入器，定时执行“读方法”，实现数据的定时存储。

以下是简单的逻辑语句：

``` java
public synchronized void writeFile(String content, String topic) {
    File file = new File(Constants.PATH + topic + ".txt");
    BufferedWriter bw;
    try {
        if (!file.exists()) {
            file.createNewFile();
        }
        bw = new BufferedWriter(new OutputStreamWriter(
                new FileOutputStream(file, true)));
        bw.write(content);
        System.out.println(System.currentTimeMillis());
        bw.newLine();
        bw.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

public synchronized void loadFile(String topic) {
    File file = new File(Constants.PATH + topic + ".txt");
    if (file.exists()) {
        HiveConnector _connector = new HiveConnector();
        Connection _connection = _connector.getConnection();
        Statement _statement = _connector.getStatement(_connection);
        try {
            _statement.execute("load data local inpath '" + Constants.PATH + topic + ".txt" + "' into table `hivelol." + topic + "`");
            file.delete();
            logger.info(topic+" has already loaded, file is deleted! "+System.currentTimeMillis());
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            _statement.close();
            _connector.releaseConnection(_connection);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## HIVE 离线分析组件

定时对写入进 HIVE 的定时进行离线分析，可以在传输同时看到分析效果。

主要分析场景给了两个，类似于：

+ 全国各个地级市，分男女统计各种文化程度的数量。
+ 统计每个省或直辖市的各类职业的人数。

都是用 GROUP BY 处理就可以了，并将处理结果写入 MySQL。

涉及到字段名称等敏感信息就不举例了。

## 展示结果组件

使用 ECharts 对 HIVE 分析出的结果简单的绘表展示，麻烦在于字段数量比较多。结果类似于下图（数据都是假的）：

![结果](/images/projects/gonganbu/结果.png)

# 3. 实现成果

全部开发部署结束后，对 Oracle 数据库做了迁移到 HIVE 数据库中，迁移速度经计算为：每分钟 20 - 30 万条左右。速度短板在接收 topic 2 的 HIVE 存储组件，毕竟这是一个但线程接收 Kafka 的 JAVA 应用。其实这里还有改进空间，但已经达到了验证要求，项目便结束了。