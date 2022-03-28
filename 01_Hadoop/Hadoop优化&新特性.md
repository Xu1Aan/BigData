# Hadoop优化&新特性

## 1 Hadoop数据压缩

### 1.1 概述

- **概述**

  压缩技术能够有效减少底层存储系统（HDFS）读写字节数。压缩提高了网络带宽和磁盘空间的效率。在运行MR程序时，I/O操作、网络数据传输、 Shuffle和Merge要花大量的时间，尤其是数据规模很大和工作负载密集的情况下，因此，使用数据压缩显得非常重要。

   鉴于磁盘I/O和网络带宽是Hadoop的宝贵资源，数据压缩对于节省资源、最小化磁盘I/O和网络传输非常有帮助。可以在任意MapReduce阶段启用压缩。不过，尽管压缩与解压操作的CPU开销不高，其性能的提升和资源的节省并非没有代价。

- **压缩策略和原则**

  压缩是提高Hadoop运行效率的一种优化策略。

  通过对Mapper、Reducer运行过程的数据进行压缩，以减少磁盘IO，提高MR程序运行速度。

  注意：采用压缩技术减少了磁盘IO，但同时增加了CPU运算负担。所以，压缩特性运用得当能提高性能，但运用不当也可能降低性能。

  **压缩基本原则：**（1）运算密集型的job，少用压缩。（2）IO密集型的job，多用压缩

### 1.2 MR支持的压缩编码

| 压缩格式 | hadoop自带？ |  算法   | 文件扩展名 | 是否可切分 | 换成压缩格式后，原来的程序是否需要修改 |
| :------: | :----------: | :-----: | :--------: | :--------: | :------------------------------------: |
| DEFLATE  | 是，直接使用 | DEFLATE |  .deflate  |     否     |       和文本处理一样，不需要修改       |
|   Gzip   | 是，直接使用 | DEFLATE |    .gz     |     否     |       和文本处理一样，不需要修改       |
|  bzip2   | 是，直接使用 |  bzip2  |    .bz2    |     是     |       和文本处理一样，不需要修改       |
|   LZO    | 否，需要安装 |   LZO   |    .lzo    |     是     |     需要建索引，还需要指定输入格式     |
|  Snappy  | 是，直接使用 | Snappy  |  .snappy   |     否     |       和文本处理一样，不需要修改       |

为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器，如下表所示。

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

压缩性能的比较

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

### 1.3 压缩方式选择

#### 1.3.1 Gzip压缩

优点：压缩率比较高，而且压缩/解压速度也比较快；Hadoop本身支持，在应用中处理Gzip格式的文件就和直接处理文本一样；大部分Linux系统都自带Gzip命令，使用方便。

缺点：不支持Split。

应用场景：当每个文件压缩之后在130M以内的（1个块大小内），都可以考虑用Gzip压缩格式。例如说一天或者一个小时的日志压缩成一个Gzip文件。

#### 1.3.2 Bzip2压缩

优点：支持Split；具有很高的压缩率，比Gzip压缩率都高；Hadoop本身自带，使用方便

缺点：压缩/解压速度慢。

应用场景：适合对速度要求不高，但需要较高的压缩率的时候；或者输出之后的数据比较大，处理之后的数据需要压缩存档减少磁盘空间并且以后数据用得比较少的情况；或者对单个很大的文本文件想压缩减少存储空间，同时又需要支持Split，而且兼容之前的应用程序的情况。

#### 1.3.3 Lzo压缩

优点：压缩/解压速度也比较快，合理的压缩率；支持Split，是Hadoop中最流行的压缩格式；可以在Linux系统下安装lzop命令，使用方便。

缺点：压缩率比Gzip要低一些；Hadoop本身不支持，需要安装；在应用中对Lzo格式的文件需要做一些特殊处理（为了支持Split需要建索引，还需要指定InputFormat为Lzo格式）。

应用场景：一个很大的文本文件，压缩之后还大于200M以上的可以考虑，而且单个文件越大，Lzo优点越越明显。

#### 1.3.4 Snappy压缩

优点：高速压缩速度和合理的压缩率。

缺点：不支持Split；压缩率比Gzip要低；Hadoop本身不支持，需要安装。

应用场景：当MapReduce作业的Map输出的数据比较大的时候，作为Map到Reduce的中间数据的压缩格式；或者作为一个MapReduce作业的输出和另外一个MapReduce作业的输入。

### 1.4 压缩位置选择

压缩可以在MapReduce作用的任意阶段启用。

![](.\picture\MapReduce数据压缩.png)

### 1.5 压缩参数配置

要在Hadoop中启用压缩，可以配置如下参数：

|                             参数                             |                     默认值                     |    阶段     |                     建议                      |
| :----------------------------------------------------------: | :--------------------------------------------: | :---------: | :-------------------------------------------: |
|      io.compression.codecs    （在core-site.xml中配置）      | 无，这个需要在命令行输入hadoop checknative查看 |  输入压缩   | Hadoop使用文件扩展名判断是否支持某种编解码器  |
|   mapreduce.map.output.compress（在mapred-site.xml中配置）   |                     false                      | mapper输出  |           这个参数设为true启用压缩            |
| mapreduce.map.output.compress.codec（在mapred-site.xml中配置） |   org.apache.hadoop.io.compress.DefaultCodec   | mapper输出  | 企业多使用LZO或Snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置） |                     false                      | reducer输出 |           这个参数设为true启用压缩            |
| mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置） |   org.apache.hadoop.io.compress.DefaultCodec   | reducer输出 |    使用标准工具或者编解码器，如gzip和bzip2    |
| mapreduce.output.fileoutputformat.compress.type（在mapred-site.xml中配置） |                     RECORD                     | reducer输出 |  SequenceFile输出使用的压缩类型：NONE和BLOCK  |

### 1.6 压缩实操案例

#### 1.6.1 数据流的压缩和解压缩

CompressionCodec有两个方法可以用于轻松地压缩或解压缩数据。

要想对正在被写入一个输出流的数据进行压缩，我们可以使用createOutputStream(OutputStreamout)方法创建一个CompressionOutputStream，将其以压缩格式写入底层的流。

相反，要想对从输入流读取而来的数据进行解压缩，则调用createInputStream(InputStreamin)函数，从而获得一个CompressionInputStream，从而从底层的流读取未压缩的数据。

测试一下如下压缩方式：

| DEFLATE | org.apache.hadoop.io.compress.DefaultCodec |
| ------- | ------------------------------------------ |
| gzip    | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2   | org.apache.hadoop.io.compress.BZip2Codec   |

#### 1.6.2 Map输出端采用压缩

即使你的MapReduce的输入输出文件都是未压缩的文件，你仍然可以对Map任务的中间结果输出做压缩，因为它要写在硬盘并且通过网络传输到Reduce节点，对其压缩可以提高很多性能，这些工作只要设置两个属性即可，我们来看下代码怎么设置。

1）给大家提供的Hadoop源码支持的压缩格式有：BZip2Codec 、DefaultCodec

2）Mapper保持不变

3）Reducer保持不变

#### 1.6.3 Reduce输出端采用压缩

基于WordCount案例处理。

1）修改驱动

2）Mapper和Reducer保持不变（

---

## 2 Hadoop企业优化

### 2.1 MapReduce 跑的慢的原因

MapReduce 程序效率的瓶颈在于两点：

1. 计算机性能

   CPU、内存、磁盘健康、网络

2. I/O 操作优化

   （1）数据倾斜

   （2）Map和Reduce数设置不合理

   （3）Map运行时间太长，导致Reduce等待过久

   （4）小文件过多

   （5）大量的不可切片的超大压缩文件

   （6）Spill次数过多

   （7）Merge次数过多等。

### 2.2 MapReduce优化方法

MapReduce优化方法主要从六个方面考虑：数据输入、Map阶段、Reduce阶段、IO传输、数据倾斜问题和常用的调优参数。 

