---
layout: post
title: Sqoop 与 Datax 对比
categories: Knowledge
description: Sqoop 与 Datax 对比
keywords: Sqoop,Datax
---
[参考来源](https://chu888chu888.gitbooks.io/hadoopstudy/content/Content/9/datax/datax.html)

目录

* TOC
{:toc}

## sqoop 特点

1. 可以将关系型数据库中的数据导入 hdfs、hive 或者 hbase 等 hadoop 组件中，也可将 hadoop 组件中的数据导入到关系型数据库中；

2. sqoop 在导入导出数据时，充分采用了 map-reduce 计算框架，**根据输入条件生成一个 map-reduce 作业**，在hadoop集群中运行。采用 map-reduce 框架**同时在多个节点进行** import 或者 export 操作， 速度比单节点运行多个并行导入导出效率高，同时提供了良好的并发性和容错性；

3. 支持 insert、update 模式，可以选择参数，若内容存在就更新，若不存在就插入；

4. 对国外的主流关系型数据库支持性更好。

## datax 主要特点

1. 异构数据库和文件系统之间的数据交换；

2. 采用 Framework + plugin 架构构建，Framework 处理了缓冲，流控，并发，上下文加载等高速数据交换的大部分技术问题，提供了简单的接口与插件交互， 插件仅需实现对数据处理系统的访问；

3. 数据传输过程在**单进程内完成，全内存操作，不读写磁盘**，也没有 IPC；

4. 开放式的框架，开发者可以在极短的时间开发一个新插件以快速支持新的数据库/文件系统。

## sqoop 和 datax 的区别：

1. sqoop 采用 map-reduce 计算框架进行导入导出，而 datax 仅仅在运行 datax 的单台机器上进行数据的抽取和加载，速度比sqoop慢了许多；

2. sqoop **只可以在关系型数据库和 hadoop 组件之间进行数据迁移**，而在 hadoop 相关组件之间，比如 hive 和 hbase 之间就无法使用 sqoop 互相导入导出数据，同时在关系型数据库之间，比如 mysql 和 oracle 之间也无法通过 sqoop 导入导出数据。与之相反，datax 能够分别实现关系型数据库和 hadoop 组件之间、关系型数据库之间、hadoop 组件之间的数据迁移；

3. sqoop 是专门为 hadoop 而生，对 hadoop 支持度好，而 datax 可能会出现不支持高版本 hadoop 的现象；

4. sqoop **只支持官方提供的指定几种关系型数据库和 hadoop 组件之间的数据交换**，而在 datax 中，用户只需根据自身需求修改文件，生成相应 rpm 包，自行安装之后就可以使用自己定制的插件；