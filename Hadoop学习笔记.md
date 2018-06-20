

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [Hadoop核心](#hadoop核心)
* [HDFS](#hdfs)
		* [概念](#概念)
		* [实现机制](#实现机制)
		* [Namenode](#namenode)
		* [Datanode](#datanode)
		* [HA](#ha)
* [MapReduce](#mapreduce)
		* [基本思想](#基本思想)
		* [map和reduce](#map和reduce)
		* [job提交过程](#job提交过程)
		* [MapReduce过程](#mapreduce过程)
			* [1. 切片机制](#1-切片机制)
			* [2. combiner机制](#2-combiner机制)
			* [3. 序列化机制](#3-序列化机制)
			* [4. 分区机制](#4-分区机制)
			* [5. shuffle机制](#5-shuffle机制)
* [YARN](#yarn)
		* [ResourceManager](#resourcemanager)
		* [NodeManager](#nodemanager)
		* [ApplicationMaster](#applicationmaster)
		* [Container](#container)
		* [总结：YARN执行流程](#总结yarn执行流程)
* [Hive](#hive)
* [HBase](#hbase)
		* [定义](#定义)
		* [表结构](#表结构)
		* [存储架构](#存储架构)
		* [集群搭建](#集群搭建)
* [ZooKeeper](#zookeeper)
		* [`Zookeeper`的最主要功能](#zookeeper的最主要功能)
		* [`Zookeeper`的应用](#zookeeper的应用)

<!-- /code_chunk_output -->


# Hadoop核心
`HDFS`：海量数据的存储
`MapReduce`:海量数据的分析
`YARN`:资源管理调度

举例说明：
`YARN`将`HDFS`存储的数据分配到不同的机器节点，运用`MapReduce`进行运算分析

# HDFS

### 概念
- 文件系统：即存储文件（数据）的系统。屏蔽掉硬盘存储的细节，向上层提供统一的数据访问接口。访问文件，复制文件等操作都可以看成一个API的方法处理。
- 分布式文件系统：即数据存储在多个机器上的文件系统，用于解决数据太大，一台机器无法存储所有数据的问题

### 实现机制

![hdfs实现机制](/assets/hdfs实现机制.png)

1. 文件被切块存储在多台服务器的本地文件系统中
2. 每个文件块都可以保存多个副本
3. hdfs中的文件和具体存储位置直接的对应关系交由一个专门的服务器来管理——nameNode

优点：
1. 容量可以线性拓展
2. 有了副本机制，存储可靠性高，吞吐量增大
3. 有了namenode后，客户端访问文件就只需要指定hdfs上的路径

### Namenode
执行文件打开、关闭、重命名等操作，同时保存分文件块（block）在`Datanode`上的具体存储位置的映射（存在namenode的元数据中），响应客户端读写文件的请求。

**问题：namenode如何保证目录的快读读写的呢？**

![nameNode实现机制](/assets/nameNode实现机制.png)
1. 客户端申请上传文件，通知`namenode`
2. `namenode`将记录上传文件的block位置信息记录在`edits.log`中
3. 客户端根据`namenode`的block位置信息，将文件block写入对应的`datanode`
4. 写入成功后返回成功信息给`namenode`
5. `namenode`将该文件的block位置写入内存元数据（方便读取）

如此，`namenode`便将上传文件的元数据写入了内存。但此刻还未写入磁盘中。当`edits.log`记录的数据条数达到一定数量（小文件，为了保持快速读写）或到达指定时间时，会触发`Secondary NameNode`合并元数据再写入磁盘

（注意，之所以触发`Secondary NameNode`进行操作是因为该合并可能是十几G文件的合并，十分耗时和资源，为了不影响`namenode`的功能，所以设计出了`Secondary NameNode`）

具体步骤：
1. `edits log`条数达到触发条件，运行`Secondary NameNode`的`checkpoint`操作。
2. `Secondary NameNode`通知`namenode`切换`edits log`文件。即`namenode`此刻采用一个全新的`edits.new`文件进行读写
3. `Secondary NameNode`通过http下载`namenode`中旧的`edits log`文件和硬盘元数据（Fsimage）
4. `Secondary NameNode`通过`edits log`的日志信息，生成新的元数据文件（Fsimage.chkpoint），并传给`namenode`
5. `namenode`用`Fsimage.chkpoint`和`edits.new`替换旧的元数据文件和`edits log`文件

### Datanode
提供真实文件数据的存储服务，定期向`Namenode`发送心跳信息，汇报本身及其所有的block信息，健康状况

- 文件块（block）：最基本的存储单位。从文件的0偏移开始，按照固定的大小，顺序对文件进行划分并编号，划分好的每一个块称为一个Block。HDFS默认Block大小是128MB。
（注意，如果文件小于一个数据库大小，并不占用整个数据块存储空间）
- Replication
副本策略，默认每个`block`有三个复本。

**副本存放策略：**
1、先在客户端所连接的datanode上存放一个副本
2、再在另一个机架上选择一个datanode存放第二个副本
3、最后在本机架上根据负载情况随机挑选一个datanode存放第三个副本

**副本数量的配置优先级：**
1、服务端hdfs-site.xml中可以配置
2、在客户端指定dfs.replication的值
客户端所指定的值优先级更高!!!

**汇报机制：**
为了防止`Datanode`上文件的改变（如删除，损坏），`Datanode`还会定期向`NameNode`汇报自身所存储的block信息

### HA
基于`zookeeper`，对`namenode`的集群管理。

即为防止单个`namenode`挂了后，`HDFS`无法运行，配置一个`namenode`集群，当`active`的`namenode`挂的时候，备用的`namenode`从`standby`状态转换为`active`状态，继续提供服务

HA机制有2种方式，一种是`NFS`（Network File System）方式，另外一种是`QJM`（Quorum Journal Manager）方式

`QJM`的原理如下图：

![HA](/assets/HA机制.png)

备用的`namenode`和主`namenode`之间需要保持数据的一致（主要是`edits.log`和硬盘元数据`Fsimage`），而在初始化时，所有`namenode`的`Fsimage`都是一致的，所以主要是`edits.log`的一致。

为保持`edits.log`的一致，`HA`机制中直接将其拿了出来变成单独的一个服务：**EDITS管理服务**。所有的`namenode`读取`edits.log`信息时都在这个服务中获取。
（**EDITS管理服务**：可以看成把多个`edits.log`放在多个机器上，基于`Zookeeper`形成主写从读的集群。每个节点的服务名：`journalNode`）

数据保存一致后，当`active`状态的`namenode`挂了的时候，仍需要进行切换，负责这个切换的机制叫做**ZKFC**，它也是基于`Zookeeper`实现的，服务名：`DFSZKFailoverController`。

`ZKFC`切换`namenode`时，可能出现原来`active`状态的`namenode`并不是挂了，只是一段时间失去通讯，等它恢复时，`ZKFC`已经切换出了一个新的`active`的`namenode`，造成此时`HDFS`出现双`active`状态的`namenode`。

为防止这种情况的出现，`ZKFC`切换`namenode时`会有一个**fencing 机制**来保证原`active`状态的`namenode`一定不会再次激活

**fencing 机制**：
1. 发送 `ssh kill namenode进程号`
2. 如果上一步命令响应超时，调用用户自定义的脚本程序

（HA集群搭建步骤可看：@import /assets/hadoop2.4.1集群搭建.txt）

# MapReduce
类似的框架还有`Storm`（擅长流式计算），`Spark`（擅长内存迭代计算）。区别在于`MapReduce`可做离线数据的批量计算，而后两者可做实时处理。

### 基本思想
1. 将一个业务处理需求分成两个阶段进行:`map`阶段，`reduce`阶段
2. 将分布式计算中面临的公共问题封装成框架来实现（`jar`包分发，任务的启动，任务的容错、调度，中间结果的分组传递……）
3. 应用开发人员只需要开发业务逻辑

### map和reduce
`Map`过程会将数据计算映射到各个服务器上进行处理，而`Reduce`会将各个服务器上的计算结果取出来进行二次处理。
![map、reduce](/assets/map、reduce.png)

### job提交过程
当执行`hadoop jar jar包路径 jar包执行主类 执行结果输出目录`命令时，job提交过程如下：

![nameNode实现机制](/assets/job提交过程.png)

（`Yarn`集群中有两个角色节点：`ResourceManager`和`NodeManager`）
1. 客户端执行命令行后，Hadoop开启一个 “job 客户端” 进程：`RunJar`
2. `RunJar`向`Yarn`中的`ResourceManager`申请运行一个job , `ResourceManager`返回执行job的 "jobId" 和 "存放job资源的hdfs路径"
3. `Runjar`将job需要的文件资源（jar包，jobid，jar存放的位置，配置信息等等）写入`HDFS`，通知`ResourceManager`提交完成
4. `ResourceManager`读取`HDFS`上的job信息，开始初始化一个Job任务，并将其放入任务队列
5. `NodeManager`读取`ResourceManager`的任务队列，发现对应任务分配给自己时，领取任务信息（心跳机制）
6. `NodeManager`根据任务信息，下载所需要的jar包，分配执行job的资源容器。分配完毕后向`ResourceManager`汇报分配结果
7. `ResourceManager`根据各个机器的分配结果，选取一个机器上的`NodeManager`运行`MapReduce`的主管进程：`MRAppMaster`
8. `MRAppMaster`主进程协调其余`NodeManager`一起运行`MapReduce`程序来完成job任务，并将结果写入`HDFS`对应目录

整个过程也可以简化为以下三步：
1. `RunJar`客户端和`ResourceManager`协作一起完成Job的提交
2. `ResourceManager`和`NodeManager`协作一起完成Job运行所需要的资源分配
3. `MapReduce`框架的主管进程`MRAppMaster`负责整个Job运行过程的协调控制

### MapReduce过程
![MapReduce过程](/assets/MapReduce原理.png)

#### 1. 切片机制
文件被划分为多个切片，每个切片对应一个`MapperTask`进行运行。

划分规则需要考虑两点：1. 切片不能太大，太大`Map`过程会很耗时；2. 也不能太小，太小会导致`MapperTask`过多，影响效率

默认的划分机制是：1. 对每个文件单独规划切片 2. 对每个文件的每一个分块（block）设置为一个切片（split）

#### 2. combiner机制
在`MapperTask`和`Shuffle`过程之间，做一个数据的局部汇总，使`MapperTask`传到`Shuffle`的数据大幅精简，减少`Shuffle`过程的IO。


![MapReduce过程](/assets/combiner机制.png)

如图，在`MapReduce`上运行 "单词个数统计" 的程序时，某个`MapperTask`最后传给`Shuffle`的可能是`<hello,1,1,1,重复10万次……1>`的长哈希map。而`Shuffle`接受多个这种长字符串后，再进行合并传给`Reduce`的哈希map只会更长。

通过在`combiner`中对`MapperTask`的长哈希map进行累加处理得`<hello,100000>`，再传给`Shuffle`，将减少它大量的IO工作。

**总结**：
1. 在每一个`MapperTask`的本地服务器上运行，能收到`MapperTask`输出的每一个key的valuelist，所以可以做局部汇总处理
2. 因为在`MapperTask`的本地环境进行了局部汇总，就会让`MapperTask`端的输出数据量大幅精简，减小shuffle过程的网络IO
3. `combiner`其实就是一个`reducer`组件，跟真实的`reducer`的区别就在于，`combiner`运行`MapperTask`的本地环境
4. `combiner`在使用时需要注意，输入输出KV数据类型要跟`map`和`reduce`的相应数据类型匹配
5. 要注意业务逻辑不能因为`combiner`的加入而受影响

#### 3. 序列化机制
1. 跟jdk自带的比较起来，更加精简，只传递对象中的数据，而不传递如继承结构等额外信息
2. 要想让自定义的数据类型在hadoop集群中传递，需要实现hadoop的序列化接口`Writable`或者 `WritableComparable<T>`
3. 自定义的数据类型bean实现了`Writable`接口后，要实现其中的两个方法
(注意：写入数据和读出数据的顺序和类型要保持一致)
```
//序列化，将数据写入字节流
public void write(DataOutput out) throws IOException   

//反序列化，从字节流中读出数据
public void readFields(DataInput in) throws IOException  
```

#### 4. 分区机制
每一个reduce task输出一个结果文件。

**场景：**
对一个大文本的电话簿数据，想按省份的输出成不同的分文本

**方法步骤：**
1. 自定义一个类`AreaPartitioner` 继承  `Partitioner` 抽象类，实现其中的方法 `int getPartition(K,V)`
```
public class AreaPartitioner<KEY, VALUE> extends Partitioner<KEY, VALUE>{

	private static HashMap<String, Integer> areaMap =  new HashMap<>();

	static {

		areaMap.put("136", 0);
		areaMap.put("137", 1);
		areaMap.put("138", 2);
		areaMap.put("139", 3);

	}

	@Override
	public int getPartition(KEY key, VALUE value, int numPartitions) {

		Integer provinceCode = areaMap.get(key.toString().substring(0,3));
		return provinceCode==null?4:provinceCode;
	}
}
```

2. 在job的描述中设置使用自定义的`partitioner`
```
job.setPartitionerClass(AreaPartitioner.class)
```
3. 在job的描述中指定作业的reduce task并发数量，job.setNumReduceTasks(5)，数量要与partitioner中的分区数一致。
```
//分区设定了5个省份，所以此处填5
job.setNumReduceTasks(5);
```
要注意的是：
- 如果reduce task的数量比partitioner中分组数多，就会产生多余的几个空文件
- 如果reduce task的数量比partitioner中分组数少，就会发生异常，因为有一些key没有对应reducetask接收
- 如果reduce task的数量为1，也能正常运行，所有的key都会分给这一个reduce task
（reduce task 或 map task 指的是，reuder和mapper在集群中运行的实例）


#### 5. shuffle机制
shuffle的主要工作是从Map结束到Reduce开始之间的过程。即它是`map task`的输出数据 到 `reduce task`之间的一种数据调度机制

`shuffle`阶段又可以分为`Map端的shuffle`和`Reduce端的shuffle`
![shuffle](/assets/shuffle机制.png)
`Map端的shuffle`：分组，排序，`Combiner`
`Reduce端的shuffle`:归并，排序

# YARN
`Yarn`由1个`ResourceManager`和多个`NodeManager`组成
![yarn架构](/assets/yarn架构.gif)

### ResourceManager
 一个集群active状态的RM只有一个，负责整个集群的资源管理和调度
1. 处理客户端的请求(启动/杀死作业)
2. 启动/监控ApplicationMaster(一个作业对应一个AM)
3. 监控`NodeManager`（心跳机制）
4. 系统的资源分配和调度


### NodeManager
整个集群中有N个，负责单个节点的资源管理和使用以及task的运行情况
1. 定期向RM汇报本节点的资源使用请求和各个Container的运行状态
2. 接收并处理RM的container启停的各种命令
3. 单个节点的资源管理和任务管理

### ApplicationMaster
每个应用/作业对应一个，负责应用程序的管理，在`Container`中启动。
1. 数据切分
2. 为应用程序向RM申请资源(跑在container)，并分配给内部任务
3. 与NM通信以启停task， task是运行在container中的
4. task的监控和容错

### Container
作业的运行容器，含有对任务运行情况的描述：cpu、memory、环境变量

### 总结：YARN执行流程
1. 用户向YARN提交作业
2. RM为该作业分配第一个container(ApplicationMaster)
3. RM会与对应的NM通信，要求NM在这个container上启动应用程序的AM
4.  AM首先向RM注册，然后AM将为各个任务申请资源，并监控运行情况
5. AM采用轮训的方式通过RPC协议向RM申请和领取资源
6. AM申请到资源以后，便和相应的NM通信，要求NM启动任务
7. NM启动我们作业对应的task

# Hive

>Hive是一个数据仓库基础工具在Hadoop中用来处理结构化数据。它架构在Hadoop之上，使数据查询和分析更方便。

简单来说，Hive就是将sql语句转换为MapReduce任务或其余引擎任务运行。

Hive底层的执行引擎有：MapReduce、Tez、Spark


# HBase

### 定义
HBase（Hadoop Database）：
一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用`HBase`技术可在廉价PC Server上搭建起大规模结构化存储集群。`HBase`利用`Hadoop HDFS`作为其文件存储系统，利用`Hadoop MapReduce`来处理`HBase`中的海量数据，利用`Zookeeper`作为协调工具。

### 表结构
HBase介于`NOSQL`和`RDBMS`之间，同`RDBMS`一样，它也是用表结构存储数据，表结构主要由**Row Key（行键）**，**列族**，**Cell**，**时间戳** 组成：

![HBase表结构](/assets/HBase表结构.png)

- Row Key：检索记录的主键
- 列族：hbase表中的每个列，都归属与某个列族。也就是一个列族存储多列数据
- Cell：键值对数据
- 时间戳：每个 cell都保存着同一份数据的多个版本。版本通过时间戳来索引。时间戳的类型是 64位整型。

### 存储架构

![HBase存储结构](/assets/HBase存储结构.png)

如图，HBase由两部分组成：`HMaster`，`Region server`
- `HMaster`:为`Region server`分配region；负责`region server`的负载均衡；发现失效的`region server`并重新分配其上的region
- `Region server`：维护`Master`分配给它的region，处理对这些region的IO请求。即负责读写表数据
- `Region`:`HBase`中分布式存储和负载均衡的最小单元，分布在不同的`HRegion server`上。实质为在行的方向上分割为多个`Hregion`。

客户端访问某个表的"xx行键-xx列族-xx字段名-xx版本号"数据时，通过`Zookeeper`中的信息访问对应的元数据表，知道数据存储在哪台`Region server`中，然后对应`Region server`读取存在`HDFS`中的数据返回

### 集群搭建
@import /assets/hbase集群搭建.txt

# ZooKeeper
为分布式程序提供协调服务，作为第三方管理各个节点的共享数据，本身非常可靠，它其实就是一个分布式集群来提供服务

###`Zookeeper`的最主要功能
保管客户端提交的数据（极其少量的数据），每一份数据在zookeeper叫做一个znode

znode之间形成一种树状结构（类似于文件树）
比如：
`/lock(znode名) 001(znode中存的数据)`
`/lock/last_acc(znode名)  008(znode中存的数据)`

### `Zookeeper`的应用
通过`Zookeeper`的共享数据的管理，能实现分布式应用的许多功能：
1. 配置信息的集中式管理和动态更新
2. 负载均衡
3. 统一命名服务（如阿里基于`Zookeeper`封装的`Dubbo`）
4. 集群管理和Master选举
5. 分布式锁
6. 分布式队列

具体可看：
@import /assets/ZooKeeper应用场景.docx
