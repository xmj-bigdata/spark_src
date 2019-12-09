## 练习日志

生产者
本地消息 dispatcher  postMessage
远端rpc NettyRPCHandler dispatchor inbox message
inbox.post(data)

消费者

线程池

MessageLoop


dispatchor
recieve
inbox.process(data)

receivers.take()

inbox.data.process(Dispatchor.this)

receivers取数
inbox的process处理
使用匹配模式

消息存储

Dispatchor receiver EndPointData，endpoints 当前RPC处在的所有Endpoint，endpointsRef

InBox Message:Endpoint
