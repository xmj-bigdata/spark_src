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


### 20191216
- SparkSubmit
    - 1、submit向RestServer提交submission 请求
    - 2、RestServer向Master发送registerDriver请求
- Driver
    - 通过APPClient向Master注册Driver，带有executor信息，CoarseGrainedExecutorBackend
- Master
    - 选举leader
    - 对worker和application管理
        - 接收worker注册并管理worker
        - 接收SparkSubmit提交application，发送launchDriver指令
        - 接收Driver通过AppClient发送的ApplicationRegist请求，发送launchExecutor指令
- Worker
    - 向Master发送心跳
    - 接收Master launchDriver指令
    - 接收Master launchExecutor指令
    
### 20191216-Driver与Executor启动过程
- CoarseGrainedSchedulerBackend
  - start
  - makeOffers（）
  - launchExecutor（）
  - executorDataMap
- DriverEndPoint
  - 接收Executor信息
  - 接收TaskScheduleImpl信息
  - 提交任务
 - CoarseGrainedExecutorBackend
      - Executor
      - Driver
      - onStart()
      - receive()
        - 注册 executor
        - 启动 任务 launchTasks
        - stopTask
        - shutdown 
### 20191216- RPC通讯过程
- 生产者
  - 本地模式 dispatchor postMessage
  - 远端调用 NettyRPCHandler，receiver（）方法，调用dispatchor处理
  endpointdata->InBox->postMessage->receive.offers(data)
- 消费者
    data.inbox.process()
- 消息存储
  - receivers


        

