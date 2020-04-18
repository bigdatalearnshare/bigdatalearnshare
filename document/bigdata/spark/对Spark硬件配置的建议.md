## 对Spark硬件配置的建议

对于Spark开发人员来说，一个比较普遍的问题就是如何合理的配置Spark的硬件？当然如何合理的对Spark集群进行硬件配置要视情况而定，在这里给出以下建议：

### 存储系统

在大数据领域，有一句"名言"：移动数据不如移动计算。主要因为数据量是庞大的，如果将数据从一个节点移动到另外一个节点甚至从一个局域网移动到另外一个局域网，就必然会牵涉到大量的磁盘IO和网络IO，这是非常影响性能的。而这里的计算可以理解为封装了你的业务处理代码的jar包，这个是很轻量的，相对于移动数据可有效缓解IO带来的弊端。

因此，将Spark集群节点尽可能部署到靠近存储系统的节点是非常重要的，因为大多数据Spark jobs通常从外部存储系统，如Hadoop文件系统、HBase获取数据。

具体可参考以下建议：

1.以HDFS作为存储系统为例，建议在与HDFS相同的节点上运行Spark。最简单的方式就是将Spark的standalone集群和Hadoop进群部署在相同节点，同时配置好Spark和Hadoop的内存、CPU使用以避免相互干扰。

```
在Hadoop中，一些参数（注意Hadoop新版本中下列参数可能有所变化，具体根据自己使用的版本查看Hadoop官网）
每个task的内存配置参数：mapred.child.java.opts，如设置为-Xmx1024m

单个节点map task数目配置参数：mapreduce.tasktracker.map.tasks.maximum

单个节点reduce task数目配置参数：mapreduce.tasktracker.reduce.tasks.maximum
```
此外，你也可以将Spark和Hadoop运行在共同的集群资源管理器上，如Yarn和Meso。

2.如果不能满足1中的条件，请将Spark和HDFS部署在同一局域网下的不同节点上。

3.对于低延迟数据存储如HBase，可能优先在与存储系统不同的节点上运行计算任务以避免干扰【计算引擎在处理任务时，比较消耗服务器资源，可能影响低延迟存储系统的即时响应】

### 本地磁盘

尽管Spark可以在内存中处理大量的计算，但它仍然需要使用本地磁盘来存储不适合RAM的数据、以及在stage之间即shuffle的中间结果。建议每个节点配备4-8块磁盘，并且这些磁盘是作为独立的磁盘挂在节点即可，不需要做磁盘阵列。

在Linux中，使用noatime选项安装磁盘，以减少不必要的写操作。在Spark中，通过参数spark.local.dir可以配置多个本地磁盘目录，多个目录之间以逗号分开。如果Spark任务运行在hdfs上，与hdfs保持一致就好。

使用noatime选项安装磁盘，要求当挂载文件系统时，可以指定标准Linux安装选项，这将停止该文件系统上的atime更新。

磁盘挂载命令：mount -t gfs BlockDevice MountPoint -o noatime（BlockDevice：指定GFS文件系统驻留的块设备；MountPoint：指定GFS文件系统应安装的目录）。
示例：mount -t gfs /dev/vg00/lvol00 /gfs_dir -o noatime

### 内存

通常情况下，每台机器的内存配置从8G到数百G，Spark都能良好的运行。但建议最多分配给Spark75%的内存，剩余的留给操作系统和buffer cache。

当然，具体需要多少内存取决于你的应用。要确定你的应用使用的特定数据集需要多大内存，请加载部分数据集到内存缓存起来，然后在Spark UI（http://<driver-node>:4040）的Storage界面去看它的内存占用量。

注意：内存使用多少受到存储级别和序列化格式的影响，可以参考http://spark.apache.org/docs/latest/tuning.html的建议。

最后，请注意，对于超过200GB的内存的RAM，JAVA VM运行状态并不一直表现良好。如果你的机器内存超过了200GB，那么可以在一个节点上运行多个worker。在Spark standalone模式下，可以在配置文件conf/spark-env.sh中设置SPARK_WORKER_INSTANCES的值来设置每个节点worker的数目，通过SPARK_WORKER_CORES参数来设置每个Worker的核数。

### 网络

根据以往的经验，如果数据是在内存中，那么Spark应用的瓶颈往往就在网络。用10 Gigabit或者更高的网络，是使Spark应用跑的更快的最佳方式。特别是针对"distributed reduce"操作，如group-bys,reduce-bys和SQL joins，就表现的更加明显。在任何给定的应用程序中，都可以通过Spark UI查看Spark shuffle过程中跨网络传输了多少数据。

### CPU cores

因为Spark在线程之间执行最小的共享CPU，因此它可以很好的扩展到每台机器几十个CPU核。建议每台机器至少配置8-16个内核。当然，具体根据你任务的CPU负载，可能需要更多的CPU：一旦数据在内存中，大多数应用程序的瓶颈就在CPU和网络。

本文主要参译于官网，笔者在此基础上做了一些解释说明，利于大家理解。



### [阅读原文](https://mp.weixin.qq.com/s/UrDCiyd4nrlBdUfDI3KoVw)