# HBase

## 1 HBase简介

### 1.1 HBase定义

HBase是一种分布式、可扩展、支持海量数据存储的NoSQL数据库。

### 1.2 HBase数据模型

逻辑上，HBase的数据模型同关系型数据库很类似，数据存储在一张表中，有行有列。但从HBase的底层物理存储结构（K-V）来看，HBase更像是一个multi-dimensional map。

#### 1.2.1 HBase逻辑结构

![](.\picture\HBase 逻辑结构3.png)

#### 1.2.2 HBase物理存储结构

![](.\picture\HBase 物理存储结构.png)

#### 1.2.3 数据模型

**1）Name Space**

命名空间，类似于关系型数据库的database概念，每个命名空间下有多个表。HBase有两个自带的命名空间，分别是hbase和default，hbase中存放的是HBase内置的表，default表是用户默认使用的命名空间。

**2）Table**

类似于关系型数据库的表概念。不同的是，HBase定义表时只需要声明列族即可，不需要声明具体的列。这意味着，往HBase写入数据时，字段可以动态、按需指定。因此，和关系型数据库相比，HBase能够轻松应对字段变更的场景。

**3）Row**

HBase表中的每行数据都由一个**RowKey**和多个**Column**（列）组成，数据是按照RowKey的字典顺序存储的，并且查询数据时只能根据RowKey进行检索，所以RowKey的设计十分重要。

**4）Column**

HBase中的每个列都由Column Family(列族)和Column Qualifier（列限定符）进行限定，例如info：name，info：age。建表时，只需指明列族，而列限定符无需预先定义。

**5）Time Stamp**

用于标识数据的不同版本（version），每条数据写入时，系统会自动为其加上该字段，其值为写入HBase的时间。

**6）Cell**

由{rowkey, column Family：column Qualifier, time Stamp} 唯一确定的单元。cell中的数据全部是字节码形式存贮。

### 1.3 HBase基本架构

![](.\picture\HBase 架构（不完整版）.png)

**架构角色：**

**1）Region Server**

Region Server为 Region的管理者，其实现类为HRegionServer，主要作用如下:

对于数据的操作：get, put, delete；

对于Region的操作：splitRegion、compactRegion。

**2）Master**

Master是所有Region Server的管理者，其实现类为HMaster，主要作用如下：

  对于表的操作：create, delete, alter

对于RegionServer的操作：分配regions到每个RegionServer，监控每个RegionServer的状态，负载均衡和故障转移。

**3）Zookeeper**

HBase通过Zookeeper来做master的高可用、RegionServer的监控、元数据的入口以及集群配置的维护等工作。

**4）HDFS**

HDFS为Hbase提供最终的底层数据存储服务，同时为HBase提供高可用的支持。

## 2 HBase快速入门

### 2.1 HBase安装部署

#### 2.1.1 Zookeeper正常部署

首先保证Zookeeper集群的正常部署，并启动之：

```
[xu1an@hadoop201 zookeeper-3.5.7]$ bin/zkServer.sh start
[xu1an@hadoop202 zookeeper-3.5.7]$ bin/zkServer.sh start
[xu1an@hadoop203 zookeeper-3.5.7]$ bin/zkServer.sh start
```

#### 2.1.2 Hadoop正常部署

Hadoop集群的正常部署并启动：

```
[xu1an@hadoop201 hadoop-3.1.3]$ sbin/start-dfs.sh
[xu1an@hadoop202 hadoop-3.1.3]$ sbin/start-yarn.sh
```

#### 2.1.3 HBase的解压

解压Hbase到指定目录：

```
[xu1an@hadoop201 software]$ tar -zxvf hbase-2.0.5-bin.tar.gz -C /opt/module
[xu1an@hadoop201 software]$ mv /opt/module/hbase-2.0.5 /opt/module/hbase
```

配置环境变量

```
[xu1an@hadoop201 ~]$ sudo vim /etc/profile.d/my_env.sh
添加
#HBASE_HOME
export HBASE_HOME=/opt/module/hbase
export PATH=$PATH:$HBASE_HOME/bin
```

#### 2.1.4 HBase的配置文件

修改HBase对应的配置文件。

1.hbase-env.sh修改内容：

```
export HBASE_MANAGES_ZK=false
```

2.hbase-site.xml修改内容：

```
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://hadoop201:8020/hbase</value>
    </property>

    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>

    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>hadoop201,hadoop202,hadoop203</value>
    </property>
</configuration>
```

3.regionservers：

```
hadoop201
hadoop202
hadoop203
```

#### 2.1.5 HBase远程发送到其他集群

```
[xu1an@hadoop201 module]$ xsync hbase/
```

#### 2.1.6 HBase服务的启动

1.单点启动

```
[xu1an@hadoop201 hbase]$ bin/hbase-daemon.sh start master
[xu1an@hadoop201 hbase]$ bin/hbase-daemon.sh start regionserver
```

提示：如果集群之间的节点时间不同步，会导致regionserver无法启动，抛出ClockOutOfSyncException异常。

修复提示：

a、同步时间服务

请参看帮助文档：《尚硅谷大数据技术之Hadoop入门》

b、属性：hbase.master.maxclockskew设置更大的值

```
<property>
        <name>hbase.master.maxclockskew</name>
        <value>180000</value>
        <description>Time difference of regionserver from master</description>
</property>
```

2.群启

```
[xu1an@hadoop201 hbase]$ bin/start-hbase.sh
```

对应的停止服务：

```
[xu1an@hadoop201 hbase]$ bin/stop-hbase.sh
```

#### 2.1.7 查看HBase页面

启动成功后，可以通过“host:port”的方式来访问HBase管理页面，例如：

[http://hadoop201:16010](http://linux01:16010/) 

#### 2.1.8 高可用(可选)

在HBase中HMaster负责监控HRegionServer的生命周期，均衡RegionServer的负载，如果HMaster挂掉了，那么整个HBase集群将陷入不健康的状态，并且此时的工作状态并不会维持太久。所以HBase支持对HMaster的高可用配置。

1.关闭HBase集群（如果没有开启则跳过此步）

```
[xu1an@hadoop201 hbase]$ bin/stop-hbase.sh
```

2.在conf目录下创建backup-masters文件

```
[xu1an@hadoop201 hbase]$ touch conf/backup-masters
```

3.在backup-masters文件中配置高可用HMaster节点

```
[xu1an@hadoop201 hbase]$ echo hadoop202 > conf/backup-masters
```

4.将整个conf目录scp到其他节点

```
[xu1an@hadoop201 hbase]$ scp -r conf/ hadoop202:/opt/module/hbase/
[xu1an@hadoop201 hbase]$ scp -r conf/ hadoop203:/opt/module/hbase/
```

5.打开页面测试查看

[http://hadooo102:16010](http://linux01:16010/) 

### 2.2 HBase Shell操作

#### 2.2.1 基本操作

**1．进入HBase客户端命令行**

```
[xu1an@hadoop201 hbase]$ bin/hbase shell
```

**2．查看帮助命令**

```
hbase(main):001:0> help
```

#### 2.2.2 namespace的操作

**1．查看当前Hbase中有哪些namespace**

```
hbase(main):002:0> list_namespace
NAMESPACE                                                                                        
default(创建表时未指定命名空间的话默认在default下)                     
hbase(系统使用的，用来存放系统相关的元数据信息等，勿随便操作)  
```

**2．创建namespace**

```
hbase(main):010:0>  create_namespace "test"
hbase(main):010:0> create_namespace "test01", {"author"=>"wyh", "create_time"=>"2020-03-10 08:08:08"}
```

**3.查看namespace**

```
hbase(main):010:0>  describe_namespace "test01"
```

**4 .修改namespace的信息（添加或者修改属性）**

```
hbase(main):010:0> alter_namespace "test01", {METHOD => 'set', 'author' => 'weiyunhui'}
```

**添加或者修改属性:**

alter_namespace 'ns1', {METHOD => 'set', 'PROPERTY_NAME' => 'PROPERTY_VALUE'} 

**删除属性:**     

alter_namespace 'ns1', {METHOD => 'unset', NAME => ' PROPERTY_NAME '} 

**5.删除namespace**

```
hbase(main):010:0> drop_namespace "test01"
```

注意: 要删除的namespace必须是空的，其下没有表。

#### 2.2.3 表的操作

**0．查看当前数据库中有哪些表**

```
hbase(main):002:0> list
```

**1．创建表**

```
hbase(main):002:0> create 'student','info'
```

**2．插入数据到表**

```
hbase(main):003:0> put 'student','1001','info:sex','male'
hbase(main):004:0> put 'student','1001','info:age','18'
hbase(main):005:0> put 'student','1002','info:name','Janna'
hbase(main):006:0> put 'student','1002','info:sex','female'
hbase(main):007:0> put 'student','1002','info:age','20'
```

**3．扫描查看表数据**

```
hbase(main):008:0> scan 'student'
hbase(main):009:0> scan 'student',{STARTROW => '1001', STOPROW  => '1001'}
hbase(main):010:0> scan 'student',{STARTROW => '1001'}
```

**4．查看表结构**

```shell
hbase(main):011:0> describe 'student'
```

或者直接使用 desc

**5．更新指定字段的数据**

```
hbase(main):012:0> put 'student','1001','info:name','Nick'
hbase(main):013:0> put 'student','1001','info:age','100'
```

**6．查看“指定行”或“指定列族:列”的数据**

```
hbase(main):014:0> get 'student','1001'
hbase(main):015:0> get 'student','1001','info:name'
```

**7．统计表数据行数**

```
hbase(main):021:0> count 'student'
```

**8．删除数据**

删除某rowkey的全部数据：

```
hbase(main):016:0> deleteall 'student','1001'
```

删除某rowkey的某一列数据：

```
hbase(main):017:0> delete 'student','1002','info:sex'
```

**9．清空表数据**

```
hbase(main):018:0> truncate 'student'
```

提示：清空表的操作顺序为先disable，然后再truncate。

**10．删除表**

首先需要先让该表为disable状态：

```
hbase(main):019:0> disable 'student'
```

然后才能drop这个表：

```
hbase(main):020:0> drop 'student'
```

提示：如果直接drop表，会报错：ERROR: Table student is enabled. Disable it first.

**11．变更表信息**

```
hbase(main):022:0> alter 'student',{NAME=>'info',VERSIONS=>3}
hbase(main):022:0> get 'student','1001',{COLUMN=>'info:name',VERSIONS=>3}
```

## 3 HBase进阶

### 3.1 RegionServer 架构

![](.\picture\RegionServer 详细架构.png)

**1）StoreFile**

保存实际数据的物理文件，StoreFile以Hfile的形式存储在HDFS上。每个Store会有一个或多个StoreFile（HFile），数据在每个StoreFile中都是有序的。

**2）MemStore**

写缓存，由于HFile中的数据要求是有序的，所以数据是先存储在MemStore中，排好序后，等到达刷写时机才会刷写到HFile，每次刷写都会形成一个新的HFile。

**3）WAL**

由于数据要经MemStore排序后才能刷写到HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做Write-Ahead logfile的文件中，然后再写入MemStore中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。

**4）BlockCache**

读缓存，每次查询出的数据会缓存在BlockCache中，方便下次查询。

### 3.2 写流程

![](.\picture\HBase 写流程.png)

写流程：

1）Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。

2）访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。

3）与目标Region Server进行通讯；

4）将数据顺序写入（追加）到WAL；

5）将数据写入对应的MemStore，数据会在MemStore进行排序；

6）向客户端发送ack；

7）等达到MemStore的刷写时机后，将数据刷写到HFile。

### 3.3 MemStore Flush

<img src=".\picture\MemStore Flush.png" style="zoom:33%;" />

**MemStore刷写时机：**

1.当某个memstore的大小达到了**hbase.hregion.memstore.flush.size（默认值128M）**，其所在region的所有memstore都会刷写。

当memstore的大小达到了

**hbase.hregion.memstore.flush.size（默认值128M）**

**hbase.hregion.memstore.block.multiplier（默认值4）**

时，会阻止继续往该memstore写数据。

2.当region server中memstore的总大小达到

**java_heapsize**

**hbase.regionserver.global.memstore.size（默认值0.4）**

**hbase.regionserver.global.memstore.size.lower.limit（默认值0.95）**，

region会按照其所有memstore的大小顺序（由大到小）依次进行刷写。直到region server中所有memstore的总大小减小到上述值以下。

当region server中memstore的总大小达到

**java_heapsize**

**hbase.regionserver.global.memstore.size（默认值0.4）**

时，会阻止继续往所有的memstore写数据。

3.到达自动刷写的时间，也会触发memstore flush。自动刷新的时间间隔由该属性进行配置**hbase.regionserver.optionalcacheflushinterval（默认1小时）**。

4.当WAL文件的数量超过**hbase.regionserver.max.logs**，region会按照时间顺序依次进行刷写，直到WAL文件数量减小到**hbase.regionserver.max.logs**以下（该属性名已经废弃，现无需手动设置，最大值为32）。

### 3.4 读流程

1）整体流程

![](.\picture\Hbase 读流程（1）.png)

2）Merge细节

![](.\picture\Hbase 读流程（2）.png)

**读流程**

1）Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。

2）访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。

3）与目标Region Server进行通讯；

4）分别在MemStore和Store File（HFile）中查询目标数据，并将查到的所有数据进行合并。此处所有数据是指同一条数据的不同版本（time stamp）或者不同的类型（Put/Delete）。

5）将查询到的新的数据块（Block，HFile数据存储单元，默认大小为64KB）缓存到Block Cache。

6）将合并后的最终结果返回给客户端。

### 3.5 StoreFile Compaction

由于memstore每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp）和不同类型（Put/Delete）有可能会分布在不同的HFile中，因此查询时需要遍历所有的HFile。为了减少HFile的个数，以及清理掉过期和删除的数据，会进行StoreFile Compaction。

Compaction分为两种，分别是Minor Compaction和Major Compaction。Minor Compaction会将临近的若干个较小的HFile合并成一个较大的HFile，并清理掉部分过期和删除的数据。Major Compaction会将一个Store下的所有的HFile合并成一个大HFile，并且**会清理掉所有过期和删除的数据。**

![](.\picture\StoreFile Compaction.png)

### 3.6 Region Split

默认情况下，每个Table起初只有一个Region，随着数据的不断写入，Region会自动进行拆分。刚拆分时，两个子Region都位于当前的Region Server，但处于负载均衡的考虑，HMaster有可能会将某个Region转移给其他的Region Server。

Region Split时机：

1.当1个region中的某个Store下所有StoreFile的总大小超过hbase.hregion.max.filesize，该Region就会进行拆分（0.94版本之前）。

2.当1个region中的某个Store下所有StoreFile的总大小超过Min(initialSize*R^3 ,hbase.hregion.max.filesize")，该Region就会进行拆分。其中initialSize的默认值为2*hbase.hregion.memstore.flush.size，R为当前Region Server中属于该Table的Region个数（0.94版本之后）。

具体的切分策略为：

第一次split：1^3 * 256 = 256MB 

第二次split：2^3 * 256 = 2048MB 

第三次split：3^3 * 256 = 6912MB 

第四次split：4^3 * 256 = 16384MB > 10GB，因此取较小的值10GB 

后面每次split的size都是10GB了。

3.Hbase 2.0引入了新的split策略：如果当前RegionServer上该表只有一个Region，按照2 * hbase.hregion.memstore.flush.size分裂，否则按照hbase.hregion.max.filesize分裂。

![](.\picture\Region Split.png)

## 4 HBase API 

### 4.1 环境准备

新建项目后在pom.xml中添加依赖