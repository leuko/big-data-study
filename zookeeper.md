# ZooKeeper学习笔记

## 1. Zookeeper概述

Zookeeper是一个工具，可以实现集群中的分布式协调服务。
所谓的分布式协调服务，就是在集群的节点中进行可靠的消息传递，来协调集群的工作。

Zookeeper之所以能够实现分布式协调服务，靠的就是它能够保证分布式数据一致性。
所谓的分布式数据一致性，指的就是可以在集群中保证数据传递的一致。

Zookeeper能够提供的分布式协调服务包括：数据发布订阅、负载均衡、命名服务、分布式协调/通知、集群管理、分布式锁、分布式队列等功能

## 2. Zookeeper的特点

Zookeeper工作在集群中，对集群提供分布式协调服务，它提供的分布式协调服务具有如下的特点：
	顺序一致性
从同一个客户端发起的事务请求，最终将会严格按照其发起顺序被应用到zookeeper中
	原子性
所有事物请求的处理结果在整个集群中所有机器上的应用情况是一致的，即，要么整个集群中所有机器都成功应用了某一事务，要么都没有应用，一定不会出现集群中部分机器应用了改事务，另外一部分没有应用的情况。
	单一视图
无论客户端连接的是哪个zookeeper服务器，其看到的服务端数据模型都是一致的。
	可靠性
一旦服务端成功的应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会一直保留下来，除非有另一个事务又对其进行了改变。
	实时性
zookeeper并不是一种强一致性，只能保证顺序一致性和最终一致性，只能称为达到了伪实时性。

## 3.zookeeper集群的搭建

安装jdk，并且配置jdk的环境变量
下载zookeeper的安装包，上传到linux集群环境下
	http://zookeeper.apache.org/
解压安装包
	tar -zxvf zookeeper-3.4.7.tar.gz
进入conf目录，复制zoo-sample.cfg为zoo.cfg,通过修改zoo.cfg来对zookeeper进行配置
	修改1：dataDir 指定zookeeper将数据保存在哪个目录下，如果不修改，默认在/tmp下，这个目录下的数据有可能会在磁盘空间不足或服务器重启时自动被linux清理，所以一定要修改这个地址
	修改2：修改服务器列表
单机模式：在zoo.cfg中只配置一个server.id就是单机模式了
伪分布式：在zoo.cfg中配置多个server.id，其中ip都是当前机器，而端口各部相同，启动时就是伪集群模式了
完全分布式：多台机器各自配置
	server.1=xxx.xxx.xxx.xxx:2888:3888	
	server.2=xxx.xxx.xxx.xxx:2888:3888	
	server.3=xxx.xxx.xxx.xxx:2888:3888
	修改3：到之前配置的zookeeper数据文件所在的目录下生成一个文件叫myid，其中写上一个数字表明当前机器是哪一个编号的机器
启动zookeeper
	启动ZK服务:bin/zkServer.sh start
	停止ZK服务:bin/zkServer.sh stop
	重启ZK服务:bin/zkServer.sh restart
	查看ZK服务状态:	bin/zkServer.sh status

## 4.zookeeper的数据模型

zookeeper中可以保存数据，正是利用zookeeper可以保存数据这一特点，我们的集群通过在zookeeper里存取数据来进行消息的传递。
zookeeper中保存数据的结构非常类似于文件系统。都是由节点组成的树形结构。不同的是文件系统是由文件夹和文件来组成的树，而zookeeper中是由ZNODE来组成的树。
每一个ZNODE里都可以存放一段数据，ZNODE下还可以挂载零个或多个子ZNODE节点，从而组成一个树形结构。

|          | 顺序节点     | 普通节点     |
| -------- | ------------ | ------------ |
| 临时节点 | 顺序临时节点 | 普通临时节点 |
| 持久节点 | 顺序持久节点 | 普通持久节点 |

顺序节点：指定叫什么，除了前缀是指定的名字外，在名字后会会自带一个独一无二自动增长的的编号 普通节点：指定叫什么就叫什么
临时节点：一个客户端连接创建的临时节点，会在当客户端会话结束时立即自动删除。 持久节点：创建出来后只要不删除就不会消失，无论客户端是否连接。
1.每个节点成为znode节点
2.多个znode节点共同组成一个znode树
3.每个znode节点都可以存储数据
4.每个znode节点都具有唯一性。基于这个特点，可以通过zookeeper做统一命名服务
5.znode树是维系在内存中，工用户快速查询
6.znode节点，4种①create /park01 ②create -e /park01 ③create -s /park01 
④creaste -e -s /park01
7.临时节点特点:当创建此临时节点的客户端断开连接后，节点被删除
8.Zxid（事务id），是一个递增的id，事务id越大，事务越新。选举时会用到。
9.选举id，server.1=ip:2888,3888，如果Zxid比较不出来，谁打谁当Leader
10.客户端连接zookeeper服务的超时的阈值：下限：2*ticktime 上限：20*ticktime

## 5.zookeeper的shell下操作

执行bin/zkCli.sh [-server ip:port] #如不指定，则连接本机

列出：`ls path [watch]	#列出所有数据节点`

练习：列出根目录下所有数据节点 `ls /`

创建：`create [-s][-e] path data acl #创建数据节点`
--其中 -s表示顺序节点 -e表示临时节点，两个都不加则是持久节点
--acl 指定权限控制，不赋值则不进行任何权限控制

练习：创建/zk-book 其中数据为123：`Create /zk-book 123`

获取：`get path [watch]`

练习：列出 /zk-book中的数据 `get /zk-book`

更新：
	set path data [version]
--data 为要更新的内容 version指定要基于哪个版本的数据进行更新

删除：
	delete path [version]
--path 为要删除的路径 version指定要基于哪个版本进行删除
--注意，要删除的节点下必须没有任何子节点才能删除成功

## 6.zookeeper的java api操作

创建java项目 并导入zookeeper相关的包

```xml
<dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.4.12</version>
      <type>pom</type>
    </dependency>
```

​	
创建会话：

```java
	Zookeeper(String connectString,int sessionTimeout,Watcher watcher)

	Zookeeper(String connectString,int sessionTimeout,Watcher watcher,boolean canBeReadOnly)

	Zookeeper(String connectString,int sessionTimeout,Watcher watcher,long sessionId,byte[] sessionPasswd)

	Zookeeper(String connectString,int sessionTimeout,Watcher watcher,long sessionId,byte[] sessionPasswd,boolean canBeReadOnly)

```

参数说明：
connectString -- host:port[,host:port][basePath] 指定的服务器列表，多个host:port之间用英文逗号分隔。还可以可选择的指定一个基路径，如果指定了一个基路径，则所有后续操作基于这个及路径进行。
sessionTimeOut -- 会话超时时间。以毫秒为单位。客户端和服务器端之间的连接通过心跳包进行维系，如果心跳包超过这个指定时间则认为会话超时失效。
watcher -- 指定默认观察者。如果为null表示不需要观察者。
canBeReadOnly -- 是否支持只读服务。只当一个服务器失去过半连接后不能再进行写入操作时，是否继续支持读取操作。略
sessionId、SessionPassword -- 会话编号 会话密码，用来实现会话恢复。

**注意，整个创建会话的过程是异步的，构造方法会在初始化连接后即返回，并不代表真正建立好了一个会话，此时会话处于"CONNECTING"状态。
**当会话真正创建起来后，服务器会发送事件通知给客户端，只有客户端获取到这个通知后，会话才真正建立。

代码：

```java
public void conn() throws Exception{
    
	final CountDownLatch cdl = new CountDownLatch(1);
    
	ZooKeeper zk = new ZooKeeper("hadoop01:2181,hadoop02:2181,hadoop03:2181", 30
	, new Watcher(){
        @Override
        public void process(WatchedEvent event) {
            if(event.getState() == KeeperState.SyncConnected){
                cdl.countDown();
            }
        }
	});
	cdl.await();
	System.out.println("连接zk成功~~~");
}
```

创建节点:

```java
String create(final String path,byte data[],List<ACL> acl,CreateMode createMode);
//同步方式创建	
void create(final String path,byte data[],List<ACL> acl,CreateMode createMode,StringCallback cb,Object ctx);//异步方式创建

```


参数说明:
path 要创建的数据节点的路径
data [] 节点创建时初始数据内容
acl 节点acl安全策略
createMode 创建模式
	PERSISTENT 持久
	PERSISTENT_SEQUENTIAL 持久顺序
	EPHEMERAL 临时
	EPHEMERAL_SEQUENTIAL 临时顺序
cb 回调接口
ctx 传递对象，用来在回调方法中使用 通常是个上下文对象

>  **注意：不支持递归创建，即不能在无父节点的情况下创建出子节点

> **尝试创建已经存在的节点将失败并抛出异常

> **在不需要进行任何权限控制时，只需传入Ids.OPEN_ACL_UNSAFE即可

```java
//更新数据
//同步方式
Stat setData(final String path,byte data[],int version);
//version可以传入-1，表明要基于最新版本进行更新操作
//异步方式
void setData(final String path,byte data[],int version,StatCallback cb,Object ctx)
// 读取数据:getChildren

//同步方式
List<String> getChildren(final String path,Watcher watcher)
List<String> getChildren(String path,boolean watch)
List<String> getChildren(final String path,Watcher watcher,Stat stat)
List<String> getChildren(String path,boolean watch,Stat stat)

//异步方式
void getChildred(final String path,Watcher watcher,ChildrenCallback cb,Object ctx)
void getChildred(String path,boolean watch,ChildrednCallback cb,Object ctx)
void getChildred(final String path,Watcher watcher,Children2Callback cb,Object ctx)
void getChildred(String path,boolean watch,Children2Callback cb,Object ctx)

```



参数说明:
	path 要创建的数据节点的路径
	watcher 观察者，一旦在本子节点获取之后，子节点列表发生变更，服务器端向客户端发送消息，触发watcher中的回调。注意，仅仅是通知而已，如果需要新的子节点列表，需要自己再次去获取。允许传入null。
	watch 表明是否需要注册一个Watcher。为true则通知默认到默认watcher，如果为false则不使用
	cb 回掉函数
	ctx 上下文对象
	stat 指定数据节点的状态信息。用法是在接口中传入一个旧的stat变量，该stat变量会在方法执行过程中，被来自服务端响应的新stat对象替换。

```java
//同步方式
byte [] getData(final String path,Watcher watcher, Stat stat)
byte [] getData(String path,boolean watch, Stat stat)

//异步方式
void getData(final String path,Watcher watcher, DataCallback cb,Object ctx)	
void getData(String path,boolean watch, DataCallback cb,Object ctx)

```

​	

*可以通过注册Watcher进行监听，一旦该节点数据被更新会通知客户端

删除节点:

```java
public void delete(final String path,int version)
public void delete(final String path,int version,VoidCallback cb,Object ctx)

```



**注意：无法删除存在子节点的节点，即如果要删除一个节点，必须要先删除其所有子节点

检查节点是否存在

```java
//同步方式
	public Stat exists(final String path,Watcher watcher)
	public Stat exists(String path,boolean watch)

	//异步方式
	public Stat exists(final String path,Watcher watcher,StatCallback cb,Object ctx)
	public Stat exists(String path,boolean watch,StatCallback cb,Object ctx)

```

*可以通过注册Watcher进行监听，一旦节点被创建、删除、数据被更新都会通知客户端

zookeeper权限控制：

```java
addAuthInfo(String schema,byte [] auth) 
```

参数说明;
schema 权限模式，可以是world auth digest ip super，我们使用digest
byte[] auth 权限控制标识，由"foo:123".getByte()方式组成，后续操作只有auth值相同才可以进行

**注意删除操作，如果在增加节点时指定了权限，则对于删除操作，认为权限加在了子节点上，删除当前结点不需要权限，删除子节点需要权限。

## 7.zookeeper原理

zookeeper为了保证可靠性，不能用一台机器，而应该是一个集群

为了保证zookeeper集群数据能够一致，必须有一个拍板说了算的人，这就是leader，其他的是follower。
某一时刻集群里只能有且仅有一个leader。
leader可以执行增删改和查询操作，而follower只能进行查询操作。
所有的更新操作都会被转交给leader来处理，leader批准的任务，再发送给follower去执行来保证和leader的一致性。
由于网络是不稳定的，为了保证执行顺序的一致，所有的任务都会被赋予一个唯一的顺序的编号，一定是按照这个编号来执行任务，保证任务顺序的一致性。

那么什么时候leader可以认为一个客户端的请求可以算是处理成功了呢？
如果只有leader或少数机器来认可这个任务，则leader和这些少量机器如果挂掉，则选出来的新的leader并不知道之前批准过的这个任务，最终会违反数据的可靠性。
所以要求leader在批准一个任务之前应该保证集群里大部分的机器应该是知道这个提案的，这样即使自己挂掉，根据过半同意选出来的leader肯定是知道这个提案的。
而如果leader一定要等到所有follower都同意才执行提案也不好，因为知道有一个机器挂掉，leader就无法工作，也相当于单节点了，无法保证集群可靠性。
所以，只要过半通过leader就可以认为一个提案通过。

所以，
leader在收到客户端提交过来的任务后，会向集群中所有的follower发送提案等待follower的投票，follower们收到这个提议后，会进行投票，同意或者不同意，
leader会回收follower的投票，一旦收到过半的投票表示同意，则leader认为这个提案通过，再发送命令要求所有的follower都进行这个提案中的任务。

因为采用过半同意机制，所以最极端的情况下集群中有过半的机器直到最新提案，而如果过半的机器挂掉，则剩下的机器可能不知道最新提案，则无法保证新选出的leader知道最新提案，所以zookeeper中，集群的及其至少要保证过半的存活才能才能正常工作。

从而可以推导出：

> zookeeper集群必须保证过半存活才能工作
>
> zookeeper的集群中的机器数量最好应该是奇数个，因为需要过半存活集群才能工作，所以偶数个机器提供的集群可靠性其实和偶数-1个机器提供的集群可靠性是一样的。

leader选举的问题：
> `(serverid, ZXID)` 优先级： ZXID > serverid
> 最开始集群启动时，达到过半条件时，serverid大的作为leader。
>
> 当leader挂掉后，会通过过半投票选出具有最高任务编号的机器成为新的leader。如果有相同的任务编号，那么在配置文件中，谁的serverid大谁为leader。

## 8.zookeeper工具

​	
​	
