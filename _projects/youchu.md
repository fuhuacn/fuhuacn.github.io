---
layout: wiki
title: 中国邮政储蓄银行北京分行 — 大数据项目
categories: 大数据
description: 中国邮政储蓄银行北京分行 — 大数据项目
---

目录

* TOC
{:toc}

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

JAVA Web 项目，基于 Spring Boot 开发。

大数据平台管理平台提供生产用大数据平台的健康监控、日志管理、告警管理、数据管理和用户管理功能。首页界面如下：

![首页](/images/peojects/youchu/portal.png)

### JMX 监控

在大数据平台的每台物理机中，均部署了 JMX 监控脚本，该脚本会每隔 60 秒收集一次部署机的各组件（主要为 HDFS 和 YARN）的 JMX 监控日志。下面为请求 NameNode 的 JMX 部分日志（实验室集群）。

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

### 难点

+ 数据表多（2000 以上），数据量巨大（最高流水类接近百亿级）。
+ 需要保证数据格式对应。
+ 迁移数据需保留分区（可能会有动态分区，要求 oracle 分区可以对应 HIVE 分区，或按一定规则分区），支持 HIVE 中的分桶操作，提高效率。
+ 迁移数据需要同时保留中文 comments 。
+ 不仅要满足一次性迁移，还需可以在之后追加数据。
+ 追加周期区分粒度，可以按日/月方式进行。
+ 追加方式分为三种：全部覆盖，追加分区，按某一字段值追加。
+ 所有追加任务尽可以在晚间进行。

### 概述

迁移流程图如下：

![ETL 任务](/images/peojects/youchu/ETL.png)

ETL 迁移脚本由 python3 编写，由 crontab 控制每分钟执行一次，脚本启动后会读取各任务的当前信息，根据**阶段**和**状态**两个字段（后面详细解释）决定接下来的操作。

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
  + 删除阶段：将 HIVE 中数据表删除。
+ 状态：当前迁移所处状态，分为以下：
  + 等待中：当前阶段还没有开始，等待开始，当开始后状态切换为运行中。
  + 运行中：当前阶段正在运行，等待检查，当检查完毕后切换为下一阶段的等待中、错误或结束。
  + 已完成：该任务已结束，并且目前没有配置追加。
  + 错误：当前任务出错或检查数量不符出错，需要人为查看原因。
  + 追加：检查是否达到追加时间。
+ 追加周期：追加的方式（日/月）。
+ 追加种类：
  + 条件追加：按某种特定的条件追加数据，比如两个表之间有相互关系时，必须表 1 满足条件后，表 2 再迁移。
  + 分区追加：仅迁移新的分区的数据。
  + 全量覆盖：把源表全部覆盖到 HIVE 的表中。
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

*关于为什么区分 ODS 阶段 和 DW 阶段：*

由于涉及到了动态分区， oracle 中的分区需与 hive 中的分区对应，甚至 oracle 中无分区，但导入到 hive 中需要有分区，比如按照时间字段 20180101 中的 201801 月来分区，这时候使用 sqoop 的分区（只可选定数据导入到某单个分区）便显得力不从心。**HIVE 中的分区是与字段名必须不同的，如何设定分区名称也需要交给用户选择。如果只填写了 partition 字段，则会按照填写字段名称增加“ _par ”作为分区名称。如果填写了 partition_select 字段，便是填写自定义函数，如 substring()，以这个字段将作为分区名称。**

所以我将整个导入过程拆成了两部分：
+ ODS（Operational Data Store）：操作性数据导入，即中间表，从 Sqoop 全量导入 Oracle 中的数据到这张表。
+ DW（Data Warehouse）：数据仓库，即把中间表按分区导入到了最终的数据仓库中。

接下来将分各阶段介绍迁移流程。

### 创建阶段

创建迁移的目的数据表。

对于一个新任务，需要创建两个数据表：分别是 ODS 数据表和 DW 数据表。

对于一个追加任务，只需要创建 ODS 数据表，因为追加任务证明已完成过此任务，有最终的数据表。

创建数据表时首先需要确认源 Oracle 中各数据表的字段类型，并做字段对应。

**为什么不实用 sqoop 的 –create-hive-table 参数：** 因为在邮储中仍然需要保留如 timestamp 之类的时间戳字段，如果使用自动创建，他可能会出现无法完全对应的字段。同时我们的迁移过程还需要保留中文注释。

**HIVE 中文 COMMENTS：** HIVE 的中文注释需要修改元数据库（HIVE 的 MySQL）的字符信息，否则从 Oracle 导过来会乱码。

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
      # 如果排序了，必须补充默认的分区分桶字段
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

当创建表完成并且检验通过后，进入下一阶段 —— ODS 同时状态会被标记为等待中。

### ODS

即将全部源 Oracle 数据表导入到中间表的过程。

ODS 任务会执行 Sqoop 脚本程序，并将日志写入到对应日志文件夹中，如下所示：

  ``` python
  def ods(job):
      # 获取日志路径
      path = get_log_path(job)
      # 检查是否达到ods任务的并发数量，最大并发数为ORACLE_CONCURRENCY
      if dao.get_ods_running_count() == ORACLE_CONCURRENCY:
          return

      dao.set_status(job.database, job.table, 'RUNNING')
      log.info('%s.%s ods running', job.database, job.table)

      # 如果追加类型不是overwrite并且还有追加要求时，需要选择特定的源数据去追加
      if job.append and job.append != 'overwrite':
          # 只追加一个分区
          if job.append == 'partition':
              query = '"SELECT * FROM ' + job.src_table + ' PARTITION(P' + job.latest + ') WHERE \$CONDITIONS"'
          # 按条件追加
          elif job.append == 'where':
              query = '"' + check_condition.get_query(job) + ' and \$CONDITIONS"'
          else:
              dao.set_status(job.database, job.table, 'ERROR')
              log.error('%s.%s append type not supported', job.database, job.table)
              return
      # 否则将是全量覆盖追加
      else:
          query = '"SELECT * FROM ' + job.src_table + ' WHERE \$CONDITIONS"'
      # 提交sqoop任务
      p = subprocess.Popen('sqoop import --hive-import' +
                          ' --connect ' + 'jdbc:oracle:thin:@%s:%s/%s' % (
                              job.src_host, job.src_port, job.src_service_names) +
                          ' --username ' + job.src_username +
                          ' --password ' + job.src_password +
                          ' --query ' + query +
                          ' --hive-database ods' +
                          ' --hive-table ' + job.table +
                          ' --target-dir ' + job.table +
                          ' --delete-target-dir' +
                          ' --null-string \'\\\\N\' --null-non-string \'\\\\N\'' +
                          ' --hive-delims-replacement \'|_|\'' +
                          ' --num-mappers 1 ' + job.parameter +
                          ' > ' + path + 'ods.log' + ' 2>&1', shell=True)
      dao.set_ods_pid(job.database, job.table, p.pid)
  ```

这里需要注意的是是否是属于追加任务，如果是追加任务，则只选择追加部分的数据内容，并且会根据追加的形式选择迁移的数据量，例如全量覆盖会选择全部数据而分区追加仅选择增加的分区的数据。

当迁移开始后，该条任务的状态会被标记为运行中，之后程序读此任务时会首先读取日志中的 mapreduce.Job: Running job: 字段，记录 ODS 任务的 yarn job id 和开始时间。再之后的每次检查时，会执行 yarn application -status 命令检查当前任务状态。任务完成时证明 ODS 任务已经完成。接着可以进入下一阶段 DW 阶段并再次将状态标为等待中。

### DW

这一部的主要目的是对中间表的数据按分区形式导入到数据仓库中。

这里先给出一个分区语言事例，可以跟之后代码中的 sql 比对一下：
  ``` sql
  SET hive.exec.dynamic.partition=true;  
  SET hive.exec.dynamic.partition.mode=nonstrict; 
  SET hive.exec.max.dynamic.partitions.pernode = 1000;
  SET hive.exec.max.dynamic.partitions=1000;
  
  INSERT overwrite TABLE t_123_partitioned PARTITION (month,day) 
  SELECT url,substr(day,1,6) AS month,day 
  FROM t_123;
  ```

程序中代码如下：

  ``` python
  def dw(job):
      path = get_log_path(job)

      dao.set_status(job.database, job.table, 'RUNNING')
      log.info('%s.%s dw running', job.database, job.table)

      sql = ''
      # 如果追加类型不是overwrite并且还有追加要求时，需要选择特定的源数据去追加
      if job.append and job.append != 'overwrite':
          sql += 'INSERT INTO TABLE `' + job.database + '`.`' + job.table + '` '
      else:
          sql += 'INSERT OVERWRITE TABLE `' + job.database + '`.`' + job.table + '` '
      if job.partition:
          sql += 'PARTITION(`par_' + job.partition + '`) '
          if job.partition_select:
              sql += 'SELECT *,' + job.partition_select + ' AS `par_' + job.partition + '` '
          else:
              sql += 'SELECT *,`' + job.partition + '` AS `par_' + job.partition + '` '
      else:
          sql += 'SELECT * '
      sql += 'FROM `ods`.`' + job.table + '`;'

      with open(path + 'dw.sql', 'w') as file:
          file.write(sql)

      # 提交hive任务，并写入指定日志文件中
      p = subprocess.Popen('hive -f ' + path + 'dw.sql ' + '> ' + path + 'dw.log' + ' 2>&1', shell=True)
      dao.set_dw_pid(job.database, job.table, p.pid)
      dao.set_dw_time(job.database, job.table, time.strftime('%Y-%m-%d %H:%M:%S', time.localtime()))
  ```

这里的重点也是追加的形式，究竟是全量追加覆盖数据，还是按分区/条件追加追加数据。同时要注意分区名称的配置。

同样开始 insert 的任务后，会把日志写入指定文件夹内，程序会一直检查日志的状态。在第一次读取日志时，通过 Executing on YARN cluster with App id 字段记录 YARN 的 job id 。在之后的检查中，仍然通过 yarn application -status 命令读取 yarn 任务状态，查看任务是否完成。当任务完成后，会将程序阶段标记为检查，状态标记为等待中。 

### 检查阶段

检查阶段主要是检查迁移数量和最终表中增加数量是否相等。

分别在 Oracle 和 HIVE 中执行两次 SQL 语句。检查 Oracle 中的数量和 HIVE 中的数量是否一致。这里的麻烦点还是在于追加的条件判断。下面列出关键部分代码：

``` python
# 在 Oracle 中检查语句
if job.append == 'partition':
    query = 'SELECT COUNT(*) FROM ' + job.src_table + ' PARTITION(P' + job.latest + ')'
elif job.append == 'where':
    query = check_condition.get_query_count(job)
else:
    query = 'SELECT COUNT(*) FROM ' + job.src_table

# 在 HIVE 中的检查语句
if job.append == 'partition':
    query = 'SELECT COUNT(*) FROM ' + job.database + '.' + job.table + ' WHERE par_' + job.partition + '=' + job.latest
elif job.append == 'where':
    query = check_condition.generate_count_hive_query(job)
else:
    query = 'SELECT COUNT(*) FROM ' + job.database + '.' + job.table
```

检查结束后，如果数量一致，需要删除 ODS 数据表，记着同时也要删除 ODS 日志。

此阶段完成后，任务阶段切换为完成或追加，状态根据不同阶段为空或追加。

### 追加

任务完成后，可以配置为追加。追加分为按日追加或按月追加。

由于所有追加任务必须在夜间进行，所有处于此阶段的任务判断只会在晚间启动。

读取当前系统时间，与任务的下次运行时间比对，即可确定是否需要追加任务。当需要追加任务时，会将状态切换为等待中，即可等待追加检查。

  ``` python
  def append(job):
      path = get_log_path(job)

      if not job.append:
          return
      if not job.next:
          return
      # 判断是否到达追加的时间
      if time.strptime(str(job.next), '%Y-%m-%d %H:%M:%S') > time.localtime():
          return

      # 判断追加的间隔，如果是按天追加，最新数据加一天，如果是按月追加，最新数据增加一个月
      if job.append_period == 'D':
          oneday = datetime.timedelta(days=1)
          old_latest = datetime.datetime.strptime(str(job.latest), '%Y%m%d')
          latest = (old_latest + oneday).strftime("%Y%m%d")
          old_next = datetime.datetime.strptime(str(job.next), '%Y-%m-%d %H:%M:%S')
          next = (old_next + oneday).strftime("%Y-%m-%d %H:%M:%S")

      else:
          year = int(job.latest[0:4])
          month = int(job.latest[4:6])
          month += 1
          if month > 12:
              year += 1
              month -= 12
          if month < 10:
              month = '0' + str(month)
          latest = '%s%s' % (year, month)

          next = time.strptime(str(job.next), '%Y-%m-%d %H:%M:%S')
          year = next.tm_year
          month = next.tm_mon
          month += 1
          if month > 12:
              year += 1
              month -= 12
          if month < 10:
              month = '0' + str(month)
          next = '%s-%s-%s %s:%s:%s' % (year, month, next.tm_mday, next.tm_hour, next.tm_min, next.tm_sec)

      log.info('%s.%s will append %s', job.database, job.table, latest)
      dao.set_append(job.database, job.table, latest, next)
  ```

当一个任务被设定为开始追加后，同时要设定他的下次运行日期为下一天/月，以备下次追加使用。

### 追加检查

可能在上面已经看见了，很多代码中出现了 check_condition 这个类，他的作用就是对于特殊配置了的表，起到特殊的选择和检查作用。

例如在 Oracle 中一个表是否可以迁移取决于另一个标记表。只有当条件满足标记表中有此待迁移表记录时，才可以迁移此表。对于这种类型的迁移都会在 check_condition 中判断。 check_condition 中配置有特殊表的检查方法和迁移时的选择语句。

在 check_condition 中配置的数据表，在开始任务前都会经过此部检查是否满足迁移条件。没有配置的表可以直接通过此部。

当此部通过后，表的阶段和状态又会重新被标记为创建阶段和等待中，这样就可以再一次的开始追加迁移了。

这部分代码涉及过多邮储数据表内容，不能公开。

### 删除

出错后或需要删除某张数据表使用的字段，删除 HIVE 中该数据表并清空日志。

### 配置及使用

配置任务的迁移同样在大数据管理平台中，如下图：

![任务配置](/images/peojects/youchu/WX20191109-173521.png)

用户可以新建任务并对任务配置。同时可以搜索查看任务配置信息。

![新建任务](/images/peojects/youchu/WX20191109-173753.png)

日志查看：通过 SSH 到目标机，读取对应任务的日志文件。

### 安全性

Apache Ranger 是 Hadoop 中最有名的权限管理组件。此项目中最重要的 HIVE 权限管理就是 Ranger 完成的。

但 HIVE 默认为不鉴权，开启鉴权后，最简单的鉴权方式为 ldap 。于是选取了一台虚拟机做 ldap 用户服务器，使用 ldap 作为用户管理体系完成全部的鉴权管理。

## 虚拟化平台

虚拟化平台使用 kvm 内核做虚拟机。虚拟机主要用于功能模块的安装和大数据实验平台的搭建。

### 功能模块的安装

即之前所提到的大数据管理平台中的组件等。由于组件较多，使用虚拟化技术分离环境比较方便。

### 大数据实验平台

这也是真正使用虚拟化技术的核心原因。

跑 HIVE 分析任务时首先不可能在生产用大数据平台中进行，所以我们创立了数据实验室这个概念。

数据实验室即一键搭建 Hadoop ，最开始的方案是使用 docker 搭建。这样方便快捷，配置简单。但是 docker 下是不好做 CPU 完全性能隔离的。试想如果一个用户进行了错误操作，导致 CPU 被跑满，这是完全不可接受的。所以最终放弃了 docker 。

而虚拟化技术是可以完全做到性能的隔离，但是这一过程相对麻烦：
+ 创建 NameNode 的镜像， DataNode 的镜像。
+ 以之前两台虚拟机镜像作为模版，创建一个 1 台 NameNode ， 2 台 DataNode 的集群。
+ 登陆 NameNode 修改免密登陆配置，修改 hosts 信息，同时将 hosts 发送给两台 DataNode 。

### 从大数据平台到数据实验室的迁移

通过这一功能可以实现从生产化大数据平台到任意实验大数据平台的迁移，方便使用人员在各自环境中分析数据。

**这一部分在大数据实验室中的 HIVE 表均为外部表，因为实际上使用的是拷贝 hdfs 文件，所以在删除表的时候，同时需要删除 hdfs 文件。**

这部分其实实现过程与从“定制化 ETL 迁移工具”类似，区别在于：

+ 创建阶段，由从 Oracle 中读取源表信息，更改为从 HIVE 生产大数据平台中读取表信息（已保存在日志文件中），无需格式转换。
+ 不再区分 ODS 阶段和 DW 阶段，统一为 DISTCP 阶段。
+ 增加分区阶段。
+ 删除表的时候，同时需要删除 hdfs 文件。

下面对重点部分进行讲解：

+ DISTCP阶段：
  
  由于是 HIVE 到 HIVE 的传输，其实最简单的办法便是直接拷贝 HDFS 文件，于是想到了使用 DISTCP （分布式拷贝，同样适用 MapReduce）传输。

  对于 HIVE 来讲，分区是一个一个的文件夹，分桶是一个一个的文件，所以说完全拷贝结构或只拷贝传输分区文件夹的结构，便实现了 HIVE 数据表的传输。

  ``` python
  def distcp(job):
      path = get_log_path(job)

      dao.set_status(job.database, job.table, 'RUNNING')
      logger.log.info('%s.%s distcp running', job.database, job.table)

      hadoop_ip = configer.config['hadoop']['ip']
      hadoop_port = configer.config['hadoop']['port']
      lab_ip = configer.config['lab']['ip']
      lab_port = configer.config['lab']['port']

      parameter = ''
      if job.partition and job.partition != 'ALL':
          parameter += 'hdfs://' + hadoop_ip + ':' + hadoop_port
          parameter += '/apps/hive/warehouse/' + job.src_database + '.db/' + job.src_table.lower() + '/' + job.partition + '/' + ' '
          parameter += 'hdfs://' + lab_ip + ':' + lab_port
          parameter += job.location + job.partition + '/' + ' '
      else:
          parameter += 'hdfs://' + hadoop_ip + ':' + hadoop_port
          parameter += '/apps/hive/warehouse/' + job.src_database + '.db/' + job.src_table.lower() + '/' + ' '
          parameter += 'hdfs://' + lab_ip + ':' + lab_port
          parameter += job.location + ' '

      p = subprocess.Popen('hadoop distcp -update -skipcrccheck ' + parameter + '> ' + path + 'distcp.log' + ' 2>&1', shell=True)
      dao.set_distcp_pid(job.database, job.table, p.pid)
  ```

+ 分区阶段：
  
  对导入后的 HIVE 数据表使用 ALTER 创建分区。

  首先会登陆 HIVE 所在的机器中，对于追加一个分区的任务，可以只声明一个分区。

  对于没有分区的任务可以跳过此部。

  对于多个分区的任务，则需要通过 show partitions 操作获取全部分区并使用 for 循环执行全部的分区声明操作。

  ``` python
  def add_partition(job):
    path = get_log_path(job)

    if not job.partition:
        dao.set_stage_and_status(job.database, job.table, '', 'FINISH')
        dao.add_log(job.database, job.table, time.strftime('%Y-%m-%d %H:%M:%S', time.localtime()))
        logger.log.info('%s.%s finish', job.database, job.table)
        return

    dao.set_status(job.database, job.table, 'RUNNING')
    logger.log.info('%s.%s addp running', job.database, job.table)

    sql = ''
    if job.partition == 'ALL':
        p = subprocess.Popen('ssh root@' + configer.config['hadoop']['ip'] + ' "hive -e \\\"SHOW PARTITIONS ' + job.src_database + '.' + job.src_table + ';\\\"" > ' + path + 'partitions.sql' + ' 2>&1', shell=True)
        p.wait()
        partitions = []
        with open(path + 'partitions.sql', 'r', encoding='utf-8') as file:
            for line in file:
                if line.startswith('par_'):
                    partitions.append(line[0:-1])
        for partition in partitions:
            sql += 'ALTER TABLE `' + job.database + '`.`' + job.table + '` ADD PARTITION (' + partition + ') LOCATION \'' + job.location + partition + '/\';\n'
    else:
        sql += 'ALTER TABLE `' + job.database + '`.`' + job.table + '` ADD PARTITION (' + job.partition + ') LOCATION \'' + job.location + job.partition + '/\';\n'

    with open(path + 'addp.sql', 'w', encoding='utf-8') as file:
        file.write(sql)

    p = subprocess.Popen('hive -f ' + path + 'addp.sql ' + '> ' + path + 'addp.log' + ' 2>&1', shell=True)
    dao.set_addp_pid(job.database, job.table, p.pid)
    dao.set_addp_time(job.database, job.table, time.strftime('%Y-%m-%d %H:%M:%S', time.localtime()))
  ```

## 人脸识别高并发接口

这一部分与大数据平台的关系并不大，邮储银行自己拥有自己的事件平台（貌似是 Rabbit Mq 改的），他们的人脸识别应用是第三方公司做的。我负责封装一个事件平台接口供人脸识别调用，通过调用我这个接口返回客户的个人信息。

这部分听起来并不难，高并发也就是做一个线程池，当收到一个新时间时，扔到线程池里查询数据库，获取个人信息再扔回事件平台就 OK 了，但真正的坑点就是这个线程池！

**邮储官方给的说明中提到，JAVA SDK 在多线程条件下无法使用。**

这就很难受了，我亲自试了一下，当在线程池中发送回事件平台时，SDK 直接卡死。

解决方案：

对于这种大批量接口调用的，肯定不能但线程下进行啊，尤其查询 Oracle 这部操作并不快。

之后选择了一种神奇的方案：Socket 进程池。

这方面多多少少受了我[阿里中间件初赛](http://www.fuhuacn.top/2019/10/26/alimiddleware/)的影响，我将收到了事件也看作是一个 Gateway 的操作，查询和发送回事件平台看作是 Provider 操作。只不过 Gateway 和 Provider 均换成了最简单的 Socket 连接。

这样就相当于下图：

![进程池](/images/peojects/youchu/事件平台接口.png)

这里以三个 Provider 代替了，实际部署上使用了 10 个 Provider 。

同时在 Gateway 记录着每个 Provider 当前的任务处理信息（当选定一个 Provider 发送后，Gateway 会等待 Provider 结束并收集他的处理时间在关闭 Socket 连接）。之后通过参数计算得分（其实就比了一下任务数量），下次发送给得分最高的机器。这部分的公式都与比赛时类似：

得分 = 100 - 当前剩余任务数量 - 任务延迟时间（毫秒）/5

但其实最后算下来，就是挨个轮着发了，因为都部署在了同一台机器上，也没啥差别了。

### 关于这部分使用 NIO ：

其实在一切都开发接近结束时，又想到了用 NIO 解决这一问题，即线程池多线程查询到结果后，通过 NIO Socket 发送给服务器，服务器使用选择器管理多个 Channel 。据说比多线程效率还高。

附一个 NIO Socket [链接](https://juejin.im/post/5b947ffa6fb9a05d2b6d9a68)，以后可以试试。

# 4. 总结

邮储项目持续了接近一年的时间，他也是我研究生两年时间的一个缩影。这个项目从大框架设计，到每个功能模块的拆解，再到到代码的研发都是我主导并参与（除了虚拟化底层部分是同组其他人制作）。其中的数据迁移部分更是耗时最多也调整部分最大的地方。整个项目虽然说叫大数据项目，但更多的其实还是开发项目（数据迁移的设计，大数据平台的监控与告警，人脸接口的设计），他让我更加熟悉了最主流的几个大数据工具（Hadoop 、HIVE 、Kafka 、Storm 等），为后面的两个开发项目（公安部一所数据迁移项目，批流一体的自适应异常检测框架）打下了良好的基础。