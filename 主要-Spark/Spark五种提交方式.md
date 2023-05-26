## Spark五种提交方式
@[toc]
### 1.Spark相关端口号

1.Spark 查看当前 Spark-shell 运行任务情况端口号：`4040`（计算） 

2.Spark Master 内部通信服务端口号：`7077`

3.Standalone 模式下，Spark Master Web 端口号：`8080`（资源）

4.Spark 历史服务器端口号：`18080`

5.Hadoop YARN 任务运行情况查看端口号：`8088`

### 2.本地模式

**提交方式**：

    bin/spark-submit \
    
    --class org.apache.spark.examples.SparkPi \
    
    --master local[2] \
    
    ./examples/jars/spark-examples_2.12-3.0.0.jar \
    
    10

### 3.standalone模式

 第一步：`sbin/start-all.sh`启动所有集群

第二步：`sbin/start-history-server.sh`启动历史服务，首先要启动hdfs

 **提交方式：**

    bin/spark-submit \
    
    --class org.apache.spark.examples.SparkPi \
    
    --master spark://hadoop102:7077 \
    
    ./examples/jars/spark-examples_2.12-3.0.0.jar \10

### 4.高可用

正常启动spark，然后在hadoop103上启动一个master，进入备用网页（hadoop102->hadoop103），可以看到standby状态的master

**提交方式：**

    bin/spark-submit \
    --class org.apache.spark.examples.SparkPi \
    --master spark://hadoop102:7077,hadoop103:7077 \
    ./examples/jars/spark-examples_2.12-3.0.0.jar \10

在高可用模式下当hadoop102挂掉以后，hadoop103的Master会变为活动状态

### 5.yarn模式，要在hadoop103(yarn所在节点)上提交任务

 1.spark历史服务器要在hadoop102上启动
 2.启动HDFS 以及 YARN 集群

**第一种提交方式：**

    bin/spark-submit \
    --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    ./examples/jars/spark-examples_2.12-3.0.0.jar \10

**第二种提交方式**

    bin/spark-submit \
    --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode client \
    ./examples/jars/spark-examples_2.12-3.0.0.jar \
    10

### 6.在windows环境下

1.Windows环境下，配置好jdk，scala，hadoop（以及hadoop扩展包，放在hadoop的bin目录下）的环境变量

2.启动bin目录下的`spark-shell.cmd`

3.在bin目录下cmd，可以执行提交任务脚本

    spark-submit --class org.apache.spark.examples.SparkPi --master 
    local[2] ../examples/jars/spark-examples_2.12-3.0.0.jar 10
