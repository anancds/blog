## NameNode 启动

源码分析一般先看启动脚本，这样就能找到main函数，HDFS的启动脚本就是start-dfs.sh，然后配合日志来跟踪源码。
在hadoop_usage变量可以看出脚本支持三个入参：-upgrade用来升级，-rollback用来升级不成功的回滚操作，clusterId是集群id。
通过start-dfs.sh然后传变量到HDFS脚本，然后启动HDFS的NameNode。

和NameNode启动相关的类有三个：NameNode、NameNodeRpcServer、FSNamesystem。
* NameNodeRpcServer用于接收和处理所有的RPC请求；
* FSNamesystem负责实现NameNode的所有逻辑实现；
* NameNode负责管理并解析配置、RPC接口和HTTP接口。

### main函数

main方法中最重要的函数就是createNameNode，会根据用户传入的参数来解析。
* FORMAT：格式化当前NameNode，调用format方法执行格式化操作；
* BOOTSTRAPSTANDBY：拷贝Active NameNode的最新命名空间到StandBy NameNode；
* GENCLUSTERID：生成一个新的集群ID；
* ROLLBACK：回滚上一次升级；
* INITIALIZESHAREDEDITS：初始化editlog的共享存储空间，并从Active NameNode中拷贝足够的editlog数据，使得StandBy节点能够顺利启动；
* BACKUP：启动BackUp节点；
* CHECKPOINT：同上；
* RECOVER：恢复损坏的元数据以及文件系统；
* METADATAVERSION：确认配置文件夹存在，并且答应fsImage文件和文件系统的元数据信息；
* UPGRADEONLY：升级NameNode。

### NameNode构造方法

主要是根据配置确认是否开启了HA，然后初始化NameNode，完成后，NameNode进入StandBy状态。
初始化NameNode主要是构造Http服务，构造RPC服务，初始化FSNamesystem，最后启动服务。

### NameNode停止

StringUtils.startupShutdownMessage(NameNode.class, argv, LOG);
也就是说NameNode在启动时添加了一个关闭的钩子程序，NameNode关闭或者JVM退出时就会调用。

start-dfs.sh的解析在[start-dfs.sh](https://github.com/anancds/hadoop/blob/trunk/hadoop-hdfs-project/hadoop-hdfs/src/main/bin/start-dfs.sh)
