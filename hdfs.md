# HDFS学习笔记

## NameNode:

 `NameNode`的内存中记录的元数据信息为：文件目录结构，副本数，block的编号，block编号对应的DataNode地址，例如 */test/a.log, 3, {b1,b2,b3},[{b1:[h0,h1,h2]},…]*。

`fsimage`中记录的元数据信息为：文件目录结构，副本数，block的编号（没block对应的地址）。

`edits`操作日志文件。

`fstime`记录最后一次checkpoint的时间。



## HDFS读流程：

使用HDFS提供的客户端，先向NameNode查询文件是否存在，NameNode同时会检查客户端是否有权限，否则抛异常。NameNode会视情况返回文件的部分和全部block，读到块信息，直接去DataNode读，直到读完



## HDFS写流程：

使用HDFS提供的客户端，先向NameNode写RPC请求，NameNode会检查要创建的文件是否存在，同时是否有写权限，成功则创建文件，否则抛异常。当客户端开始写文件时，客户端会将文件切分成多个packets，并在内部以数据队列data queue的形式管理这些packets，并向NameNode的请求blocks，然后直接向一个DataNode传文件，DataNode再与pipeline管道流形式向其他DataNode传，最后一个datanode成功后向上返回ack.packet，当所有block成功后，客户端通知NameNode文件上传成功，NameNode将文件设为可用状态。



## HDFS删除流程：

使用HDFS提供的客户端，先向NameNode发删除请求，NameNode会检查要文件是否存在，同时是否有删除权限。NameNode执行MetaData数据的删除（将该元数据设为已删除状态，只标记，不主动联系datanode删除），向客户端返回删除成功。当DataNode节点向NameNode发送心跳时，NameNode才会通知DataNode删除标记的block。



## HDFS启动流程：

NameNode启动时，先合并edits和fsimage，生成新的fsimage和空的edits文件（这个过程不需secondNameNode参与），然后加载元数据到内存，等待datanode上报块信息，当达到最小启动条件，即每个文件至少有一个block信息，进行必要的副本复制和删除工作，整个过程中，HDFS无法对外工作，处于安全模式，客户端访问会抛出SafeModeException



## DataNode：

数据以block的形式存放在DataNode中，DataNode每三秒向NameNode发送心跳报告，在心跳报告中，向NameNode报告信息，从心跳响应中接收NameNode的指令，执行对块的复制移动删除等操作，如果NameNode10分钟都没收到DataNode的心跳，则认为DN已经lost，并copy其上的block到其他DN



## 端口：

- DSF

> namenode  端口50070
>
> secondnamenode 端口50090
>
> datanode 端口50075

- YARN（MapReduce）

> resourceManager  端口8088
>
> datanodeManager 端口8042



￼
￼
