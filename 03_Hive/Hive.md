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

```
hive (default)> select sal +1 from emp;
```

#### 6.1.4 常用函数