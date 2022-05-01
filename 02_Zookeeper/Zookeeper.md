# Zookeeper 

## 1  Zookeeper入门

### 1.1 概述

Zookeeper是一个开源的分布式的，为分布式应用提供协调服务的Apache项目。

Zookeeper从设计模式角度来理解，是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生了变化，Zookeeper就负责通知已经在Zookeeper上注册的那些观察者做出相应的反应.

**Zookeeper = 文件系统 + 通知机制**

Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

![](.\picture\Zookeeper工作机制.png)

### 1.2 特点

![](.\picture\Zookeeper特点.png)

1）Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。

2）集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。

3）全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。

4）更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。

5）数据更新原子性，一次数据更新要么成功，要么失败。

6）实时性，在一定时间范围内，Client能读到最新数据。

### 1.3 数据结构

ZooKeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树，每个节点称做一个ZNode。每一个ZNode默认能够存储1MB的数据，每个ZNode都可以通过其路径唯一标识。

![](.\picture\zookeeper数据结构.png)

### 1.4 应用场景

提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等。

**统一命名服务**

![](.\picture\统一命名服务.png)

**统一配置管理**

1）分布式环境下，配置文件同步非常常见。

​	（1）一般要求一个集群中，所有节点的配置信息是一致的，比如 Kafka 集群。

​	（2）对配置文件修改后，希望能够快速同步到各个节点上。

2）配置管理可交由ZooKeeper实现。

​	（1）可将配置信息写入ZooKeeper上的一个Znode。

​	（2）各个客户端服务器监听这个Znode。

​	（3）一旦Znode中的数据被修改，ZooKeeper将通知各个客户端服务器。

<img src=".\picture\统一配置管理.png" style="zoom:22%;" />

**统一集群管理**

1）分布式环境中，实时掌握每个节点的状态是必要的。

 	（1）可根据节点实时状态做出一些调整。

2）ZooKeeper可以实现实时监控节点状态变化

​	（1）可将节点信息写入ZooKeeper上的一个ZNode。

​	（2）监听这个ZNode可获取它的实时状态变化。

<img src=".\picture\统一集群管理.png" style="zoom: 33%;" />

**服务器动态上下线**

![](.\picture\服务器动态上下线.png)

**软负载均衡**

在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求

<img src="E:\learning\04_java\01_笔记\BigData\02_Zookeeper\picture\软负载均衡.png" style="zoom:50%;" />

### 1.5 下载地址

**1）官网首页：**

https://zookeeper.apache.org/

---

## 2 Zookeeper安装

### 2.1 本地模式安装部署

**1）安装前准备**

（1）安装Jdk

（2）拷贝Zookeeper安装包到Linux系统下

（3）解压到指定目录
