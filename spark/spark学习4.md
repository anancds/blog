## SparkContext初始化

SparkContext是Spark的主要入口点，如果把Spark集群当作服务端，那么Spark Driver就是客户端，SparkContext就是客户端的核心。SparkContext用于连接Spark集群、创建RDD、累加器和广播变量，如下图所示：

![](./images/SparkContext.png)

### SparkContext相关组件

#### SparkConf
Spark配置类，以键值对的形式存储在ConcurrentHashMap，配置项包含：master、appName、Jars、ExecutorEnv等。

#### SparkEnv

SparkEnv维护着Spark的执行环境，包含有：serializer、RpcEnv、block Manager、map output tracker等；所有的线程都可以通过SparkCotext访问到同一个SparkEnv对象；SparkContext通过SparkEnv.createDriverEnv创建SparkEnv实例；在SparkEnv中包含了如下主要对象：

* RpcEnv: Rpc通信环境，以前是AKKA，现在默认是netty。Spark中有RpcEnvFactory trait，默认实现为NettyRpcEnvFactory，在Factory中默认使用了Jdk的Serializer作为序列化工具。

* SerializerManager： 管理Spark的压缩和序列化。

* MapOutputTracker: 跟踪Map阶段结果的输出状态，用于在reduce阶段获取地址与输出结果，如果当前为Driver则创建MapOutputTrackerMaster对象否则创建的是MapOutputTrackerWorker两者都继承了MapOutputTracker类。

* ShuffleManager：用于管理远程和本地Block数据shuffle操作，默认使用了SortShuffleManager实例。

* BroadcastManager：用于管理广播对象，默认使用了TorrentBroadcastFactory广播工厂。

* blockManagerMaster： 用于对Block的协调与管理

* BlockManager：为Spark存储系统重要组成部分，用于管理Block。

* SecurityManager：用于对权限、账号进行管理、Hadoop YARN模式下的证书管理等。

* MetricsSystem： Spark测量系统。

* BlockTransferService： 块传输服务，默认使用了Netty的实现，用于获取网络节点的Block或者上传当前结点的Block到网络节点

* MemoryManager：用于管理Spark的内存使用策略，有两种模式StaticMemoryManager、UnifiedMemoryManager，第一种为1.6版本之前的后面那张为1.6版本时引入的，当前模式使用第二种模式；两种模式区别为粗略解释为第一种是静态管理模式，而第二种为动态分配模式，execution与storage之间可以相互“借”内存。

#### LiveListenerBus

异步传递Spark事件监听与SparkListeners监听器的注册。

#### JobProgressListener

JobProgressListener监听器用于监听Spark中任务的进度信息，SparkUI上的任务数据既是该监听器提供的，监听的事件包括有，Job：active、completed、failed；Stage：pending、active、completed、skipped、failed等；JobProgressListener最终将注册到LiveListenerBus中。

#### SparkUI

SparkUI为Spark监控Web平台提供了Spark环境、任务的整个生命周期的监控。

#### TaskScheduler

TaskScheduler为Spark的任务调度器，Spark通过他提交任务并且请求集群调度任务；TaskScheduler通过Master匹配部署模式用于创建TashSchedulerImpl与根据不同的集群管理模式（local、local[n]、standalone、local-cluster、mesos、YARN）创建不同的SchedulerBackend实例。

#### DAGScheduler

DAGScheduler为高级的、基于stage的调度器，为提交给它的job计算stage，将stage作为tasksets提交给底层调度器TaskScheduler执行；DAGScheduler还会决定着stage的最优运行位置。

#### ExecutorAllocationManager

根据负载动态的分配与删除Executor，可通过ExecutorAllcationManager设置动态分配最小Executor、最大Executor、初始Executor数量等配置，调用start方法时会将ExecutorAllocationListener加入到LiveListenerBus中监听Executor的添加、移除等。

#### ContextClearner

ContextClearner为RDD、shuffle、broadcast状态的异步清理器，清理超出应用范围的RDD、ShuffleDependency、Broadcast对象；清理操作由ContextClearner启动的守护线程执行。

#### SparkStatusTracker

低级别的状态报告API，对job、stage的状态进行监控；包含有一个jobProgressListener监听器，用于获取监控到的job、stage事件信息、Executor信息

#### HadoopConfiguration

Spark默认使用HDFS来作为分布式文件系统，HadoopConfigguration用于获取Hadoop配置信息，通过SparkHadoopUtil.get.newConfiguration创建Configuration对象，SparkHadoopUtil 会根据SPARK_YARN_MODE配置来判断是用SparkHadoopUtil或是YarnSparkHadoopUtil，创建该对象时会将spark.hadoop.开头配置都复制到HadoopConfugration中。


### SparkContext初始化流程

根据上面的组件，那么可以描述SparkContext的初始化流程：

* 创建CallSite：CallSite存储了线程栈中最靠近栈顶的用户类及最靠近栈底的scala或者spark核心类。
* 读取配置判断是否可以有多个SparkCotext，默认是只有一个。
* 确保只有一个SparkCotext运行中。
* 克隆一份SparkCotext的Configuration，也就是说Configuration在spark运行时无法改变。
* 通过isLocalMaster判断spark是否运行在本地模式。
* 设置spark event log目录。
* 设置spark event log编码。
* 创建异步的spark监听模式，监听job progress。
* 创建spark的执行环境：SparkEnv。
* 创建spark progress bar。
* 创建并初始化spark UI。
* 设置hadoop相关配置和executor环境相关配置。
* 创建heartbeat receiver。
* 创建并启动TaskScheduler。
* 初始化BlockManager。
* 启动测量系统MetricsSystem。
* 启动日志监听。
* 判断根据负载情况动态分配executor是否开启。
* 创建和启动Executor的ExecutorAllocationManager。
* 创建并启动ContextCleaner。
* Spark环境更新。
* 创建BlockManagerSource、DAGSchedulerSource和ExecutorAllocationManagerSource。
* 创建钩子函数，并在钩子函数中关闭环境。
* 将SparkContext标记为激活状态。
