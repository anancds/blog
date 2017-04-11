## 源码阅读准备

* jdk1.8
* elasticsearch-5.3.0

## 创建用户组

因为ElasticSearch运行在root用户下会抛出异常：java.lang.RuntimeException: can not run elasticsearch as root，所以就创建一个elasticsearch的用户。

    groupadd elasticsearch
    useradd elasticsearch -g elasticsearch -p elasticsearch

进入ElasticSearch的目录，比如说就是elasticsearch：

    chown -R elsearch:elsearch  elasticsearch
    su elsearch cd elasticsearch
    ES_JAVA_OPTS="-Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=4000,suspend=y" ./bin/elasticsearch

这样就开启了远程调试模式。

还有一个远程调试方法是

    gradle run --debug-jvm
