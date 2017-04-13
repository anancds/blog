### ElasticSearch组件生命周期

关于生命周期封装在component这个package下，一共有这么几个生命周期。

* INITIALIZED
* STOPPED
* STARTED
* CLOSED

所有的服务组件都需要继承AbstractLifecycleComponent，主要就是实现doStart、doStop和doClose。

### ElasticSearch服务起来流程

* 通过elasticsearch脚本可以知道，主函数是Elasticsearch，这个函数主要是用来解析命令行参数，继承了EnvironmentAwareCommand。这个类主要是用来解析命令行参数的，用的库是jopt，个人感觉如果对c语言的getopt熟悉的话，那么就用这个库来解析，当然也可以用commons-cli这个库来解析。
