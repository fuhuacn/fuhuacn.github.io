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

## 定制化 ETL 迁移工具

如前文所述，定制化 ETL 工具是基于 Sqoop 开发的，具体版本是 1.4.6 。

迁移流程图如下：

![ETL 任务](/images/peojects/youchu/ETL.png)

### 难点

+ 需要保证数据格式对应。
+ 迁移数据需保留分区和分桶。
+ 迁移数据需要同时保留中文 comments 。
+ 不仅要满足一次性迁移，还需可以在之后追加数据。
+ 追加周期区分粒度，可以按日/月方式进行。
+ 追加方式分为三种：全部覆盖，追加分区，按某一字段值追加。
+ 所有追加任务尽可以在晚间进行。

### 概述

ETL 迁移脚本由 python3 编写，由 crontab 控制每分钟执行一次。

在 MySQL 中记录了迁移任务的迁移信息，信息如下：

+ 目标数据库：记录迁移到 HIVE 中的目标数据库名称。
+ 目标数据表：记录迁移到 HIVE 中的目标数据包名称。
+ 源数据库（用户信息）：从哪个 Oracle 用户下的表迁移。
+ 源数据表：Oracle 中数据表的名称。
+ 阶段：当前所处的迁移阶段，按顺序分为以下 6 种：
  + 创建阶段：创建数据表。
  + ODS迁移阶段：使用 Sqoop 迁移到 ODS 中间表。
  + DW阶段：使用 HIVE insert 命令从中间表迁移到目的表。
  + 校验数量阶段：检验最终数量是否完成。
  + 追加阶段：数据表等待到达追加时间。
  + 追加检查阶段：数据表等待是否满足追加的条件。
+ 状态：当前迁移所处状态，分为以下：
  + 等待中：当前阶段还没有开始，等待开始，当开始后状态切换为运行中。
  + 运行中：当前阶段正在运行，等待检查，当检查完毕后切换为下一阶段的等待中、错误或结束。
  + 已完成：该任务已结束，并且目前没有配置追加。
  + 错误：当前任务出错或检查数量不符出错，需要人为查看原因。
  + 追加：检查是否达到追加时间。
+ 追加周期：追加的方式（日/月）。
+ 追加种类：分区追加/条件追加/全部覆盖。
+ 上次任务进行时间。
+ 下次任务进行时间。
+ 分区字段。
+ 分区名称选择方法：在原 Oracle 数据表库，有些表的分区名称很诡异例如时间是以（8位或6位）字符串方式记录，需要手动调整分区字段名称。
+ 分桶数量。
+ 分桶字段。
+ 排序字段。
+ 附加参数：对于部分原表含有特殊参数的如：BLOB ， ROWID 等，需要在 sqoop 中设置 --map-column-java 或 --map-column-hive 对应关系。
+ 创建时间。
+ ods_pid：通过 pid 检测 ods 任务是否完成。
+ ods_job：ods 对应的 yarn 中 job 的 id 。
+ ods 任务开始时间。
+ ods 任务迁移大小。
+ ods 任务迁移条数
+ ods 任务运行时间。
+ ods 任务速度。
+ dw_pid：dw 任务的进程号。
+ dw_job：dw 任务的 yarn 中 job 的 id 。
+ dw_time：dw 任务的开始时间。
+ dw_seconds：dw 任务的运行时间。

接下来将分各阶段介绍迁移流程。

### 创建阶段

创建迁移的目的数据表。

对于一个新任务，需要创建两个数据表：分别是 ODS 数据表和 DW 数据表。

对于一个追加任务，只需要创建 ODS 数据表，因为追加任务证明已完成过此任务，有最终的数据表。

创建数据表时首先需要确认源 Oracle 中各数据表的字段类型，并做字段对应。

**为什么不实用 sqoop 的 –create-hive-table 参数：** 因为在邮储中仍然需要保留如 timestamp 之类的时间戳字段，如果使用自动创建，他可能会出现无法完全对应的字段。同时我们的迁移过程还需要保留中文注释。

**HIVE 中文注释：** HIVE 的中文注释需要修改元数据库（ HIVE 的 MySQL ）的字符信息。

部分创建对应字段代码：

  ``` python
  def get_fields(job, change_parameter):
      fields = ''  # 创建ods和dw表的核心sql语句
      map_column_java = ''  # Oracle转为非hive类型文件时需要添加的参数，在使用sqoop时，该参数能够改变数据表列的默认映射类型
      map_column_hive = ''  # Oracle转为hive类型文件时需要添加的参数，在使用sqoop时，该参数能够改变数据表列的默认映射类型
      # 在Oracle中，视图SYS.USER_TAB_COLS和SYS.USER_TAB_COLUMNS都保存了当前用户的表、视图和Clusters中的列信息
      # 因此在根据任务类中的Oracle源数据库相关信息连接Oracle源数据库之后
      # 可以通过检索这两个表，方便地获取到表的结构，用于创建所需的ods和dw表
      oracle = cx_Oracle.connect(job.src_username, job.src_password,
                                job.src_host + ':' + str(job.src_port) + '/' + job.src_service_names)
      cursor = oracle.cursor()
      cursor.execute(
          "SELECT USER_TAB_COLUMNS.COLUMN_NAME, USER_TAB_COLUMNS.DATA_TYPE, USER_TAB_COLUMNS.DATA_LENGTH, USER_TAB_COLUMNS.DATA_PRECISION, USER_TAB_COLUMNS.DATA_SCALE ,USER_COL_COMMENTS.COMMENTS FROM USER_TAB_COLUMNS, USER_COL_COMMENTS WHERE USER_TAB_COLUMNS.TABLE_NAME='%s' AND USER_TAB_COLUMNS.TABLE_NAME = USER_COL_COMMENTS.TABLE_NAME AND USER_TAB_COLUMNS.COLUMN_NAME = USER_COL_COMMENTS.COLUMN_NAME ORDER BY USER_TAB_COLUMNS.COLUMN_ID" % job.src_table)
      # 根据查找到的表项类型，构造创建表的sql语句
      for row in cursor.fetchall():
          # row[0]为列名
          fields += '`' + row[0] + '` '
          # row[1]为数据类型，由于导入前后数据类型表示方法不同，因此需要做相应转换
          if row[1] == 'DATE':
              fields += 'timestamp'
          elif row[1] == 'NUMBER':
              if str(row[3]) == 'None':
                  if row[4] == 0:
                      fields += 'decimal(38,0)'
                  else:
                      fields += 'double'
              else:
                  fields += 'decimal(' + str(row[3]) + ',' + str(row[4]) + ')'
          elif row[1] == 'CHAR':
              fields += 'varchar(' + str(row[2]) + ')'
          elif row[1] == 'VARCHAR':
              fields += 'varchar(' + str(row[2]) + ')'
          elif row[1] == 'VARCHAR2':
              fields += 'varchar(' + str(row[2]) + ')'
          elif row[1] == 'CLOB':
              fields += 'string'
              map_column_java += row[0] + '=String,'
          elif row[1] == 'BLOB':
              fields += 'string'
              map_column_hive += row[0] + '=string,'
          elif row[1] == 'RAW':
              fields += 'string'
              map_column_hive += row[0] + '=string,'
          elif row[1] == 'ROWID':
              fields += 'string'
              map_column_java += row[0] + '=String,'
              map_column_hive += row[0] + '=string,'
          elif row[1] == 'TIMESTAMP':
              fields += 'timestamp'
          else:
              dao.set_status(job.database, job.table, 'ERROR')
              log.error('%s.%s field type %s not supported', job.database, job.table, row[1])
              return
          if row[5]:
              fields += ' COMMENT ' + '\'' + row[5].replace(';', '；') + '\''
          fields += ',\n'
      cursor.close()
      oracle.close()
      # 在使用sqoop时，需要--map-column-hive和--map-column-java参数改变数据表列的默认映射类型，因此将该参数写入当前任务表格中
      parameter = ''
      if map_column_java != '':
          parameter += ' --map-column-java ' + map_column_java[:-1]
      if map_column_hive != '':
          parameter += ' --map-column-hive ' + map_column_hive[:-1]
      if change_parameter and parameter != '':
          dao.set_parameter(job.database, job.table, job.parameter + parameter)
      return fields[:-2]
  ```

在创建完成数据表后，还需创建日志文件夹，该任务的所有日志都将保留在此文件夹中。该项目设计的日志文件夹是：/etl_log/${database_name}/${table_name}/

再获得字段对应信息后，生成 create sql 文件，并使用 hive -f 命令执行文件，并将执行结果写入到对应文件夹的日志中，检查创建结果。下面给出创建 dw 数据表（所以包含了分区、分桶信息）的代码：

  ``` python
  def create_dw(job, path, fields):
      # 补充默认的分区分桶字段
      if job.sort and not job.cluster:
          job.bucket = 1
          if job.partition:
              job.cluster = job.partition
          else:
              job.cluster = fields[1:fields[1:].index('`') + 1]  # 获取首列名
      # 构造完整建表语句
      sql = ''
      sql += 'CREATE TABLE `' + job.database + '`.`' + job.table + '` (\n'
      sql += fields + ')\n'
      if job.partition:
          sql += 'PARTITIONED BY (`par_' + job.partition + '` string)\n'
      if job.cluster:
          sql += 'CLUSTERED BY (`' + job.cluster + '`)\n'
      if job.sort:
          sql += 'SORTED BY (' + job.sort + ')\n'
      if job.bucket:
          sql += 'INTO ' + str(job.bucket) + ' BUCKETS\n'
      sql += 'ROW FORMAT DELIMITED\nFIELDS TERMINATED BY \'\\u0001\'\nLINES TERMINATED BY \'\\n\'\n'
      sql += 'STORED AS ORC\n'
      sql += ';\n'
      # 写入sql文件准备执行
      with open(path + 'create_dw.sql', 'w') as file:
          file.write(sql)
      # 创建子进程执行建表语句，并等待子进程的完成
      p = subprocess.Popen('hive -f ' + path + 'create_dw.sql' + ' > ' + path + 'create_dw.log' + ' 2>&1', shell=True)
      p.wait()
      # 检查建表后的日志文件，若存在‘OK’则表示建表成功，否则失败
      with open(path + 'create_dw.log', 'r') as file:
          for line in file:
              if line == 'OK\n':
                  return True
      return False
  ```

当创建表完成并且检验通过后，进入下一阶段 —— ods。