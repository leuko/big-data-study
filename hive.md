# 一、HIVE概述

## 1.mapreduce的问题
		只能用java开发，对于不会java甚至不会编程的人员来说，这是一个门槛 - 数据仓库开发工程师
		需要对hadoop的底层及api比较了解才能开发复杂而高效的代码 - shuffle
		开发调试麻烦

		能不能想办法解决以上问题，最好能够用一种通用的方式开发大数据处理的程序，可以屏蔽掉底层细节，简化开发测试。

## 2.hive的本质
		HIVE - 在hadoop的基础上架设了一层sql操作的接口，使我们可以通过类sql的 - hql - 来操作hive 由hive将这些hql语句翻译成mapreduce来处理海量数据。

		HIVE基于HQL来操作海量数据，可不可认为它是基于hadoop的数据库？

		不可以！

		Hive并不是基于Hadoop的数据库工具 而是一种基于hadoop的数据仓库工具
		
			数据库
				为线上系统提供实时数据
				提供完整的增删改查的能力
				具有完整的事务支持
				尽量的避免数据的冗余 提高存储 和处理效率

			数据仓库
				为离线的数据分析服务
				只能一次写入多次查询 不支持行级别的增删改
				不强调事务特性
				人为的制造冗余 来提高数据查询的效率
		
		HIVE将结构化的数据文件映射为一张数据库表，并提供完整的sql查询能力。但是只能一次写入 多次查询 不支持行级别的增删改(hadoop2.0后可以追加了) 这是受限于底层的hdfs。 本质上只是在hadoop的基础上加了一层sql的壳，仍然是一种离线数据分析工具。不支持事务的特性。通常会通过制造冗余来 提高数据的查询能力。

		Hive是基于Hadoop的一个数据仓库工具。

## 3.优缺点
		优点：
			优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。
			提供了一系列的工具，可以用来进行数据提取转化加载（ETL），这是一种可以存储、查询和分析存储在 Hadoop 中的大规模数据的机制
			Hive 定义了简单的类 SQL 查询语言，称为 HiveQL，它允许熟悉 SQL 的用户查询数据。同时，这个语言也允许熟悉 MapReduce 开发者的开发自定义的 mapper 和 reducer 来处理内建的 mapper 和 reducer 无法完成的复杂的分析工作。

		缺点：
			Hive不支持在线事务处理
			不支持行级的插入和更新和删除
			相对来说速度比较慢

# 二、HIVE的安装配置
	前提条件：
		安装好JDK 并且配置JAVA_HOME环境变量
		需要hadoop的支持，安装好hadoop 并配置HADOOP_HOME环境变量

	下载：
		从apache官网下载新版本hive，要注意和hadoop版本的匹配。

	安装：
		将下载好的hive安装包上传到linux中。
		解压：tar -zxvf apache-hive-1.2.0-bin.tar.gz
		
	启动：
		进入hive/bin目录，直接运行hive命令，即可进入hive提示符。
		hive不需要任何配置就可以运行，因为它可以通过HADOOP_HOME环境变量获知hadoop的配置信息。

	------------
	可能的安装冲突解决：
		问题描述：
			在使用hadoop2.5.x环境下，启动hive发现报错：
				java.lang.IncompatibleClassChangeError: Found class jline.Terminal, but interface was expected
		问题分析：
			造成这个错误的原因是因为 jline.Terminal这个类有错误。
			经过检查发现，在hadoop/share/hadoop/yarn/lib目录下存在jline-0.9.x.jar
			而在hive/lib/目录下存在jline-2.12.jar
			重复的包不兼容造成了此问题。
		解决方法：
			复制hive/lib/jline-2.12.jar替换hadoop/share/hadoop/yarn/lib中的jline-0.9.x.jar，重启hadoop和hive即可。
			或
			直接将hadoop升级到更高版本，如2.7.x中已经解决此问题。
	------------

# 三、HIVE入门
	$show databases;
		执行后发现默认有一个库default
	$show tables;
		发现没有任何表，证明不use其他库时，默认就是default库。
	$create database tedu;
		发现在hdfs中多出了/user/hive/warehouse/tedu.db目录
		结论1:hive中的数据库对应hdfs中/user/hive/warehouse目录下以.db结尾的目录。
	$use tedu;
	$create table student (id int,name string);
	$show tables;
	$desc student;
	$show create table student;
		发现正确创建出来了表。
		发现在hdfs中多出了/user/hive/warehouse/tedu.db/sutdent目录
		结论2：hive中的表对应hdfs/user/hive/warehouse/[db目录]中的一个目录
	$load data local inpath '../mydata/student.txt' into table student;
		发现/user/hive/warehouse/tedu.db/sutdent下多出了文件
	$select * from student;
		发现查出的数据不正确，原因是建表时没有指定分隔符。默认的分隔符是空格。
	$create table student2 (id int,name string) row format delimited fields terminated by '\t';
	$load data local inpath '../mydata/student.txt' into table student2;
	$select * from student2;
		发现正确查询出了数据。
		结论3：hive中的数据对应当前hive表对应的hdfs目录中的文件中的数据。
	$select count(*) from student;
		发现执行了mapreduce作业，最终现实了结果
		结论4：hive会将命令转换为mapreduce执行。
	$use default;
	$create table teacher(id int,name string);
		发现在hive对应的目录下多出了 tedu.db 文件夹,其中包含user文件夹。
		结论5：hive默认的default数据库直接对应/user/hive/warehouse目录，在default库中创建的表直接会在该目录下创建对应目录。
	
# 四、HIVE配置mysql metastore 元数据库
	hive中除了保存真正的数据以外还要额外保存用来描述库、表、列的数据，称为hive的元数据。
	这些元数据又存放在何处呢？hive需要将这些元数据存放在另外的关系型数据库中。
	如果不修改配置hive默认使用内置的derby数据库存储元数据。
	derby是apache开发的基于java的文件型数据库。
	可以检查之前执行命令的目录，会发现其中产生了一个metastore.db的文件，这就是derby产生的用来保存元数据的数据库文件。
	derby数据库仅仅用来进行测试，真正使用时会有很多限制。
	最明显的问题是不能支持并发。
	经测试可以发现，在同一目录下使用无法同时开启hive，不同目录下可以同时开启hive但是会各自产生metastore.db文件造成数据无法共同访问。
	所以真正生产环境中我们是不会使用默认的derby数据库保存hive的元数据的。

	hive目前支持derby和mysql来存储元数据。

	配置hive使用mysql保存元数据信息：
		删除hdfs中的/user/hive
			hadoop fs -rmr /user/hive	
		复制hive/conf/hive-default.xml.template为hive-site.xml
			cp hive-default.xml.template hive-site.xml 
		在<configuration>中进行配置
			<property>
			  <name>javax.jdo.option.ConnectionURL</name>
			  <value>jdbc:mysql://hadoop01:3306/hive?createDatabaseIfNotExist=true</value>
			  <description>JDBC connect string for a JDBC metastore</description>
			</property>

			<property>
			  <name>javax.jdo.option.ConnectionDriverName</name>
			  <value>com.mysql.jdbc.Driver</value>
			  <description>Driver class name for a JDBC metastore</description>
			</property>

			<property>
			  <name>javax.jdo.option.ConnectionUserName</name>
			  <value>root</value>
			  <description>username to use against metastore database</description>
			</property>

			<property>
			  <name>javax.jdo.option.ConnectionPassword</name>
			  <value>root</value>
			  <description>password to use against metastore database</description>
			</property>

		!!手动创建hive元数据库，注意此库必须是latin1，否则会出现奇怪问题！所以推荐手动创建！并且创建库之前不能有任意的hive操作，否则自动创建出来的库表将使用mysql默认的字符集，仍然报错！
		!!另一种方法是修改mysql的配置文件，让mysql默认编码集就是latin1，这样hive自动创建的元数据库就是latin1的了，但是这已修改将会影响整个mysql数据库，如果mysql中有其他库，这种方式并不好。
			create database hive character set latin1;

		将mysql的连接jar包拷贝到$HIVE_HOME/lib目录下
		
		如果出现没有权限的问题，在mysql授权(在安装mysql的机器上执行)
			mysql -uroot -p
			#(执行下面的语句  *.*:所有库下的所有表   %：任何IP地址或主机都可以连接)
			GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
			FLUSH PRIVILEGES;

		再进入hive命令行，试着创建库表发现没有问题。
				
		测试发现开启多个连接没有问题。

		连接mysql，发现多了一个hive库。其中保存有hive的元数据。DBS-数据库的元数据信息，TBLS-表信息。COLUMNS_V2表中字段信息，SDS-表对应hdfs目录

# 五、HIVE内部表 外部表 分区表 分桶表
	内部表 外部表
		创建hive表，经过检查发现TBLS表中，hive表的类型为MANAGED_TABLE，即 所谓的内部表.

		内部表的特点是，先有表  后有数据，数据被上传到表对应的hdfs目录下进行管理。

		但是在真实开发中，很可能在hdfs中已经有了数据，希望通过hive直接使用这些数据作为表内容。
		此时可以创建hive表关联到该位置，管理其中的数据，这种方式创建出来的表叫做外部表。

		外部表的特点是，先有数据后有表，hive表关联到该位置管理其中的数据

		创建外部表的语法：
			create external table .... location 'xxxx';

		案例：
			#向hdfs中准备文件
			hadoop fs -mkdir /hdata
			hadoop fs -put stu.txt /hdata/stu.txt 
			hadoop fs -put teacher.txt /hdata/teacher.txt		 
			#在hive中创建外部表管理已有数据
			create external table ext_stu(id int ,name string) row format delimited fields terminated by '\t' location '/hdata';

			通过观察发现：
				并不会在/usr/hive/warehouse/[db]/下创建表文件夹
				在元数据库的TBLS里多出了记录，且表的类型为EXTERNAL_TABLE
				在元数据库的COLUMSV2里多出了列相关的记录
				在元数据库的SDS里多出了记录，指向了真正的数据存放的位置
				经过检查发现可以使用其中的数据。成功的建立了一张外部表。

		在删除表时：
			内部表删除了元数据库中相关的元数据，删除了hdsf中表对应的文件夹及其中的数据
			外部表删除了元数据库中相关的元数据，并不会删除关联到的文件夹及其内部的数据

	分区表
		hive也支持分区表
		利用分区表可以对数据进行分区来提高查询的效率，在大量数据经常要按照某些指定特定字段查询时可以设计分区表来提高效率。

		create table book (id bigint, name string) partitioned by (country string) row format delimited fields terminated by '\t'; 
		在创建分区表时，partitioned字段可以不在字段列表中。生成的文件自动就会具有该字段。
		
		分区表加载数据
		load data local inpath './book_china.txt' overwrite into table book partition (country='china');
		load data local inpath './book_english.txt' overwrite into table book partition (country='english');
		load data local inpath './book_jp.txt' overwrite into table book partition (country='jp');

		从分区表 中查询数据
		select * from book;
		select * from book where pubdate='2010-08-22';

		当创建分区表并向分区表写入数据后，会在表对应的文件夹下创建出子一级分区文件夹来存放数据，并且将该目录加入元数据库中的SDS中作为数据来源文件夹。

		当按照分区字段作为条件进行查询时，hive可以直接找到该分区字段值对应的文件夹，直接将该文件夹下的数据返回，效率非常的高

		所以，如果hive表中有大量数据，且经常按照某些字段做查询，则可以将该字段设计为表得到分区字段提升效率

		hive支持一个表中有多个分区字段，在创建表时 依次声明即可。
		create table book2 (id bigint, name string) partitioned by (country string,category string) row format delimited fields terminated by '\t'; 
		load data local inpath '/root/work/china.txt' into table book2 partition (country='china',category='xs');

		多个分区会形成 多级子目录 来分开存放数据 同样可以提高按照分区字段查询数据时的效率
		
		此时手动创建目录是无法被hive使用的，因为元数据库中没有记录该分区。
		如果需要将自己创建的分区也能被识别，需要执行：
			ALTER TABLE book2 add  PARTITION (country='jp',category = 'xs') location '/user/hive/warehouse/park2.db/book2/country=jp/category=xs';
			ALTER TABLE book2 add  PARTITION (country='jp',category = 'it') location '/user/hive/warehouse/park2.db/book2/country=jp/category=it';			

	分桶表
		hive也支持分桶表
		分桶操作是更细粒度的分配方式
		一张表可以同时分区和分桶
		分桶的原理是根据指定的列的计算后将数据分开存放
		分桶的主要作用是：对于一个庞大的数据集我们经常需要拿出来一小部分作为样例，然后在样例上验证我们的查询，优化我们的程序,利用分桶表可以实现数据的抽样。

		注意：
			hive默认关闭分桶功能，需要手动开启set hive.enforce.bucketing=true; 
			分桶的操作需要在底层mr执行的过程中起作用，所以通常不会在原始的数据表上加分桶，而是专门创建一个测试表，将原始表中的数据导入到测试表，再导入过程触发的mr中实现分桶。

		开启分桶功能，强制多个 reduce 进行输出：
			set hive.enforce.bucketing=true; 
		准备主表：
			create table teacher(id int,name string) row format delimited fields terminated by '|';
		向主表加载数据:
			load data local inpath '/root/work/teachers.txt' into table teacher;
		创建带桶的 table ：
			create table teacher_temp(id int,name string) clustered by (id) into 2 buckets  row format delimited fields terminated by '|';
		从主表向分桶表导入数据：
			insert overwrite/into table teacher_temp select * from tacher;

		分桶表其实就是将表中的数据按照hash分桶的方式分桶存放，而所谓的桶就是表文件夹下不同的文件

		在分桶表中基于部分数据做测试:
			select * from teacher_temp tablesample(bucket 1 out of 2 on id);
			//桶的个数从1开始计数，前面的查询从2个桶中的第一个桶获取数据。其实就是二分之一。
		从分桶表中获取1/4的数据:
			select * from bucketed_user tablesample(bucket 1 out of 4 on id);
			//tablesample函数是一种逻辑抽样过程 即使物理分桶时 并不是4个桶一样可以得到数据 但是最好在抽样时保证抽样的总数 和 物理分桶的总数相同 这样可以保证最好的效率
			
# 六、HIVE语法
## 0.数据类型
		TINYINT - byte
		SMALLINT - short
		INT	- int
		BIGINT - long
		BOOLEAN - boolean
		FLOAT - float
		DOUBLE - double
		STRING - String
		TIMESTAMP - TimeStamp
		BINARY - byte[]
		
## 1.create table
		CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXIST 选项来忽略这个异常。
		EXTERNAL 关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION），Hive 创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。
		LIKE 允许用户复制现有的表结构，但是不复制数据。	
		有分区的表可以在创建的时候使用 PARTITIONED BY 语句。一个表可以拥有一个或者多个分区，每一个分区单独存在一个目录下。
		CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
		  [(col_name data_type [COMMENT col_comment], ...)]
		  [COMMENT table_comment]
		  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
		  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
		  [
		   [ROW FORMAT row_format] [STORED AS file_format]
		   | STORED BY 'storage.handler.class.name' [ WITH SERDEPROPERTIES (...) ]  (Note:  only available starting with 0.6.0)
		  ]
		  [LOCATION hdfs_path]
		  [TBLPROPERTIES (property_name=property_value, ...)]  (Note:  only available starting with 0.6.0)
		  [AS select_statement]  (Note: this feature is only available starting with 0.5.0.)

		CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
		  LIKE existing_table_name
		  [LOCATION hdfs_path]

		data_type
		  : primitive_type
		  | array_type
		  | map_type
		  | struct_type

		primitive_type
		  : TINYINT
		  | SMALLINT
		  | INT
		  | BIGINT
		  | BOOLEAN
		  | FLOAT
		  | DOUBLE
		  | STRING

		array_type
		  : ARRAY < data_type >

		map_type
		  : MAP < primitive_type, data_type >

		struct_type
		  : STRUCT < col_name : data_type [COMMENT col_comment], ...>

		row_format
		  : DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char]
		        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
		  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]

		file_format:
		  : SEQUENCEFILE
		  | TEXTFILE
		  | RCFILE     (Note:  only available starting with 0.6.0)
		  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname

		练习：
			创建一张内部表
				create table xx (id int,name string) row format DELIMITED FIELDS TERMINATED BY '\t';
			创建一张外部表
				create external table xx (id int,name string) row format DELIMITED FIELDS TERMINATED BY '\t';
			创建一张带有分区的外部表
				create external table xx (id int,name string) row format DELIMITED FIELDS TERMINATED BY '\t' partitioned by 'ccc';
				
## 2.Alter Table
		(1)Add Partitions
			ALTER TABLE table_name ADD [IF NOT EXISTS] partition_spec [ LOCATION 'location1' ] partition_spec [ LOCATION 'location2' ] ...
			partition_spec:
			  PARTITION (partition_col = partition_col_value, partition_col = partiton_col_value, ...)
			练习：修改表增加分区
		(2)Drop Partitions
			ALTER TABLE table_name DROP partition_spec, partition_spec,...

		(3)Rename Table
			ALTER TABLE table_name RENAME TO new_table_name

		(4)Change Column
			ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]
			这个命令可以允许改变列名、数据类型、注释、列位置或者它们的任意组合

		(5)Add/Replace Columns
			ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...)

			ADD是代表新增一字段，字段位置在所有列后面(partition列前);REPLACE则是表示替换表中所有字段。

## 3.Show
		查看库表
		SHOW DATABASES;
		SHOW TABLES;

		查看表名，部分匹配
		SHOW TABLES 'page.*';
		SHOW TABLES '.*view';

		查看某表的所有Partition，如果没有就报错：
		SHOW PARTITIONS page_view;

		查看某表结构：
		DESCRIBE invites;
		DESC invites;
		
		查看分区内容
		SELECT a.foo FROM invites a WHERE a.ds='2008-08-15';

		查看有限行内容，同Greenplum，用limit关键词
		SELECT a.foo FROM invites a limit 3;

		查看表分区定义
		DESCRIBE EXTENDED page_view PARTITION (ds='2008-08-08');

## 4.Load
		LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
		Load 操作只是单纯的复制/移动操作，将数据文件移动到 Hive 表对应的位置。
		

## 5.Insert
		(1)向hive表中插入一个查询的结果
			INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement
		(2)将查询结果写出到外部文件系统中
			INSERT OVERWRITE [LOCAL] DIRECTORY directory1 SELECT ... FROM ...

## 6.Drop
		删除一个内部表的同时会同时删除表的元数据和数据。
		删除一个外部表，只删除元数据而保留数据。

## 7.Limit 
		Limit 可以限制查询的记录数。

## 8.Select
		SELECT [ALL | DISTINCT] select_expr, select_expr, ...
		FROM table_reference
		[WHERE where_condition] 
		[GROUP BY col_list]
		[   CLUSTER BY col_list
		  | [DISTRIBUTE BY col_list] [SORT BY col_list]
		]
		[LIMIT number]

## 9.JOIN
		join_table:
		    table_reference JOIN table_factor [join_condition]
		  | table_reference {LEFT|RIGHT|FULL} [OUTER] JOIN table_reference join_condition
		  | table_reference LEFT SEMI JOIN table_reference join_condition

		table_reference:
		    table_factor
		  | join_table

		table_factor:
		    tbl_name [alias]
		  | table_subquery alias
		  | ( table_references )

		join_condition:
		    ON equality_expression ( AND equality_expression )*

		equality_expression:
		    expression = expression

# 七、HIVE内置函数
	HIVE除了提供了类似mysql的sql的语法外 还提供了大量内置的函数 方便开发者来调用 编写功能丰富的处理程序。
	
# 八、HIVE的UDF
	新建java工程，导入hive相关包。
	创建类继承UDF
	自己编写一个evaluate方法，返回值和参数任意。
	为了能让mapreduce处理，String要用Text处理。
	将写好的类打成jar包，上传到linux中
	在hive命令行下，向hive注册UDF：add jar /xxxx/xxxx.jar
	为当前udf起一个名字：create temporary function fname as '类的全路径名';
	之后就可以在hql中使用该自定义函数了。
	
# 九、hive的jdbc编程
	hive实现了jdbc接口，所以可以非常方便用jdbc技术通过java代码操作。

	1.在服务器端开启HiveServer2服务
		./hive --service hiveserver2

	2.创建本地工程，导入jar包
		导入hive\lib目录下的hive-jdbc-1.2.0-standalone.jar
		导入hadoop-2.7.1\share\hadoop\common下的hadoop-common-2.7.1.jar

	3.编写jdbc代码执行
		Class.forName("org.apache.hive.jdbc.HiveDriver");
		Connection conn = DriverManager.getConnection("jdbc:hive2://192.168.242.101:10000/park","root","root");
		Statement statement = conn.createStatement();
		//statement.executeUpdate("create table x2xx (id int,name string)");
		//其他sql语句...
		statement.close();
		conn.close();
	
# 十、案例
## 1.flume收集日志
		(1)客户端Agent

		```properties	#声明Agent
			a1.sources = r1
			a1.sinks = k1 k2
			a1.channels = c1

			#声明Source
			a1.sources.r1.type = spooldir
			a1.sources.r1.spoolDir = /root/work/data/zdata
			a1.sources.r1.interceptors = i1
			a1.sources.r1.interceptors.i1.type = timestamp

			#声明Sink
			a1.sinks.k1.type = avro
			a1.sinks.k1.hostname = hadoop01
			a1.sinks.k1.port = 44444

			a1.sinks.k2.type = avro
			a1.sinks.k2.hostname = hadoop02
			a1.sinks.k2.port = 44444

			a1.sinkgroups = g1
			a1.sinkgroups.g1.sinks = k1 k2
			a1.sinkgroups.g1.processor.type = load_balance
			a1.sinkgroups.g1.processor.backoff = true
			a1.sinkgroups.g1.processor.selector = random
			
			#声明channel
			a1.channels.c1.type = memory
			a1.channels.c1.capacity = 1000
			a1.channels.c1.transactionCapacity = 100

			#绑定关系
			a1.sources.r1.channels = c1
			a1.sinks.k1.channel = c1
			a1.sinks.k2.channel = c1
```

		(2)中心服务器Agent
			```properties
      #声明Agent
			a1.sources = r1
			a1.sinks = k1
			a1.channels = c1

			#声明Source
			a1.sources.r1.type = avro
			a1.sources.r1.bind = 0.0.0.0
			a1.sources.r1.port = 44444

			#声明Sink
			a1.sinks.k1.type = hdfs
			a1.sinks.k1.hdfs.path = hdfs://hadoop01:9000/zebra/reportTime=%Y-%m-%d
			a1.sinks.k1.hdfs.timeZone = GMT+8
			a1.sinks.k1.hdfs.fileType = DataStream
			a1.sinks.k1.hdfs.rollInterval = 30
			a1.sinks.k1.hdfs.rollSize = 0
			a1.sinks.k1.hdfs.rollCount = 0

			#声明channel
			a1.channels.c1.type = memory
			a1.channels.c1.capacity = 1000
			a1.channels.c1.transactionCapacity = 100

			#绑定关系
			a1.sources.r1.channels = c1
			a1.sinks.k1.channel = c1
```

## 2.数据清洗
		数据的来源可能很复杂
		数据收集到后可能存在如下的问题：

			数据格式不统一
				-- 把格式统一起来	
			数据存在字段缺失
				--丢弃数据 补齐数据 用固定值替代 将缺失字段的数据单独拿出来处理
			数据字段存在错误
				--丢弃数据 补齐数据 用固定值替代 将缺失字段的数据单独拿出来处理
			数据可能需要进行预处理

		在进行数据清洗时，采用何种技术并没有严格的限制，哪种技术最合适就选择哪种技术。

		zebra中的数据清洗:
			77个字段 --> 21个字段
			如果 CellId 为空 则将CellId补齐为 000000000
			
					
		#建立外部分区表 管理数据
    ```mysql
		create database zebradb;
		use zebradb;
		create EXTERNAL table zebra (a1 string,a2 string,a3 string,a4 string,a5 string,a6 string,a7 string,a8 string,a9 string,a10 string,a11 string,a12 string,a13 string,a14 string,a15 string,a16 string,a17 string,a18 string,a19 string,a20 string,a21 string,a22 string,a23 string,a24 string,a25 string,a26 string,a27 string,a28 string,a29 string,a30 string,a31 string,a32 string,a33 string,a34 string,a35 string,a36 string,a37 string,a38 string,a39 string,a40 string,a41 string,a42 string,a43 string,a44 string,a45 string,a46 string,a47 string,a48 string,a49 string,a50 string,a51 string,a52 string,a53 string,a54 string,a55 string,a56 string,a57 string,a58 string,a59 string,a60 string,a61 string,a62 string,a63 string,a64 string,a65 string,a66 string,a67 string,a68 string,a69 string,a70 string,a71 string,a72 string,a73 string,a74 string,a75 string,a76 string,a77 string) partitioned by (reportTime string) row format delimited fields terminated by '|' stored as textfile location '/zebra';
		ALTER TABLE zebra add  PARTITION (reportTime='2017-09-20') location '/zebra/reportTime=2017-09-20';
    ```

		#清洗数据
    ```mysql
		create table dataclear (
				reporttime string,
				appType bigint,
				appSubtype bigint,
				userIp string,
				userPort bigint,
				appServerIP string,
				appServerPort bigint,
				host string,
				cellid string,
				
				appTypeCode bigint ,
				interruptType String,
				transStatus bigint,
				trafficUL bigint,
				trafficDL bigint,
				retranUL bigint,
				retranDL bigint,
				procdureStartTime bigint,
				procdureEndTime bigint
			) row format delimited fields terminated by '|';
			--create table dataclear (reporttime string, appType bigint, appSubtype bigint, userIp string, userPort bigint, appServerIP string, appServerPort bigint, host string, cellid string,  appTypeCode bigint , interruptType String, transStatus bigint, trafficUL bigint, trafficDL bigint, retranUL bigint, retranDL bigint, procdureStartTime bigint, procdureEndTime bigint) row format delimited fields terminated by '|';

			insert into table dataclear 
				select 
					reportTime,a23,a24,a27,a29,a31,a33,a59,if(a17 == "","000000000",a17),
					a19,a68,a55,a34,a35,a40,a41,a20,a21 
				from datasource;
			--insert into table dataclear select reportTime,a23,a24,a27,a29,a31,a33,a59,if(a17 == "","000000000",a17),a19,a68,a55,a34,a35,a40,a41,a20,a21 from zebra;	
```

## 3.业务处理 将数据清洗过后 包好原始数据的dataclear表经过规则额计算转换为业务基础数据表F_HTTP_APP_HOST
		```mysql
    create table F_HTTP_APP_HOST (
			reporttime string,
			appType bigint,
			appSubtype bigint,
			userIp string,
			userPort bigint,
			appServerIP string,
			appServerPort bigint,
			host string,
			cellid string,

			attempts bigint,
			accepts bigint,
			trafficUL bigint,
			trafficDL bigint,
			retranUL bigint,
			retranDL bigint,
			failCount bigint,
			transDelay bigint
		)row format delimited fields terminated by '|';

		--create table F_HTTP_APP_HOST (reporttime string,appType bigint,appSubtype bigint,userIp string,userPort bigint,appServerIP string,appServerPort bigint,host string,cellid string,attempts bigint,accepts bigint,trafficUL bigint,trafficDL bigint,retranUL bigint,retranDL bigint,failCount bigint,transDelay bigint)row format delimited fields terminated by '|';

		insert into F_HTTP_APP_HOST 
			select 
				reporttime,
				appType,
				appSubtype,
				userIp,
				userPort,
				appServerIP,
				appServerPort,
				host,
				cellid,

				#HTTP_ATTEMPT	attempts	if( App Type Code=103 ) then counter++
				if(appTypeCode == 103,1,0),
				#if( App Type Code=103 & HTTP/WAP事物状态 in(10,11,12,13,14,15,32,33,34,35,36,37,38,48,49,50,51,52,53,54,55,199,200,201,202,203,204,205,206,302,304,306) && Wtp中断类型==NULL) then counter++
				if(appTypeCode == 103 and find_in_set(transStatus,'10,11,12,13,14,15,32,33,34,35,36,37,38,48,49,50,51,52,53,54,55,199,200,201,202,203,204,205,206,302,304,306') != 0 and interruptType == 0,1,0),
				#trafficUL	if( App Type Code=103  ) then counter = counter + UL Data
				#trafficDL	if( App Type Code=103  ) then counter = counter + DL Data
				#retranUL	if( App Type Code=103  ) then counter = counter + 上行TCP重传报文数			
				#retranDL	if( App Type Code=103  ) then counter = counter + 下行TCP重传报文数
				if(appTypeCode == 103,trafficUL,0),
				if(appTypeCode == 103,trafficDL,0),
				if(appTypeCode == 103,retranUL,0),
				if(appTypeCode == 103,retranDL,0),
				#failCount	if( App Type Code=103 &&  HTTP/WAP事务状态==1  &&  Wtp中断类型==NULL ) then counter = counter + 1
				if(appTypeCode == 103 and transStatus == 1 and interruptType == 0,1,0),
				#transDelay	if( App Type Code=103  ) then counter = counter + (Procedure_End_time-Procedure_Start_time),
				if(appTypeCode == 103,procdureEndTime-procdureStartTime,0)
			from
				dataclear
			where 
				reportTime = '2017-09-20';

		--insert into F_HTTP_APP_HOST  select  reporttime, appType, appSubtype, userIp, userPort, appServerIP, appServerPort, host, cellid,  if(appTypeCode == 103,1,0), if(appTypeCode == 103 and find_in_set(transStatus,'10,11,12,13,14,15,32,33,34,35,36,37,38,48,49,50,51,52,53,54,55,199,200,201,202,203,204,205,206,302,304,306') != 0 and interruptType == 0,1,0), if(appTypeCode == 103,trafficUL,0), if(appTypeCode == 103,trafficDL,0), if(appTypeCode == 103,retranUL,0), if(appTypeCode == 103,retranDL,0), if(appTypeCode == 103 and transStatus == 1 and interruptType == 0,1,0), if(appTypeCode == 103,procdureEndTime-procdureStartTime,0) from dataclear where  reportTime = '2017-09-20';
```

## 4.将业务基础数据表 进一步处理 转换为 不同维度的结果表

		F_HTTP_APP_HOST
				-->基于应用的统计结果表D_H_HTTP_APPTYPE 
				-->基于网站的统计结果表D_H_HTTP_HOST 
				-->基于小区的统计结果表D_H_HTTP_CELLID 
				-->基于小区和网站的统计结果表D_H_HTTP_CELLID_HOST 

		这样未来查询时不需要查询完整的基础数据表 直接查询结果表就可以获得相应维度的数据 效率非常高

		以应用统计结果表为例:

			(1)创建应用统计结果表
      ```mysql
				create table D_H_HTTP_APPTYPE(
					hourid	string,
					appType bigint,
					appSubtype bigint,
					attempts bigint,
					accepts bigint,
					succRatio double,
					trafficUL bigint,
					trafficDL bigint,
					totalTraffic bigint,
					retranUL bigint,
					retranDL bigint,
					retranTraffic bigint,
					failCount bigint,
					transDelay bigint
				)row format delimited fields terminated by '|';

				--create table D_H_HTTP_APPTYPE(hourid string,appType bigint,appSubtype bigint,attempts bigint,accepts bigint,succRatio double,trafficUL bigint,trafficDL bigint,totalTraffic bigint,retranUL bigint,retranDL bigint,retranTraffic bigint,failCount bigint,transDelay bigint) row format delimited fields terminated by '|';
			```
      
			(2)从基础数据表F_HTTP_APP_HOST预处理数据转换为D_H_HTTP_APPTYPE
      ```mysql
				insert into D_H_HTTP_APPTYPE
					select
						reportTime,
						appType,
						appSubtype,
						sum(attempts),
						sum(accepts),
						round(sum(accepts) / sum(attempts),4),
						sum(trafficUL),
						sum(trafficDL),
						sum(trafficUL) + sum(trafficDL),
						sum(retranUL),
						sum(retranDL),
						sum(retranUL) + sum(retranDL),
						sum(failCount),
						sum(transDelay)
					from 
						F_HTTP_APP_HOST
					where
						reportTime = '2017-09-20'
					group by
						reportTime,
						appType,
						appSubtype

				--insert into D_H_HTTP_APPTYPE select reportTime, appType, appSubtype, sum(attempts), sum(accepts), round(sum(accepts) / sum(attempts),4), sum(trafficUL), sum(trafficDL), sum(trafficUL) + sum(trafficDL), sum(retranUL), sum(retranDL), sum(retranUL) + sum(retranDL), sum(failCount), sum(transDelay) from  F_HTTP_APP_HOST where reportTime = '2017-09-20' group by reportTime, appType, appSubtype
```

## 5.将结果表数据导出到mysql中
		(1)在mysql中创建表
			在mysql中准备库和表：
      ```mysql
			create database zebradb;
			use zebradb;
			create table D_H_HTTP_APPTYPE(hourid varchar(20),apptype bigint,appsubtype bigint,attempts bigint,accepts bigint,succratio double,trafficul bigint,trafficdl bigint,totaltraffic bigint,retranul bigint,retrandl bigint,retrantraffic bigint,failcount bigint,transdelay bigint);
```

		(2)将hive表中的数据通过sqoop导出到mysql中
				```shell
        ./sqoop export --connect jdbc:mysql://hadoop01:3306/zebradb --username root --password root --export-dir '/user/hive/warehouse/zebradb.db/d_h_http_apptype' --table D_H_HTTP_APPTYPE -m 1 --fields-terminated-by '|'
        ```

				====================================================================	
					sqoop 沟通hdfs和关系型数据库的桥梁，可以从hdfs导出数据到关系型数据库，也可以从关系型数据库导入数据到hdfs
					下载：
						Apache 提供的工具
					安装：
						要求必须有jdk 和 hadoop的支持，并且有版本要求。
						上传到linux中，进行解压
						sqoop可以通过JAVA_HOME找到jdk 可以通过HADOOP_HOME找到hadoop所以不需要做任何配置就可以工作。
						需要将要连接的数据库的驱动包加入sqoop的lib目录下                               

					从关系型数据库导入数据到hdfs:
						sqoop import --connect jdbc:mysql://192.168.1.10:3306/tedu --username root --password 123  --table trade_detail --columns 'id, account, income, expenses'
						
						指定输出路径、指定数据分隔符
						sqoop import --connect jdbc:mysql://192.168.1.10:3306/tedu --username root --password 123  --table trade_detail --target-dir '/sqoop/td' --fields-terminated-by '\t'
						
						指定Map数量 -m 
						sqoop import --connect jdbc:mysql://192.168.1.10:3306/tedu --username root --password 123  --table trade_detail --target-dir '/sqoop/td1' --fields-terminated-by '\t' -m 2

						增加where条件, 注意：条件必须用引号引起来
						sqoop import --connect jdbc:mysql://192.168.1.10:3306/tedu --username root --password 123  --table trade_detail --where 'id>3' --target-dir '/sqoop/td2' 

						增加query语句(使用 \ 将语句换行)
						sqoop import --connect jdbc:mysql://192.168.1.10:3306/tedu --username root --password 123 --query 'SELECT * FROM trade_detail where id > 2 AND $CONDITIONS' --split-by trade_detail.id --target-dir '/sqoop/td3'

						注意：如果使用--query这个命令的时候，需要注意的是where后面的参数，AND $CONDITIONS这个参数必须加上
						而且存在单引号与双引号的区别，如果--query后面使用的是双引号，那么需要在$CONDITIONS前加上\即\$CONDITIONS
						如果设置map数量为1个时即-m 1，不用加上--split-by ${tablename.column}，否则需要加上
						
					从hdfs导出数据到关系型数据库:
						sqoop export --connect jdbc:mysql://192.168.8.120:3306/tedu --username root --password 123 --export-dir '/td3' --table td_bak -m 1 --fields-terminated-by ','

					从关系型数据库导入数据到hive：
						sqoop import --connect jdbc:mysql://192.168.1.10:3306/tedu --username root --password 123  --table trade_detail --hive-import-hive-table hivetab1 -m 1

					从hive导出数据到关系型数据库
						就是将hive在hdfs文件夹下的文件导出到mysql中，参考从hdfs导出数据到关系型数据库的过程。
				====================================================================		

## 6.可视化展示
		(1)通过sql查询指定日期应用大类前10名
    ```mysql
			select
				hourid,
				apptype,
				sum(totaltraffic) as tt
			from
				D_H_HTTP_APPTYPE
			where
				hourid = '2017-09-20'
			group by
				hourid,apptype
			order by 
				tt desc
			limit 10;
      ```

		(2)通过查询指定日期指定应用大类中应用小类的前10名
    ```mysql
			select
				hourid,
				appType,
				appSubtype,
				sum(totaltraffic) as tt
			from
				D_H_HTTP_APPTYPE
			where
				hourid = '2017-09-20'
				and
				apptype = 1
			group by
				appSubtype
			order by
				tt desc
			limit 10;
      ```
				
