# CoarceGrainedSchedulerExecutor
## start()
- 注册DriverEndPoint 用户rpc通讯

## makeOffers()
为Executor制作伪资源
## launchExecutor
遍历所有已经由 TaskSchedulerImplresourceOffers 封装好的任务。然后将任务进行序列化。
检测序列化后的任务大小，超过 akkaFrameSize - AkkaUtils.reservedSizeBytes 128Mb-200kb。
否则直接提交。
启动Executor

# DriverEndPoint
## 接收Executor信息
- Executor注册信息 ，存储在executorDataMap中
- 接收Executor状态更新信息 TaskSchedulerImpl的updateStatus
## 接收TaskSchedulerImpl信息
- ReceiveOffs
## 提交任务给Executor



