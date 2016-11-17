---
title: GC调优
date: 2016-09-13 11:12:59
tags: java
---



# GC调优

因为应用程序跑在jetty下，所以先说下启动jetty的jvm参数。

    java -XX:+CMSParallelRemarkEnabled -XX:+CMSScavengeBeforeRemark -XX:+ParallelRefProcEnabled -XX:+UseCMSInitiatingOccupancyOnly -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=50 -XX:CMSMaxAbortablePrecleanTime=6000 -XX:ConcGCThreads=4 -XX:MaxTenuringThreshold=8 -XX:NewRatio=3 -XX:ParallelGCThreads=4 -XX:PretenureSizeThreshold=64m -XX:SurvivorRatio=4 -XX:TargetSurvivorRatio=90 -Xmx75600m -Xms75600m -XX:ParallelGCThreads=4 -XX:+UseParNewGC -XX:+UseCompressedOops -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -jar start.jar jetty.http.port=80

## GC目的

*GC的优化一般是最后一项任务*。

进行GC优化的目的一般有两个：

*  将转移到老年代的对象数量降低到最少。
* 减少Full GC的时间。

## GC优化需要考虑的Java参数

| 定义  |  参数      | 描述  |
| ---  | -----  | -----|
|堆内存空间| -Xms |Heap area size when starting JVM。启动JVM时的堆内存空间。|
||-Xmx|Maximum heap area size。堆内存最大限制。|
|新生代空间|-XX:NewRatio|Ratio of New area and Old area。新生代和老年代的占比。|
||-XX:NewSize|New area size。新生代空间|
||-XX:SurvivorRatio|Ratio ofEdenarea and Survivor area。伊甸园空间和幸存者空间的占比|

|GC分类     |参数             |
|----       |------          |
|Serial GC  |-XX:+UseSerialGC|
|Parallel GC|-XX:+UseParallelGC -XX:ParallelGCThreads=value|
|Parallel Compacting GC|-XX:+UseParallelOldGC|
|CMS GC|-XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:CMSInitiatingOccupancyFraction=value -XX:+UseCMSInitiatingOccupancyOnly|
|G1|XX:+UnlockExperimentalVMOptions -XX:+UseG1GC|

* 说明：如果NewRatio为2，意味着新生代老年代之比为1:2，因此该值越大，老年代空间越大，新生代空间越小。NewRatio参数会显著地影响整个GC的性能。如果新生代空间很小，会用更多的对象被转移到老年代空间，这样导致频繁的Full GC，增加暂停时间。

## GC优化过程

### 监控GC状态

* 打印GC日志是很好的方式，因为记录GC日志并不会特别地影响java程序性能。

      -Xloggc:$CATALINA_BASE/logs/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps。

* 监控jetty，可以通过jconsole来监控。

* 通过jstat来监控，比如：jstat -gc <vimid> 1000

一般有两个重要指标：

* 新生代GC平均时间：YGCT/YGC

* 老年代GC平均时间：FGCT/FGC

关于jstat参数的说明如下：

|列     |说明                |
|-------|----------- |
|S0C    |输出Survivor0空间的大小。单位KB。|
|S1C|输出Survivor1空间的大小。单位KB。|
|S0U|输出Survivor0已用空间的大小。单位KB。|
|S1U|输出Survivor1已用空间的大小。单位KB。|
|EC|输出Eden空间的大小。单位KB。|
|EU|输出Eden已用空间的大小。单位KB。|
|OC|输出老年代空间的大小。单位KB。|
|OU|输出老年代已用空间的大小。单位KB。|
|PC|输出持久代空间的大小。单位KB。|
|PU|输出持久代已用空间的大小。单位KB。|
|YGC|新生代空间GC时间发生的次数。|
|YGCT|新生代GC处理花费的时间。|
|FGC|full GC发生的次数。|
|FGCT|full GC操作花费的时间|
|GCT|GC操作花费的总时间。|
|NGCMN|新生代最小空间容量，单位KB。|
|NGCMX|新生代最大空间容量，单位KB。|
|NGC|新生代当前空间容量，单位KB。|
|OGCMN|老年代最小空间容量，单位KB。|
|OGCMX|老年代最大空间容量，单位KB。|
|OGC|老年代当前空间容量制，单位KB。|
|PGCMN|持久代最小空间容量，单位KB。|
|PGCMX|持久代最大空间容量，单位KB。|
|PGC|持久代当前空间容量，单位KB。|
|PC|持久代当前空间大小，单位KB|
|PU|持久代当前已用空间大小，单位KB|
|LGCC|最后一次GC发生的原因|
|GCC|当前GC发生的原因|
|TT|老年化阈值。被移动到老年代之前，在新生代空存活的次数。|
|MTT|最大老年化阈值。被移动到老年代之前，在新生代空存活的次数。|
|DSS|幸存者区所需空间大小，单位KB。|


* -verbosegc

-verbosegc的可用参数如下：

* -XX:+PrintGCDetails
* -XX:+PrintGCTimeStamps
* -XX:+PrintHeapAtGC
* -XX:+PrintGCDateStamps

GC的格式如下：

      [GC [<collector>: <starting occupancy1> -> <ending occupancy1>, <pause time1> secs] <starting occupancy3> -> <ending occupancy3>, <pause time3> secs]

|collector    | minor gc使用的收集器的名字|
|-------|-----------|
|starting occupancy1	|GC执行前新生代空间大小。|
|ending occupancy1	| GC执行后新生代空间大小。|
|pause time1| 因为执行minor GC，Java应用暂停的时间。|
|starting occupancy3	|GC执行前堆区域总大小|
|ending occupancy3	|GC执行后堆区域总大小|
|pause time3| Java应用由于执行堆空间GC（包括major GC）而停止的时间。|

真实环境下的例子如下：

      [GC (CMS Final Remark) [ParNew: 9496241K->543569K(12902400K), 0.5521501 secs] 9496241K->543569K(74833920K), 0.5522263 secs] [Times: user=2.09 sys=0.19, real=0.56 secs]

别的监控工具还有(Java) VisualVM  + Visual GC和[HPJMeter](https://h20392.www2.hpe.com/portal/swdepot/displayProductInfo.do?productNumber=HPJMETER)。


### 分析结果

如果结果表明执行GC的时间只有0.1-0.3s，那么就没有必要浪费时间来进行优化，如果是1s以上，那么就非常有必要了。

如果需要GC优化，那么就要调整GC类型和内存空间了。

一般内存空间与GC执行次数和GC执行时间的关系如下：

* 如果是大内存空间，那么就会减少GC执行次数并且增加GC执行时间。
* 如果是小内存空间，那么就会减小GC执行时间并且增加GC执行次数。

一般优化GC比较重要的参数有堆内存大小，XX:NewRatio的大小，使用的GC类型。
