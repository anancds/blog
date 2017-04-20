### guice

这里简单介绍下google guice库的相关的一些概念，主要有：

* Binder
* injector
* Module
* guice

并且假设有这么一个例子：
**Add.java**

    public interface Add {

      public int add(int a, int b);

    }

**SimpleAdd.java**

    public class SimpleAdd implements Add{

      public int add(int a, int b) {
        return a + b;
      }

    }

**AddModule.java**

    public class AddModule implements Module {

      public void configure(Binder binder) {
        binder.bind(Add.class).to(SimpleAdd.class);
      }

    }

#### Binder

Binder接口主要是由与Bindings相关的信息组成的。一个Bingd其实就是一个接口和其相应的实现类的映射，比如上面的例子中的

    binder.bind(Add.class).to(SimpleAdd.class)

同样也可以将一个接口直接映射到一个具体的实例对象，代码如下。

    binder.bind(Add.class).to(new SimpleAdd())

当然也可以将一个接口绑定到一个相应的Provider类

    binder.bind(Add.class).to(new AddProvider<Add>())

#### Injector

Injectors 通常会在客户端 （Clients） 使用，它只关心如何创建 （Creating）和维护 (Maintaining) 对象（生命周期）。Injectors 会去维护一组默认的 Bindings (Default Bindings)，这里我们可以获取创建和维护不同对象间关系的配置信息 （Configuration information）。以下代码将会返回 Add 的实现类对象。

    Add addObject = injector.getInstance(Add.class)

#### Module

Module 对象会去维护一组 Bindings。在一个应用中可以有多个 Module  。反过来 Injectors 会通过 Module 来获取可能的 Bindings。Module 是通过一个包含需要被重写 override 的 Module.configure() 方法的接口去管理 Bindings。 简单地说，就是你要继承一个叫做 AbstractModule的类，这个类实现了 Module 接口，并且重写 configure()方法。

#### Guice

客户端 （Clients） 是通过 Guice 类直接和其他 Objects 进行交互的。Injector 和不同的 Modules 之间的联系是通过 Guice 建立的。例如下面的代码。

    MyModule module = new MyModule();
    Injector injector = Guice.createInjector(module);

### ElasticSearch模块

ElasticSearch对guice框架进行了简单封装，通过ModulesBuilder类构建ElasticSearch的模块，一个ElasticSearch节点包括以下几个模块：

* PluginsModule：插件模块
* SettingsModule：设置参数模块
* NodeModule：节点模块
* NetworkModule：网络模块
* NodeCacheModule：缓存模块
* ScriptModule：脚本模块
* JmxModule：jmx模块
* EnvironmentModule：环境模块
* NodeEnvironmentModule：节点环境模块
* ClusterNameModule：集群名模块
* ThreadPoolModule：线程池模块
* DiscoveryModule：自动发现模块
* ClusterModule：集群模块
* RestModule：rest模块
* TransportModule：tcp模块
* HttpServerModule：http模块
* RiversModule：river模块
* IndicesModule：索引模块
* SearchModule：搜索模块
* ActionModule：行为模块
* MonitorModule：监控模块
* GatewayModule：持久化模块
* NodeClientModule：客户端模块


### ElasticSearch组件生命周期

关于生命周期封装在component这个package下，一共有这么几个生命周期。

* INITIALIZED
* STOPPED
* STARTED
* CLOSED

所有的服务组件都需要继承AbstractLifecycleComponent，主要就是实现doStart、doStop和doClose。

### ElasticSearch服务起来流程

* 通过elasticsearch脚本可以知道，主函数是Elasticsearch，这个函数主要是用来解析命令行参数，继承了EnvironmentAwareCommand。这个类主要是用来解析命令行参数的，用的库是jopt，个人感觉如果对c语言的getopt熟悉的话，那么就用这个库来解析，当然也可以用commons-cli这个库来解析。
