## Spark-RPC

### 术语解释

#### RpcEndpoint

消息通信体，主要是用来接收消息、处理消息，实现了RpcEndPoint接口就是一个消息通信体（Master、Work），如果想要与一个RpcEndpoint端进行通信，一定需要获取到该RpcEndpoint一个RpcEndpointRef，通过RpcEndpointRef与RpcEndpoint进行通信，只能通过一个RpcEnv环境对象来获取RpcEndpoint对应的RPCEndpointRef，所以RpcEndpoint 需要向RpcEnv注册。

客户端通过RpcEndpointRef发消息，首先通过RpcEnv来处理这个消息，找到这个消息具体发给谁，然后路由给RpcEndpoint实体。Spark默认使用更加高效的NettyRpcEnv。

实际上对于Spark框架来说RpcEndpoint主要是接收消息并处理，具体功能如下：

* 管理一个RpcEndpoint生命周期的操作（constructor-> onStart -> receive* ->onStop）。
* 管理通信过程中一个RpcEndpoint所具有的基于事件驱动的行为（连接、断开、网络异常）。

#### RpcEnv

Rpc通信的上下文环境，管理着整个RpcEndpoint的生命周期，消息发送过来首先经过RpcEnv然后路由给对应的RpcEndPoint，得到RpcEndPoint，其主要功能有：

* 根据name或uri注册endpoints。
* 管理各种消息的处理。
* 停止endpoints

### 具体流程

在Spark
