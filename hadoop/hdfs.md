# HDFS分布式文件系统

## 1 HDFS概述

- **HDFS定义**

  HDFS（Hadoop Distributed File System）hadoop分布式文件系统，它是一个文件系统，用于存储文件，通过目录树来定位文件；其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。

  HDFS的使用场景：适合一次写入，多次读出的场景，且不支持文件的修改。适合用来做数据分析，并不适合用来做网盘应用。

- **HDFS优点**

  1. 高容错性
     - 数据自动保存多个副本。它通过增加副本的形式，提高容错性。
     - 某一个副本丢失以后，它可以自动恢复。
  2. 适合处理大数据
     - 数据规模：能够处理数据规模达到GB、TB、甚至PB级别的数据；
     - 文件规模：能够处理百万规模以上的文件数量，数量相当之大。
  3. 可构建在廉价机器上，通过多副本机制，提高可靠性。

- **HDFS缺点**

  1. 不适合低延时数据访问，比如毫秒级的存储数据，是做不到的。
  2. 无法高效的对大量小文件进行存储。
     - 存储大量小文件的话，它会占用NameNode大量的内存来存储文件目录和块信息。这样是不可取的，因为NameNode的内存总是有限的；
     - 小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标
  3. 不支持并发写入、文件随机修改。
     - 一个文件只能有一个写，不允许多个线程同时写；
     - 仅支持数据append（追加），不支持文件的随机修改。

- **HDFS组成架构**

  1. NameNode（nn）：就是Master，它是一个主管、管理者。
     - 管理HDFS的名称空间
     - 配置副本策略
     - 管理数据块（Block）映射信息
     - 处理客户端读写请求
  2. DataNode：就是Slave。NameNode下达命令，DataNode执行实际的操作。
     - 存储实际的数据块
     - 执行数据块的读/写操作
  3. Client：就是客户端。
     - 文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传
     - 与NameNode交互，获取文件的位置信息
     - 与DataNode交互，读取或者写入数据
     - Client提供一些命令来管理HDFS，比如NameNode格式化
     - Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作
  4. Secondary NameNode：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务
     - 辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode 
     - 在紧急情况下，可辅助恢复NameNode

- **HDFS文件块大小**

  - HDFS中的文件在物理上是分块存储（Block），块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在Hadoop2.x版本中是128M，老版本中是64M。

  ![](.\picture\hdf文件块大小.png)

- **思考：为什么块的大小不能设置太小，也不能设置太大？**

  - HDFS的块设置太小，会增加寻址时间，程序一直在找块的开始位置
  - 如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。导致程序在处理这块数据时，会非常慢。

  **总结：HDFS块的大小设置主要取决于磁盘传输速率**

  ---

## 2 HDFS的Shell操作

### 2.1 hdfs命令

- 基本语法

  ```shell
  hadoop fs 具体命令   
  # OR  
  hdfs dfs 具体命令
  ```

  以上两个命令是完全相同的。

- 命令大全

  ```shell
  $ bin/hadoop fs
  
  [-appendToFile <localsrc> ... <dst>]
          [-cat [-ignoreCrc] <src> ...]
          [-checksum <src> ...]
          [-chgrp [-R] GROUP PATH...]
          [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
          [-chown [-R] [OWNER][:[GROUP]] PATH...]
          [-copyFromLocal [-f] [-p] <localsrc> ... <dst>]
          [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
          [-count [-q] <path> ...]
          [-cp [-f] [-p] <src> ... <dst>]
          [-createSnapshot <snapshotDir> [<snapshotName>]]
          [-deleteSnapshot <snapshotDir> <snapshotName>]
          [-df [-h] [<path> ...]]
          [-du [-s] [-h] <path> ...]
          [-expunge]
          [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
          [-getfacl [-R] <path>]
          [-getmerge [-nl] <src> <localdst>]
          [-help [cmd ...]]
          [-ls [-d] [-h] [-R] [<path> ...]]
          [-mkdir [-p] <path> ...]
          [-moveFromLocal <localsrc> ... <dst>]
          [-moveToLocal <src> <localdst>]
          [-mv <src> ... <dst>]
          [-put [-f] [-p] <localsrc> ... <dst>]
          [-renameSnapshot <snapshotDir> <oldName> <newName>]
          [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
          [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
          [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
          [-setrep [-R] [-w] <rep> <path> ...]
          [-stat [format] <path> ...]
          [-tail [-f] <file>]
          [-test -[defsz] <path>]
          [-text [-ignoreCrc] <src> ...]
          [-touchz <path> ...]
          [-usage [cmd ...]]
  ```

  

