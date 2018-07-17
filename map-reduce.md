# MapReduce学习笔记

## 1. map任务处理

文件逻辑切片，每一个切片对应一个Mapper
Mapper读取输入切片内容，每行解析成一个k1、v1对(默认情况)。
每一个键值对调用一次map函数
执行map中的逻辑，对输入的k1、v1处理，转换成新的k2、v2输出。

> MR内部执行流程
>
> MR可以单独运行 也可以经由YARN分配资源运行 
>
> YARN框架的组成
> 	JobTracker -> ResourceManager
> 	TaskTracker -> NodeManager
> 	Mapper Reducer
>
> 客户端提交一个mr的jar包给JobClient(提交方式：hadoop jar ...)
> JobClient通过RPC和ResourceManager进行通信，表明要发起一个作业，ResourceMnager返回一个存放jar包的地址（HDFS）和jobId
> client将jar包和相关配置信息写入到HDFS指定的位置中当中(path = hdfs上的地址 + jobId)
> client联系ResourceManager提交任务(任务的描述信息，不是jar, 包括jobid，jar存放的位置，配置信息等等)
> ResourceManager进行初始化作业
> ResourceManager读取HDFS上的要处理的文件信息，开始计算输入分片，算好有多少个split，规划出Mapper和Reducer的数量
> 再在NodeManager的心跳响应中分配任务给NodeManager执行
> 分配任务通常采用本地化策略 将任务分配给数据所在DataNode同一个节点的NodeManager中
> NodeManager通过心跳机制领取任务（任务的描述信息）
> 访问hdfs下载所需的jar，配置文件等
> NodeManager启动一个单独的java child子进程 worker进程，用来执行具体的任务（MapperTask或ReducerTask） 
> 之所以单独启动一个worker进程，而不是让NodeManager自己执行运算，是担心如果程序崩溃，可能造成进程的退出，如果是NodeManger执行任务崩溃，就会丢失一个计算节点，所以还不如单独启动一个 Worker进程，这样崩溃的也只是Worker而不是NodeManager
> Worker中运行指定的Mapper或Reducer  task 最终将结果写入到HDFS当中
>
> 整个过程传递的是代码 而不是数据，即，数据在哪里，就让运算发生在哪里，减少对数据的移动，提高效率	
>
> 



## 2. shuffle阶段

分配map输出的数据到reduce 其中会将k2 v2 转换为k3 v3
中间包括 buffer splill group parition sort combiner(合并多个spill,同样执行partition sort combiner)等操作

- 排序 
  Map执行过后，在数据进入reduce操作之前，数据将会按照K3进行排序，利用这个特性可以实现大数据场景下排序的需求

- Partitioner 
  分区操作是shuffle操作中的一个重要过程，作用就是将map的结果按照规则分发到不同reduce中进行处理，从而按照分区得到多个输出结果

  Partitioner是partitioner的基类，如果需要定制partitioner也需要继承该类
  HashPartitioner是mapreduce的默认partitioner。计算方法是
    which reducer=(key.hashCode() & Integer.MAX_VALUE) % numReduceTasks
  注：默认情况下，reduceTask数量为1

  很多时候MR自带的分区规则并不能满足我们需求，为了实现特定的效果，可以需要自己来定义分区规则。

- Combiner -- 	合并

  每一个MapperTask可能会产生大量的输出，combiner的作用就是在MapperTask端对输出先做一次合并，以减少传输到reducerTask的数据量。
   combiner是实现在Mapper端进行key的归并，combiner具有类似本地的reduce功能。

  如果不用combiner，所有的结果都是reduce完成，效率会相对低下。使用combiner，先完成在Mapper的本地聚合，从而提升速度。

  ```java job.setCombinerClass(WCReducer.class);```

  注意：
  	Combiner的目的是为了提升程序效率，而不是改变程序的结果，所以要保证一个combiner无论是否执行 无论执行多少次都不应该改变最终的结果 。

  

## 3. reduce任务处理

输入shuffle得到的k3 v3 执行reduce处理得到k4 v4
把reduce的输出的k4 v4写出到目的地



## 4. Mapper的数量

  默认情况下是不可干预的，不过可以通过配置mapred.min.split.size来控制split的size的最小值

## 5. InputFormat （默认TextInputFormat）

  类用来产生InputSplit， 并基于RecordReader切分成record，形成Mapper的输入。
  Hadoop内置提供的有：

`KeyValueTextInputFormat`，`SequenceFileInputFormat`，`NLineInputFormat`，`CompositeInputFormat`

## 6. 多数据来源

```java
MultipleInputs.addInputPath(Job job, Path path, Class<? extends InputFormat> inputFormatClass, Class<? extends Mapper> mapperClass)
```

## 7. 多输出文件

```java
MultipleOutputs.addNamedOutput(Job job, String namedOutput, Class<? extends OutputFormat> outputFormatClass, Class<?> keyClass, Class<?> valueClass)
```

## 8. 二次排序

## 9. 数据倾斜

  分区设置不合理 - 重新设置分区合理分配数据
  业务数据本身倾斜 - 使用Combiner减轻倾斜
    - 单独拿出来处理量
    - 将一个MR 拆分为多个MR 降低倾斜造成的危害

## 10. 小文件处理

  Hadoop Archive
  命令：```shell hadoop archive -archiveName <NAME>.har -p <parent path> <dest>
