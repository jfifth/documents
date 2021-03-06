# N-Streaming系统Release Notes

[TOC]

# 1.    功能概述

系统主要功能：

N-Streaming基于N-Insight运行，目前已完成VoLTE、IoT、路路通等项目统一版本的开发。后续根据实际需求，还会加入其他统一版本的N-Streaming项目。

系统结构图：

![1555378686630](assets/1555378686630.png)

# 2.    模块构成

N-Streaming安装包主要包涵三种类型：项目数据适配模块、指标计算模块、N-Insight Web列表模块。

- **项目数据适配模块**：指标计算模块完成后会部署到不同省份，每个省份对集团的xdr规范执行程度及xdr规范版本不一致，甚至同一个N-Streaming项目会部署到不同的运营商。项目数据适配模块的作用就是把不同运营商、不同xdr版本的数据进行格式化。把不同的xdr数据转换后的标准数据输出给指标计算模块。该模块的名称com.nsn.datamining.support.xdr.(xdr运营商).(N-Streadming名称).(项目所在地)，如河南移动VoLTE项目名称则为：`com.nsn.datamining.support.xdr.normal.volte.hncmcc`。该模块在每个项目上需要独立开发。
- **指标计算模块：**指标计算模块是N-Streaming的核心模块，主要用来实现算法。对于相同集团规范版本的xdr可以通过适配模块解决，但不同xdr版本的字段数量和指标均有不同。所以在指标计算模块需要针对不同版本的xdr分开开发，通过配置的方式对不同的xdr版本使用对应的开发版本。该模块的名称格式：com.nsn.do.tbox.spark.(子模块称称).(时间维度)，如VoLTE项目的VoLTE模块小时维度则为：`com.nsn.do.tbox.spark.volte.hour`。
- **N-Insight Web列表模块**：该模块主要用于把N-Streaming相关功能注册到N-Insight。能够进一步在N-Insight中展示功能列表并进行调度。该模块的名称格式：com.nsn.web.do.tbox.spark.(子模块乐称).(时间维度)，如VoLTE项目的VoLTE模块小时维度则为：`com.nsn.web.do.tbox.spark.volte.hour`。

# 3.    各个模块介绍

## 3.1    N-Streaming模块

VoLTE项目标准版有两个版本：Spark大数据版、KPI的java版本。有单接口、多接口、CSFB、分片时延和落地方案共5个子模块。按照时间维度有15分钟级、小时级、天级、周级。目前已在河南移动(大数据版)、福建移动(KPI版)、北京移动(KPI版)项目部署。

IoT项目根据2/4G XDR接口数据过滤出IoT相关数据，对过滤后的数据进行指标计算并展示。过滤规则根据项目上的数据Apn的数据整理出非IoT的Apn作为黑名单，然后再使用黑名单对原始数据进行过滤，对过滤出的IoT数据进行相关的计算。目前已在安徽电信、江苏移动、浙江移动(未果)、上海移动、河北移动部署。

路路通项目是截至目前算法最复杂的一个项目。主要是针对http和mme XDR数据对城市道路、高速公路及高速铁路沿线的用户、小区进行计算。对用户进行感知计算，对小区进行模拟路测。目前已在浙江移动、北京移动部署。

# 4.    更新说明

N-Streaming基于N-Insight，模块开发完成后指标计算模块和N-Insight Web列表模块会被统一，无论在哪个项目上都会使用统一版本。只有项目数据适配模块在每个项目上会有新增的开发。