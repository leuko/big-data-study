# Spark 转载笔记

* [Spark基础RDD+DAG](Spark第一天.pdf)

* [Spark运行架构](Spark第二天.pdf)

* [参数配置+调优策略](Spark第三天.pdf)

* [Spark SQL](Spark第四天.pdf)

* [数据挖掘和机器学习](Spark第五天.pdf)

* [算法](Spark第六天.pdf)


# Spark学习笔记



## SparkContext

三大核心对象：DAGScheduler、TaskScheduler、ShedulerBackend





## Spark standalone

> Spark runtime：Driver、Master、Worker、Executor、CoarseGrainedExecutorBackend



1、Driver启动时，核心是SparkContext，会向Master注册。Driver进程有2个重要的Endpoint，ClientEndpoint、DriverEndpoint



2、SparkContext在实例化的时候会构造：

`StandaloneSchedulerBackend`(继承`CoarsdGrainedSchedulerBackend`)负责集群Worker硬件资源的管理和调度、

`DAGScheduler`负责高层调度（如Job中Stage的划分、数据本地性等内容）、

`TaskSchedulerImpl`负责具体Stage内部的底层调度（如具体每个Task的调度、容错），根据计算本地性原则确定Task具体要运行在哪个ExecutorBackend、

`MapOutputTrackerMaster`负责Shuffle中数据输出和读取管理。



3、Task工作线程

Worker接到任务，启用进程`CoarseGrainedExecutorBackend`，`CoarseGrainedExecutorBackend`启动后向Driver的`CoarseGrainedSchdulerBackend`发消息，Driver回复注册成功信息给Worker，然后Worker启动`new Executor`并构造线程池，线程池执行具体的`Task`

`Master`接收用户提交的程序，并发送指令给`Worker`为当前程序分配资源，每个`Worker`所在节点 默认 为当前程序分配一个`Executor`，在`Executor`中通过线程池并发执行 ,实际工作中会通过TaskRunner来封装Task，然后从ThreadPool中获取一条线程来执行Task，执行完后线程被回收复用。

`Worker`为集群中的节点，`Worker`进程通过一个`Proxy`为`ExecutorRunner`的对象实例来远程启动`ExecutorBackend`进程

`Master`分配资源参考来源 `spark-env.sh` `spark-defaults.sh` `spark-submit提供的参数` `SparkConf配置的参数`。

最后一个Stage中的Task称为ResultTask，其他前面的Stage中的Task都是ShuffleMapTask，为下一阶段的Stage数据准备



4、Master对组件的注册：Drivers、Workers、Application

ExecutorBackend不会注册给Master，ExecutorBackend是注册给Driver中的SchedulerBackend的



5、Spark资源调度分配（分配Driver、Application）

> 任务调度是通过DAGScheduler、TaskScheduler、SchedulerBackend等进行的作业调度；
>
> 资源调度是指应用程序如何获得资源；
>
> 任务调度是在资源调度的基础上进行的，没有资源调度那么任务调度就成为了无源之水无本之木

所有资源的调度都发送在`Master`，当注册程序或者资源发生改变的时候都会导致`schedule()`的调用：This method will be called every time a new app joins or resource availability changes

```scala
private def schedule(): Unit = {
    if (state != RecoveryState.ALIVE) {
      return
    }
    // Drivers take strict precedence over executors
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
```



6 、BlockManager

对整个集群的block数据进行管理



7、Spark RPC

`RpcEnv`、`RpcEndpoint`、`RpcEndpointRef`



大总结：

在`SparkContext`实话的时候调用`createTaskScheduler`来创建`TaskSchedulerImpl`和`StandaloneSchedulerBackend`(继承自`CoarseGrainedSchedulerBackend`)，同时在`SparkContent`实例化的时候会调用`TaskSchedulerImpl`的`start`，在`start`方法中会调用`StandaloneSchedulerBackend`的`start`，在该`start`方法中会创建`StandaloneAppClient`并启动`start`，在改start会创建`ClientEndpoint`，在创建`ClientEndpoint`会传入`Command`来指定具体为当前应用程序启动的`Executor`进行的入口类的名称为`CoarseGrainedExectorBackend`，然后`ClientEndpoint`启动并通过`tryRegisterMaster`来注册当前的应用程序到Master中，Master接收到注册信息后，则会为该程序生成Job ID并通过Scheduler来分配计算资源，Master会发送指令给Worker，Worker首先会分配ExecutorRunner，ExecutorRunner内部通过Thread的方式构建ProcessBuilder来启动另外一个JVM进程，这个JVM进程就是前面说的Command指定的`CoarseGrainedExectorBackend`的类，改`ExectorBackend`会向`DriverEndpoint`发送`RegisterExector`来注册当前的信息，当`DriverEndpoint`收到注册信息会保存在StandaloneSchedulerBackend实例的内存数据结构中。



## Spark on Yarn

`Client`客户端向`ResourceManager`提交application，RM接收后，并在具体的某个`NodeManager`中启动`Driver`(App Master)，当app maseter启动（会下载当前app相关的jar等资源）的时候会首先向RM注册说明并申请资源，RM接收到app master的资源分配请求，会最大化分配资源，并发资源元数据信息给app master，app master收到资源的元数据后，发指令要求`NodeManager`启动具体的`Container`， `Container`启动后向app maseter注册，当app master收到container后，开始进行任务调度和计算



### 两种运行模式

`Yarn`的`ResourceManager`就相当于`Spark Standalone`模式下的`Master`

1、Cluster模式

`Driver`运行在Yarn集群下的某台NodeManager上的JVM进程中



2、Client模式

`Driver`运行在当前提交程序的机器上，一般在测试中使用，方便查看运行过程中的信息

```shell
./spark-submit --class org.apache.spark.examples.SparkPi  --master yarn --deploy-mode client  /path/to/your.jar
```



> Spark on Yarn 的模式下 Hadoop Yarn的配置`yarn.nodemanager.local-dirs`会覆盖 Spark的`spark.local.dir`配置
>
> 查看运行的日志信息，可以用命令 `yarn logs -applicationId <app ID>`







## Shuffle可能面临的问题

1、数据量非常大

2、数据如何分类：即如何Partition：Hash、Sort、钨丝计算

3、负载均衡（数据倾斜）

4、网络传输效率，需压缩和解压缩、序列化和反序列化中权衡



### Hash Shuffle & Sort Shuffle

1、Shuffle前会产生很多小文件，文件数为：Partition数 * Reduce数，为改善此问题，Spark推出了Consolidate机制合并小文件，一个Task就一个文件，Task也即并行度，文件数为：Task数 * Reduce数，但此问题还是存在

2、为了让spark在更大规模的集群上更高性能的处理数据，貌似从1.1开始 spark引入了sort-based shuffle，sort产生的文件数为：2  * Task数。1.5开始引入钨丝计划，spark推上一个巅峰



