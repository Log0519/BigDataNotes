@[toc]
➢ Spark 查看当前 Spark-shell 运行任务情况端口号：4040（计算） 
➢ Spark Master 内部通信服务端口号：7077
➢ Standalone 模式下，Spark Master Web 端口号：8080（资源）
➢ Spark 历史服务器端口号：18080
➢ Hadoop YARN 任务运行情况查看端口号：8088

## 本地模式
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10

## standalone模式
sbin/start-all.sh启动所有集群
sbin/start-history-server.sh启动历史服务，首先要启动hdfs
提交方式：
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077 \
./examples/jars/spark-examples_2.12-3.0.0.jar \10

## 高可用
正常启动spark，然后在hadoop103上启动一个master，进入备用网页（hadoop102->hadoop103），可以看到standby状态的master
提交应用
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077,hadoop103:7077 \
./examples/jars/spark-examples_2.12-3.0.0.jar \10
当hadoop102挂掉以后，hadoop103的Master会变为活动状态

## yarn模式，要在hadoop103上提交任务
spark历史服务器要在hadoop102上启动
启动HDFS 以及 YARN 集群
第一种提交方式
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode cluster \
./examples/jars/spark-examples_2.12-3.0.0.jar \10
第二种提交方式
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10


Windows环境下，配置好jdk，scala，hadoop（以及hadoop扩展包，放在hadoop的bin目录下）的环境变量
启动bin目录下的spark-shell.cmd
在bin目录下cmd，可以执行提交任务脚本
spark-submit --class org.apache.spark.examples.SparkPi --master local[2] ../examples/jars/spark-examples_2.12-3.0.0.jar 10


## 我的提交示例：
[log@hadoop102 spark-standalone]$ bin/spark-submit \
> --master spark://hadoop102:7077\
> --class com.log.spark.core.wc.spark02_WordCount_spark\
> --executor-memory 1g \
> --total-executor-cores 4 \
> myWorkCommit/sparkcore-1.0-SNAPSHOT.jar \
> hdfs://hadoop102:9870/input/word.txt \
> hdfs://hadoop102:9870/output
