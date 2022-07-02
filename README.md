# 大数据技术学习路线、入门指南

> 大数据学习&资料整理&实战项目 by `Xu1Aan`  **欢迎加入，联系方式前往[首页](https://github.com/Xu1Aan)**

## 学习路线

### 大数据平台技术

大数据平台是为了计算，现今社会所产生的越来越大的 数据量，以存储、运算、展现作为目的的平台。大数据技术是指从各种各样类型的数据中，快速获得有价值信息的能力。适用于大数据技术的，有分布式数据库，云计算平台，互联网，和可扩展的存储系统。 

---

### 一、Hadoop

<div align=left><img src=".\picture\hadoop_logo.png" width="25%"/></div>

- [hadoop集群搭建](./01_Hadoop/hadoop.md)
- [HDFS分布式文件系统](./01_Hadoop/hdfs.md)
- [MapReduce和Yarn](./01_Hadoop/MapReduce&Yarn.md)
- [优化&新特性&HA](./01_Hadoop/Hadoop优化&新特性.md)



### 二、Zookeeper

<div align=left><img src=".\picture\Apache_ZooKeeper_logo.png" width="20%"/></div>

- [Zookeeper分布式协调框架](./02_Zookeeper/Zookeeper.md)



### 三、Hive

<div align=left><img src=".\picture\Hive_logo.png" width="23%"/></div>

- [Hive数据仓库工具](./03_Hive/Hive.md)



### 四、Flume

<div align=left><img src=".\picture\flume-logo.png" width="150px" height="150px"/></div>

- [Flume海量日志采集](./04_Flume/Flume.md)



### 五、Kaflka

<div align=left><img src=".\picture\kafka-logo.jpg" width="20%"/></div>

- [Kaflka消息队列](./05_Kaflka/Kaflka.md)



### 六、HBase

<div align=left><img src=".\picture\hbase_logo_with_orca_large.png" width="25%"/></div>

- [HBase列式存储分布式数据库](./06_HBase)



- [ ] Scala
- [ ] Spark
- [ ] Flink

### 大数据平台技术（练习）

- [x] 数仓采集
  - [电商数仓采集项目（用户行为数据采集）](./10_Porject/01_数仓采集/数仓采集(用户行为数据仓库).md)
  - [电商数仓采集项目（业务数据采集）](./10_Porject/01_数仓采集/数仓采集(系统业务数据仓库).md)
- [ ] 电商数仓
- [ ] 实时数仓

---

### **技术分类**

> 上面我们介绍了很多大数据框架，分类总结如下：

**日志收集框架**：Flume、Logstash、Filebeat

**分布式文件存储系统**：Hadoop HDFS

**数据库系统**：Mongodb、HBase

**分布式计算框架**：

+ 批处理框架：Hadoop MapReduce
+ 流处理框架：Storm
+ 混合处理框架：Spark、Flink

**查询分析框架**：Hive 、Spark SQL 、Flink SQL、 Pig、Phoenix 

**集群资源管理器**：Hadoop YARN

**分布式协调服务**：Zookeeper

**数据迁移工具**：Sqoop

**任务调度框架**：Azkaban、Oozie

**集群部署和监控**：Ambari、Cloudera Manager

---

### 机器学习+大数据技术项目（推荐系统、用户画像）

> 欢迎加入...

- [ ] 工作招聘推荐系统（计划）

  预计开发技术：Hadoop、Spark、SpringBoot、echarts、PySpark、Python、MySQL、Vue

  > 推荐系统框架图：

  <img src=".\01_Hadoop\picture\推荐系统框架图.png" style="zoom:25%;" />

- [ ] 待续...

