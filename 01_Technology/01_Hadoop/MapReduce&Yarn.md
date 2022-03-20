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

（1）编写流量统计的Bean对象

```java
package com.xu1an.mr.writable;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: Xu1Aan
 * @Date: 2022/03/17/20:22
 * @Description:
 * 流量对象（实现hadoop序列化）
 */
public class FlowBean implements Writable {

    private Integer upFlow;
    private Integer downFlow;
    private Integer sumFlow;

    public void setUpFlow(Integer upFlow) {
        this.upFlow = upFlow;
    }

    public Integer getUpFlow() {
        return upFlow;
    }

    public Integer getDownFlow() {
        return downFlow;
    }

    public Integer getSumFlow() {
        return sumFlow;
    }

    public void setDownFlow(Integer downFlow) {
        this.downFlow = downFlow;
    }

    public void setSumFlow(Integer sumFlow) {
        this.sumFlow = sumFlow;
    }

    @Override
    public String toString() {
        return upFlow+"\t"+downFlow+"\t"+sumFlow;
    }

    /**
     * 序列化方法
     * @param out
     * @throws IOException
     */
    @Override
    public void write(DataOutput out) throws IOException {
       out.writeInt(upFlow);
       out.writeInt(downFlow);
       out.writeInt(sumFlow);
    }

    /**
     *反序列化方法
     * @param in
     * @throws IOException
     */
    @Override
    public void readFields(DataInput in) throws IOException {
        upFlow = in.readInt();
        downFlow = in.readInt();
        sumFlow = in.readInt();
    }
}
```

（2）编写Mapper类

```java
package com.xu1an.mr.writable;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: Xu1Aan
 * @Date: 2022/03/17/20:32
 * @Description:
 */
public class FlowMapper extends Mapper<LongWritable, Text, Text, FlowBean> {

    private Text outKey = new Text();
    private FlowBean outValue = new FlowBean();

    /**
     * 核心业务逻辑处理
     * @param key
     * @param value
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, FlowBean>.Context context) throws IOException, InterruptedException {
        //获取当前行数据
        String line = value.toString();
        //切割数据
        String[] phoneData = line.split("\t");

        //当前数据： 7   13560436666  120.196.100.99  1116    954      200
        //获取输出数据的key(手机号)
        outKey.set(phoneData[1]);
        //获取输出数据的value
        int up = Integer.parseInt(phoneData[phoneData.length - 3]);
        int down = Integer.parseInt(phoneData[phoneData.length - 2]);
        outValue.setUpFlow(up);
        outValue.setDownFlow(down);
        outValue.setSumFlow(up+down);

        //将数据输出
        context.write(outKey,outValue);
    }
}
```

（3）编写Reducer类

```java
package com.xu1an.mr.writable;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: Xu1Aan
 * @Date: 2022/03/17/20:32
 * @Description:
 */
public class FlowReduce extends Reducer<Text, FlowBean, Text, FlowBean> {

    private Text outKey = new Text();
    private FlowBean outValue = new FlowBean();

    /**
     * 核心业务逻辑处理
     * @param key
     * @param values
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Reducer<Text, FlowBean, Text, FlowBean>.Context context) throws IOException, InterruptedException {
        //遍历一组相同key的values
        int totalUpFlow = 0;
        int totalDownFlow = 0;
        int totalSumFlow = 0;

        for (FlowBean value : values) {
            totalUpFlow += value.getUpFlow();
            totalDownFlow += value.getDownFlow();
            totalSumFlow += value.getSumFlow();
        }

        //封装输出数据的key
        outKey.set(key);
        //封装输出数据的value
        outValue.setUpFlow(totalUpFlow);
        outValue.setDownFlow(totalDownFlow);
        outValue.setSumFlow(totalSumFlow);

        //将数据输出
        context.write(outKey,outValue);
    }
}
```

（4）编写Driver驱动类

```java
package com.xu1an.mr.writable;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: Xu1Aan
 * @Date: 2022/03/17/20:32
 * @Description:
 */
public class FlowDriver {
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(FlowDriver.class);
        job.setMapperClass(FlowMapper.class);
        job.setReducerClass(FlowReduce.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        FileInputFormat.setInputPaths(job,new Path("E:\\learning\\04_java\\02_大数据资料\\00_hadoop\\资料\\07_测试数据\\phone_data"));
        FileOutputFormat.setOutputPath(job,new Path("E:\\learning\\04_java\\02_大数据资料\\00_hadoop\\out\\data_1"));

        job.waitForCompletion(true);
    }
}
```

----

## 3  MapReduce框架原理

### 3.1 InputFormat数据输入

MapReduce的执行大概流程

```
简易版： InputFormat  -->    Mapper    -->        Reducer       --> OutputFormat
详细版： InputFormat  -->   map sort   -->  copy sort reduce    --> OutputFormat
```

<img src=".\picture\MapReduce框架原理.png" style="zoom: 33%;" />

```java
//MapTask源码：
if (isMapTask()) {
    // If there are no reducers then there won't be any sort. Hence the map 
    // phase will govern the entire attempt's progress.
    //如果没有reducer，将不会进行排序
    if (conf.getNumReduceTasks() == 0) {
        mapPhase = getProgress().addPhase("map", 1.0f);
    } else {
        // If there are reducers then the entire attempt's progress will be 
        // split between the map phase (67%) and the sort phase (33%).
        // 如果有reducer，map阶段和排序阶段分配67%和33%
        mapPhase = getProgress().addPhase("map", 0.667f);
        sortPhase  = getProgress().addPhase("sort", 0.333f);
    }
}
```

```java
//ReduceTask源码

if (isMapOrReduce()) {
	copyPhase = getProgress().addPhase("copy");//将集群中多个MapTask任务输出数据拷贝
	sortPhase  = getProgress().addPhase("sort");//将数据进行归并排序
	reducePhase = getProgress().addPhase("reduce");//执行reduce方法
}
```

#### 3.1.1 切片与MapTask并行度决定机制

**1）问题引出**

MapTask的并行度决定Map阶段的任务处理并发度，进而影响到整个Job的处理速度。

思考：1G的数据，启动8个MapTask，可以提高集群的并发处理能力。那么1K的数据，也启动8个MapTask，会提高集群性能吗？MapTask并行任务是否越多越好呢？哪些因素影响了MapTask并行度？

**2）MapTask并行度决定机制**

**数据块：**Block是HDFS物理上把数据分成一块一块。数据块是HDFS存储数据单位。

**数据切片：**数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。数据切片是MapReduce程序计算输入数据的单位，一个切片会对应启动一个MapTask。

```
- 切片的概念：从文件的逻辑上的进行大小的切分，一个切片多大，将来一个MapTask的处理的数据就多大。
- 一个切片就会产生一个MapTask
- 切片时只考虑文件本身，不考虑数据的整体集。
- 切片大小和切块大小默认是一致的，这样设计目的为了避免将来切片读取数据的时候有跨机器的情况
```

<img src="E:\learning\04_java\01_笔记\BigData\hadoop\picture\数据切片与MapTask并行度决定机制.png" style="zoom: 50%;" />

#### 3.1.2 InpuFormat的体系结构

InputFormat是一个抽象类

- **FileInputFormat:**  InputFormat的子实现类，实现切片逻辑。实现了getSplits() 负责切片。（FileInputFormat也是一个抽象类）
- **TextInputFormat**:  FileInputFormat的子实现类， 实现读取数据的逻辑。createRecordReader() 返回一个RecordReader，在RecordReader中实现了读取数据的方式：按行读取。
- **CombineTextInputFormat:**  FileInputFormat的子实现类，此类中也实现了  一套切片逻辑 （处理：适用于小文件计算场景。）

#### 3.1.3 FileInputFormat切片源码解析

**FileInputFormat负责切片（getSplits）**

（1）源码中计算切片大小的公式

​	 Math.max(minSize, Math.min(maxSize, blockSize)); 

​	mapreduce.input.fileinputformat.split.minsize=1 默认值为1

​	mapreduce.input.fileinputformat.split.maxsize= Long.MAXValue 默认值Long.MAXValue

（2）切片大小设置

​	maxsize（切片最大值）：参数如果调得比blockSize小，则会让切片变小，而且就等于配置的这个参数的值。

​	minsize（切片最小值）：参数调的比blockSize大，则可以让切片变得比blockSize还大。

（3）获取切片信息API

```
// 获取切片的文件名称
String name = inputSplit.getPath().getName();
// 根据文件类型获取切片信息
FileSplit inputSplit = (FileSplit) context.getInputSplit();
```

**getSplits源码**

```java
// 切片源码
public List<InputSplit> getSplits(JobContext job) throws IOException {
    
    //java中的计时器
    StopWatch sw = new StopWatch().start();

    // minSize = 1(默认情况) 
    // 但是我们也可以通过改变mapreduce.input.fileinputformat.split.minsize 配置项来改变minSize大小
    long minSize = Math.max(getFormatMinSplitSize(), getMinSplitSize(job));

    // maxSize = Long类型的最大值(默认情况)
    // 但是我们也可以通过改变mapreduce.input.fileinputformat.split.maxsize 配置项来改变maxSize大小
    long maxSize = getMaxSplitSize(job);

    // 管理最终切完片的对象的集合 最终返回的就是此集合
    List<InputSplit> splits = new ArrayList<InputSplit>();

    // 获取当前文件的详情
    List<FileStatus> files = listStatus(job);

    boolean ignoreDirs = !getInputDirRecursive(job)
            && job.getConfiguration().getBoolean(INPUT_DIR_NONRECURSIVE_IGNORE_SUBDIRS, false);

    // 遍历获取到的文件列表，一次按照文件为单位进行切片  
    for (FileStatus file: files) {

        // 如果是忽略文件以及是文件夹就不进行切片
        if (ignoreDirs && file.isDirectory()) {
            continue;
        }

        // 获取文件的路径
        Path path = file.getPath();
        // 获取文件的内容大小
        long length = file.getLen();
        // 如果不是空文件 继续切片
        if (length != 0) {

            // 获取文件的具体的块信息
            BlockLocation[] blkLocations;
            if (file instanceof LocatedFileStatus) {
                blkLocations = ((LocatedFileStatus) file).getBlockLocations();
            } else {
                FileSystem fs = path.getFileSystem(job.getConfiguration());
                blkLocations = fs.getFileBlockLocations(file, 0, length);
            }

            // 核心逻辑：判断是否要进行切片（主要判断当前文件是否是压缩文件，有一些压缩文件时不能够进行切片）
            if (isSplitable(job, path)) {
                // 获取HDFS中的数据块的大小
                long blockSize = file.getBlockSize();
                // 计算切片的大小--> 128M 默认情况下永远都是块大小
                long splitSize = computeSplitSize(blockSize, minSize, maxSize);
                -- 内部方法：
                    protected long computeSplitSize(long blockSize, long minSize,long maxSize) {
                        return Math.max(minSize, Math.min(maxSize, blockSize));
                    }

                long bytesRemaining = length;

                // 判断当前的文件的剩余内容是否要继续切片 SPLIT_SLOP = 1.1
                // 判断公式：bytesRemaining)/splitSize > SPLIT_SLOP
                // 用文件的剩余大小/切片大小 > 1.1 才继续切片（这样做的目的是为了让我们每一个MapTask处理的数据更加均衡）
                while (((double) bytesRemaining)/splitSize > SPLIT_SLOP) {
                    int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
                    splits.add(makeSplit(path, length-bytesRemaining, splitSize,
                            blkLocations[blkIndex].getHosts(),
                            blkLocations[blkIndex].getCachedHosts()));
                    bytesRemaining -= splitSize;
                }

                // 如果最后文件还有剩余且不足一个切片大小，最后再形成最后的一个切片
                if (bytesRemaining != 0) {
                    int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
                    splits.add(makeSplit(path, length-bytesRemaining, bytesRemaining,
                            blkLocations[blkIndex].getHosts(),
                            blkLocations[blkIndex].getCachedHosts()));
                }
            } else { // not splitable 不能切分
                if (LOG.isDebugEnabled()) {
                    // Log only if the file is big enough to be splitted
                    if (length > Math.min(file.getBlockSize(), minSize)) {
                        LOG.debug("File is not splittable so no parallelization "
                                + "is possible: " + file.getPath());
                    }
                }
                splits.add(makeSplit(path, 0, length, blkLocations[0].getHosts(),
                        blkLocations[0].getCachedHosts()));
            }
        } else {
            //Create empty hosts array for zero length files
            splits.add(makeSplit(path, 0, length, new String[0]));
        }
    }
    // Save the number of input files for metrics/loadgen
    job.getConfiguration().setLong(NUM_INPUT_FILES, files.size());
    
    //计时器停止
    sw.stop();
    if (LOG.isDebugEnabled()) {
        LOG.debug("Total # of splits generated by getSplits: " + splits.size()
                + ", TimeTaken: " + sw.now(TimeUnit.MILLISECONDS));
    }
    return splits;
}
```

#### 3.1.4 TextInputFormat读取数据

**TextInputFormat实现按行读取（createRecordReader）**

`TextInputFormat`实现读取数据的逻辑`createRecordReader()` 返回一个`RecordReader`，在`RecordReader`中实现了读取数据的方式：安行读取。源码如下：

```java
  @Override
  public RecordReader<LongWritable, Text> 
    createRecordReader(InputSplit split,
                       TaskAttemptContext context) {
    String delimiter = context.getConfiguration().get(
        "textinputformat.record.delimiter");
    byte[] recordDelimiterBytes = null;
    if (null != delimiter)
      recordDelimiterBytes = delimiter.getBytes(Charsets.UTF_8);
      //按行读取
    return new LineRecordReader(recordDelimiterBytes);
  }
```

TextInputFormat是默认的FileInputFormat实现类。按行读取每条记录。键是存储该行在整个文件中的起始字节偏移量， LongWritable类型。值是这行的内容，不包括任何行终止符（换行符和回车符），Text类型。

以下是一个示例，比如，一个分片包含了如下4条文本记录

```
Rich learning form
Intelligent learning engine
Learning more convenient
From the real demand for more close to the enterprise
```

每条记录表示为以下键/值对：

```
(0,Rich learning form)
(19,Intelligent learning engine)
(47,Learning more convenient)
(72,From the real demand for more close to the enterprise)
```

#### 3.1.5 CombineTextInputFormat小文件计算

框架默认的TextInputFormat切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个MapTask，这样如果有大量小文件，就会产生大量的MapTask，处理效率极其低下。

**1）应用场景：**

CombineTextInputFormat用于小文件过多的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个MapTask处理。

**2）虚拟存储切片最大值设置**

CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m

注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

**3）切片机制**

生成切片过程包括：虚拟存储过程和切片过程二部分。

<img src="E:\learning\04_java\01_笔记\BigData\hadoop\picture\CombineTextInputFormat切片机制.png" style="zoom:30%;" />

（1）虚拟存储过程：

将输入目录下所有文件大小，依次和设置的setMaxInputSplitSize值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值2倍，此时将文件均分成2个虚拟存储块（防止出现太小切片）。

例如setMaxInputSplitSize值为4M，输入文件大小为8.02M，则先逻辑上分成一个4M。剩余的大小为4.02M，如果按照4M逻辑划分，就会出现0.02M的小的虚拟存储文件，所以将剩余的4.02M文件切分成（2.01M和2.01M）两个文件。

（2）切片过程：

​	（a）判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片。

​	（b）如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。

​	（c）测试举例：有4个小文件大小分别为1.7M、5.1M、3.4M以及6.8M这四个小文件，则虚拟存储之后形成6个文件块，大小分别为：

​		1.7M，（2.55M、2.55M），3.4M以及（3.4M、3.4M）

​		最终会形成3个切片，大小分别为：

​		（1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M

### 3.2 Shuffle机制

Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。

![](.\picture\shuffle机制.png)
