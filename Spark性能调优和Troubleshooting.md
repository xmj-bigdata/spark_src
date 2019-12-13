#Spark性能调优和问题解决
## 常规性能调优
- 1、提升资源 cpu 内存 task资源
- 2、提高并行度
- 3、广播大变量
- 4、序列化 Kryo
- 5、RDD优化 复用、持久化
- 6、调节本地等待时长
    - 场景：一个task，blockManager在Executor中，性能最好
    - 解决方法：设置参数 1）spark.local.waite.process；2）spark.local.waite.node；3）spark.local.waite.rack；
##算子调优
- 1、mapPartition
- 2、foreachePartition
- 3、filter和coalesce配合使用
- 4、repartition解决sparkSQL并行度低问题
- 5、reduceByKey的本地聚合
##shuffle调优
- 1、Map文件聚合
- 2、缓存区设置
    - spark.shuffle.file.buffer 32k
    - spark.shuffle.memoryFaction 0.2
    - 原则：1）程序跑通；2）减少连接数造成的IO性能消耗；
##jvm调优
- 1、降低cache操作的内存占比
- 2、调节Executor堆外内存
    - 场景：大量数据，频繁出现 shufflefile not found；task lost；oom；
    - 原因：executor不足， 从上个stage task shuffle map output file拉数据；executor 内存溢出，挂掉，shuffle file can not found
    - 解决方法：spark-submit 参数调整：1）conf  park.yarn.executor.memoryOverHeap;2）等待时间连接超时，导致达不到 BlockManager的数据
- 3、调节连接等待操作时长
## 数据倾斜
- 1、聚合源数据
- 2、过滤导致倾斜的Key
- 3、提高shuffle操作的reduce并行度
- 4、使用随机key实现双重聚合
- 5、将reduce join转为map join
- 6、sample采样对倾斜的key单独join
- 7、使用随机数以及扩容 进行join
## Spark TroubleShooting
- 1、控制reduce缓冲区大小避免OOM
  - 原因：map数据输出到reduce端，并不是等所有数据都传输完了，reduce再去拉取数据，拉取数据的大小有buffer决定，
reduce缓冲区大小默认值是48M，reduce是边计算变拉数据，可能buffer到10M就开始拉数据了，但是如果数据量特别大，
每次都写满48M，reduce每次拉取的大小都差不多达到极值，这时候聚合就会形成很多的大对象，从而造成了OOM
   - 解决方法：1）调小缓冲区大小；2）资源允许的情况下 ，调大缓冲区，减少拉取次数和IO消耗 配置参数：spark.reducer.maxSizeInflight
- 2、JVM GC导致的reduce拉数失败
   - 场景：shuffle file not found；经常出现，重新提交没有复现
   - 原因：当前executor 在执行 GC，同时下个stage的executor去拉上个stage的task数据；
   - 解决方法：
        - 调整配置参数：spark.shuffle.io.maxRetries 3;spark.shuffle.io.retryWait 5;保证下个stage的task可以拉到上个stage输出的文件
- 3、解决各种序列化导致的报错
    - 场景：用client模式提交spark作业，观察本地打出的log，如果出现Serializable,Serialize等字段，报错的log，那就出现了序列化问题导致的错误。
    client spark 作业，Serializable Serialize
    - 原因：1）序列化 2）可序列化 3）第三方不可用
- 4、解决算子函数返回NULL导致的问题
    - 解决方法： 1）不返回值；2）RDD算子过滤 3）使用压缩算子，提升性能
   
- 5、解决YARN_CLIENT模式导致网卡流量激增问题
    - 原因：使用本地提交任务，导致大量的网络IO资源占用，导致网卡流量激增
    - 解决方法：调试小量可以使用，线上使用CLUSTER模型提交job
- 6、解决YARN_CLUSTER模式JVM栈内存溢出问题
    - 场景：client模式可以，cluster出现jvm（永久代）内存溢出，复杂SparkSQL(大量使用OR)解析，产生大量大对象
    - 原因：cluster模型，JVM默认配置是82M
    - 解决方法：spark-submit提交参数设置 spark.driver.extraJavaOption="-XX:PemSize=128M;-XX:MaxPermSize=256M"
- 7、解决SparkSQL导致JVM栈内存溢出问题
    - 解决方法：同上
