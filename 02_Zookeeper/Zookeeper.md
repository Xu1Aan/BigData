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

数据结构
	   ZK中的数据保存的格式（树状结构）
	   注意：ZK中没有文件的概念，节点下直接存的就是内容。

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

​	（1）可根据节点实时状态做出一些调整。

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

安装流程

1. **安装ZK:**
   ① 把软件包上传的Linux的 /opt/software 下
   ② 加压ZK到 /opt/module 下
   ③ 将加压后的目录名称修改一下（选做）
   ④ 将zk的安装目录下 conf/zoo_sample.cfg 文件改名为 zoo.cfg
   ⑤ 在ZK的安装目录下创建一个新的目录，作为zk的数据持久化目录
   ⑤ 修改zoo.cfg配置文件 
       dataDir=/opt/module/zookeeper-3.5.7/zkData		   
  ⑥ 配置ZK的环境变量 （选做）
   
2. **单点模式的简单操作**
   ① 启停zk服务端 和 zk客户端
     zkServer.sh start    zkCli.sh -server host:port
   ② 查看一下zk的服务端和客户端对应的进程
      QuorumPeerMain --> 服务端
      ZooKeeperMain  --> 客户端
	
   ③ 退出客户端
      quit

### 2.1 本地模式安装部署

**1）安装前准备**

（1）安装Jdk

（2）拷贝Zookeeper安装包到Linux系统下

（3）解压到指定目录/opt/module 下

```shell
tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module/
```

**2）配置修改**

（1）将/opt/module/zookeeper-3.5.7/conf这个路径下的zoo_sample.cfg修改为zoo.cfg；

```shell
mv zoo_sample.cfg zoo.cfg
```

（2）打开zoo.cfg文件，修改dataDir路径：

```shell
 vim zoo.cfg
```

修改如下内容：

```
dataDir=/opt/module/zookeeper-3.5.7/zkData
```

 （3）在/opt/module/zookeeper-3.5.7/这个目录上创建zkData文件夹

```shell
mkdir zkData
```

**3）操作Zookeeper**

（1）启动Zookeeper

```shell
bin/zkServer.sh start
```

（2）查看进程是否启动

```shell
jps
```

（3）查看状态：

```shell
bin/zkServer.sh status

ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: standalone
```

（4）启动客户端：

```shell
bin/zkCli.sh
```

（5）退出客户端：

```shell
[zk: localhost:2181(CONNECTED) 0] quit
```

（6）停止Zookeeper

```shell
bin/zkServer.sh stop
```

### 2.2 配置参数解读

Zookeeper中的配置文件zoo.cfg中参数含义解读如下：

**1）tickTime =2000：通信心跳数，Zookeeper服务器与客户端心跳时间，单位毫秒**

Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。

它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2*tickTime)

**2）initLimit =10：LF初始通信时限**

集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。

**3）syncLimit =5：LF同步通信时限**

集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。

**4）dataDir：数据文件目录+数据持久化路径**

主要用于保存Zookeeper中的数据。

**5）clientPort =2181：客户端连接端口**

监听客户端连接的端口。

---

## 3 Zookeeper实战

**搭建流程：**

注意事项：如果不是第一次搭建集群，那么就把zk安装目录下的zkData目录
	      删除，并且把logs目录也删除

	1. 在ZK的安装目录下创建 zkData 
	
	2. 修改zoo.cfg 配置文件
	   -- 
	     dataDir=/opt/module/zookeeper-3.5.7/zkData
	   -- 
	     server.2=hadoop102:2888:3888
		 server.3=hadoop103:2888:3888
	     server.4=hadoop104:2888:3888
		 
	3. 在ZK的安装目录下的zkData目录中创建一个myid的文件
	   用于标记当前服务器的编号
	   hadoop102 --> 2
	   hadoop103 --> 3
	   hadoop104 --> 4
	   
	5. 将hadoop02的整个ZK的安装目录分发到其他机器
	
	6. 在不同机器上修改myid文件中的值 

### 3.1 分布式安装部署

**1）集群规划**

在hadoop102、hadoop103和hadoop104三个节点上部署Zookeeper。

**2）解压安装**

（1）解压Zookeeper安装包到/opt/module/目录下

```shell
 tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module/
```

（2）同步/opt/module/zookeeper-3.5.7目录内容到hadoop103、hadoop104

```shell
my_rsync zookeeper-3.5.7/
```

**3）配置服务器编号**

（1）在/opt/module/zookeeper-3.5.7/这个目录下创建zkData

```shell
mkdir -p zkData
```

（2）在/opt/module/zookeeper-3.5.7/zkData目录下创建一个myid的文件

```shell
touch myid
```

添加myid文件，注意一定要在linux里面创建，在notepad++里面很可能乱码

（3）编辑myid文件

```shell
vi myid
```

在文件中添加与server对应的编号：2

（4）拷贝配置好的zookeeper到其他机器上

```shell
my_rsync myid
```

并分别在hadoop103、hadoop104上修改myid文件中内容为3、4

**4）配置zoo.cfg文件**

（1）重命名/opt/module/zookeeper-3.5.7/conf这个目录下的zoo_sample.cfg为zoo.cfg

```shell
mv zoo_sample.cfg zoo.cfg
```

（2）打开zoo.cfg文件

```
vim zoo.cfg
```

修改数据存储路径配置

```
dataDir=/opt/module/zookeeper-3.5.7/zkData
```

增加如下配置

```
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
```

（3）同步zoo.cfg配置文件

```shell
my_rsync zoo.cfg
```

（4）配置参数解读

server.A=B:C:D。

**A**是一个数字，表示这个是第几号服务器；

集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。

**B**是这个服务器的地址；

**C**是这个服务器Follower与集群中的Leader服务器交换信息的端口；

**D**是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

**5）集群操作**

（1）分别启动Zookeeper

```shell
[hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh start
[hadoop103 zookeeper-3.5.7]$ bin/zkServer.sh start
[hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh start
```

（2）查看状态

```
[xu1an@hadoop102 zookeeper-3.5.7]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: follower
[xu1an@hadoop103 zookeeper-3.5.7]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: leader
[xu1an@hadoop104 zookeeper-3.5.7]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: follower
```

### 3.2 客户端命令行操作

| 命令基本语法 | 功能描述                                                     |
| ------------ | ------------------------------------------------------------ |
| help         | 显示所有操作命令                                             |
| ls path      | 使用 ls 命令来查看当前znode的子节点  -w 监听子节点变化  -s  附加次级信息 |
| create       | 普通创建  -s 含有序列  -e 临时（重启或者超时消失）           |
| get path     | 获得节点的值  -w 监听节点内容变化  -s  附加次级信息          |
| set          | 设置节点的具体值                                             |
| stat         | 查看节点状态                                                 |
| delete       | 删除节点                                                     |
| deleteall    | 递归删除节点                                                 |

**1）启动客户端**

```shell
bin/zkCli.sh
```

**2）显示所有操作命令**

```
help
```

**3）查看当前znode中所包含的内容**

```shell
[zk: localhost:2181(CONNECTED) 1] ls /

[zookeeper]
```

**4）查看当前节点详细数据**

```
[zk: localhost:2181(CONNECTED) 1] ls -s /
[zookeeper]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

**5）分别创建2个普通节点**

```
[zk: localhost:2181(CONNECTED) 3] create /sanguo "diaochan"
Created /sanguo
[zk: localhost:2181(CONNECTED) 4] create /sanguo/shuguo "liubei"
Created /sanguo/shuguo
```

**6）获得节点的值**

```
[zk: localhost:2181(CONNECTED) 5] get /sanguo
diaochan
[zk: localhost:2181(CONNECTED) 6] get -s /sanguo
diaochan
cZxd = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000003
mtime = Wed Aug 29 00:03:23 CST 2018
pZxid = 0x100000004
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 1
[zk: localhost:2181(CONNECTED) 7]
[zk: localhost:2181(CONNECTED) 7] get -s /sanguo/shuguo
liubei
cZxid = 0x100000004
ctime = Wed Aug 29 00:04:35 CST 2018
mZxid = 0x100000004
mtime = Wed Aug 29 00:04:35 CST 2018
pZxid = 0x100000004
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

**7）创建临时节点**

```
[zk: localhost:2181(CONNECTED) 7] create -e /sanguo/wuguo "zhouyu"
Created /sanguo/wuguo
```

（1）在当前客户端是能查看到的

```
[zk: localhost:2181(CONNECTED) 3] ls /sanguo 
[wuguo, shuguo]
```

（2）退出当前客户端然后再重启客户端

```
[zk: localhost:2181(CONNECTED) 12] quit
[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkCli.sh
```

（3）再次查看根目录下短暂节点已经删除

```
[zk: localhost:2181(CONNECTED) 0] ls /sanguo
[shuguo]
```

**8）创建带序号的节点**

​    （1）先创建一个普通的根节点/sanguo/weiguo

```
[zk: localhost:2181(CONNECTED) 1] create /sanguo/weiguo "caocao"
Created /sanguo/weiguo
```

​    （2）创建带序号的节点

```
[zk: localhost:2181(CONNECTED) 2] create /sanguo/weiguo "caocao"
Node already exists: /sanguo/weiguo
[zk: localhost:2181(CONNECTED) 3] create -s /sanguo/weiguo "caocao"
Created /sanguo/weiguo0000000000
[zk: localhost:2181(CONNECTED) 4] create -s /sanguo/weiguo "caocao"
Created /sanguo/weiguo0000000001
[zk: localhost:2181(CONNECTED) 5] create -s /sanguo/weiguo "caocao"
Created /sanguo/weiguo0000000002
[zk: localhost:2181(CONNECTED) 6] ls /sanguo
[shuguo, weiguo, weiguo0000000000, weiguo0000000001, weiguo0000000002, wuguo]
[zk: localhost:2181(CONNECTED) 6]
```

如果节点下原来没有子节点，序号从0开始依次递增。如果原节点下已有2个节点，则再排序时从2开始，以此类推。

**9）修改节点数据值**

```
[zk: localhost:2181(CONNECTED) 6] set /sanguo/weiguo "caopi"
```

**10）节点的值变化监听**

​    （1）在hadoop104主机上注册监听/sanguo节点数据变化

```
[zk: localhost:2181(CONNECTED) 26] [zk: localhost:2181(CONNECTED) 8] get -w /sanguo
```

​    （2）在hadoop103主机上修改/sanguo节点的数据

```
[zk: localhost:2181(CONNECTED) 1] set /sanguo "xishi"
```

​    （3）观察hadoop104主机收到数据变化的监听

```
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged path:/sanguo
```

**11）节点的子节点变化监听（路径变化）**

​    （1）在hadoop104主机上注册监听/sanguo节点的子节点变化

```
[zk: localhost:2181(CONNECTED) 1] ls -w /sanguo
[aa0000000001, server101]
```

​    （2）在hadoop103主机/sanguo节点上创建子节点

```
[zk: localhost:2181(CONNECTED) 2] create /sanguo/jin "simayi"
Created /sanguo/jin
```

​    （3）观察hadoop104主机收到子节点变化的监听

```
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/sanguo
```

**12）删除节点**

```
[zk: localhost:2181(CONNECTED) 4] delete /sanguo/jin
```

**13）递归删除节点**

```
[zk: localhost:2181(CONNECTED) 15] deleteall /sanguo/shuguo
```

**14****）查看节点状态**

```
[zk: localhost:2181(CONNECTED) 17] stat /sanguo
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000011
mtime = Wed Aug 29 00:21:23 CST 2018
pZxid = 0x100000014
cversion = 9
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 1
```

### 3.3 API应用

#### 3.3.1 IDEA环境搭建

**1）创建一个Maven Module**

**2）添加pom文件**

```xml
<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.8.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.5.7</version>
		</dependency>
</dependencies>
```

**3）拷贝log4j.properties文件到项目根目录**

需要在项目的src/main/resources目录下，新建一个文件，命名为“log4j.properties”，在文件中填入。

```
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

#### 3.3.2 初始化ZooKeeper客户端

```java
public class Zookeeper {

    private String connectString;
    private int sessionTimeout;
private ZooKeeper zkClient;

    @Before   //获取客户端对象
public void init() throws IOException {

        connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
        int sessionTimeout = 10000;
        
       //参数解读 1集群连接字符串  2连接超时时间 单位:毫秒  3当前客户端默认的监控器
        zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
            }
        });
    }

    @After //关闭客户端对象
    public void close() throws InterruptedException {
        zkClient.close();
    }
}

```

#### 3.3.3 获取子节点列表,不监听

```java
@Test
public void ls() throws IOException, KeeperException, InterruptedException {
  //用客户端对象做各种操作
  List<String> children = zkClient.getChildren("/", false);
  System.out.println(children);
}
```

#### 3.3.4 获取子节点列表,并监听

```java
@Test
public void lsAndWatch() throws KeeperException, InterruptedException {
    List<String> children = zkClient.getChildren("/atguigu", new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            System.out.println(event);
        }
    });
System.out.println(children);
    //因为设置了监听,所以当前线程不能结束
    Thread.sleep(Long.MAX_VALUE);
}

```

#### 3.3.5 创建子节点

```java
@Test
public void create() throws KeeperException, InterruptedException {
//参数解读 1节点路径  2节点存储的数据  
//3节点的权限(使用Ids选个OPEN即可) 4节点类型 短暂 持久 短暂带序号 持久带序号
     String path = zkClient.create("/atguigu", "shanguigu".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

    //创建临时节点
//String path = zkClient.create("/atguigu2", "shanguigu".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);

System.out.println(path);

    //创建临时节点的话,需要线程阻塞
    //Thread.sleep(10000);
}
```

#### 3.3.6 判断Znode是否存在

```java
Test
public void exist() throws Exception {

	Stat stat = zkClient.exists("/atguigu", false);

	System.out.println(stat == null ? "not exist" : "exist");
}

```

#### 3.3.7 获取子节点存储的数据,不监听

```java
@Test
public void get() throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists("/atguigu", false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }

    byte[] data = zkClient.getData("/atguigu", false, stat);
    System.out.println(new String(data));
}
```

#### 3.3.8 获取子节点存储的数据,并监听

```java
@Test
public void getAndWatch() throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists("/atguigu", false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }

    byte[] data = zkClient.getData("/atguigu", new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            System.out.println(event);
        }
    }, stat);
    System.out.println(new String(data));
    //线程阻塞
    Thread.sleep(Long.MAX_VALUE);
}

```

#### 3.3.9 设置节点的值

```java
@Test
public void set() throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists("/atguigu", false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }
    //参数解读 1节点路径 2节点的值 3版本号
    zkClient.setData("/atguigu", "sgg".getBytes(), stat.getVersion());
}
```

#### 3.3.10 删除空节点

```java
@Test
public void delete() throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists("/aaa", false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }
    zkClient.delete("/aaa", stat.getVersion());
}
```

#### 3.3.11 删除非空节点,递归实现

```java
//封装一个方法,方便递归调用
public void deleteAll(String path, ZooKeeper zk) throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists(path, false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }
    //先获取当前传入节点下的所有子节点
    List<String> children = zk.getChildren(path, false);
    if (children.isEmpty()) {
        //说明传入的节点没有子节点,可以直接删除
        zk.delete(path, stat.getVersion());
    } else {
        //如果传入的节点有子节点,循环所有子节点
        for (String child : children) {
            //删除子节点,但是不知道子节点下面还有没有子节点,所以递归调用
            deleteAll(path + "/" + child, zk);  
        }
        //删除完所有子节点以后,记得删除传入的节点
        zk.delete(path, stat.getVersion());
    }
}
//测试deleteAll
@Test
public void testDeleteAll() throws KeeperException, InterruptedException {
    deleteAll("/atguigu",zkClient);
}
```

---

## 4 Zookeeper内部原理
