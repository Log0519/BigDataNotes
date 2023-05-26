# Spark、Flink主要需求
@[toc]
## 一、Spark

spark连接hive，hive表存在hdfs，提交任务到spark-yarn执行

### 1、spark连接hive

需要加入hive-site.xml

```scala
 val conf: SparkConf = new SparkConf()
    val sparkSession: SparkSession = SparkSession.builder()
      .appName("SparkSQLReadHive")
      .config(conf)
      //开启对Hive的支持，支持连接Hive的MetaStore、序列化、自定义函数
      .enableHiveSupport()
      .getOrCreate()

    sparkSession.sql("select * from dept").show()
    sparkSession.close()
```

### 2、spark存到hive

```scala

```

### 3、提交jar包到spark-yarn

(1) 启动hadoop集群，在spark-yarn执行./sbin/start-all.sh和

./sbin.start-history.sh

(2)提交：

```shell
bin/spark-submit \
--master yarn \
--class com.log.spark.core.wc.spark02_WordCount_spark\
--deploy-mode client \
--executor-memory 1g \
--total-executor-cores 4 \
--driver-memory 1g \
--numexecutors 3 \
--executor-cores 2 \
--executor-memory 6g \
--class
myWorkCommit/sparkcore-1.0-SNAPSHOT.jar
```

### 4、SparkSQL初始化

```scala
  val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
    val spark = new SparkSession.Builder().config(sparkConf).getOrCreate()
    //DataFrame
    val df: DataFrame = spark.read.json("datas/user.json")
    df.show()

    //DataFrame=>SQL
    //View可查不可改，table可查可改
    df.createOrReplaceTempView(("user"))
```

## 二、Flink

(1) flink作为消费者连接kafka，先处理格式，再用flinksql实现需求.

(2) 如果只需要ETL，就再作为生产者连接kafka.
(3) 通过提交jar包到yarn的per-job模式下运行.

### 1、作为消费者连接kafka

```scala
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    //构建ETL管道
    //从Kafka读取数据
    val properties = new Properties()
    properties.setProperty("bootstrap.servers","hadoop102:9092")
    properties.setProperty("group.id","consumer-group")
    val stream: DataStream[String]
 = env.addSource(new FlinkKafkaConsumer[String]("clicks"
, new SimpleStringSchema(), properties))

    stream.map(data=>{
          val fields=data.split(",")
      Event(fields(0).trim,fields(1).trim,fields(2).trim.toLong)
      })
      .map(_.toString)
```

### 2、作为生产者连接kafka

```scala
   //数据写入Kafka
    //noinspection DuplicatedCode
    stream.addSink(new FlinkKafkaProducer[String]("hadoop102:9092",
   "events",new SimpleStringSchema()))
    env.execute()
```

### 3、per-job单作业模式提交jar包

(1) 启动hadoop集群

在 hadoop102 节点服务器上执行 start-cluster.sh 启动 Flink 集群

如果是用yarn-per-job提交模式，不需要启动flink集群，只需要启动hadoop集群，那么应该在hadoop的application页面查看flink的web页面

(2) 提交：

```shell
bin/flink run \
-t yarn-per-job \
-d \
-c com.log.sink.KafkaToFlinkAndSinkToKafka(类路径) \
-p 2(并行度) \
./FlinkTutorial-1.0-SNAPSHOT.jar(任务jar包) \
-Drest.flamegraph.enabled=true(开启火焰图) \
-Dyarn.application.queue=test \
-Djobmanager.memory.process.size=1024mb \
-Dtaskmanager.memory.process.size=2048mb \
-Dtaskmanager.numberOfTaskSlots=2 \
(开启性能监控)
-Dstate.backend.latency-track.keyed-state-enabled=true \
```

```shell
bin/flink run \
-t yarn-per-job \
-d \
-c com.log.sink.KafkaToFlinkAndSinkToKafka \
-p 2 \
./myword/test2022-11-17/FlinkTutorial-1.0-SNAPSHOT.jar\
-Drest.flamegraph.enabled=true \
-Dyarn.application.queue=test \
-Djobmanager.memory.process.size=1024mb \
-Dtaskmanager.memory.process.size=2048mb \
-Dtaskmanager.numberOfTaskSlots=2 \
-Dstate.backend.latency-track.keyed-state-enabled=true \
```

### 4、CheckPoint和Status设置

```scala
//Configuration conf = new Configuration();
//conf.set(RestOptions.ENABLE_FLAMEGRAPH, true);开启火焰图
//StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(conf);
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
EmbeddedRocksDBStateBackend embeddedRocksDBStateBackend = new EmbeddedRocksDBStateBackend();
//下面一句的意思是开启基于磁盘和内存的优化
//embeddedRocksDBStateBackend.setPredefinedOptions(PredefinedOptions.SPINNING_DISK_OPTIMIZED_HIGH_MEM);
env.setStateBackend(embeddedRocksDBStateBackend);
// env.setStateBackend(new HashMapStateBackend());
// env.getConfig().enableObjectReuse();
env.enableCheckpointing(TimeUnit.SECONDS.toMillis(3), CheckpointingMode.EXACTLY_ONCE);

CheckpointConfig checkpointConfig = env.getCheckpointConfig();
checkpointConfig.setCheckpointStorage("hdfs://hadoop1:8020/flink-tuning/ck");
checkpointConfig.setMinPauseBetweenCheckpoints(TimeUnit.SECONDS.toMillis(3));
checkpointConfig.setTolerableCheckpointFailureNumber(5);
checkpointConfig.setCheckpointTimeout(TimeUnit.MINUTES.toMillis(1));
checkpointConfig.enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
```

## 三、JDBCUtil

```scala
package com.log.spark.util

import com.alibaba.druid.pool.DruidDataSourceFactory

import java.sql.{Connection, PreparedStatement}
import java.util.Properties
import javax.sql.DataSource

object JDBCUtil {
  //初始化连接池
  var dataSource: DataSource = init()
  //初始化连接池方法
  def init(): DataSource = {
    val properties = new Properties()
    properties.setProperty("driverClassName", "com.mysql.jdbc.Driver")
    properties.setProperty("url", "jdbc:mysql://hadoop102:3306/spark_streaming?useUnicode=true&characterEncoding=UTF-8")
    properties.setProperty("username", "root")
    properties.setProperty("password", "fuxiaoluo")
    properties.setProperty("maxActive","50")
    DruidDataSourceFactory.createDataSource(properties)
  }
  //获取 MySQL 连接
  def getConnection: Connection = {
    dataSource.getConnection
  }
  //执行 SQL 语句,单条数据插入
  def executeUpdate(connection: Connection, sql: String, params: Array[Any]): Int
  = {
    var rtn = 0
    var pstmt: PreparedStatement = null
    try {
      connection.setAutoCommit(false)
      pstmt = connection.prepareStatement(sql)
      if (params != null && params.length > 0) {
        for (i <- params.indices) {
          pstmt.setObject(i + 1, params(i))
        }
      }
      rtn = pstmt.executeUpdate()
      connection.commit()
      pstmt.close()
    } catch {
      case e: Exception => e.printStackTrace()
    }
    rtn
  }
  //判断一条数据是否存在
  def isExist(connection: Connection, sql: String, params: Array[Any]): Boolean =
  {
    var flag: Boolean = false
    var pstmt: PreparedStatement = null
    try {
      pstmt = connection.prepareStatement(sql)
      for (i <- params.indices) {
        pstmt.setObject(i + 1, params(i))
      }
      flag = pstmt.executeQuery().next()
      pstmt.close()
    } catch {
      case e: Exception => e.printStackTrace()
    }
    flag
  }

}
```

## 四、log4j.properties

```properties
log4j.rootCategory=ERROR, console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/ddHH:mm:ss} %p %c{1}: %m%n
# Set the default spark-shell log level to ERROR. When running the spark-shell,the
# log level for this class is used to overwrite the root logger's log level, so that
# the user can have different defaults for the shell and regular Spark apps.
log4j.logger.org.apache.spark.repl.Main=ERROR
# Settings to quiet third party logs that are too verbose
log4j.logger.org.spark_project.jetty=ERROR
log4j.logger.org.spark_project.jetty.util.component.AbstractLifeCycle=ERROR
log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=ERROR
log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=ERROR
log4j.logger.org.apache.parquet=ERROR
log4j.logger.parquet=ERROR
# SPARK-9183: Settings to avoid annoying messages when looking up nonexistentUDFs in SparkSQL with Hive support
log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL
log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR
```

## 五、hive-site.xml

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- jdbc 连接的 URL -->
 <property>
 <name>javax.jdo.option.ConnectionURL</name>
 <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
</property>
 <!-- jdbc 连接的 Driver-->
 <property>
 <name>javax.jdo.option.ConnectionDriverName</name>
 <value>com.mysql.jdbc.Driver</value>
</property>
<!-- jdbc 连接的 username-->
 <property>
 <name>javax.jdo.option.ConnectionUserName</name>
 <value>root</value>
 </property>
 <!-- jdbc 连接的 password -->
 <property>
 <name>javax.jdo.option.ConnectionPassword</name>
 <value>fuxiaoluo</value>
</property>
 <!-- Hive 元数据存储版本的验证 -->
 <property>
 <name>hive.metastore.schema.verification</name>
 <value>false</value>
</property>
 <!--元数据存储授权-->
 <property>
 <name>hive.metastore.event.db.notification.api.auth</name>
 <value>false</value>
 </property>
 <!-- Hive 默认在 HDFS 的工作目录 -->
 <property>
 <name>hive.metastore.warehouse.dir</name>
 <value>/user/hive/warehouse</value>
 </property>
<!--当前库 和 表头-->
<property>
 <name>hive.cli.print.header</name>
 <value>true</value>
 </property>
 <property>
 <name>hive.cli.print.current.db</name>
 <value>true</value>
 </property>
<!-- 指定存储元数据要连接的地址 -->
<!-- <property>
 <name>hive.metastore.uris</name>
 <value>thrift://hadoop102:9083</value>
 </property>
-->
<!-- 指定 hiveserver2 连接的 host -->
<property>
 <name>hive.server2.thrift.bind.host</name>
 <value>hadoop102</value>
 </property>
 <!-- 指定 hiveserver2 连接的端口号 -->
<property>
 <name>hive.server2.thrift.port</name>
<value>10000</value>
</property>
</configuration>
```
