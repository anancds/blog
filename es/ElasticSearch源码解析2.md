## ElasticSearch基本概念

这里只给出基本概念，后面会根据具体代码分析每个流程。

### index(索引)

索引是具有相似特征的文档的集合，相当于SQL里的数据库，相当于Lucene里的一个或者多个索引，这是由ElasticSearch的shard、replica机制决定的。
索引根据小写名称来区分，名称相当于索引的引用，当索引、搜索、更新、删除文档时都是根据名称来定位索引，集群中的索引数量是没有限制的。

### Document(文档)

文档由字段构成，每个字段有它的字段名和一个或者多个字段值，文档之间可以有各自不同的字段合集，且文档并没有固定的模式，可以认为文档就是一个json对象。

### Type(类型)

在索引内，可以定义一个或者多个类型。类型是索引的逻辑分类，通常来说，类型可以定义为拥有共同字段的集合。

### (Mapping)映射

用户可以配置如何将输入文本分割为词条，哪些词条被过滤，或者哪些附加处理是有必要的，排序时所需的字段内容信息等。

### Node(节点)

单个的ElasticSearch服务实例就是节点，有自己的节点名称(启动时赋予一个默认的随机UUID)。节点根据配置的cluster name加入某个集群，默认为elasticsearch。节点一般分为三种类型：data node、master node、tribe node。

### Cluster(集群)

集群是一个或者多个节点的集合，这些节点共享同一个cluster name。

### Shard(分片)

ElasticSearch针对超大的索引提供了分片机制，即shards。创建一个索引的时候，你可以指定分片个数。每个分片是一个完全独立可用的索引。分片的意义在于：水平拆分和扩展、支持并发操作。分片如何分发，文档如何聚合成搜索请求等操作都在ElasticSearch内部完成。可以用官方博客[Sizing Elasticsearch](https://www.elastic.co/blog/found-sizing-elasticsearch)上的一幅图来描述。

![](https://www.elastic.co/assets/bltff2cbc243e62aed0/scaling-stories.svg)

如图所示,开始在单个节点上设置5个分片，后来扩容到两个节点，那么一个三个分片，另外一个2个分片。再后来扩容到5个节点，那么每个节点一个分片。如果继续扩容，是不能自动切分进行数据迁移的，因为分片切分成本和重新索引的成本差不多，所以可以通过接口重新索引。

ElasticSearch的分片默认是基于id哈希的，id可以用户指定，也可以自动生成。

ElasticSearch禁止同一个分片的主分片和副本分片在同一个节点上，所以如果是一个节点的集群是不能有副本的。


### Replica(备份/副本)

为了提高可靠性，ElasticSearch针对分片提供了备份机制，即replica shards。
备份的意义在于：

* 节点/分片宕掉之后的高可用。需要说明的是，replica和它的primary shard不在同一个节点。
* 提高查询的并发，因为可以在备份上执行查询。

综上所述，每个索引都被分成多个分片。索引可以设置是否备份，一旦设置了备份，每个索引都有primary shards和replica shards。备份机制是索引级别的，索引创建之时，备份机制即生效，之后可以动态修改备份的个数，但是不能修改分片的个数。默认情况下，每个索引分配了5个primary shards和1个replica。这意味着集群中至少需要两个节点。

### Allocation(分配)

将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。

### 恢复以及容灾

分布式系统的一个要求就是要保证高可用。就是不但要考虑正常退出服务，还需要考虑故障导致节点挂掉，ElasticSearch会主动allocation，但如果节点丢失后立刻allocation，稍后节点恢复又立刻加入，会造成浪费。ElasticSearch的恢复流程大致如下：

* 集群中的某个节点丢失网络连接。
* master提升该节点上的所有主分片的在其他节点上的副本为主分片
* cluster集群状态变为 yellow ,因为副本数不够
* 等待一个超时设置的时间，如果丢失节点回来就可以立即恢复（默认为1分钟，通过 index.unassigned.node_left.delayed_timeout 设置）。如果该分片已经有写入，则通过translog进行增量同步数据。
* 否则将副本分配给其他节点，开始同步数据。

但如果该节点上的分片没有副本，则无法恢复，集群状态会变为red，表示可能要丢失该分片的数据了。


### NRT(Near Realtime 准实时)

ElasticSearch是一个准实时的搜索引擎，这就意味着从索引一个文档到该文档可被查询之间有轻微的延迟。

## 分布式以及Elastic

分布式系统要解决的第一个问题就是节点之间互相发现以及选主的机制，可以理解为服务的注册与发现，spring boot、zookeeper、Etcd都提供了成熟的服务发现工具。但是ElasticSearch自己实现了一套工具，好处就是部署服务的成本和复杂度降低了，因为不需要预先安装服务发现的集群，缺点就是将复杂度带入了ElasticSearch内部中。

### 服务发现以及选主 ZenDiscovery

* 节点启动后先ping（这里的ping是 Elasticsearch 的一个RPC命令。如果 discovery.zen.ping.unicast.hosts 有设置，则ping设置中的host，否则尝试ping localhost 的几个端口， Elasticsearch 支持同一个主机启动多个节点）
* Ping的response会包含该节点的基本信息以及该节点认为的master节点。
* 选举开始，先从各节点认为的master中选，规则很简单，按照id的字典序排序，取第一个。
* 如果各节点都没有认为的master，则从所有节点中选择，规则同上。这里有个限制条件就是 discovery.zen.minimum_master_nodes，如果节点数达不到最小值的限制，则循环上述过程，直到节点数足够可以开始选举。
* 最后选举结果是肯定能选举出一个master，如果只有一个local节点那就选出的是自己。
* 如果当前节点是master，则开始等待节点数达到 minimum_master_nodes，然后提供服务。
* 如果当前节点不是master，则尝试加入master。

Elasticsearch 将以上服务发现以及选主的流程叫做 ZenDiscovery 。由于它支持任意数目的集群（1-N）,所以不能像 Zookeeper/Etcd 那样限制节点必须是奇数，也就无法用投票的机制来选主，而是通过一个规则，只要所有的节点都遵循同样的规则，得到的信息都是对等的，选出来的主节点肯定是一致的。但分布式系统的问题就出在信息不对等的情况，这时候很容易出现脑裂（Split-Brain）的问题，大多数解决方案就是设置一个quorum值，要求可用节点必须大于quorum（一般是超过半数节点），才能对外提供服务。而 Elasticsearch 中，这个quorum的配置就是 discovery.zen.minimum_master_nodes 。 说到这里要吐槽下 Elasticsearch 的方法和变量命名，它的方法和配置中的master指的是master的候选节点，也就是说可能成为master的节点，并不是表示当前的master，我就被它的一个 isMasterNode 方法坑了，开始一直没能理解它的选举规则。

###  弹性伸缩 Elastic

Elasticsearch 的弹性体现在两个方面：

* 服务发现机制让节点很容易加入和退出。
* 丰富的设置以及allocation API。

Elasticsearch 节点启动的时候只需要配置discovery.zen.ping.unicast.hosts，这里不需要列举集群中所有的节点，只要知道其中一个即可。当然为了避免重启集群时正好配置的节点挂掉，最好多配置几个节点。节点退出时只需要调用 API 将该节点从集群中排除 （Shard Allocation Filtering），系统会自动迁移该节点上的数据，然后关闭该节点即可。当然最好也将不可用的已知节点从其他节点的配置中去除，避免下次启动时出错。

## 系统架构

ElasticSearch的依赖注入用的是guice,网络使用netty，提供http rest和RPC两种协议，对外提供rest，内部是RPC。
具体请求流程可以描述为：

* 用户发起http请求，ElasticSearch的9200端口接受请求后，传递给对应的RestAction。
* RestAction做的事情很简单，将rest请求转换为RPC的TransportRequest,然后调用NodeClient，相当于用客户端的方式请求RPC服务，只不过transport层会对本节点的请求特殊处理。

ElasticSearch之所以用guice，而不是用spring做依赖注入，关键的一个原因是guice可以帮它很容易实现模块化，通过代码进行模块组装，可以很精确的控制依赖注入的管理范围。比如ElasticSearch给每个shard单独生成一个injector，可以将该shard相关的配置以及组件注入进去，降低编码和状态管理的复杂度，同时删除shard的时候也方便回收相关对象。
