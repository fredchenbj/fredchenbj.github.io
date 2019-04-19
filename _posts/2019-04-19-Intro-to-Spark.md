---
layout: post
title:  "Introduction to Spark"
date: 2019-04-19 19:06:26 +0800
categories: Spark
---


## Spark
本文从概述、历史、生态圈、应用场景和劣势等五个方面对 Spark 进行简要介绍。

<!--description-->

### 概述
> Apache Spark™ is a unified analytics engine for large-scale data processing.

Spark 是一个用于大规模数据处理的统一分析引擎。

- 速度：比 Map-Reduce 快 100x 多倍。Spark 使用先进的 DAG 调度器、查询优化器和物理执行引擎来实现高性能的批流处理。
- 易用：可使用 Java/Scala/Python/R/SQL 来快速实现应用。Spark 提供了超过 80 个高级操作来方便地构建并行应用。也可以使用 Scala、Python、R、SQL shell 来进行交互。
- 通用：包含 SQL、流、复杂分析。提供功能强大的库，如：用于 SQL 的 DataFrames、用于机器学习的 MLlib、图处理 Graphx 和 流处理，同时这些库可以无缝地集成在同一个应用里。
- 运行环境多样：可以借助 Hadoop YARN、Kubernetes、standalone 或 cloud 等进行资源调度，同时可访问多种数据源，如 HDFS、Alluxio、Cassandra、HBase、Hive 等 100 多种。

### 历史
- 2009 年，UC Berkeley's AMPLab 的 Matei Zaharia 开发了 Spark，并于 2010 年使用 BSD license 开源。
- 2013 年，Matei Zaharia 等人创立 Databricks 公司，进行 Spark 商业化开发运营。接着，将项目捐献给 ASF，更换为 Apache 2.0 license。
- 2014 年，成为 Apache 顶级项目。
- 版本：1.0（2014-05），2.0（2016-07），2.4（2018-11）。
- 2019 年，公司估值 27.5 亿美元。

### 生态圈
- Spark 团队在一开始设计 Spark 产品的时候，就遵循了一条原则：和 Hadoop 整个体系保持松耦合，所有的相关服务都抽象成接口，再通过接口调用。这样的好处是，Spark 和另一套类似的系统对接时，只需要针对另一套重新实现一下接口就可以了，Spark 自身的代码则完全不需要改动。
- 和 Hadoop 做到有效整合，借助业已壮大的 Hadoop 平台和生态圈推广自己。
- 可针对企业内部系统（比如微软、谷歌内部大数据系统）进行整合。
- 可针对云平台（如AWS、IBM 的云平台）进行整合。

### 应用场景
- 以 Spark Core 为核心，实现 "One Stack to Rule Them All"。
    - 批处理：Spark submit
    - 即席查询：Spark SQL
    - 实时流处理：Spark streaming
    - 机器学习：MLlib
    - 图处理：Graphx
    - 数学计算：SparkR

### 优劣势
- Databricks 对 Spark 开源社区的发展掌控非常强势，重大 feature 若没有 Databricks 的推动和首肯，几乎不可能进到 Spark 的发行版。
- Spark 是用 Scala 写的，并运行在 JVM 上。因此，基于 Spark 的机器学习平台并不能高效地利用 GPU。
- 在流处理模式方面弱于 Flink，并与 Flink 在大部分应用场景形成竞争。
