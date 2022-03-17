# MapReduce&Yarn

## 1 MapReduce概述

### 1.1 MapReduce定义

MapReduce是一个分布式运算程序的编程框架，是用户开发“基于Hadoop的数据分析应用”的核心框架。

MapReduce核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并行运行在一个Hadoop集群上。

### 1.2 MapReduce优缺点

#### 1.2.1 优点

1）MapReduce易于编程

它简单的实现一些接口，就可以完成一个分布式程序，这个分布式程序可以分布到大量廉价的PC机器上运行。也就是说你写一个分布式程序，跟写一个简单的串行程序是一模一样的。就是因为这个特点使得MapReduce编程变得非常流行。

2）良好的扩展性

当你的计算资源不能得到满足的时候，你可以通过简单的增加机器来扩展它的计算能力。

3）高容错性

MapReduce设计的初衷就是使程序能够部署在廉价的PC机器上，这就要求它具有很高的容错性。比如其中一台机器挂了，它可以把上面的计算任务转移到另外一个节点上运行，不至于这个任务运行失败，而且这个过程不需要人工参与，而完全是由Hadoop内部完成的。

4）适合PB级以上海量数据的离线处理

可以实现上千台服务器集群并发工作，提供数据处理能力。

#### 1.2.2 缺点

1）不擅长实时计算

MapReduce无法像MySQL一样，在毫秒或者秒级内返回结果。会产生很多IO操作。

2）不擅长流式计算

流式计算的输入数据是动态的，而MapReduce的输入数据集是静态的，不能动态变化。这是因为MapReduce自身的设计特点决定了数据源必须是静态的。

3）不擅长DAG（有向无环图）计算

多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，MapReduce并不是不能做，而是使用后，每个MapReduce作业的输出结果都会写入到磁盘，会造成大量的磁盘IO，导致性能非常的低下。

### 1.3 MapReduce核心思想

<img src="E:\learning\04_java\01_笔记\BigData\hadoop\picture\MapReduce核心编程思想.png" style="zoom: 33%;" />

（1）分布式的运算程序往往需要分成至少2个阶段。

（2）第一个阶段的MapTask并发实例，完全并行运行，互不相干。

（3）第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出。

（4）MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行。

总结：分析WordCount数据流走向深入理解MapReduce核心思想。

### 1.4 MapReduce进程

一个完整的MapReduce程序在分布式运行时有三类实例进程：

（1）MrAppMaster：负责整个程序的过程调度及状态协调。

（2）MapTask：负责Map阶段的整个数据处理流程。

（3）ReduceTask：负责Reduce阶段的整个数据处理流程。

### 1.5 官方WordCount源码

采用反编译工具反编译源码，发现WordCount案例有Map类、Reduce类和驱动类。且数据的类型是Hadoop自身封装的序列化类型。

### 1.6 常用数据序列化类型

| Java类型 | Hadoop Writable类型 |
| -------- | ------------------- |
| Boolean  | BooleanWritable     |
| Byte     | ByteWritable        |
| Int      | IntWritable         |
| Float    | FloatWritable       |
| Long     | LongWritable        |
| Double   | DoubleWritable      |
| String   | Text                |
| Map      | MapWritable         |
| Array    | ArrayWritable       |
| Null     | NullWritable        |

### 1.7 MapReduce编程规范

用户编写的程序分成三个部分：Mapper、Reducer和Driver。

1. Mapper阶段

   （1）用户自定义的Mapper要继承自己的父类（继承Hadoop提供的Mapper类）

   （2）Mapper的输入数据是KV对的形式（KV的类型可自定义）

   （3）Mapper中的业务逻辑写在map()方法中

   （4）Mapper的输出数据是KV对的形式（KV的类型可自定义）

   （5）map()方法（MapTask进程）对每一个<K,V>调用一次

2. Reducer阶段

   （1）用户自定义的Reducer要继承自己的父类（继承Hadoop提供的Reducer类）

   （2）Reducer的输入数据类型对应Mapper的输出数据类型，也是KV

   （3）Reducer的业务逻辑写在reduce()方法中

   （4）ReduceTask进程对每一组相同k的<k,v>组调用一次reduce()方法

3. Driver阶段

   相当于YARN集群的客户端，用于提交我们整个程序到YARN集群，提交的是封装了MapReduce程序相关运行参数的job对象

### 1.8 WordCount案例实操

**1）需求**

在给定的文本文件中统计输出每一个单词出现的总次数

**2）编程思路**

按照MapReduce编程规范，分别编写Mapper，Reducer，Driver。

**3）环境准备**

（1）创建maven工程

（2）在pom.xml文件中添加如下依赖

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.12.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>3.1.3</version>
    </dependency>
<dependencies>    
```

（3）在项目的`src/main/resources`目录下，新建一个文件，命名为`log4j2.xml`，在文件中填入。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="error" strict="true" name="XMLConfig">
    <Appenders>
        <!-- 类型名为Console，名称为必须属性 -->
        <Appender type="Console" name="STDOUT">
            <!-- 布局为PatternLayout的方式，
            输出样式为[INFO] [2018-01-22 17:34:01][org.test.Console]I'm here -->
            <Layout type="PatternLayout"
                    pattern="[%p] [%d{yyyy-MM-dd HH:mm:ss}][%c{10}]%m%n" />
        </Appender>
    </Appenders>
    
    <Loggers>
        <!-- 可加性为false -->
        <Logger name="test" level="info" additivity="false">
            <AppenderRef ref="STDOUT" />
        </Logger>

        <!-- root loggerConfig设置 -->
        <Root level="info">
            <AppenderRef ref="STDOUT" />
        </Root>
    </Loggers>

</Configuration>
```

**4）编写程序**

（1）编写Mapper类

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: Xu1Aan
 * @Date: 2022/03/15/20:15
 * @Description:
 * 以WordCount案例为例：
 * 自定义的Mapper类 需要继承Hadoop提供的Mapper 并且根据具体业务指定输入数据和输出数据的数据类型
 *
 * 输入数据类型
 * KEYIN,  读取文件的偏移量 数字(LongWritable)
 * VALUEIN, 读取文件的一行数据 文本(Text)
 * 输出数据类型
 * KEYOUT, 输出数据key的类型 就是一个单词(Text)
 * VALUEOUT 输出数据value的类型 给单词的标记1数字(IntWritable)
 */
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

    private Text outKey = new Text();

    private IntWritable outValue = new IntWritable(1);

    /**
     * Map端的核心处理方法，每输入一行数据会调用一次map方法
     * @param key 输入数据的key
     * @param value 输入数据的value
     * @param context 上下文对象
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context) throws IOException, InterruptedException {
        //获取当前输入的数据
        String line = value.toString();
        //切割数据
        String[] datas = line.split(" ");
        //遍历集合 封装 输出数据的key和value
        for (String data : datas) {
           outKey.set(data);
           context.write(outKey,outValue);
        }
    }
}
```

（2）编写Reducer类

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;


import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: Xu1Aan
 * @Date: 2022/03/15/20:15
 * @Description:
 * 以WordCount案例为例：
 * 自定义的Reducer类 需要继承Hadoop提供的Reducer 并且根据具体业务指定输入数据和输出数据的数据类型
 *
 * 输入数据类型
 * KEYIN,  Map端输出的key的数据类型(Text)
 * VALUEIN, Map端输出的value的数据类型(IntWritable)
 * 输出数据类型
 * KEYOUT, 输出数据key的类型 就是一个单词(Text)
 * VALUEOUT 输出数据value的类型 单词出现的总次数(IntWritable)
 */
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    private Text outKey = new Text();
    private IntWritable outValue = new IntWritable();

    /**
     * Reduce阶段的核心业务处理方法， 一组相同的key的values会调用一次reduce方法
     * @param key
     * @param values
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
        int totalCount = 0;
        // 遍历values
        for (IntWritable value : values) {
            //对value进行累加 输出结果
            totalCount += value.get();
        }
        //封装key和value
        outKey.set(key);
        outValue.set(totalCount);
        context.write(outKey,outValue);
    }
}
```

（3）编写Driver驱动类

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: Xu1Aan
 * @Date: 2022/03/15/20:36
 * @Description:
 * MR程序的驱动类，主要用于提交MR任务
 */
public class WordCountDriver {
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
        //声明配置对象
        Configuration conf = new Configuration();
        //声明Job对象
        Job job = Job.getInstance(conf);
        //指定当前Job的驱动类
        job.setJarByClass(WordCountDriver.class);
        //指定当前Job的Mapper和Reducer
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);
        //指定Map端输出数据key的类型和输出数据value的类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        //指定最终输出结果的key的类型和value的类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //指定输入数据的目录 和 输出数据的目录
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        //提交job
        job.waitForCompletion(true);
    }
}
```

5）**测试**

- **本地测试**

  （1）需要首先配置好HADOOP_HOME变量以及Windows运行依赖

  （2）在IDEA/Eclipse上运行程序

- **集群测试**

  （1）将程序打成jar包，然后拷贝到Hadoop集群中

  步骤详情：右键->Run as->maven install。等待编译完成就会在项目的target文件夹中生成jar包。如果看不到。在项目上右键->Refresh，即可看到。修改不带依赖的jar包名称为wc.jar，并拷贝该jar包到Hadoop集群。

  （2）启动Hadoop集群

  （3）执行WordCount程序

  ```shell
  hadoop jar MapReduce-1.0-SNAPSHOT.jar com.xu1an.mr.WordCountDriver /wcinput /wcoutput
  ```

  

## 2 Hadoop序列化

### 2.1 序列化概述

- **什么是序列化**

  **序列化**就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储到磁盘（持久化）和网络传输。

  **反序列化**就是将收到字节序列（或其他数据传输协议）或者是磁盘的持久化数据，转换成内存中的对象。

- **为什么要序列化**

  一般来说，“活的”对象只生存在内存里，关机断电就没有了。而且“活的”对象只能由本地的进程使用，不能被发送到网络上的另外一台计算机。 然而序列化可以存储“活的”对象，可以将“活的”对象发送到远程计算机。

- **为什么不用Java的序列化**

  Java的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，Header，继承体系等），不便于在网络中高效传输。所以，Hadoop自己开发了一套序列化机制（Writable）。

- **Hadoop序列化特点**

  **（1）紧凑** **：**高效使用存储空间。

  **（2）快速：**读写数据的额外开销小。

  **（3）可扩展：**随着通信协议的升级而可升级。

  **（4）互操作：**支持多语言的交互。

### 2.2 自定义bean对象实现序列化接口（Writable）

在企业开发中往往常用的基本序列化类型不能满足所有需求，比如在Hadoop框架内部传递一个bean对象，那么该对象就需要实现序列化接口。

具体实现bean对象序列化步骤如下7步。

（1）必须实现Writable接口

（2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造

```java
public FlowBean() {
	super();
}
```

（3）重写序列化方法

```java
@Override
public void write(DataOutput out) throws IOException {
	out.writeLong(upFlow);
	out.writeLong(downFlow);
	out.writeLong(sumFlow);
}
```

（4）重写反序列化方法

```java
@Override
public void readFields(DataInput in) throws IOException {
	upFlow = in.readLong();
	downFlow = in.readLong();
	sumFlow = in.readLong();
}
```

（5）注意反序列化的顺序和序列化的顺序完全一致

（6）要想把结果显示在文件中，需要重写toString()，可用”\t”分开，方便后续用。

（7）如果需要将自定义的bean放在key中传输，则还需要实现Comparable接口，因为MapReduce框中的Shuffle过程要求对key必须能排序。**详见后面排序案例。**

```java
@Override
public int compareTo(FlowBean o) {
	// 倒序排列，从大到小
	return this.sumFlow > o.getSumFlow() ? -1 : 1;
}
```

### 2.3 序列化案例实操 

**1）需求**

统计每一个手机号耗费的总上行流量、总下行流量、总流量

**2）需求分析**

1. 输入数据格式

   ```
   7   13560436666  120.196.100.99  1116    954      200
   Id    手机号码        网络ip      上行流量  下行流量  网络状态码
   ```

   期望输出数据格式

   ```
   13560436666    1116     954     2070
    手机号码      上行流量  下行流量   总流量
   ```

2. Map阶段

   （1）读取一行数据，切分字段

   ```
   7  13560436666  120.196.100.99  1116    954   200
   ```

   （2）抽取手机号、上行流量、下行流量

   ```
     13560436666   1116      954      
      手机号码     上行流量   下行流量 
   ```

   （3）以手机号为key，bean对象为value输出，即context.write(手机号,bean);

3. Reduce阶段

   （1）累加上行流量和下行流量得到总流量。

   ```
   13560436666   1116  +  954   =    2070   
   手机号码	  上行流量   下行流量     总流量 
   ```

**3）编写MapReduce程序**



