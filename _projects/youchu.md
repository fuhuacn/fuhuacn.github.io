---
layout: wiki
title: 中国邮政储蓄银行北京分行 — 大数据项目
categories: 机器视觉
description: 中国邮政储蓄银行北京分行 — 大数据项目
---

中国邮政储蓄银行北京分行 — 大数据项目是为邮储银行北京分行定制化搭建的生产大数据分析、管理应用项目。该项目从立项到完成耗时接近一年，由于银行的安全级别特殊性，期间几个月连续每周至少一次前往邮储银行北京分行（玺萌大厦）。从项目的设计到项目的开发使用及维护，一直由我主导并推进完成。可以说这也是我研究生期间耗费精力最多的项目了。

# 1. 项目需求

经过几轮的沟通最终确定邮储银行的需求主要有以下几方面：

+ 搭建生产用大数据平台。
+ 完成生产用大数据平台的系统运行状况监督和预警。
+ 提供一键安装实验用大数据平台，可以完成与生产化大数据平台的性能隔离。
+ 确保大数据平台的权限机制。
+ 完成数据集市（邮储银行的原 Oracle 数据库）到生产用大数据平台的自动 ETL 迁移。
+ 完成生产用大数据平台到指定实验用大数据平台的按需 ETL 迁移。
+ 完成人脸识别接口的高并发调用（与大数据平台关系不大）。

邮储银行提供的硬件配置大概如下：

共提供22台物理机，每台配置约 20 核心，256 G 内存，6 TB 硬盘存储

每台机器提供可连入邮储内网的IP地址。

# 2.框架设计

框架图如下所示：

![邮储架构](/images/peojects/youchu/邮储架构图.png)

框架分为三部分：

+ 生产用大数据平台：为邮储银行搭建的生产用大数据平台及定制化迁移 ETL 工具。
+ 虚拟化平台：方便于自研组件及公用数据库（MySQL）的安装。
+ 人脸识别高并发接口：单独安装于邮储银行内部一台物理机上。

# 3. 功能设计

## 安装时无外网

其实整体架构之前在实验室内已经模拟过一次，安装起来整体虽然烦琐，但都有文档记录。唯一的变动就是无法连入外网，所以所有资源都需要提前准备好包括：

+ CentOS 完整版 yum 源，以光盘形式配置一台机器的 yum 源。
+ 配置光盘 yum 源的机器安装httpd服务，并拷入以下文件作为其余机器 yum 源。
  + CentOS 完整版 yum 源
  + Ambari 所需 yum 源。
  + Hadoop 2.6 yum源。
+ 配置一台 NTP 服务器（保证时间同步）。
+ 配置所有机器的 JAVA 版本为 jdk 1.8。

## 生产用大数据平台

大数据平台物理机均安装 CentOS 6.4 服务器系统。

为了方便于安装及日后的维护，考虑到之后邮储银行运维方便，选择使用 Apache Ambari 安装和管理hadoop。安装版本为 hadoop 2.7.1。同时安装的组件还有：
+ HIVE：生产用大数据平台数据仓库。方便实现数据的分析计算。
+ Sqoop：使用 Sqoop 作为定制化 ETL 迁移的底层工具。Sqoop 基于 MapReduce 提供分布式迁移能力。
+ Kafka：消息队列，做大数据平台收集日志的中转平台。
+ Storm：实时计算平台，负责收集日志后的实时分数计算，告警规则判断。
+ Ranger：生产用大数据平台的权限管理组件。

## 乐志

实验室曾经的一个自研项目，用于存储 NoSQL 日志。主体思想是基于 HBase 封装了一个可存储、查询的可视化接口及界面。并且给 HBase 设置了 TTL 。超过一周的日志可以自动被删除。

**注意：** 这里的 HBase 是不使用生产用大数据平台的。生产用的数据平台只用来存储邮储的内部数据。日志数据用一个新的 hadoop 集群存储。实现数据的分离。

## 大数据平台管理平台

大数据平台管理平台提供生产用大数据平台的健康监控、日志管理、告警管理、数据管理和用户管理功能。首页界面如下：

![首页](/images/peojects/youchu/portal.png)

### JMX 监控

在大数据平台的每台物理机中，均部署了 JMX 监控脚本，该脚本会每隔 60 秒收集一次部署机的各组件（主要为 HDFS 和 YARN ）的 JMX 监控日志。下面为请求 NameNode 的 JMX 部分日志（实验室集群）。

``` json
{
  "beans" : [ {
    "name" : "Hadoop:service=NameNode,name=JvmMetrics",
    "modelerType" : "JvmMetrics",
    "tag.Context" : "jvm",
    "tag.ProcessName" : "NameNode",
    "tag.SessionId" : null,
    "tag.Hostname" : "ambari-namenode.com",
    "MemNonHeapUsedM" : 96.710556,
    "MemNonHeapCommittedM" : 98.82031,
    "MemNonHeapMaxM" : -1.0,
    "MemHeapUsedM" : 318.44983,
    "MemHeapCommittedM" : 1011.25,
    "MemHeapMaxM" : 1011.25,
    "MemMaxM" : 1011.25,
    "GcCountParNew" : 6438,
    "GcTimeMillisParNew" : 116989,
    "GcCountConcurrentMarkSweep" : 2,
    "GcTimeMillisConcurrentMarkSweep" : 108,
    "GcCount" : 6440,
    "GcTimeMillis" : 117097,
    "GcNumWarnThresholdExceeded" : 0,
    "GcNumInfoThresholdExceeded" : 9,
    "GcTotalExtraSleepTime" : 69786,
    "ThreadsNew" : 0,
    "ThreadsRunnable" : 7,
    "ThreadsBlocked" : 0,
    "ThreadsWaiting" : 8,
    "ThreadsTimedWaiting" : 118,
    "ThreadsTerminated" : 0,
    "LogFatal" : 0,
    "LogError" : 0,
    "LogWarn" : 639503,
    "LogInfo" : 2793915
  }
  ...
```

*该模块后来进行了版本迭代，更改为在 StandbyNameNode 上部署唯一监控脚本，该脚本会请求所有物理机的 JMX 地址，这样可以更直观发现是否有物理机掉线。若 60 秒内无新的监控日志，则证明至少 StandbyNameNode 掉线。*

收集完各机 JMX 日志后，会将日志发送至 Kafka 做健康状况打分评估、判断是否告警及日志存储。

### 健康状况打分评估

在首页中的左上角的分数（65分）即为该集群当前的健康状态评分。

使用 Storm （其实对于这种小的程序可能写个 JAVA 小程序也够了）开一个 Kafka 消费者组，消费收集到的 JMX 日志。通过计算公式计算出当前打分，并写入乐志。

### 告警预测

同样使用 Storm 作为 Kafka 消费者完成。可以使用正则表达式等形式完成预警判断，如下图：

![告警](/images/peojects/youchu/alarm.png)

同时可以设定告警邮箱（在邮储内网环境是连接了邮储的短信通知平台），当超过阈值时发送告警信息。

### 日志存储

所有的日志数据可以配置存储到两个地方： Kafka （如果存储在 Kafka 将只支持顺序读取）或乐志（即单独的 hbase 集群）。这里使用一个 java 消费 Kafka 数据。这个 java 程序会定时从 MySQL 读取应用信息并更新要存储的 topic 。

下图是管理平台的三个组件的 Kafka 消费示意图。

![管理平台](/images/peojects/youchu/邮储管理平台组件.png)