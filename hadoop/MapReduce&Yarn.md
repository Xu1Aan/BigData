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
