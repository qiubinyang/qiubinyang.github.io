# Hadoop基础知识


# Hadoop
组成：
- 文件系统 HDFS
- 并行计算框架 MapReduce

## 一、HDFS
### 1.设计架构
- 块(Block)
  - 存储和处理的逻辑单元
  - 默认64MB
- NameNode
  - 管理节点
  - 存放文件元数据
    - 文件与数据块的映射表
    - 数据块与数据节点的映射表
- DataNode
  - 工作节点
  - 存放具体数据

![](https://raw.githubusercontent.com/qiubinyang/qiubinyang.github.io/imgs/hdfs%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84.png)

### 2.数据管理策略
1. 高可靠：
   - 每个数据块3个副本，分布在两个机架内的三个节点上。

2. 心跳检测
![](https://raw.githubusercontent.com/qiubinyang/qiubinyang.github.io/imgs/%E5%BF%83%E8%B7%B3%E6%A3%80%E6%B5%8B.png)

3. 二级NameNode
- 定期同步NameNode中的元数据映像文件和修改日志，NameNode发生故障时，备胎转正。
- 只做备份，不参与NameNode操作

4. HDFS Federation
- 我们知道NameNode的内存会制约文件数量，HDFS Federation提供了一种横向扩展NameNode的方式。在Federation模式中，每个NameNode管理命名空间的一部分，例如一个NameNode管理/user目录下的文件， 另一个NameNode管理/share目录下的文件。
- 每个NameNode管理一个namespace volumn，所有volumn构成文件系统的元数据。每个NameNode同时维护一个Block Pool，保存Block的节点映射等信息。各NameNode之间是独立的，一个节点的失败不会导致其他节点管理的文件不可用。
- 客户端使用mount table将文件路径映射到NameNode。mount table是在Namenode群组之上封装了一层，这一层也是一个Hadoop文件系统的实现，通过viewfs:协议访问。

5. HDFS HA(High Availability高可用性)

启动新的Namenode之后，需要重新配置客户端和DataNode的NameNode信息。另外重启耗时一般比较久，稍具规模的集群重启经常需要几十分钟甚至数小时，造成重启耗时的原因大致有： 
- 元数据镜像文件载入到内存耗时较长。 
- 需要重放edit log 
- 需要收到来自DataNode的状态报告并且满足条件后才能离开安全模式提供写服务。
- HA方案：
  - 采用HA的HDFS集群配置两个NameNode，分别处于Active和Standby状态。当Active NameNode故障之后，Standby接过责任继续提供服务，用户没有明显的中断感觉。一般耗时在几十秒到数分钟。

6. HDFS为每一个用户创建了一个回收站
   1. 在/usr/用户名/.Trash/中
   2. 要使用需要在core-site.xml中配置




### 3.HDFS文件读写操作
#### 读取
![](https://raw.githubusercontent.com/qiubinyang/qiubinyang.github.io/imgs/%E6%96%87%E4%BB%B6%E8%AF%BB%E5%8F%96.png)

#### 写入
![](https://raw.githubusercontent.com/qiubinyang/qiubinyang.github.io/imgs/%E6%96%87%E4%BB%B6%E5%86%99%E5%85%A5.png)

### 4.总结HDFS特点
1. 数据冗余，硬件容错
2. 流式的数据访问
   1. 一次写入，多次读取
   2. 如果出错需要修改，不能直接修改，需要删除后重新写入
3. 适合存储大文件
   1. 储存小文件，仍然需要分块，效率低下
4. 适合数据批量读写，吞吐量高
5. 不适合交互式应用，**延迟太高**
6. 不支持多用户并发写同一个文件

### 5.HDFS常用操作命令
>HDFS提供了各种交互方式，例如通过Java API、HTTP、shell命令行

#### Shell命令
可查文档：http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html

#### Java命令
实际的应用中，对HDFS的大多数操作还是通过FileSystem来操作，这部分重点介绍一下相关的接口，主要关注HDFS的实现类DistributedFileSystem及相关类。



## 二、MapReduce
>分而治之的思想：一个大的任务分成多个小的任务(Map)，并行执行后，合并结果(Reduce)

![](https://raw.githubusercontent.com/qiubinyang/qiubinyang.github.io/imgs/MapReduce.png)

### 区分
MapReduce阶段和DataNode存数据阶段都会有一个分割。数据在DataNode上是物理意义的分割成了一个个Block。而在MapReduce的Map阶段，会有一个split操作分割一个文件来分块取出来分别Map，一般情况下split的大小和block的大小是一样的。