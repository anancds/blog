Mesos相当于一个数据中心的操作系统内核，管理很多机器，Framework相当于这个操作系统的应用程序，每当应用程序需要执行，Framework就会在mesos中选择一台或者多台有合适资源(cpu、内存)的从机来运行。

### 资源调度

Mesos实现了一个两层的调度系统：Mesos slave将它的可用资源汇报给master，然后master通过可插拔的分配模块（Allocation module）向Framework的schedule发出资源邀约，这个schedule可以接受整个、部分或者拒绝这个资源邀约。如下图所示：
![](http://img2.tuicool.com/qYZb22u.png!web)

* Mesos slave向master通告它的资源：8CPUs，16GB内存，64GB磁盘可用空间。星号(* ) 表示这些资源属于默认角色(default role)。
* Mesos的分配模块决定master应该向Framework A的schedule发出资源邀约。
* Framework A的schedule接受了资源邀约的一半的资源，剩下4CPUs，8GB内存及32GB的可用磁盘空间给其它程序。
* Mesos的分配模块决定master应该将所有剩下的未分配的资源通告给Framework B的schedule。

Mesos slave的资源通告包括：
* cpus
* memory
* disk
* ports

上面的过程一直重复不断地进行，因为每过一段时间，slaves会因为任务运行完毕而空出可用的资源来。当slave持续不断地将自己的可用资源通告给master时，Mesos的另一个部分，可插拔分配模块，负责决定哪个Framework应该得到给定的资源邀约。

### 资源分配

Mesos master的分配模块（Allocation module）决定了将资源分配给某个Framework。此模块可插拔的特性，允许系统工程师根据自己组织的需要，来实现他们自己的分配策略和算法。内置的分配模块使用的是Dominant Resource Fairnes（DRF）算法，它可以满足大部分Mesos用户的需要。

Mesos默认提供了几种方法来对资源分配进行调整，而不需要替换或者重写整个分配模块。这些方法包括 角色 ， 权重 和 资源保留 。

### 角色

在Mesos集群中，角色允许您将Frameworks和资源分成任意的组。

要使用角色，需要在启动master时加入–roles的配置选项。例如，下面的配置允许Frameworks以三个在数据中心里常见的角色注册：

    --roles="dev,stage,prod"

之后，当Framework要在master上注册时，Framework可以指定为这些角色中的任意一种。这就允许许多团队或许多环境来共享一个大的Mesos集群，而不用创建多个小的集群。您也可以使用角色来确保特定类型的工作只运行在特定的一组机器上，例如负载均衡器或反向代理只运行在专用的边缘节点上。

### 权重

集群也可以给每个角色配置权重，来使不同的角色对于资源的分配有不同的优先权。当Mesos决定将资源先分配给哪个Framework时，它先将资源分配给最低于该权重的合理份额的Framework。例如，参考如下配置：

    --weights="dev=10,stage=20,prod=30"

具有prod角色的Frameworks将得到比具有dev角色的Frameworks的三倍的资源。

### 资源保留

虽然权重可以确保某个角色得到比其它角色更多的资源，但Mesos也提供资源保留的方法。资源保留确保某个角色总是可以得到slave的一定的资源，但这样做可能会导致整个集群使用率的降低。

假如有一台机器有16CPUs，32GB内存，128GB磁盘，你想要确保这台机器有一半的资源总是可以提供给以prod角色注册的Frameworks，可以这样在这台Mesos slave机器上配置：

    --resources="cpus(prod):8; mem(prod):16384; disk(prod):65536"

### 手动配置Mesos slave的资源和属性

Mesos提供了三种数据类型来指定资源： 标量，范围，和集合 。
* 标量：cpus资源给定值8；mem资源给定值16384。
* 范围：端口资源给定从10000到20000的值。
* 集合：磁盘资源给定值ssd1，ssd2，和ssd3。

    --resources="cpu(*):4; mem(*):8192; disk(*):32768; ports(*):[40000-50000];cpu(prod):8; mem(prod):16384; disk(prod):65536"

### 资源隔离

Mesos中的一个基础理念是使用容器隔离进程是使用计算资源最有效率的方式。

Mesos默认支持Linux cgroups和Docker，两种当下最流行的容器技术。通过在容器中运行executors和任务，Mesos slave允许多个Framework的executor一起运行，而不影响其它负载。这有点像在每个物理机上同时运行多个虚拟机，容器不用启动整个操作系统而比虚拟机轻量化许多。

Mesos的一个基础的组件称为containerizer。可以使用 –containerizers配置选项在Mesos slave上配置，目前包括两个containerizer：mesos和docker。mesos配置使用cgroups隔离和监视负载；docker配置调用Docker容器运行时，允许您在Mesos集群上启动已经编译好的镜像。

除了containerizer，Mesos也提供其它方法隔离资源。包括posix/cpu和posix/mem（默认），cgroups/cpu和cgroups/mem。怎样使用对资源的隔离和监视呢？是通过在slave上，给–isolation配置选项提供参数列表，来实现的。
