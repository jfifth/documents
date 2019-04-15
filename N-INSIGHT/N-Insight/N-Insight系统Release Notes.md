# N-Insight系统Release Notes

版本 1.0

目录

[TOC]



# 修订历史

| **版本编号** | **变化状态** | **简要说明**   **（变更内容和变更范围）** | **日期**   | **变更人** |      |
| ------------ | ------------ | ----------------------------------------- | ---------- | ---------- | ---- |
| 1.0          | C            | 新增Release   Notes文档                   | 2019-03-21 | 张保全     |      |
|              |              |                                           |            |            |      |

# 1.功能概述

系统结构图：

![1555312581635](./assets/1555312581635.png)

# 2.系统主要特性

​     N-Insight使用Java语言开发，所有的模块基于OSGI插件化，可根据项目需要选择不同的插件组合。目前支持Oracle、Greenplum、Spark计算模型，专题可以基于三种计算模型进行指标计算。专题也可以插件化，并且支持热插拔。  

​    在N-Insight中可以周期、定时、消息等方式调度专题。专题计算完成后会把结果数据根据需要写入到HDFS/Hive或者关系库。

# 3.模块构成

##  3.1N-Insight系统由以下模块构成：

`com.nsn.configurator：`配置文件管理插件。

com.nsn.datamining：N-Insight集群化基础功能组件插件。

com.nsn.datamining.mysql：Mysql计算引擎插件。

com.nsn.datamining.spark：Spark计算引擎插件。

com.nsn.datamining.support.xdr.cmcc：中国移动XDR数据源基础插件。

com.nsn.datamining.support.xdr.ctc：中国电信XDR数据源基础插件。

com.nsn.datamining.support.xdr.cuc：中国联通XDR数据源基础插件。

com.nsn.datamining.support.xdr.normal：通用(统一版本)XDR数据源基础插件。

com.nsn.do.cluster：N-Insight集群化服务启动插件。

com.nsn.executer：多线程工具插件。

com.nsn.io：I/O工具插件

com.nsn.logger：日志工具插件。

com.nsn.messages：MQ消息队列工具插件

com.nsn.scanner：(ftp、操作系统、hdfs等)文件、kafka数据扫描插件。

com.nsn.scheduler：任务调度插件。

com.nsn.util：常用工具类插件。

com.nsn.web：基于jetty的Web服务插件。

com.nsn.web.configurator：管理配置文件的web服务插件。

com.nsn.web.do：Web界面框架插件。

com.nsn.web.do.login：Web登陆管理插件。

com.nsn.web.do.nologin：Web免登陆插件。

com.nsn.web.do.tbox：N-Insight Web服务插件。

com.nsn.web.do.tbox.shell：shell调度插件。

## 3.2 模块间交互及整体的数据流向

![](./assets/11.png)

# 4.    更新说明

## 4.1    xml中添加代码片功能

在指标计算的项目中，经常会有相同的指标出现在多个降维组合中，这样指标就会重复出现，一旦某个指标需要修改，需要重复修改多个维度中的同一个指标.

## 4.2    result节点支持输出到多个库

支持把计算结果输出到多种类型的库。

## 4.3    hive库表自动创建功能

原有入库方式入hive库时，hive创建表ddl脚本需要手动维护，一旦新增字段或调整字段顺序还需额外维护ddl脚本，修改分库以后，HiveStreamSource的save方法可以判断hive表是否存在，如果不存在则自动创建。并且会自动添加p_hour的分区。

## 4.4    专题执行结束后的清理功能

每个专题在计算过程中会产生很多中间结果，临时存放在数据库中或hdfs上，如需要导入到关系库的数据、parquet缓存文件、临时表、广播数据等，一旦专题执行结束后，这些中间数据将不再需要，如果不进行清理，将越积越多。之前需要人工手动清理，目前在StreamSource中添加了cleanup方法，具体的实现中根据实际需要可以做清理。

## 4.5    在专题的xml添加source节点中添加对关系库的支持

在Spark计算过程中添加读取关系库数据源功能。

## 4.6    Shell 调度功能

目前N-Insight仅支持Oracle、GP、Spark做为计算引擎。很多项目上因历史因素或基它特殊原因，不适合使用NInsight的开发方式进行调度，所以在N-Insight中添加了Shell 调度功能。可以把计算逻辑通过shell实现，然后再使用 N-Insight进行调度。

## 4.7    专题插件热插拔

N-Insight在生产过程中如果对插件进行修改，需要重新启动N-Insight。重启操作会影响到其他专题的生产。热插拔 功能功可以新增或更新插件而不需要重新启动N-Insight。

## 4.8    算法xml加密功能

部分项目中业务算法是我们业务专家辛苦设计出来的，保密级别较高。并且很多项目的大数据平台不归我们自己管 理，为了保证算法不被泄漏，N-Insight添加了算法加密功能。