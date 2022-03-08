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

     ​	负责执行有关 `文件系统命名空间` 的操作，例如打开，关闭、重命名文件和目录等。它同时还负责集群元数据的存储，记录着文件中各个数据块的位置信息。

     - 管理HDFS的名称空间
     - 配置副本策略
     - 管理数据块（Block）映射信息
     - 处理客户端读写请求

  2. DataNode：就是Slave。NameNode下达命令，DataNode执行实际的操作。

     ​	负责提供来自文件系统客户端的读写请求，执行块的创建，删除等操作。

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

  <img src=".\picture\hdf文件块大小.png" style="zoom:22%;" />

- **数据复制及其实现原理**

  由于 Hadoop 被设计运行在廉价的机器上，这意味着硬件是不可靠的，为了保证容错性，HDFS 提供了数据复制机制。HDFS 将每一个文件存储为一系列**块**，每个块由多个副本来保证容错，块的大小和复制因子可以自行配置（默认情况下，块大小是 128M，默认复制因子是 3）。

  <img src=".\picture\数据复制.png" style="zoom:75%;" />

  **数据复制的实现原理**

  大型的 HDFS 实例在通常分布在多个机架的多台服务器上，不同机架上的两台服务器之间通过交换机进行通讯。在大多数情况下，同一机架中的服务器间的网络带宽大于不同机架中的服务器之间的带宽。因此 HDFS 采用机架感知副本放置策略，对于常见情况，当复制因子为 3 时，HDFS 的放置策略是：

  在写入程序位于 `datanode` 上时，就优先将写入文件的一个副本放置在该 `datanode` 上，否则放在随机 `datanode` 上。之后在另一个远程机架上的任意一个节点上放置另一个副本，并在该机架上的另一个节点上放置最后一个副本。此策略可以减少机架间的写入流量，从而提高写入性能。

  <img src="E:\learning\04_java\01_笔记\BigData\hadoop\picture\数据复制原理.png" style="zoom:75%;" />

  如果复制因子大于 3，则随机确定第 4 个和之后副本的放置位置，同时保持每个机架的副本数量低于上限，上限值通常为 `（复制系数 - 1）/机架数量 + 2`，需要注意的是不允许同一个 `dataNode` 上具有同一个块的多个副本。

- **架构的稳定性**

  1. 心跳机制和重新复制

     每个 DataNode 定期向 NameNode 发送心跳消息，如果超过指定时间没有收到心跳消息，则将 DataNode 标记为死亡。NameNode 不会将任何新的 IO 请求转发给标记为死亡的 DataNode，也不会再使用这些 DataNode 上的数据。 由于数据不再可用，可能会导致某些块的复制因子小于其指定值，NameNode 会跟踪这些块，并在必要的时候进行重新复制。

  2. 数据的完整性

     由于存储设备故障等原因，存储在 DataNode 上的数据块也会发生损坏。为了避免读取到已经损坏的数据而导致错误，HDFS 提供了数据完整性校验机制来保证数据的完整性，具体操作如下：

     当客户端创建 HDFS 文件时，它会计算文件的每个块的 `校验和`，并将 `校验和` 存储在同一 HDFS 命名空间下的单独的隐藏文件中。当客户端检索文件内容时，它会验证从每个 DataNode 接收的数据是否与存储在关联校验和文件中的 `校验和` 匹配。如果匹配失败，则证明数据已经损坏，此时客户端会选择从其他 DataNode 获取该块的其他可用副本。

  3. 元数据的磁盘故障

     `FsImage` 和 `EditLog` 是 HDFS 的核心数据，这些数据的意外丢失可能会导致整个 HDFS 服务不可用。为了避免这个问题，可以配置 NameNode 使其支持 `FsImage` 和 `EditLog` 多副本同步，这样 `FsImage` 或 `EditLog` 的任何改变都会引起每个副本 `FsImage` 和 `EditLog` 的同步更新。

  4. 支持快照

     快照支持在特定时刻存储数据副本，在数据意外损坏时，可以通过回滚操作恢复到健康的数据状态。

- **思考：为什么块的大小不能设置太小，也不能设置太大？**

  - HDFS的块设置太小，会增加寻址时间，程序一直在找块的开始位置
  - 如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。导致程序在处理这块数据时，会非常慢。

  **总结：HDFS块的大小设置主要取决于磁盘传输速率**

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

  

