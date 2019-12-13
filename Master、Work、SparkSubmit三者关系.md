## SparkSubmit
- 1、SparkClientSubmit 向 StandaloneRestServer提交 Submission请求；StandaloneRestServer接收请求并向Master发送RegisterDriver请求
- 2、Master接收StandaloneRestServer提交的RegisterDriver请求，遍历worker资源，寻找合适的worker并且向他发送launchDriver指令
## Driver
- Driver端通过，appClient向Master注册Driver信息，带有Executor，CoarseGrainedSchedulerExecutorBackend信息
## Master
- 1、leader选举
- 2.1、接收worker注册，并且管理worker
- 2.2、接收SparkSubmit的 RegisterDriver请求，并且向worker发送launchDriver指令
- 2.3、接收Driver端通过AppClient发送的ApplicationRegister请求，并向worker发送launchExecutor指令
## Worker
- 1、心跳
- 接收Master的launchExecutor指令
- 接收Master的launchDriver指令

#调度代码
Master调度启动Driver和Executor过程

```
 private def schedule(): Unit = {
    if (state != RecoveryState.ALIVE) {
      return
    }
    // Drivers take strict precedence over executors driver拥有严格超过executor的优先权
    val shuffledAliveWorkers = Random.shuffle(workers.toSeq.filter(_.state == WorkerState.ALIVE))
    val numWorkersAlive = shuffledAliveWorkers.size
    var curPos = 0
    for (driver <- waitingDrivers.toList) { // iterate over a copy of waitingDrivers
      // We assign workers to each waiting driver in a round-robin fashion. For each driver, we
      // start from the last worker that was assigned a driver, and continue onwards until we have
      // explored all alive workers.
      var launched = false
      var numWorkersVisited = 0
      while (numWorkersVisited < numWorkersAlive && !launched) {
        val worker = shuffledAliveWorkers(curPos)
        numWorkersVisited += 1
        if (worker.memoryFree >= driver.desc.mem && worker.coresFree >= driver.desc.cores) {
          launchDriver(worker, driver)
          waitingDrivers -= driver
          launched = true
        }
        curPos = (curPos + 1) % numWorkersAlive
      }
    }
    startExecutorsOnWorkers()
  }
  
  
  startExecutorsOnWorkers()
  private def startExecutorsOnWorkers(): Unit = {
    // Right now this is a very simple FIFO scheduler. We keep trying to fit in the first app
    // in the queue, then the second app, etc.
    for (app <- waitingApps if app.coresLeft > 0) {
      val coresPerExecutor: Option[Int] = app.desc.coresPerExecutor
      // Filter out workers that don't have enough resources to launch an executor
      val usableWorkers = workers.toArray.filter(_.state == WorkerState.ALIVE)
        .filter(worker => worker.memoryFree >= app.desc.memoryPerExecutorMB &&
          worker.coresFree >= coresPerExecutor.getOrElse(1))
        .sortBy(_.coresFree).reverse
      val assignedCores = scheduleExecutorsOnWorkers(app, usableWorkers, spreadOutApps)

      // Now that we've decided how many cores to allocate on each worker, let's allocate them
      for (pos <- 0 until usableWorkers.length if assignedCores(pos) > 0) {
        allocateWorkerResourceToExecutors(
          app, assignedCores(pos), coresPerExecutor, usableWorkers(pos))
      }
    }
  }
   private def allocateWorkerResourceToExecutors(
      app: ApplicationInfo,
      assignedCores: Int,
      coresPerExecutor: Option[Int],
      worker: WorkerInfo): Unit = {
    // If the number of cores per executor is specified, we divide the cores assigned
    // to this worker evenly among the executors with no remainder.
    // Otherwise, we launch a single executor that grabs all the assignedCores on this worker.
    val numExecutors = coresPerExecutor.map { assignedCores / _ }.getOrElse(1)
    val coresToAssign = coresPerExecutor.getOrElse(assignedCores)
    for (i <- 1 to numExecutors) {
      val exec = app.addExecutor(worker, coresToAssign)
      launchExecutor(worker, exec)
      app.state = ApplicationState.RUNNING
    }
  }
```