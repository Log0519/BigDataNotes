# flink调优总结：
@[toc]
### 检查点：

source插入检查点分界线，如果有多个并行下游会被广播同时传给下游，如果有多个并行上游source传来不同的检查点分界线，需要进行分界线对齐操作，如果一个分区barrier先到来，另外一个分区继续传来数据，就不能处理。flink1.11后提供了不对齐分界线的检查点保存方式，将未处理的缓冲数据也保存进去，就不需要等待对齐可以直接保存了，保存更多信息，提升性能。

### 精准一次：

精确一次有个2PC，两阶段提交，多了一个预提交kafka，FlinkKafkaProducer已经帮我们实现了

### 调优

#### 资源方面

1、slot数＝任务最大并行度，一个task要启一个容器，jobmanager要起一个容器，如果一个task使用两个slot，就是两个核，加上jobmanager就是七个，但实际上默认总共只有四个，不会根据你的slot设置来改变，所以要改配置文件参数，能够使用slot相同个数的核数。

2、通过**压测**检测单并行度最大处理能力，来设置全局并行度，最好通过提交任务的时候提交参数设置

精细并行度：如果连了kafka，就设置和kafka一样，内部一般是keyby之后并行度可以调大，keyby之前和sorce保持一样就可以了，代码中设置。

#### 大状态调优

1、开启State性能监控

2、(1)rocksDB是先写到内存，再刷写到磁盘，所以写的效率比较高。获取的时候先在内存中找，没有再去磁盘查询

(2) 可以开启增量检查点和本地恢复，开启增量检查点后checkpoint速度不一定比基于内存慢。

(3) 增加目录数，必须在不同磁盘上增加。

(4) 调整预定义选项。

(5)增大block缓存

#### checkpoint调优

间隔：(1~5)分钟，对于状态很大的，访问hdfs很耗时，就设置5~10分钟，并且调大暂停间隔，例如暂停4~8分钟;如果想要延迟小：就用秒，结合end-to-end的周期，不要比endtoend小，设置成秒级、毫秒级

最小等待间隔：参考间隔，一半

超时时间：默认10分钟，参考间隔    

失败此时

保留ck，取消作业后，checkpoint依旧在hdfs上。好处是除了用保存点恢复还可以用checkpoint来恢复。

web看日志报错可以搜索snapshot

### 反压

Flink 网络流控及反压的介绍：

https://flinklearning.org.cn/article/detail/138316d1556f8f9d34e517d04d670626

上游下游都各有buffer，上游用来发送，下游用来接收，下游每次接收会告知上游是否还有空间，当下游存满的时候，上游不发数据了，这时上游堆积也存满，反压产生

#### 后果

1、checkpoint时间变长，可能会超时

2、status变大

3、OOM、

4、内存过大被yarnKill掉

#### 检查方法

1、webui

2、禁用env.disableOperatorChaining算子链，让一个算子一个框

(上游都是反压找第一个OK的算子)

3、一般是下游第一个好的的问题，处理能力不行，导致上游堵死

另一种情况是第一个反压的算子的问题，例如flatmap(一进多出)数据膨胀，导致反压，现象是flatmap之前开始是好的，后面的包括flatmap全部反压

(上游都是好的，找第一个不OK的，或者结合Metrics进一步判断)

#### Metrics

有用的Metrics：outPoolUsage发送端buffer的使用率、inPoolUsage接收端buffer的使用率

1.9版本以上：

floatingBuffersUsage接收端Floating Buffer的使用率，可能被多个task使用。exclusiveBuffersUsage接收端ExlusiveBuffer的使用率，被一个task单独占用

inPoolUsage=后两个相加

反-反-反-OK-OK

定位第一个OK，看他的输入缓冲区是不是很高，输入很低，那他就是罪魁祸首

OK-OK-反-反-反

定位第一个反，看他的输入缓冲区是不是正常(如果高就可能被下游反压)，输出很高，那他就是罪魁祸首

inputGate和Task一对一，inputChannel的个数取决于上游有多少个实例发送的数据，用于接收数据，满了就申请本地缓存，再满了就申请公用的，只有部分inputChannel反压就有数据倾斜

#### 反压原因

1、数据倾斜

2、资源太少

3、长时间的GC

4、跟外部系统交互，外部系统跟不上

5、代码性能太差

##### (1)先看有没有数据倾斜

 看<mark>SubTasks</mark>

##### (2)火焰图

 开启火焰图参数：-D**rest.flamegraph.enabled=true**,如果不是数据倾斜，就分析性能瓶颈，看**火焰图**(通常需要进行CPU profile，看一个Task是否跑满了一个cpu核，如果是的话就看cpu主要花在哪些函数里面)1.13版本之后提供了JVM的CPU火焰图，大大简化性能瓶颈的分析，在<mark>FlameGraph</mark>里面。实时更新，颜色没有意义，从    下往上看是调用栈，看每个地方的最顶上的是当前正在运行的线程，横向是执行时长(次数)，找最顶层的最长的，就可能是引起问题的地方。上面三个标签，第一个查看运行中的线程，第二个查看阻塞的线程，第三个是混合状态，一般看第一个。

##### (3)分析GC

先打印GC日志，参数-D**env.java.opts="-XX:+PrintGCDetails -XX:+PrintGCDateStamps"**。然后在<mark>TaskManager</mark>点开一个LOG，Stdout里面，如果没配置路径的话Logs里面有默认路径，点击Stdout右边的下载，使用工具GCViewer，资料里面的jar包，双击打开，打开后选择刚才下载下来的文件，重点关注老年代，和fullGC。**设FUllGC之后老年代剩余空间大小为M，那么堆的大小建议是3~4倍M，新生代为1~1.5倍M，老年代应为2~3倍M**

#### 外部组件的交互

1、source读取性能问题，和sink端写入性能差，需要检验第三方组件是否遇到瓶颈，还有就是做维表join时的性能问题，例如

kafka集群是否要扩容，kafka连接器是否并行度较低

hbase的rowkey热点问题，是否请求处理不过来

clickhouse并发能力弱，是否达到瓶颈

**第三方组件性能问题通常思路：**

1、异步io+热缓存来优化读写性能

2、先攒批再写

**维表join文章：**

https://flinklearning.org.cn/article/detail/b8df32fbc6542257a5b449114e137cc3

https://www.jianshu.com/p/a62fa483ff54
