# Hive

## 1 Hive基本概念

### 1.1 什么是Hive

**1）hive简介**

Hive：由Facebook开源用于解决海量结构化日志的数据统计工具。

Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。

**2）Hive本质**：将HQL转化成MapReduce程序

（1）Hive处理的数据存储在HDFS

（2）Hive分析数据底层的实现是MapReduce

（3）执行程序运行在Yarn上

<img src="E:\learning\04_java\01_笔记\BigData\03_Hive\picture\hive.png" style="zoom: 25%;" />

### 1.2 Hive的优缺点

#### 1.2.1 优点

（1）操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）。

（2）避免了去写MapReduce，减少开发人员的学习成本。

（3）Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合。

（4）Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。

（5）Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。

#### 1.2.2 缺点

**1）Hive的HQL表达能力有限**

（1）迭代式算法无法表达

（2）数据挖掘方面不擅长，由于MapReduce数据处理流程的限制，效率更高的算法却无法实现。

**2）Hive的效率比较低**

（1）Hive自动生成的MapReduce作业，通常情况下不够智能化

（2）Hive调优比较困难，粒度较粗

### 1.3 Hive架构原理

<img src="E:\learning\04_java\01_笔记\BigData\03_Hive\picture\Hive 架构.png" style="zoom: 67%;" />

**1）用户接口：Client**

CLI（command-line interface）、JDBC/ODBC(jdbc访问hive)、WEBUI（浏览器访问hive）

**2）元数据：Meta store**

元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；

默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore

**3）Hadoop**

使用HDFS进行存储，使用MapReduce进行计算。

**4）驱动器：Driver**

（1）解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。

（2）编译器（Physical Plan）：将AST编译生成逻辑执行计划。

（3）优化器（Query Optimizer）：对逻辑执行计划进行优化。

（4）执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。

<img src="E:\learning\04_java\01_笔记\BigData\03_Hive\picture\Hive的运行机制.png" style="zoom: 25%;" />

Hive通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，使用自己的Driver，结合元数据(MetaStore)，将这些指令翻译成MapReduce，提交到Hadoop中执行，最后，将执行返回的结果输出到用户交互接口。

### 1.4 Hive和 数据库比较

由于 Hive 采用了类似SQL 的查询语言 HQL(Hive Query Language)，因此很容易将 Hive 理解为数据库。其实从结构上来看，Hive 和数据库除了拥有类似的查询语言，再无类似之处。本文将从多个方面来阐述 Hive 和数据库的差异。数据库可以用在 Online 的应用中，但是Hive 是为数据仓库而设计的，清楚这一点，有助于从应用角度理解 Hive 的特性。

#### 1.4.1 查询语言

由于SQL被广泛的应用在数据仓库中，因此，专门针对Hive的特性设计了类SQL的查询语言HQL。熟悉SQL开发的开发者可以很方便的使用Hive进行开发。

#### 1.4.2 数据更新

由于Hive是针对数据仓库应用设计的，而数据仓库的内容是读多写少的。因此，Hive中不建议对数据的改写，所有的数据都是在加载的时候确定好的。而数据库中的数据通常是需要经常进行修改的，因此可以使用 INSERT INTO … VALUES 添加数据，使用 UPDATE … SET修改数据。

#### 1.4.3 执行延迟

Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。另外一个导致 Hive 执行延迟高的因素是 MapReduce框架。由于MapReduce 本身具有较高的延迟，因此在利用MapReduce 执行Hive查询时，也会有较高的延迟。相对的，数据库的执行延迟较低。当然，这个低是有条件的，即数据规模较小，当数据规模大到超过数据库的处理能力的时候，Hive的并行计算显然能体现出优势。

#### 1.4.4 数据规模

由于Hive建立在集群上并可以利用MapReduce进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。

---

## 2 Hive安装

### 2.1 Hive安装地址

**1）Hive官网地址**

http://hive.apache.org/

**2）文档查看地址**

https://cwiki.apache.org/confluence/display/Hive/GettingStarted

**3）下载地址**

http://archive.apache.org/dist/hive/

**4）github地址**

https://github.com/apache/hive

### 2.2 MySql安装

**1）为什么需要Mysql**

原因在于Hive默认使用的元数据库为derby，开启Hive之后就会占用元数据库，且不与其他客户端共享数据，如果想多窗口操作就会报错，操作比较局限。以我们需要将Hive的元数据地址改为MySQL，可支持多窗口操作。

**2）检查当前系统是否安装过Mysql**

```shell
[xu1an@hadoop102 ~]$ rpm -qa|grep mariadb
mariadb-libs-5.5.56-2.el7.x86_64 //如果存在通过如下命令卸载
[xu1an@hadoop102 ~]$ sudo rpm -e --nodeps  mariadb-libs   //用此命令卸载mariadb
```

1. 卸载Linux自带的mysql
   
   1)查自带的软件 
   
   ```shell
   CentOS6->mysql   CentOS7 ->mariadb
   rpm -qa | grep  -i  -E mysql\|mariadb
   ```
   
   2 )卸载
   
   ```shell
   sudo rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64
   sudo rpm -e --nodeps mysql-community-common-5.7.16-1.el7.x86_64
   rpm -qa | grep  -i  -E mysql\|mariadb | xargs -n1 sudo rpm -e --nodeps 
   ```

**3) 将MySQ安装包拷贝到/opt/software目录下**

```shell
[xu1an @hadoop102 software]# ll
总用量 528384
-rw-r--r--. 1 root root 609556480 3月  21 15:41 mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
```

**4）在安装目录下执行rpm安装**

```shell
tar -xf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
```

**5）在安装目录下执行rpm安装**

安装 common、libs、libs-compact、server四个rpm

```shell
[xu1an @hadoop102 software]$ sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
[xu1an @hadoop102 software]$ sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
[xu1an @hadoop102 software]$ sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
[xu1an @hadoop102 software]$ sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
[xu1an @hadoop102 software]$ sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
```

注意:按照顺序依次执行

如果Linux是最小化安装的，在安装mysql-community-server-5.7.28-1.el7.x86_64.rpm时可能会出现如下错误

```shell
[xu1an@hadoop102 software]$ sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
警告：mysql-community-server-5.7.28-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
错误：依赖检测失败：
        libaio.so.1()(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64 需要
        libaio.so.1(LIBAIO_0.1)(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64 需要
        libaio.so.1(LIBAIO_0.4)(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64 需要
```

通过yum安装缺少的依赖,然后重新安装mysql-community-server-5.7.28-1.el7.x86_64 即可

```shell
[xu1an@hadoop102 software] yum install -y libaio
```

**6）删除/etc/my.cnf文件中datadir指向的目录下的所有内容,如果有内容的情况下:**

 查看datadir的值：

```shell
[mysqld]
datadir=/var/lib/mysql
```

  删除/var/lib/mysql目录下的所有内容:

```shell
[xu1an @hadoop102 mysql]# cd /var/lib/mysql
[xu1an @hadoop102 mysql]# sudo rm -rf ./*  //注意执行命令的位置
```

**7）初始化数据库**

```shell
[xu1an @hadoop102 opt]$ sudo mysqld --initialize --user=mysql
```

**8）查看临时生成的root用户的密码** 

```shell
[xu1an @hadoop102 opt]$ cat /var/log/mysqld.log
```

**9）启动MySQL服务**

```shell
[xu1an @hadoop102 opt]$ sudo systemctl start mysqld
```

**10）登录MySQL数据库**

```shell
[xu1an @hadoop102 opt]$ mysql -uroot -p
Enter password:  输入临时生成的密码
```

**11）必须先修改root用户的密码,否则执行其他的操作会报错**

```mysql
mysql> set password = password("新密码")
```

**12）修改mysql库下的user表中的root用户允许任意ip连接**

```mysql
mysql> update mysql.user set host='%' where user='root';
mysql> flush privileges;
```

### 2.3 Hive安装部署

**1）把apache-hive-3.1.2-bin.tar.gz上传到linux的/opt/software目录下**

**2）解压apache-hive-3.1.2-bin.tar.gz到/opt/module/目录下面**

```shell
[xu1an@hadoop102 software]$ tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/
```

**3）修改apache-hive-3.1.2-bin.tar.gz的名称为hive**

```shell
[xu1an@hadoop102 software]$ mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
```

**4）修改/etc/profile.d/my_env.sh，添加环境变量**

```shell
[xu1an@hadoop102 software]$ sudo vim /etc/profile.d/my_env.sh
```

**5）添加内容**

```
#HIVE_HOME
HIVE_HOME=/opt/module/hive

PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin
export PATH JAVA_HOME HADOOP_HOME HIVE_HOME
```

**6）解决日志Jar包冲突**

```
[xu1an@hadoop102 software]$ mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.bak
```

### 2.4 Hive元数据配置到MySql

#### 2.4.1 拷贝驱动

将MySQL的JDBC驱动拷贝到Hive的lib目录下

```
[xu1an@hadoop102 software]$ cp /opt/software/mysql-connector-java-5.1.48.jar $HIVE_HOME/lib
```

#### 2.4.2 配置Metastore到MySql

在$HIVE_HOME/conf目录下新建hive-site.xml文件

```
[xu1an@hadoop102 software]$ vim $HIVE_HOME/conf/hive-site.xml
```

添加如下内容

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <!-- jdbc连接的URL -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
</property>

    <!-- jdbc连接的Driver-->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
</property>

	<!-- jdbc连接的username-->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <!-- jdbc连接的password -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
</property>

    <!-- Hive默认在HDFS的工作目录 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    
   <!-- Hive元数据存储的验证 -->
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
   
    <!-- 元数据存储授权  -->
    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
</configuration>
```

### 2.5 启动Hive

#### 2.5.1 初始化元数据库

**1）登陆MySQL**

```
[xu1an@hadoop102 software]$ mysql -uroot -p000000
```

**2）新建Hive元数据库**

```
mysql> create database metastore;
mysql> quit;
```

**3）初始化Hive元数据库**

```
[xu1an@hadoop102 software]$ schematool -initSchema -dbType mysql -verbose
```

#### 2.5.2 启动Hive

**0）先启动hadoop集群**

```
zk_culster.sh start
my_ha.sh start
```

**1）启动Hive**

```
[xu1an@hadoop102 hive]$ bin/hive
```

**2）使用Hive**

```
hive> show databases;
hive> show tables;
hive> create table test (id int);
hive> insert into test values(1);
hive> select * from test;
```

**3）开启另一个窗口测试开启hive**

```
[xu1an@hadoop102 hive]$ bin/hive
```

#### 2.5.3 使用元数据服务的方式访问Hive

**1）在hive-site.xml文件中添加如下配置信息**

```xml
    <!-- 指定存储元数据要连接的地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://hadoop102:9083</value>
    </property>
```

**2）启动metastore**

```
[xu1an@hadoop202 hive]$ hive --service metastore
2020-04-24 16:58:08: Starting Hive Metastore Server
注意: 启动后窗口不能再操作，需打开一个新的shell窗口做别的操作
```

**3）启动hive**

```
[xu1an@hadoop202 hive]$ bin/hive
```

#### 2.5.4 使用JDBC方式访问Hive

**1）在hive-site.xml文件中添加如下配置信息**

```xml
    <!-- 指定hiveserver2连接的host -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hadoop102</value>
    </property>

    <!-- 指定hiveserver2连接的端口号 -->
    <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
    </property>
```

**2）启动hiveserver2**

```
[xu1an@hadoop102 hive]$ bin/hive --service hiveserver2
```

**3）启动beeline客户端（需要多等待一会）**

```
[xu1an@hadoop102 hive]$ bin/beeline -u jdbc:hive2://hadoop102:10000 -n xu1an
```

**4）看到如下界面**

```
Connecting to jdbc:hive2://hadoop102:10000
Connected to: Apache Hive (version 3.1.2)
Driver: Hive JDBC (version 3.1.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 3.1.2 by Apache Hive
0: jdbc:hive2://hadoop102:10000>
```

#### 2.5.5 编写启动metastore和hiveserver2脚本(了解)

**1）Shell命令介绍**

前台启动的方式导致需要打开多个shell窗口，可以使用如下方式后台方式启动

nohup: 放在命令开头，表示不挂起,也就是关闭终端进程也继续保持运行状态

0:标准输入

1:标准输出

2:错误输出

2>&1 : 表示将错误重定向到标准输出上

&: 放在命令结尾,表示后台运行

一般会组合使用: nohup [xxx命令操作]> file 2>&1 & ， 表示将xxx命令运行的

结果输出到file中，并保持命令启动的进程在后台运行。

如上命令不要求掌握。 

```
[xu1an@hadoop202 hive]$ nohup hive --service metastore 2>&1 &
[xu1an@hadoop202 hive]$ nohup hive --service hiveserver2 2>&1 &
```

**2）编写脚本**

```shell
[xu1an@hadoop102 hive]$ vim $HIVE_HOME/bin/hiveservices.sh

#!/bin/bash
HIVE_LOG_DIR=$HIVE_HOME/logs
if [ ! -d $HIVE_LOG_DIR ]
then
	mkdir -p $HIVE_LOG_DIR
fi
#检查进程是否运行正常，参数1为进程名，参数2为进程端口
function check_process()
{
    pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
    ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
    echo $pid
    [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
}

function hive_start()
{
    metapid=$(check_process HiveMetastore 9083)
    cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 &"
    cmd=$cmd" sleep 4; hdfs dfsadmin -safemode wait >/dev/null 2>&1"
    [ -z "$metapid" ] && eval $cmd || echo "Metastroe服务已启动"
    server2pid=$(check_process HiveServer2 10000)
    cmd="nohup hive --service hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
    [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2服务已启动"
}

function hive_stop()
{
    metapid=$(check_process HiveMetastore 9083)
    [ "$metapid" ] && kill $metapid || echo "Metastore服务未启动"
    server2pid=$(check_process HiveServer2 10000)
    [ "$server2pid" ] && kill $server2pid || echo "HiveServer2服务未启动"
}

case $1 in
"start")
    hive_start
    ;;
"stop")
    hive_stop
    ;;
"restart")
    hive_stop
    sleep 2
    hive_start
    ;;
"status")
    check_process HiveMetastore 9083 >/dev/null && echo "Metastore服务运行正常" || echo "Metastore服务运行异常"
    check_process HiveServer2 10000 >/dev/null && echo "HiveServer2服务运行正常" || echo "HiveServer2服务运行异常"
    ;;
*)
    echo Invalid Args!
    echo 'Usage: '$(basename $0)' start|stop|restart|status'
    ;;
esac
```

**3）添加执行权限**

```
[xu1an@hadoop102 hive]$ chmod u+x $HIVE_HOME/bin/hiveservices.sh
```

**4）启动Hive后台服务**

```
[xu1an@hadoop102 hive]$ hiveservices.sh start
```

### 2.6 Hive常用交互命令

```
[xu1an@hadoop102 hive]$ bin/hive -help
usage: hive
 -d,--define <key=value>          Variable subsitution to apply to hive
                                  commands. e.g. -d A=B or --define A=B
    --database <databasename>     Specify the database to use
 -e <quoted-query-string>         SQL from command line
 -f <filename>                    SQL from files
 -H,--help                        Print help information
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable subsitution to apply to hive
                                  commands. e.g. --hivevar A=B
 -i <filename>                    Initialization SQL file
 -S,--silent                      Silent mode in interactive shell
 -v,--verbose                     Verbose mode (echo executed SQL to the console)
```

**1）“-e”不进入hive的交互窗口执行sql语句**

```
[xu1an@hadoop102 hive]$ bin/hive -e "select id from student;"
```

**2）“-f”执行脚本中sql语句**

（1）在/opt/module/hive/下创建datas目录并在datas目录下创建hivef.sql文件

```
[xu1an@hadoop102 datas]$ touch hivef.sql
```

（2）文件中写入正确的sql语句

```
select *from student;
```

（3）执行文件中的sql语句

```
[xu1an@hadoop102 hive]$ bin/hive -f /opt/module/hive/datas/hivef.sql
```

（4）执行文件中的sql语句并将结果写入文件中

```
[xu1an@hadoop102 hive]$ bin/hive -f /opt/module/hive/datas/hivef.sql > /opt/module/datas/hive_result.txt
```

### 2.7 Hive其他命令操作

**1）退出hive窗口：**

hive(default)>exit;

hive(default)>quit;

在新版的hive中没区别了，在以前的版本是有的：

exit:先隐性提交数据，再退出；

quit:不提交数据，退出；

**2）在hive cli命令窗口中如何查看hdfs文件系统**

```hive
hive(default)>dfs -ls /;
```

**3）查看在hive中输入的所有历史命令**

（1）进入到当前用户的根目录/root或/home/xu1an

（2）查看. hivehistory文件

```
[atguig2u@hadoop102 ~]$ cat .hivehistory
```

### 2.8 Hive常见属性配置

#### 2.8.1 hive窗口打印默认库和表头

**1）打印当前库和表头**

在hive-site.xml中加入如下两个配置: 

```xml
  <property>
    <name>hive.cli.print.header</name>
    <value>true</value>
  </property>
   <property>
    <name>hive.cli.print.current.db</name>
    <value>true</value>
  </property>
```

#### 2.8.2 Hive运行日志信息配置

**1）Hive的log默认存放在/tmp/xu1an/hive.log目录下（当前用户名下）**

**2）修改hive的log存放日志到/opt/module/hive/logs**

（1）修改`/opt/module/hive/conf/hive-log4j2.properties.template`文件名称为`hive-log4j2.properties`

```
[xu1an@hadoop102 conf]$ pwd
/opt/module/hive/conf
[xu1an@hadoop102 conf]$ mv hive-log4j2.properties.template hive-log4j2.properties
```

（2）在`hive-log4j.properties`文件中修改log存放位置

```
property.hive.log.dir=/opt/module/hive/logs
```

#### 2.8.3 参数配置方式

**1）查看当前所有的配置信息**

hive>set;

**2）参数的配置三种方式**

（1）配置文件方式

默认配置文件：`hive-default.xml`

用户自定义配置文件：`hive-site.xml`

注意：用户自定义配置会覆盖默认配置。另外，Hive也会读入Hadoop的配置，因为Hive是作为Hadoop的客户端启动的，Hive的配置会覆盖Hadoop的配置。配置文件的设定对本机启动的所有Hive进程都有效。

（2）命令行参数方式

启动Hive时，可以在命令行添加-hiveconf param=value来设定参数。

例如：

```
[xu1an@hadoop103 hive]$ bin/hive -hiveconf mapred.reduce.tasks=10;
```

注意：仅对本次hive启动有效

查看参数设置：  

```
hive (default)> set mapred.reduce.tasks;
```

（3）参数声明方式

可以在HQL中使用SET关键字设定参数

例如：

```
hive (default)> set mapred.reduce.tasks=100;
```

注意：仅对本次hive启动有效。

查看参数设置

```
hive (default)> set mapred.reduce.tasks;
```

上述三种设定方式的优先级依次递增。即配置文件<命令行参数<参数声明。注意某些系统级的参数，例如log4j相关的设定，必须用前两种方式设定，因为那些参数的读取在会话建立以前已经完成了。

---

## 3 Hive数据类型

### 3.1 基本数据类型

| Hive数据类型 | Java数据类型 | 长度                                               | 例子                                  |
| ------------ | ------------ | -------------------------------------------------- | ------------------------------------- |
| TINYINT      | byte         | 1byte有符号整数                                    | 20                                    |
| SMALINT      | short        | 2byte有符号整数                                    | 20                                    |
| **INT**      | int          | 4byte有符号整数                                    | 20                                    |
| **BIGINT**   | long         | 8byte有符号整数                                    | 20                                    |
| BOOLEAN      | boolean      | 布尔类型，true或者false                            | TRUE FALSE                            |
| FLOAT        | float        | 单精度浮点数                                       | 3.14159                               |
| **DOUBLE**   | double       | 双精度浮点数                                       | 3.14159                               |
| **STRING**   | string       | 字符系列。可以指定字符集。可以使用单引号或者双引号 | ‘now is the time’  “for all good men” |
| TIMESTAMP    |              | 时间类型                                           |                                       |
| BINARY       |              | 字节数组                                           |                                       |

对于Hive的String类型相当于数据库的varchar类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储2GB的字符数。

### 3.2 集合数据类型

| 数据类型 | 描述                                                         | 语法示例                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------ |
| STRUCT   | 和c语言中的struct类似，都可以通过“点”符号访问元素内容。例如，如果某个列的数据类型是STRUCT{first STRING, last  STRING},那么第1个元素可以通过字段.first来引用。 | struct()  例如struct<street:string, city:string> |
| MAP      | MAP是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是MAP，其中键->值对是’first’->’John’和’last’->’Doe’，那么可以通过字段名[‘last’]获取最后一个元素 | map()  例如map<string, int>                      |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’, ‘Doe’]，那么第2个元素可以通过数组名[1]进行引用。 | Array()  例如array<string>                       |

Hive有三种复杂数据类型ARRAY、MAP 和 STRUCT。ARRAY和MAP与Java中的Array和Map类似，而STRUCT与C语言中的Struct类似，它封装了一个命名字段集合，复杂数据类型允许任意层次的嵌套。

**1）案例实操**

（1）假设某表有如下一行，我们用JSON格式来表示其数据结构。在Hive下访问的格式为

```
{
    "name": "songsong",
    "friends": ["bingbing" , "lili"] ,       //列表Array, 
    "children": {                      //键值Map,
        "xiao song": 19 ,
        "xiaoxiao song": 18
    }
    "address": {                      //结构Struct,
        "street": "hui long guan" ,
        "city": "beijing" 
    }
}
```

（2）基于上述数据结构，我们在Hive里创建对应的表，并导入数据。 创建本地测试文件test.txt

```
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing
```

注意：MAP，STRUCT和ARRAY里的元素间关系都可以用同一个字符表示，这里用“_”。

（3）Hive上创建测试表test

```
create table test(
name string,
friends array<string>,
children map<string, int>,
address struct<street:string, city:string>
)
row format delimited fields terminated by ','
collection items terminated by '_'
map keys terminated by ':'
lines terminated by '\n';
```

字段解释：

row format delimited fields terminated by ',' -- 列分隔符

collection items terminated by '_'    --MAP STRUCT 和 ARRAY 的分隔符(数据分割符号)

map keys terminated by ':'           -- MAP中的key与value的分隔符

lines terminated by '\n';             -- 行分隔符

（4）导入文本数据到测试表

load data local inpath '/opt/module/hive/datas/test.txt' into table test; 

（5）访问三种集合列里的数据，以下分别是ARRAY，MAP，STRUCT的访问方式

```
hive (default)> select friends[1],children['xiao song'],address.city from test
where name="songsong";
OK
_c0     _c1     city
lili    18      beijing
Time taken: 0.076 seconds, Fetched: 1 row(s)
```

### 3.3 类型转换

Hive的原子数据类型是可以进行隐式转换的，类似于Java的类型转换，例如某表达式使用INT类型，TINYINT会自动转换为INT类型，但是Hive不会进行反向转化，例如，某表达式使用TINYINT类型，INT不会自动转换为TINYINT类型，它会返回错误，除非使用CAST操作。

**1）隐式类型转换规则如下**

（1）任何整数类型都可以隐式地转换为一个范围更广的类型，如TINYINT可以转换成INT，INT可以转换成BIGINT。

（2）所有整数类型、FLOAT和**STRING类型（字符串类型的数字）**都可以隐式地转换成DOUBLE。

（3）TINYINT、SMALLINT、INT都可以转换为FLOAT。

（4）BOOLEAN类型不可以转换为任何其它的类型。

**2）可以使用CAST操作显示进行数据类型转换**

例如`CAST('1' AS INT)`将把字符串'1' 转换成整数1；如果强制类型转换失败，如执行CAST('X' AS INT)，表达式返回空值 NULL。

```
0: jdbc:hive2://hadoop102:10000> select '1'+2, cast('1'as int) + 2;
+------+------+--+
| _c0  | _c1  |
+------+------+--+
| 3.0  | 3    |
+------+------+--+
```

----

## 4 DDL数据定义语言

### 4.1 创建数据库

```mysql
CREATE DATABASE [IF NOT EXISTS] database_name
[COMMENT database_comment]
[LOCATION hdfs_path]
[WITH DBPROPERTIES (property_name=property_value, ...)];
```

**1）创建一个数据库，数据库在HDFS上的默认存储路径是/user/hive/warehouse/*.db。**

```mysql
hive (default)> create database db_hive;
```

**2）避免要创建的数据库已经存在错误，增加if not exists判断。（标准写法）**

```mysql
hive (default)> create database db_hive;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Database db_hive already exists
hive (default)> create database if not exists db_hive;
```

```mysql
beeline:
create database if not exists mydb   
comment "my first db"
with dbproperties("createtime"="2021-04-24");
```

**3）创建一个数据库，指定数据库在HDFS上存放的位置**

```mysql
hive (default)> create database db_hive2 location '/db_hive2.db';
```

### 4.2 查询数据库

#### 4.2.1 显示数据库

**1）显示数据库**

```mysql
hive> show databases;
```

```mysql
desc database mydb;
```

**2）过滤显示查询的数据库**

```mysql
hive> show databases like 'db_hive*';
OK
db_hive
db_hive_1
```

#### 4.2.2 查看数据库详情

**1）显示数据库信息**

```mysql
hive> desc database db_hive;
OK
db_hive		hdfs://hadoop102:9000/user/hive/warehouse/db_hive.db	xu1an USER	
```

**2）显示数据库详细信息，extended**

```mysql
hive> desc database extended db_hive;
OK
db_hive		hdfs://hadoop102:9000/user/hive/warehouse/db_hive.db	xu1an USER	
```

#### 4.2.3 切换当前数据库

```mysql
hive (default)> use db_hive;
```

### 4.3 修改数据库

用户可以使用ALTER DATABASE命令为某个数据库的DBPROPERTIES设置键-值对属性值，来描述这个数据库的属性信息。

```mysql
hive (default)> alter database db_hive set dbproperties('createtime'='20170830');
```

在hive中查看修改结果

```mysql
hive> desc database extended db_hive;
db_name comment location        owner_name      owner_type      parameters
db_hive         hdfs://hadoop102:8020/user/hive/warehouse/db_hive.db    xu1an USER    {createtime=20170830}
```

### 4.4 删除数据库

**1）删除空数据库**

```
hive>drop database db_hive2;
```

**2）如果删除的数据库不存在，最好采用if exists判断数据库是否存在**

```
hive> drop database db_hive;
FAILED: SemanticException [Error 10072]: Database does not exist: db_hive
hive> drop database if exists db_hive2;
```

**3）如果数据库不为空，可以采用cascade命令，强制删除**

```mysql
hive> drop database db_hive;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database db_hive is not empty. One or more tables exist.)
hive> drop database db_hive cascade;
```

### 4.5 创建表

**1）建表语法**

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
[(col_name data_type [COMMENT col_comment], ...)] 
[COMMENT table_comment] 
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
[CLUSTERED BY (col_name, col_name, ...) 
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
[ROW FORMAT row_format] 
[STORED AS file_format] 
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)]
[AS select_statement]
```

**2）字段解释说明** 

（1）CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXISTS 选项来忽略这个异常。

（2）EXTERNAL关键字可以让用户创建一个外部表，在建表的同时可以指定一个指向实际数据的路径（LOCATION），在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。

（3）COMMENT：为表和列添加注释。

（4）PARTITIONED BY创建分区表

（5）CLUSTERED BY创建分桶表

（6）SORTED BY不常用，对桶中的一个或多个列另外排序

（7）ROW FORMAT 

DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char]

​    [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char] 

  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]

用户在建表的时候可以自定义SerDe或者使用自带的SerDe。如果没有指定ROW FORMAT 或者ROW FORMAT DELIMITED，将会使用自带的SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的SerDe，Hive通过SerDe确定表的具体的列的数据。

SerDe是Serialize/Deserilize的简称， hive使用Serde进行行对象的序列与反序列化。

（8）STORED AS指定存储文件类型

常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、RCFILE（列式存储格式文件）

如果文件数据是纯文本，可以使用STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE。

（9）LOCATION ：指定表在HDFS上的存储位置。

（10）AS：后跟查询语句，根据查询结果创建表。

（11）LIKE允许用户复制现有的表结构，但是不复制数据。

#### 4.5.1 管理表(内部表)

**1）理论**

默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive会（或多或少地）控制着数据的生命周期。Hive默认情况下会将这些表的数据存储在由配置项hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。  当我们删除一个管理表时，Hive也会删除这个表中数据。管理表不适合和其他工具共享数据。

**2）案例实操**

（0）原始数据

```
1001	ss1
1002	ss2
1003	ss3
1004	ss4
1005	ss5
1006	ss6
1007	ss7
1008	ss8
1009	ss9
1010	ss10
1011	ss11
1012	ss12
1013	ss13
1014	ss14
1015	ss15
1016	ss16
```

（1）普通创建表

```mysql
create table if not exists student(
id int, name string
)
row format delimited fields terminated by '\t'
stored as textfile
location '/user/hive/warehouse/student';
```

（2）根据查询结果创建表（查询的结果会添加到新创建的表中）

```mysql
create table if not exists student2 as select id, name from student;
```

（3）根据已经存在的表结构创建表

```mysql
create table if not exists student3 like student;
```

（4）查询表的类型

```mysql
hive (default)> desc formatted student2;
Table Type:             MANAGED_TABLE  
```

#### 4.5.2 外部表

**1）理论**

因为表是外部表，所以Hive并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。

**2）管理表和外部表的使用场景**

每天将收集到的网站日志定期流入HDFS文本文件。在外部表（原始日志表）的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过SELECT+INSERT进入内部表。

**3）案例实操**

分别创建部门和员工外部表，并向表中导入数据。

（0）原始数据

dept:

```
10	ACCOUNTING	1700
20	RESEARCH	1800
30	SALES	1900
40	OPERATIONS	1700
```

emp：

```
7369	SMITH	CLERK	7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-2-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-2-22	1250.00	500.00	30
7566	JONES	MANAGER	7839	1981-4-2	2975.00		20
7654	MARTIN	SALESMAN	7698	1981-9-28	1250.00	1400.00	30
7698	BLAKE	MANAGER	7839	1981-5-1	2850.00		30
7782	CLARK	MANAGER	7839	1981-6-9	2450.00		10
7788	SCOTT	ANALYST	7566	1987-4-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-9-8	1500.00	0.00	30
7876	ADAMS	CLERK	7788	1987-5-23	1100.00		20
7900	JAMES	CLERK	7698	1981-12-3	950.00		30
7902	FORD	ANALYST	7566	1981-12-3	3000.00		20
7934	MILLER	CLERK	7782	1982-1-23	1300.00		10
```

（1）上传数据到HDFS

```mysql
hive (default)> dfs -mkdir /student;
hive (default)> dfs -put /opt/module/datas/student.txt /student;
```

（2）建表语句，创建外部表

创建部门表

```mysql
create external table if not exists dept(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

创建员工表

```mysql
create external table if not exists emp(
empno int,
ename string,
job string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
row format delimited fields terminated by '\t';
```

（3）查看创建的表

```mysql
hive (default)>show tables;
```

（4）查看表格式化数据

```mysql
hive (default)> desc formatted dept;
Table Type:             EXTERNAL_TABLE
```

（5）删除外部表

```mysql
hive (default)> drop table dept;
```

#### 4.5.3 管理表与外部表的互相转换

（1）查询表的类型

```mysql
hive (default)> desc formatted student2;
Table Type:             MANAGED_TABLE
```

（2）修改内部表student2为外部表

```mysql
alter table student2 set tblproperties('EXTERNAL'='TRUE');
```

（3）查询表的类型

```mysql
hive (default)> desc formatted student2;
Table Type:             EXTERNAL_TABLE
```

（4）修改外部表student2为内部表

```mysql
alter table student2 set tblproperties('EXTERNAL'='FALSE');
```

（5）查询表的类型

```mysql
hive (default)> desc formatted student2;
Table Type:             MANAGED_TABLE
```

注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！

### 4.6 修改表

#### 4.6.1 重命名表

**1）语法**

```mysql
ALTER TABLE table_name RENAME TO new_table_name
```

**2）实操案例**

```mysql
hive (default)> alter table dept_partition2 rename to dept_partition3;
```

#### 4.6.2 增加、修改和删除表分区

详见7.1章分区表基本操作。

#### 4.6.3 增加/修改/替换列信息

**1）语法**

（1）更新列

```mysql
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]
```

（2）增加和替换列

```mysql
ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) 
```

注：ADD是代表新增一字段，字段位置在所有列后面(partition列前)，REPLACE则是表示替换表中所有字段。

**2）实操案例**

（1）查询表结构

```mysql
hive> desc dept;
```

（2）添加列

```mysql
hive (default)> alter table dept add columns(deptdesc string);
```

（3）查询表结构

```mysql
hive> desc dept;
```

（4）更新列

```mysql
hive (default)> alter table dept change column deptdesc desc string;
```

（5）查询表结构

```mysql
hive> desc dept;
```

（6）替换列

```mysql
hive (default)> alter table dept replace columns(deptno string, dname string, loc string);
```

（7）查询表结构

```mysql
hive> desc dept;
```

### 4.7 删除表

```mysql
hive (default)> drop table dept;
```

---

## 5 DML数据操作语言

### 5.1 数据导入

#### 5.1.1 向表中装载数据（Load）

**1）语法**

```mysql
hive> load data [local] inpath '数据的path' [overwrite] into table student [partition (partcol1=val1,…)];
```

（1）load data:表示加载数据

（2）local:表示从本地加载数据到hive表；否则从HDFS加载数据到hive表

（3）inpath:表示加载数据的路径

（4）overwrite:表示覆盖表中已有数据，否则表示追加

（5）into table:表示加载到哪张表

（6）student:表示具体的表

（7）partition:表示上传到指定分区

**2）实操案例**

（0）创建一张表

```mysql
hive (default)> create table student(id string, name string) row format delimited fields terminated by '\t';
```

（1）加载本地文件到hive

```mysql
hive (default)> load data local inpath '/opt/module/hive/datas/student.txt' into table default.student;
```

'/opt/module/hive/datas/student.txt' into table default.student;

（2）加载HDFS文件到hive中

上传文件到HDFS

```mysql
hive (default)> dfs -put /opt/module/hive/datas/student.txt /user/atguigu/hive;
```

加载HDFS上数据

```mysql
hive (default)> load data inpath '/user/atguigu/hive/student.txt' into table default.student;
```

（3）加载数据覆盖表中已有的数据

上传文件到HDFS

```mysql
hive (default)> dfs -put /opt/module/datas/student.txt /user/atguigu/hive;
```

加载数据覆盖表中已有的数据

```mysql
hive (default)> load data inpath '/user/atguigu/hive/student.txt' overwrite into table default.student;
```

#### 5.1.2 通过查询语句向表中插入数据（Insert）

**1）创建一张表**

```mysql
hive (default)> create table student_par(id int, name string) row format delimited fields terminated by '\t';
```

**2）基本插入数据**

```mysql
hive (default)> insert into table  student_par values(1,'wangwu'),(2,'zhaoliu');
```

**3）基本模式插入（根据单张表查询结果）**

```mysql
hive (default)> insert overwrite table student_par 
             select id, name from student ; 
```

insert into：以追加数据的方式插入到表或分区，原有数据不会删除

insert overwrite：会覆盖表中已存在的数据

注意：insert不支持插入部分字段

**4）多表（多分区）插入模式（根据多张表查询结果）**

```mysql
hive (default)> from student
              insert overwrite table student partition(month='201707')
              select id, name where month='201709'
              insert overwrite table student partition(month='201706')
              select id, name where month='201709';
```

#### 5.1.3 查询语句中创建表并加载数据（As Select）

详见4.5.1章创建表。

根据查询结果创建表（查询的结果会添加到新创建的表中）

```mysql
create table if not exists student3
as select id, name from student;
```

#### 5.1.4 创建表时通过Location指定加载数据路径

**1）上传数据到hdfs上**

```mysql
hive (default)> dfs -mkdir /student;
hive (default)> dfs -put /opt/module/datas/student.txt /student;
```

**2）创建表，并指定在hdfs上的位置**

```mysql
hive (default)> create external table if not exists student5(
              id int, name string
              )
              row format delimited fields terminated by '\t'
              location '/student;
```

**3）查询数据**

```mysql
hive (default)> select * from student5;
```

#### 5.1.5 Import数据到指定Hive表中

注意：先用export导出后，再将数据导入。

```mysql
hive (default)> import table student2  from
 '/user/hive/warehouse/export/student';
```

### 5.2 数据导出

#### 5.2.1 Insert导出

**1）将查询的结果导出到本地**

```mysql
hive (default)> insert overwrite local directory '/opt/module/hive/datas/export/student'
            select * from student;
```

**2）将查询的结果格式化导出到本地**

```mysql
hive(default)>insert overwrite local directory '/opt/module/hive/datas/export/student1'
           ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'             select * from student;
```

**3）将查询的结果导出到HDFS上(没有local)**

```mysql
hive (default)> insert overwrite directory '/user/atguigu/student2'
             ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
             select * from student;
```

#### 5.2.2 Hadoop命令导出到本地

```mysql
hive (default)> dfs -get /user/hive/warehouse/student/student.txt
/opt/module/datas/export/student3.txt;
```

#### 5.2.3 Hive Shell 命令导出

基本语法：（hive -f/-e 执行语句或者脚本 > file）

```mysql
[atguigu@hadoop102 hive]$ bin/hive -e 'select * from default.student;' >
 /opt/module/hive/datas/export/student4.txt;
```

#### 5.2.4 Export导出到HDFS上

```mysql
(defahiveult)> export table default.student to
 '/user/hive/warehouse/export/student';
```

export和import主要用于两个Hadoop平台集群之间Hive表迁移。

#### 5.2.5 Sqoop导出

后续课程专门讲。

#### 5.2.6 清除表中数据（Truncate）

注意：Truncate只能删除管理表，不能删除外部表中数据

```
hive (default)> truncate table student;
```

## 6 查询

https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select

查询语句语法

```mysql
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
  FROM table_reference
  [WHERE where_condition]
  [GROUP BY col_list]
  [ORDER BY col_list]
  [CLUSTER BY col_list
    | [DISTRIBUTE BY col_list] [SORT BY col_list]
  ]
 [LIMIT number]
```

### 6.1 基本查询（Select…From）

#### 6.1.1 全表和特定列查询

**0）数据准备**

（0）原始数据

dept:

```
10	ACCOUNTING	1700
20	RESEARCH	1800
30	SALES	1900
40	OPERATIONS	1700
```

emp：

```
7369	SMITH	CLERK	7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-2-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-2-22	1250.00	500.00	30
7566	JONES	MANAGER	7839	1981-4-2	2975.00		20
7654	MARTIN	SALESMAN	7698	1981-9-28	1250.00	1400.00	30
7698	BLAKE	MANAGER	7839	1981-5-1	2850.00		30
7782	CLARK	MANAGER	7839	1981-6-9	2450.00		10
7788	SCOTT	ANALYST	7566	1987-4-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-9-8	1500.00	0.00	30
7876	ADAMS	CLERK	7788	1987-5-23	1100.00		20
7900	JAMES	CLERK	7698	1981-12-3	950.00		30
7902	FORD	ANALYST	7566	1981-12-3	3000.00		20
7934	MILLER	CLERK	7782	1982-1-23	1300.00		10
```

（1）创建部门表

```mysql
create table if not exists dept(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

（2）创建员工表

```mysql
create table if not exists emp(
empno int,
ename string,
job string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
row format delimited fields terminated by '\t';
```

（3）导入数据

```mysql
load data local inpath '/opt/module/datas/dept.txt' into table
dept;
load data local inpath '/opt/module/datas/emp.txt' into table emp;
```

**1）全表查询**

```mysql
hive (default)> select * from emp;
hive (default)> select empno,ename,job,mgr,hiredate,sal,comm,deptno from emp ;
```

**2）选择特定列查询**

```mysql
hive (default)> select empno, ename from emp;
```

注意：

（1）SQL 语言大小写不敏感。 

（2）SQL 可以写在一行或者多行

（3）关键字不能被缩写也不能分行

（4）各子句一般要分行写。

（5）使用缩进提高语句的可读性。

#### 6.1.2 列别名

**1）重命名一个列**

**2）便于计算**

**3）紧跟列名，也可以在列名和别名之间加入关键字‘AS****’**

**4）案例实操**

查询名称和部门

```mysql
hive (default)> select ename AS name, deptno dn from emp;
```

#### 6.1.3 算术运算符

| 运算符 | 描述           |
| ------ | -------------- |
| A+B    | A和B 相加      |
| A-B    | A减去B         |
| A*B    | A和B 相乘      |
| A/B    | A除以B         |
| A%B    | A对B取余       |
| A&B    | A和B按位取与   |
| A\|B   | A和B按位取或   |
| A^B    | A和B按位取异或 |
| ~A     | A按位取反      |

案例实操：查询出所有员工的薪水后加1显示。

```mysql
hive (default)> select sal +1 from emp;
```

#### 6.1.4 常用函数

**1）求总行数（count）**

```mysql
hive (default)> select count(*) cnt from emp;
```

**2）求工资的最大值（max）**

```
hive (default)> select max(sal) max_sal from emp;
```

**3）求工资的最小值（min）**

```
hive (default)> select min(sal) min_sal from emp;
```

**4）求工资的总和（sum）**

```
hive (default)> select sum(sal) sum_sal from emp; 
```

**5）求工资的平均值（avg）**

```
hive (default)> select avg(sal) avg_sal from emp;
```

#### 6.1.5 Limit语句

典型的查询会返回多行数据。LIMIT子句用于限制返回的行数。

```
hive (default)> select * from emp limit 5;
hive (default)> select * from emp limit 2,3;
```

#### 6.1.6 Where语句

**1）使用WHERE子句，将不满足条件的行过滤掉**

**2）WHERE子句紧随FROM子句**

**3）案例实操**

查询出薪水大于1000的所有员工

```
hive (default)> select * from emp where sal >1000;
```

注意：where子句中不能使用字段别名。

#### 6.1.7 比较运算符（Between/In/ Is Null）

**1）下面表中描述了谓词操作符，这些操作符同样可以用于JOIN…ON和HAVING语句中。**

| 操作符                   | 支持的数据类型 | 描述                                                         |
| ------------------------ | -------------- | ------------------------------------------------------------ |
| A=B                      | 基本数据类型   | 如果A等于B则返回TRUE，反之返回FALSE                          |
| A<=>B                    | 基本数据类型   | 如果A和B都为NULL，则返回TRUE，如果一边为NULL，返回False      |
| A<>B, A!=B               | 基本数据类型   | A或者B为NULL则返回NULL；如果A不等于B，则返回TRUE，反之返回FALSE |
| A<B                      | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于B，则返回TRUE，反之返回FALSE |
| A<=B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于等于B，则返回TRUE，反之返回FALSE |
| A>B                      | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于B，则返回TRUE，反之返回FALSE |
| A>=B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于等于B，则返回TRUE，反之返回FALSE |
| A [NOT] BETWEEN B  AND C | 基本数据类型   | 如果A，B或者C任一为NULL，则结果为NULL。如果A的值大于等于B而且小于或等于C，则结果为TRUE，反之为FALSE。如果使用NOT关键字则可达到相反的效果。 |
| A IS NULL                | 所有数据类型   | 如果A等于NULL，则返回TRUE，反之返回FALSE                     |
| A IS NOT NULL            | 所有数据类型   | 如果A不等于NULL，则返回TRUE，反之返回FALSE                   |
| IN(数值1, 数值2)         | 所有数据类型   | 使用 IN运算显示列表中的值                                    |
| A [NOT] LIKE B           | STRING 类型    | B是一个SQL下的简单正则表达式，也叫通配符模式，如果A与其匹配的话，则返回TRUE；反之返回FALSE。B的表达式说明如下：‘x%’表示A必须以字母‘x’开头，‘%x’表示A必须以字母’x’结尾，而‘%x%’表示A包含有字母’x’,可以位于开头，结尾或者字符串中间。如果使用NOT关键字则可达到相反的效果。 |
| A RLIKE B, A  REGEXP B   | STRING 类型    | B是基于java的正则表达式，如果A与其匹配，则返回TRUE；反之返回FALSE。匹配使用的是JDK中的正则表达式接口实现的，因为正则也依据其中的规则。例如，正则表达式必须和整个字符串A相匹配，而不是只需与其字符串匹配。 |

**2）案例实操**

（1）查询出薪水等于5000的所有员工

```mysql
hive (default)> select * from emp where sal =5000;
```

（2）查询工资在500到1000的员工信息

```mysql
hive (default)> select * from emp where sal between 500 and 1000;
```

（3）查询comm为空的所有员工信息

```mysql
hive (default)> select * from emp where comm is null;
```

（4）查询工资是1500或5000的员工信息

```mysql
hive (default)> select * from emp where sal IN (1500, 5000);
```

#### 6.1.8 Like和RLike

**1）使用LIKE运算选择类似的值**

**2）选择条件可以包含字符或数字:**

% 代表零个或多个字符(任意个字符)。

_ 代表一个字符。

**3）RLIKE子句**

RLIKE子句是Hive中这个功能的一个扩展，其可以通过Java的正则表达式这个更强大的语言来指定匹配条件。

**4）案例实操**

（1）查找名字以A开头的员工信息

```mysql
hive (default)> select * from emp where ename LIKE 'A%';
```

（2）查找名字中第二个字母为A的员工信息

```mysql
hive (default)> select * from emp where ename LIKE '_A%';
```

（3）查找名字中带有A的员工信息

```mysql
hive (default)> select * from emp where ename  RLIKE '[A]';
```

#### 6.1.9 逻辑运算符（And/Or/Not）

| 操作符 | 含义   |
| ------ | ------ |
| AND    | 逻辑并 |
| OR     | 逻辑或 |
| NOT    | 逻辑否 |

**1）案例实操**

（1）查询薪水大于1000，部门是30

```mysql
hive (default)> select * from emp where sal>1000 and deptno=30;
```

（2）查询薪水大于1000，或者部门是30

```mysql
hive (default)> select * from emp where sal>1000 or deptno=30;
```

（3）查询除了20部门和30部门以外的员工信息

```mysql
hive (default)> select * from emp where deptno not IN(30, 20);
```

### 6.2 分组

#### 6.2.1 Group By语句

GROUP BY语句通常会和聚合函数一起使用，按照一个或者多个列队结果进行分组，然后对每个组执行聚合操作。

**1）案例实操：**

（1）计算emp表每个部门的平均工资

```mysql
hive (default)> select t.deptno, avg(t.sal) avg_sal from emp t group by t.deptno;
```

（2）计算emp每个部门中每个岗位的最高薪水

```mysql
hive (default)> select t.deptno, t.job, max(t.sal) max_sal from emp t group by t.deptno, t.job;
```

#### 6.2.2 Having语句

**1）having与where不同点**

（1）where后面不能写分组函数，而having后面可以使用分组函数。

（2）having只用于group by分组统计语句。

**2）案例实操**

（1）求每个部门的平均薪水大于2000的部门

求每个部门的平均工资

```mysql
hive (default)> select deptno, avg(sal) from emp group by deptno;
```

求每个部门的平均薪水大于2000的部门

```mysql
hive (default)> select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal > 2000;
```

### 6.3 Join语句

#### 6.3.1 等值Join

Hive支持通常的SQL JOIN语句。 

**1）案例实操**

（1）根据员工表和部门表中的部门编号相等，查询员工编号、员工名称和部门名称；

```
hive (default)> select e.empno, e.ename, d.deptno, d.dname from emp e join dept d on e.deptno = d.deptno;
```

#### 6.3.2 表的别名

**1）好处**

（1）使用别名可以简化查询。

（2）使用表名前缀可以提高执行效率。

**2）案例实操**

合并员工表和部门表

```mysql
hive (default)> select e.empno, e.ename, d.deptno from emp e join dept d on e.deptno = d.deptno;
```

#### 6.3.3 内连接

内连接：只有进行连接的两个表中都存在与连接条件相匹配的数据才会被保留下来。

```mysql
hive (default)> select e.empno, e.ename, d.deptno from emp e join dept d on e.deptno = d.deptno;
```

#### 6.3.4 左外连接

左外连接：JOIN操作符左边表中符合WHERE子句的所有记录将会被返回。

```mysql
hive (default)> select e.empno, e.ename, d.deptno from emp e left join dept d on e.deptno = d.deptno;
```

#### 6.3.5 右外连接

右外连接：JOIN操作符右边表中符合WHERE子句的所有记录将会被返回。

```mysql
hive (default)> select e.empno, e.ename, d.deptno from emp e right join dept d on e.deptno = d.deptno;
```

#### 6.3.6 满外连接

满外连接：将会返回所有表中符合WHERE语句条件的所有记录。如果任一表的指定字段没有符合条件的值的话，那么就使用NULL值替代。

```
hive (default)> select e.empno, e.ename, d.deptno from emp e full join dept d on e.deptno = d.deptno;
```

#### 6.3.7 多表连接

注意：连接 n个表，至少需要n-1个连接条件。例如：连接三个表，至少需要两个连接条件。

数据准备

```
1700	Beijing
1800	London
1900	Tokyo
```

**1）创建位置表**

```mysql
create table if not exists location(
loc int,
loc_name string
)
row format delimited fields terminated by '\t';
```

**2）导入数据**

```mysql
hive (default)> load data local inpath '/opt/module/datas/location.txt' into table location;
```

**3）多表连接查询**

```mysql
hive (default)>SELECT e.ename, d.dname, l.loc_name
FROM   emp e 
JOIN   dept d
ON     d.deptno = e.deptno 
JOIN   location l
ON     d.loc = l.loc;
```

大多数情况下，Hive会对每对JOIN连接对象启动一个MapReduce任务。本例中会首先启动一个MapReduce job对表e和表d进行连接操作，然后会再启动一个MapReduce job将第一个MapReduce job的输出和表l;进行连接操作。

注意：为什么不是表d和表l先进行连接操作呢？这是因为Hive总是按照从左到右的顺序执行的。

优化：当对3个或者更多表进行join连接时，如果每个on子句都使用相同的连接键的话，那么只会产生一个MapReduce job。

#### 6.3.8 笛卡尔积

**1）笛卡尔积会在下面条件下产生**

（1）省略连接条件

（2）连接条件无效

（3）所有表中的所有行互相连接

**2）案例实操**

```mysql
hive (default)> select empno, dname from emp, dept;
```

### 6.4 排序

#### 6.4.1 全局排序（Order By）

Order By：全局排序，只有一个Reducer

**1）使用 ORDER BY子句排序**

ASC（ascend）: 升序（默认）

DESC（descend）: 降序

**2）ORDER BY 子句在SELECT语句的结尾**

**3）案例实操** 

（1）查询员工信息按工资升序排列

```mysql
hive (default)> select * from emp order by sal;
```

（2）查询员工信息按工资降序排列

```mysql
hive (default)> select * from emp order by sal desc;
```

#### 6.4.2 按照别名排序

按照员工薪水的2倍排序

```mysql
hive (default)> select ename, sal*2 twosal from emp order by twosal;
```

#### 6.4.3 多个列排序

按照部门和工资升序排序

```mysql
hive (default)> select ename, deptno, sal from emp order by deptno, sal ;
```

#### 6.4.4 每个Reduce内部排序（Sort By）

Sort By：对于大规模的数据集order by的效率非常低。在很多情况下，并不需要全局排序，此时可以使用sort by。

Sort by为每个reducer产生一个排序文件。每个Reducer内部进行排序，对全局结果集来说不是排序。

**1）设置reduce个数**

```mysql
hive (default)> set mapreduce.job.reduces=3;
```

**2）查看设置reduce个数**

```mysql
hive (default)> set mapreduce.job.reduces;
```

**3）根据部门编号降序查看员工信息**

```mysql
hive (default)> select * from emp sort by deptno desc;
```

**4）将查询结果导入到文件中（按照部门编号降序排序）**

```
hive (default)> insert overwrite local directory '/opt/module/hive/datas/sortby-result' select * from emp sort by deptno desc;
```

#### 6.4.5 分区（Distribute By）

Distribute By： 在有些情况下，我们需要控制某个特定行应该到哪个reducer，通常是为了进行后续的聚集操作。**distribute by** 子句可以做这件事。**distribute by**类似MR中partition（自定义分区），进行分区，结合sort by使用。 

对于distribute by进行测试，一定要分配多reduce进行处理，否则无法看到distribute by的效果。

**1）案例实操：**

（1）先按照部门编号分区，再按照员工编号降序排序。

```
hive (default)> set mapreduce.job.reduces=3;
hive (default)> insert overwrite local directory '/opt/module/hive/datas/distribute-result' select * from emp distribute by deptno sort by empno desc;
```

注意：

- distribute by的分区规则是根据分区字段的hash码与reduce的个数进行模除后，余数相同的分到一个区。

- Hive要求DISTRIBUTE BY语句要写在SORT BY语句之前。

#### 6.4.6 Cluster By

当distribute by和sort by字段相同时，可以使用cluster by方式。

cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是升序排序，不能指定排序规则为ASC或者DESC。

（1）以下两种写法等价

```mysql
hive (default)> select * from emp cluster by deptno;
hive (default)> select * from emp distribute by deptno sort by deptno;
```

注意：按照部门编号分区，不一定就是固定死的数值，可以是20号和30号部门分到一个分区里面去。

## 7 分区表和分桶表

### 7.1 分区表

分区表实际上就是对应一个HDFS文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。Hive中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。

#### 7.1.1 分区表基本操作

**1）引入分区表（需要根据日期对日志进行管理,通过部门信息模拟）**

```
dept_20200401.log
dept_20200402.log
dept_20200403.log
……
```

**2）创建分区表语法**

```mysql
hive (default)> create table dept_partition(
deptno int, dname string, loc string
)
partitioned by (day string)
row format delimited fields terminated by '\t';
```

注意：分区字段不能是表中已经存在的数据，可以将分区字段看作表的伪列。

**3）加载数据到分区表中**

（1）数据准备

dept_20200401.log

```
10	ACCOUNTING	1700
20	RESEARCH	1800
```

dept_20200402.log

```
30	SALES	1900
40	OPERATIONS	1700
```

dept_20200403.log

```
50	TEST	2000
60	DEV		1900
```

（2）加载数据

```mysql
hive (default)> load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table dept_partition partition(day='20200401');
hive (default)> load data local inpath '/opt/module/hive/datas/dept_20200402.log' into table dept_partition partition(day='20200402');
hive (default)> load data local inpath '/opt/module/hive/datas/dept_20200403.log' into table dept_partition partition(day='20200403');
```

注意：分区表加载数据时，必须指定分区

![](E:\learning\04_java\01_笔记\BigData\03_Hive\picture\图分区表.png)

**4）查询分区表中数据**

单分区查询

```mysql
hive (default)> select * from dept_partition where day='20200401';
```

多分区联合查询

```mysql
hive (default)> select * from dept_partition where day='20200401'
              union
              select * from dept_partition where day='20200402'
              union
              select * from dept_partition where day='20200403';
hive (default)> select * from dept_partition where day='20200401' or
                day='20200402' or day='20200403' ;			
```

**5）增加分区**

创建单个分区

```mysql
hive (default)> alter table dept_partition add partition(day='20200404') ;
```

同时创建多个分区

```mysql
hive (default)> alter table dept_partition add partition(day='20200405') partition(day='20200406');
```

**6）删除分区**

删除单个分区

```mysql
hive (default)> alter table dept_partition drop partition (day='20200406');
```

同时删除多个分区

```mysql
hive (default)> alter table dept_partition drop partition (day='20200404'), partition(day='20200405');
```

**7）查看分区表有多少分区**

```
hive> show partitions dept_partition;
```

**8）查看分区表结构**

```
hive> desc formatted dept_partition;

# Partition Information          
# col_name              data_type               comment             
month                   string  
```

#### 7.1.2 二级分区

思考: 如何一天的日志数据量也很大，如何再将数据拆分?

**1）创建二级分区表**

```
hive (default)> create table dept_partition2(
               deptno int, dname string, loc string
               )
               partitioned by (day string, hour string)
               row format delimited fields terminated by '\t';
```

**2）正常的加载数据**

（1）加载数据到二级分区表中

```mysql
hive (default)> load data local inpath '/opt/module`/hive/datas/dept_20200401.log' into table
dept_partition2 partition(day='20200401', hour='12');
```

（2）查询分区数据

```
hive (default)> select * from dept_partition2 where day='20200401' and hour='12';
```

**3）把数据直接上传到分区目录上，让分区表和数据产生关联的三种方式**

（1）方式一：上传数据后修复

```shell
hive (default)> dfs -mkdir -p
 /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=13;
hive (default)> dfs -put /opt/module/datas/dept_20200401.log  /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=13;
```

查询数据（查询不到刚上传的数据）

```mysql
hive (default)> select * from dept_partition2 where day='20200401' and hour='13';
```

执行修复命令

```mysql
hive> msck repair table dept_partition2;
```

再次查询数据

```
hive (default)> select * from dept_partition2 where day='20200401' and hour='13';
```

（2）方式二：上传数据后添加分区

上传数据

```mysql
hive (default)> dfs -mkdir -p
 /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=14;
hive (default)> dfs -put /opt/module/hive/datas/dept_20200401.log  /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=14;
```

执行添加分区

```
hive (default)> alter table dept_partition2 add partition(day='201709',hour='14');
```

查询数据

```mysql
hive (default)> select * from dept_partition2 where day='20200401' and hour='14';
```

（3）方式三：创建文件夹后load数据到分区

创建目录

```mysql
hive (default)> dfs -mkdir -p
 /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=15;
```

上传数据

```mysql
hive (default)> load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table
 dept_partition2 partition(day='20200401',hour='15');
```

查询数据

```mysql
hive (default)> select * from dept_partition2 where day='20200401' and hour='15';
```

#### 7.1.3 动态分区

关系型数据库中，对分区表Insert数据时候，数据库自动会根据分区字段的值，将数据插入到相应的分区中，Hive中也提供了类似的机制，即动态分区(Dynamic Partition)，只不过，使用Hive的动态分区，需要进行相应的配置。

**1）开启动态分区参数设置**

（1）开启动态分区功能（默认true，开启）

```
hive.exec.dynamic.partition=true
```

（2）设置为非严格模式（动态分区的模式，默认strict，表示必须指定至少一个分区为静态分区，nonstrict模式表示允许所有的分区字段都可以使用动态分区。）

```
hive.exec.dynamic.partition.mode=nonstrict
```

（3）在所有执行MR的节点上，最大一共可以创建多少个动态分区。默认1000

```
hive.exec.max.dynamic.partitions=1000
```

（4）在每个执行MR的节点上，最大可以创建多少个动态分区。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。

```
hive.exec.max.dynamic.partitions.pernode=100
```

（5）整个MR Job中，最大可以创建多少个HDFS文件。默认100000

```
hive.exec.max.created.files=100000
```

（6）当有空分区生成时，是否抛出异常。一般不需要设置。默认false

```
hive.error.on.empty.partition=false
```

**2）案例实操**

需求：将dept表中的数据按照地区（loc字段），插入到目标表dept_partition的相应分区中。

（1）创建目标分区表

```
hive (default)> create table dept_partition_dy(id int, name string) partitioned by (loc int) row format delimited fields terminated by '\t';
```

（2）设置动态分区

```
set hive.exec.dynamic.partition.mode = nonstrict;
hive (default)> insert into table dept_partition_dy partition(loc) select deptno, dname, loc from dept;
```

（3）查看目标分区表的分区情况

```
hive (default)> show partitions dept_partition;
```

思考：目标分区表是如何匹配到分区字段的？

### 7.3 抽样查询

对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果。Hive可以通过对表进行抽样来满足这个需求。

语法: TABLESAMPLE(BUCKET x OUT OF y) 

查询表stu_buck中的数据。

```
hive (default)> select * from stu_buck tablesample(bucket 1 out of 4 on id);
```

注意：x的值必须小于等于y的值，否则

```
FAILED: SemanticException [Error 10061]: Numerator should not be bigger than denominator in sample clause for table stu_buck
```

## 8 函数

### 8.1 系统内置函数

**1）查看系统自带的函数**

```
hive> show functions;
```

**2）显示自带的函数的用法**

```
hive> desc function upper;
```

**3）详细显示自带的函数的用法**

```
hive> desc function extended upper;
```

### 8.2 常用内置函数

#### 8.2.1 空字段赋值

**1）函数说明**

NVL：给值为NULL的数据赋值，它的格式是NVL( value，default_value)。它的功能是如果value为NULL，则NVL函数返回default_value的值，否则返回value的值，如果两个参数都为NULL ，则返回NULL。

**2）数据准备：采用员工表**

**3）查询：如果员工的comm为NULL，则用-1代替**

```
hive (default)> select comm,nvl(comm, -1) from emp;
OK
comm    _c1
NULL    -1.0
300.0   300.0
500.0   500.0
NULL    -1.0
1400.0  1400.0
NULL    -1.0
NULL    -1.0
NULL    -1.0
NULL    -1.0
0.0     0.0
NULL    -1.0
NULL    -1.0
NULL    -1.0
NULL    -1.0
```

**4）查询：如果员工的comm为NULL，则用领导id代替**

```
hive (default)> select comm, nvl(comm,mgr) from emp;
OK
comm    _c1
NULL    7902.0
300.0   300.0
500.0   500.0
NULL    7839.0
1400.0  1400.0
NULL    7839.0
NULL    7839.0
NULL    7566.0
NULL    NULL
0.0     0.0
NULL    7788.0
NULL    7698.0
NULL    7566.0
NULL    7782.0
```

#### 8.2.2 CASE WHEN THEN ELSE END

**1）数据准备**

| name | dept_id | sex  |
| ---- | ------- | ---- |
| 悟空 | A       | 男   |
| 大海 | A       | 男   |
| 宋宋 | B       | 男   |
| 凤姐 | A       | 女   |
| 婷姐 | B       | 女   |
| 婷婷 | B       | 女   |

**2）需求**

求出不同部门男女各多少人。结果如下：

```
dept_Id     男       女
A     		2       1
B     		1       2
```

**3）创建本地emp_sex.txt，导入数据**

```
 vi emp_sex.txt
悟空	A	男
大海	A	男
宋宋	B	男
凤姐	A	女
婷姐	B	女
婷婷	B	女
```

**4）创建hive表并导入数据**

```
create table emp_sex(
name string, 
dept_id string, 
sex string) 
row format delimited fields terminated by "\t";
load data local inpath '/opt/module/hive/datas/emp_sex.txt' into table emp_sex;
```

**5）按需求查询数据**

```
select 
  dept_id,
  sum(case sex when '男' then 1 else 0 end) male_count,
  sum(case sex when '女' then 1 else 0 end) female_count
from 
  emp_sex
group by
  dept_id;
```

#### 8.2.3 行转列

**1）相关函数说明**

CONCAT(string A/col, string B/col…)：返回输入字符串连接后的结果，支持任意个输入字符串;

CONCAT_WS(separator, str1, str2,...)：它是一个特殊形式的 CONCAT()。第一个参数剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间;

注意: CONCAT_WS must be "string or array<string>

COLLECT_SET(col)：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段。

**2）数据准备**

| name   | constellation | blood_type |
| ------ | ------------- | ---------- |
| 孙悟空 | 白羊座        | A          |
| 大海   | 射手座        | A          |
| 宋宋   | 白羊座        | B          |
| 猪八戒 | 白羊座        | A          |
| 凤姐   | 射手座        | A          |
| 苍老师 | 白羊座        | B          |

**3）需求**

把星座和血型一样的人归类到一起。结果如下：

```
射手座,A            大海|凤姐
白羊座,A            孙悟空|猪八戒
白羊座,B            宋宋|苍老师
```

**4）创建本地constellation.txt，导入数据**

```
vim person_info.txt
孙悟空	白羊座	A
大海	射手座	A
宋宋	白羊座	B
猪八戒	白羊座	A
凤姐	射手座	A
苍老师	白羊座	B
```

**5）创建hive表并导入数据**

```
create table person_info(
name string, 
constellation string, 
blood_type string) 
row format delimited fields terminated by "\t";
load data local inpath "/opt/module/hive/datas/person_info.txt" into table person_info;
```

**6）按需求查询数据**

```
SELECT t1.c_b , CONCAT_WS("|",collect_set(t1.name))
FROM (
SELECT NAME ,CONCAT_WS(',',constellation,blood_type) c_b
FROM person_info
)t1 
GROUP BY t1.c_b
```

#### 8.2.4 列转行

**1）函数说明**

EXPLODE(col)：将hive一列中复杂的array或者map结构拆分成多行。

LATERAL VIEW

用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias

解释：用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

**2）数据准备**

表6-7 数据准备

| movie         | category                 |
| ------------- | ------------------------ |
| 《疑犯追踪》  | 悬疑,动作,科幻,剧情      |
| 《Lie to me》 | 悬疑,警匪,动作,心理,剧情 |
| 《战狼2》     | 战争,动作,灾难           |

**3）需求**

将电影分类中的数组数据展开。结果如下：

```
《疑犯追踪》      悬疑
《疑犯追踪》      动作
《疑犯追踪》      科幻
《疑犯追踪》      剧情
《Lie to me》   悬疑
《Lie to me》   警匪
《Lie to me》   动作
《Lie to me》   心理
《Lie to me》   剧情
《战狼2》        战争
《战狼2》        动作
《战狼2》        灾难
```

**4）创建本地movie.txt，导入数据**

```
[atguigu@hadoop102 datas]$ vi movie_info.txt
《疑犯追踪》	悬疑,动作,科幻,剧情
《Lie to me》	悬疑,警匪,动作,心理,剧情
《战狼2》	战争,动作,灾难
```

**5）创建hive表并导入数据**

```
create table movie_info(
    movie string, 
    category string) 
row format delimited fields terminated by "\t";
load data local inpath "/opt/module/hive/datas/movie_info.txt" into table movie_info;
```

**6）按需求查询数据**

```
SELECT movie,category_name 
FROM movie_info 
lateral VIEW
explode(split(category,",")) movie_info_tmp  AS category_name ;
```

#### 8.2.5 窗口函数（开窗函数）

**1）相关函数说明**

OVER()：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的改变而变化。

CURRENT ROW：当前行

n PRECEDING：往前n行数据

n FOLLOWING：往后n行数据

UNBOUNDED：起点，

​    UNBOUNDED PRECEDING 表示从前面的起点， 

   UNBOUNDED FOLLOWING表示到后面的终点

LAG(col,n,default_val)：往前第n行数据

LEAD(col,n, default_val)：往后第n行数据

NTILE(n)：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。

**2）数据准备：name，orderdate，cost**

```
jack,2017-01-01,10
tony,2017-01-02,15
jack,2017-02-03,23
tony,2017-01-04,29
jack,2017-01-05,46
jack,2017-04-06,42
tony,2017-01-07,50
jack,2017-01-08,55
mart,2017-04-08,62
mart,2017-04-09,68
neil,2017-05-10,12
mart,2017-04-11,75
neil,2017-06-12,80
mart,2017-04-13,94
```

**3）需求**

（1）查询在2017年4月份购买过的顾客及总人数

（2）查询顾客的购买明细及月购买总额

（3）上述的场景, 将每个顾客的cost按照日期进行累加

（4）查询每个顾客上次的购买时间

（5）查询前20%时间的订单信息

**4）创建本地business.txt，导入数据**

```
[atguigu@hadoop102 datas]$ vi business.txt
```

**5）创建hive表并导入数据**

```
create table business(
name string, 
orderdate string,
cost int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
load data local inpath "/opt/module/hive/datas/business.txt" into table business;
```

**6）按需求查询数据**

（1）查询在2017年4月份购买过的顾客及总人数

```
select name,count(*) over () 
from business 
where substring(orderdate,1,7) = '2017-04' 
group by name;
```

（2）查询顾客的购买明细及月购买总额

```
select name,orderdate,cost,sum(cost) over(partition by month(orderdate)) from business;
```

（3）将每个顾客的cost按照日期进行累加

```
select name,orderdate,cost, 
sum(cost) over() as sample1,--所有行相加 
sum(cost) over(partition by name) as sample2,--按name分组，组内数据相加 
sum(cost) over(partition by name order by orderdate) as sample3,--按name分组，组内数据累加 
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row ) as sample4 ,--和sample3一样,由起点到当前行的聚合 
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING and current row) as sample5, --当前行和前面一行做聚合 
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING AND 1 FOLLOWING ) as sample6,--当前行和前边一行及后面一行 
sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行 
from business;
```

rows必须跟在Order by 子句之后，对排序的结果进行限制，使用固定的行数来限制分区中的数据行数量

（4）查看顾客上次的购买时间

```
select name,orderdate,cost, 
lag(orderdate,1,'1900-01-01') over(partition by name order by orderdate ) as time1, lag(orderdate,2) over (partition by name order by orderdate) as time2 
from business;
```

（5）查询前20%时间的订单信息

```
select * from (
    select name,orderdate,cost, ntile(5) over(order by orderdate) sorted
    from business
) t
where sorted = 1;
```

#### 8.2.6 Rank

**1）函数说明**

RANK() 排序相同时会重复，总数不会变

DENSE_RANK() 排序相同时会重复，总数会减少

ROW_NUMBER() 会根据顺序计算

**2）数据准备**

| name   | subject | score |
| ------ | ------- | ----- |
| 孙悟空 | 语文    | 87    |
| 孙悟空 | 数学    | 95    |
| 孙悟空 | 英语    | 68    |
| 大海   | 语文    | 94    |
| 大海   | 数学    | 56    |
| 大海   | 英语    | 84    |
| 宋宋   | 语文    | 64    |
| 宋宋   | 数学    | 86    |
| 宋宋   | 英语    | 84    |
| 婷婷   | 语文    | 65    |
| 婷婷   | 数学    | 85    |
| 婷婷   | 英语    | 78    |

**3）需求**

计算每门学科成绩排名。

**4）创建本地score.txt，导入数据**

```
[atguigu@hadoop102 datas]$ vi score.txt
```

**5）创建hive表并导入数据**

```
create table score(
name string,
subject string, 
score int) 
row format delimited fields terminated by "\t";
load data local inpath '/opt/module/hive/datas/score.txt' into table score;
```

**6）按需求查询数据**

```
select name,
subject,
score,
rank() over(partition by subject order by score desc) rp,
dense_rank() over(partition by subject order by score desc) drp,
row_number() over(partition by subject order by score desc) rmp
from score;

name    subject score   rp      drp     rmp
孙悟空  数学    95      1       1       1
宋宋    数学    86      2       2       2
婷婷    数学    85      3       3       3
大海    数学    56      4       4       4
宋宋    英语    84      1       1       1
大海    英语    84      1       1       2
婷婷    英语    78      3       2       3
孙悟空  英语    68      4       3       4
大海    语文    94      1       1       1
孙悟空  语文    87      2       2       2
婷婷    语文    65      3       3       3
宋宋    语文    64      4       4       4
```

扩展：求出每门学科前三名的学生？

#### 8.2.7 其他常用函数

### 8.3 自定义函数

**1）**Hive 自带了一些函数，比如：max/min等，但是数量有限，自己可以通过自定义UDF来方便的扩展。

**2）**当Hive提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义函数（UDF：user-defined function）。

**3）**根据用户自定义函数类别分为以下三种：

（1）UDF（User-Defined-Function）

​    一进一出

（2）UDAF（User-Defined Aggregation Function）

​    聚集函数，多进一出

​    类似于：count/max/min

（3）UDTF（User-Defined Table-Generating Functions）

​    一进多出

​    如lateral view explode()

**4）**官方文档地址

https://cwiki.apache.org/confluence/display/Hive/HivePlugins

**5）**编程步骤：

（1）继承Hive提供的类

​        org.apache.hadoop.hive.ql.udf.generic.GenericUDF  

​         org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;

（2）实现类中的抽象方法

（3）在hive的命令行窗口创建函数

添加jar

```
add jar linux_jar_path
```

创建function

```
create [temporary] function [dbname.]function_name AS class_name;
```

（4）在hive的命令行窗口删除函数

```
drop [temporary] function [if exists] [dbname.]function_name;
```

### 8.4 自定义UDF函数

**0）需求:**

自定义一个UDF实现计算给定字符串的长度，例如：

```
hive(default)> select my_len("abcd");
4
```

**1）创建一个Maven工程Hive**

**2）导入依赖**

```
<dependencies>
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-exec</artifactId>
			<version>3.1.2</version>
		</dependency>
</dependencies>
```

**3）创建一个类**

```java
package com.atguigu.hive;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentLengthException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

/**
 * 自定义UDF函数，需要继承GenericUDF类
 * 需求: 计算指定字符串的长度
 */
public class MyStringLength extends GenericUDF {
    /**
     *
     * @param arguments 输入参数类型的鉴别器对象
     * @return 返回值类型的鉴别器对象
     * @throws UDFArgumentException
     */
    @Override
    public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
        // 判断输入参数的个数
        if(arguments.length !=1){
            throw new UDFArgumentLengthException("Input Args Length Error!!!");
        }
        // 判断输入参数的类型
        if(!arguments[0].getCategory().equals(ObjectInspector.Category.PRIMITIVE)){
            throw new UDFArgumentTypeException(0,"Input Args Type Error!!!");
        }
        //函数本身返回值为int，需要返回int类型的鉴别器对象
        return PrimitiveObjectInspectorFactory.javaIntObjectInspector;
    }

    /**
     * 函数的逻辑处理
     * @param arguments 输入的参数
     * @return 返回值
     * @throws HiveException
     */
    @Override
    public Object evaluate(DeferredObject[] arguments) throws HiveException {
       if(arguments[0].get() == null){
           return 0 ;
       }
       return arguments[0].get().toString().length();
    }

    @Override
    public String getDisplayString(String[] children) {
        return "";
    }
}
```

**4）打成jar包上传到服务器/opt/module/hive/datas/myudf.jar**

**5）将jar包添加到hive的classpath**

```
hive (default)> add jar /opt/module/hive/datas/myudf.jar;
```

**6）创建临时函数与开发好的java class关联**

```
hive (default)> create temporary function my_len as "com.atguigu.hive. MyStringLength";
```

**7）即可在hql中使用自定义的函数** 

```
hive (default)> select ename,my_len(ename) ename_len from emp;
```

### 8.5 自定义UDTF函数

**0）需求** 

自定义一个UDTF实现将一个任意分割符的字符串切割成独立的单词，例如：

```
hive(default)> select myudtf("hello,world,hadoop,hive", ",");

hello
world
hadoop
hive
```

**1）代码实现**

```
package com.atguigu.udtf;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

import java.util.ArrayList;
import java.util.List;

public class MyUDTF extends GenericUDTF {

    private ArrayList<String> outList = new ArrayList<>();

    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {


        //1.定义输出数据的列名和类型
        List<String> fieldNames = new ArrayList<>();
        List<ObjectInspector> fieldOIs = new ArrayList<>();

        //2.添加输出数据的列名和类型
        fieldNames.add("lineToWord");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames, fieldOIs);
    }

    @Override
    public void process(Object[] args) throws HiveException {
        
        //1.获取原始数据
        String arg = args[0].toString();

        //2.获取数据传入的第二个参数，此处为分隔符
        String splitKey = args[1].toString();

        //3.将原始数据按照传入的分隔符进行切分
        String[] fields = arg.split(splitKey);

        //4.遍历切分后的结果，并写出
        for (String field : fields) {

            //集合为复用的，首先清空集合
            outList.clear();

            //将每一个单词添加至集合
            outList.add(field);

            //将集合内容写出
            forward(outList);
        }
    }

    @Override
    public void close() throws HiveException {

    }
}
```

**2）打成jar包上传到服务器/opt/module/hive/data/myudtf.jar**

**3）将jar包添加到hive的classpath下**

```
hive (default)> add jar /opt/module/hive/data/myudtf.jar;
```

**4）创建临时函数与开发好的java class关联**

```
hive (default)> create temporary function myudtf as "com.atguigu.hive.MyUDTF";
```

**5）使用自定义的函数**

```
hive (default)> select myudtf("hello,world,hadoop,hive",",") ;
```

## 9 压缩和存储

### 9.1 Hadoop压缩配置

#### 9.1.1 MR支持的压缩编码

| 压缩格式 | 算法    | 文件扩展名 | 是否可切分 |
| -------- | ------- | ---------- | ---------- |
| DEFLATE  | DEFLATE | .deflate   | 否         |
| Gzip     | DEFLATE | .gz        | 否         |
| bzip2    | bzip2   | .bz2       | 是         |
| LZO      | LZO     | .lzo       | 是         |
| Snappy   | Snappy  | .snappy    | 否         |

为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器，如下表所示：

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

压缩性能的比较：

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

http://google.github.io/snappy/

On a single core of a Core i7 processor in 64-bit mode, Snappy compresses at about 250 MB/sec or more and decompresses at about 500 MB/sec or more.

#### 9.1.2 压缩参数配置

|                       参数                       | 默认值                                                       | 阶段        | 建议                                         |
| :----------------------------------------------: | ------------------------------------------------------------ | ----------- | -------------------------------------------- |
| io.compression.codecs （在core-site.xml中配置）  | org.apache.hadoop.io.compress.DefaultCodec,  org.apache.hadoop.io.compress.GzipCodec,  org.apache.hadoop.io.compress.BZip2Codec,  org.apache.hadoop.io.compress.Lz4Codec | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器 |
|          mapreduce.map.output.compress           | false                                                        | mapper输出  | 这个参数设为true启用压缩                     |
|       mapreduce.map.output.compress.codec        | org.apache.hadoop.io.compress.DefaultCodec                   | mapper输出  | 使用LZO、LZ4或snappy编解码器在此阶段压缩数据 |
|    mapreduce.output.fileoutputformat.compress    | false                                                        | reducer输出 | 这个参数设为true启用压缩                     |
| mapreduce.output.fileoutputformat.compress.codec | org.apache.hadoop.io.compress. DefaultCodec                  | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2      |
| mapreduce.output.fileoutputformat.compress.type  | RECORD                                                       | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK  |

### 9.2 开启Map输出阶段压缩

开启map输出阶段压缩可以减少job中map和Reduce task间数据传输量。具体配置如下：

**1）案例实操：**

（1）开启hive中间传输数据压缩功能

```
hive (default)>set hive.exec.compress.intermediate=true;
```

（2）开启mapreduce中map输出压缩功能

```
hive (default)>set mapreduce.map.output.compress=true;
```

（3）设置mapreduce中map输出数据的压缩方式

```
hive (default)>set mapreduce.map.output.compress.codec= org.apache.hadoop.io.compress.SnappyCodec;
```

（4）执行查询语句

```
hive (default)> select count(ename) name from emp;
```

### 9.3 开启Reduce输出阶段压缩

当Hive将输出写入到表中时，输出内容同样可以进行压缩。属性hive.exec.compress.output控制着这个功能。用户可能需要保持默认设置文件中的默认值false，这样默认的输出就是非压缩的纯文本文件了。用户可以通过在查询语句或执行脚本中设置这个值为true，来开启输出结果压缩功能。

**1）案例实操：**

（1）开启hive最终输出数据压缩功能

```
hive (default)>set hive.exec.compress.output=true;
```

（2）开启mapreduce最终输出数据压缩

```
hive (default)>set mapreduce.output.fileoutputformat.compress=true;
```

（3）设置mapreduce最终数据输出压缩方式

```
hive (default)> set mapreduce.output.fileoutputformat.compress.codec = org.apache.hadoop.io.compress.SnappyCodec;
```

（4）设置mapreduce最终数据输出压缩为块压缩

```
hive (default)> set mapreduce.output.fileoutputformat.compress.type=BLOCK;
```

（5）测试一下输出结果是否是压缩文件

```
hive (default)> insert overwrite local directory
 '/opt/module/hive/datas/distribute-result' select * from emp distribute by deptno sort by empno desc;
```

### 9.4 文件存储格式

Hive支持的存储数据的格式主要有：TEXTFILE 、SEQUENCEFILE、ORC、PARQUET。

#### 9.4.1 列式存储和行式存储

![]()![列式存储和行式存储](E:\learning\04_java\01_笔记\BigData\03_Hive\picture\列式存储和行式存储.png)

如图所示左边为逻辑表，右边第一个为行式存储，第二个为列式存储。

**1）行存储的特点**

查询满足条件的一整行数据的时候，列存储则需要去每个聚集的字段找到对应的每个列的值，行存储只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快。

**2）列存储的特点**

因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能大大减少读取的数据量；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。

TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的；

ORC和PARQUET是基于列式存储的。

#### 9.4.2 TextFile格式

默认格式，数据不做压缩，磁盘开销大，数据解析开销大。可结合Gzip、Bzip2使用，但使用Gzip这种方式，hive不会对数据进行切分，从而无法对数据进行并行操作。

#### 9.4.3 Orc格式

Orc (Optimized Row Columnar)是Hive 0.11版里引入的新的存储格式。

如下图所示可以看到每个Orc文件由1个或多个stripe组成，每个stripe一般为HDFS的块大小，每一个stripe包含多条记录，这些记录按照列进行独立存储，对应到Parquet中的row group的概念。每个Stripe里有三部分组成，分别是Index Data，Row Data，Stripe Footer：

![](E:\learning\04_java\01_笔记\BigData\03_Hive\picture\Orc格式.jpg)

1）Index Data：一个轻量级的index，默认是每隔1W行做一个索引。这里做的索引应该只是记录某行的各字段在Row Data中的offset。

2）Row Data：存的是具体的数据，先取部分行，然后对这些行按列进行存储。对每个列进行了编码，分成多个Stream来存储。

3）Stripe Footer：存的是各个Stream的类型，长度等信息。

每个文件有一个File Footer，这里面存的是每个Stripe的行数，每个Column的数据类型信息等；每个文件的尾部是一个PostScript，这里面记录了整个文件的压缩类型以及FileFooter的长度信息等。在读取文件时，会seek到文件尾部读PostScript，从里面解析到File Footer长度，再读FileFooter，从里面解析到各个Stripe信息，再读各个Stripe，即从后往前读。

#### 9.4.4 Parquet格式

Parquet文件是以二进制方式存储的，所以是不可以直接读取的，文件中包括该文件的数据和元数据，因此Parquet格式文件是自解析的。

（1）行组(Row Group)：每一个行组包含一定的行数，在一个HDFS文件中至少存储一个行组，类似于orc的stripe的概念。

（2）列块(Column Chunk)：在一个行组中每一列保存在一个列块中，行组中的所有列连续的存储在这个行组文件中。一个列块中的值都是相同类型的，不同的列块可能使用不同的算法进行压缩。

（3）页(Page)：每一个列块划分为多个页，一个页是最小的编码的单位，在同一个列块的不同页可能使用不同的编码方式。

通常情况下，在存储Parquet数据的时候会按照Block大小设置行组的大小，由于一般情况下每一个Mapper任务处理数据的最小单位是一个Block，这样可以把每一个行组由一个Mapper任务处理，增大任务执行并行度。Parquet文件的格式。

![](E:\learning\04_java\01_笔记\BigData\03_Hive\picture\Parquet格式.png)

上图展示了一个Parquet文件的内容，一个文件中可以存储多个行组，文件的首位都是该文件的Magic Code，用于校验它是否是一个Parquet文件，Footer length记录了文件元数据的大小，通过该值和文件长度可以计算出元数据的偏移量，文件的元数据中包括每一个行组的元数据信息和该文件存储数据的Schema信息。除了文件中每一个行组的元数据，每一页的开始都会存储该页的元数据，在Parquet中，有三种类型的页：数据页、字典页和索引页。数据页用于存储当前行组中该列的值，字典页存储该列值的编码字典，每一个列块中最多包含一个字典页，索引页用来存储当前行组下该列的索引，目前Parquet中还不支持索引页。

#### 9.4.5 主流文件存储格式对比实验