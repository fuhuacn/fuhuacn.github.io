---
layout: wiki
title: 中国邮储银行北京分行 - 数据发现门户
categories: 数据发现
description: 中国邮储银行北京分行 - 数据发现门户
keywords: ElasticSearch,数据发现
---

# 中国邮政储蓄银行北京分行 - 数据发现门户

本项目是邮储银行北京分行大数据平台项目之一，项目指帮助银行内部更好的管理所拥有的数据资产，协助非技术人员更好地查看数据资产。此项目我只进行了前部分数据导入、数据搜索和数据展示方面的架构设计，所以在这里只是简单的记录下。

## 项目功能

+ 提供对数据集市中数据的数据字典的检索能力，既支持逐层发现的探索式检索，也支持直接针对数据库、表、字段的关键字搜索；
+ 实现数据库、表、列的数据结构的自动追加。
+ 对[邮储大数据平台](http://www.fuhuacn.top/projects/youchu/)起到辅助作用，业务人员可以通过发现门户的发现能力，选定自己要需求的数据表进行迁移。

## 项目架构

![架构](/images/projects/searchPlatform/数据发现门户.png)

主要有以下四大部分组件：

+ Web 显示端模块：提供数据发现门户的显示界面，供技术/非技术人员使用。
+ 搜索模块：提供中/英文的分词检索能力。
+ 大数据平台数据迁移接口模块：对于存储在大数据平台（HIVE）中的数据提供发现后迁移至数据实验室的功能。
+ 元数据采集模块：定时自动的从配置的数据库采集最新的结构。

## Web 显示端模块：

基于 SpringBoot 搭建，提供对应用、数据库、数据表、数据列的搜索及结果显示。

![主页](/images/projects/searchPlatform/1.png)

可以分级查看搜索到的结果和信息，提供用户管理、权限管理功能。

![数据库介绍](/images/projects/searchPlatform/WX20191130-192437.png)

## 搜索模块

基于 ElasticSearch 搭建，特别安装 ElasticSearch IK 中文分词方便检索。

使用 LogStash 每分钟同步 MySQL 中的表结构一次，这样搜索能力由 ES 提供，但在网页中的层级内容显示依然由 MySQL 提供。

**Tips：**

自动更新其实是由 LogStash 定时执行 SQL 语句，为了实现增量/修改/删除更新，为 MySQL 中的每一个表都添加了 updateTime 和 ifDelete 两个字段。其中：

+ updateTime：在 MySQL 设定为默认的该行得修改时间（CURRENT_TIMESTAMP），这样每次更新时定时拉取上一时间戳后的数据。
+ ifDelte：在 MySQL 中删除数据时不直接删除，而是调整该列。如果时间删除则无法在 ES 中同步删除。

ES 配置的自动更新 SQL 语句(仅事例)：

``` sql
SELECT db.id AS id, db.name AS name, ds.geshi AS columnType, db.`comment` AS `comment`, db.`collation` AS `collation`
        , db.ifnull AS ifnull, db.ifkey AS ifkey, db.extra AS extra, db.tableId AS tableId, ds.`description` AS `description`
        , db.evaluation AS evaluation,db.sourceappid as sourceappid, db.remark AS remark, ds.cnname AS sourcecnname, db.databaseId AS databaseId, db.deleteFlag AS deleteFlag
        , ds.update_time AS update_time, db.defaultValue AS defaultValue
FROM data_column db
        inner JOIN data_source ds ON db.evaluation = ds.sourceid and db.sourceappid=ds.sourceappid WHERE ds.update_time >= :sql_last_value
```

## 大数据门户迁移模块

与[邮储大数据平台](http://www.fuhuacn.top/projects/youchu/)项目相关联。

用户在发现一个要需求的数据表时，可以请求迁移这个数据表到自己的大数据实验室，通过大数据实验室获取相关表的数据，并在自身的数据实验室中进行数据分析等操作。

[数据订阅](/images/projects/searchPlatform/WX20191130-224408.png)

## 元数据采集

使用 JAVA 编写，设定每日 24 点用 crontab 自动执行一次，纯 JDBC 读取，扫描所配置的新增数据库/表/列结构信息（目前不支持表结构修改）。事例代码如下：

``` java
@Override
public void run() {

    HiveConnector _connector = new HiveConnector(connect.getUrl(), connect.getUser(), connect.getPassword());
    Connection _connection = _connector.getConnection();
    Statement _statement = _connector.getStatement(_connection);

    String _query = "show databases";
    List<String> _databases = GetColumnIndex.getColumnIndex1(_statement, _query);
    log.info("get " + _databases.size() + " databases");
    for (String _database : _databases) {

        int tempDatabaseId = createDatabase(_database, connect.getId());

        try {
            _statement.execute("use `" + _database + "`");
        } catch (SQLException e) {
            e.printStackTrace();
        }
        _query = "show tables";
        List<String> _tables = GetColumnIndex.getColumnIndex1(_statement, _query);
        for (String _table : _tables) {
            int tempTableId = createTable(_table, tempDatabaseId, _statement);
            _query = "desc `" + _table + "`";
            try {
                ResultSet columnRs = _statement.executeQuery(_query);
                while (columnRs.next()) {
                    String _column = columnRs.getString(1);
                    if (!_column.equals("") && _column != null && !_column.startsWith("#")) {
                        DataColumn tempColumn = column_dao.findByNameAndTableId(_column, tempTableId);
                        String _type = columnRs.getString(2);
                        if (tempColumn == null) {
                            tempColumn.setName(_column);
                            tempColumn.setColumnType(_type);
                            tempColumn.setComment(columnRs.getString(3));
                            tempColumn.setTableId(tempTableId);
                            tempColumn.setDeleteFlag(0);
                            column_dao.saveAndFlush(tempColumn);
                        } else {
                            log.info(_column + "已创建");
                        }
                    }
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        log.info("get " + _tables.size() + " tables in " + _database);
    }
    try {
        _statement.close();
        _connector.releaseConnection(_connection);
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

## 总结

这个项目把整个搜索、采集以及自动同步结构设定完后，就交给学弟学妹了，据反馈，后台系统跑的是十分稳定的。尤其是 ES 的搜索和 Logstash 的定时更新还是十分好用的。