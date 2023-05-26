# Spark知识点汇总
@[toc]
#### 一.RDD 任务划分

  RDD 任务划分中间分为：Application、Job、Stage 和 Task

1. **Application**：初始化一个 SparkContext 即生成一个 Application； 
2. **Job**：一个 Action 算子就会生成一个 Job； 
3. **Stage**：Stage 等于宽依赖(ShuffleDependency)的个数加 1；
4. **Task**：一个 Stage 阶段中，最后一个 RDD 的分区个数就是 Task 的个数。
   `注意：Application->Job->Stage->Task 每一层都是 1 对 n 的关系`

#### 二.持久化操作

1.**Cache和Persist**：`Cache`和`Persist`可以将前面的计算结果缓存，默认情况下会把数据以缓存在**JVM的堆内存**中，不是在这两个方法被调用时立即缓存，而是触发后面的action算子时，该rdd将会被缓存在计算节点的**内存**中，并供后面使用，**使用完后会进行删除**。

2.**检查点**：`CheckPoint`，将RDD中间结果写入磁盘，由于血缘依赖过长会造成容错成本过高，这样不如在中间阶段做检查点容错，如果检查点之后有节点出现问题，可以从检查点开始重做血缘，减少开销，也是触发后面的action算子时才会使用，会**落盘**，**执行完后不会删除**

#### 三.RDD 序列化

**1) 闭包检查**
从计算的角度, **算子以外**的代码都是在 `Driver `端执行, **算子里面**的代码都是在 `Executor`端执行。那么在 scala 的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就形成了**闭包**的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给 Executor端执行，就会发生错误，所以需要在执行任务计算前，**检测闭包内的对象是否可以进行序列化**，这个操作我们称之为**闭包检测**。
`注意：Scala2.12 版本后闭包编译方式发生了改变.`
**2) Kryo 序列化框架**
参考地址: `https://github.com/EsotericSoftware/kryo`

1. Java 的序列化能够序列化任何的类。但是比较重（字节多），序列化后，对象的提交也比较大。
2. Spark 出于性能的考虑，Spark2.0 开始支持另外一种 Kryo 序列化机制。Kryo 速度是 Serializable 的 10 倍。
3. 当 RDD 在 Shuffle 数据的时候，简单数据类型、数组和字符串类型已经在 Spark 内部使用 Kryo 来序列化。
   `注意：即使使用 Kryo 序列化，也要继承 Serializable 接口。`

#### 四.零散知识点

1. spark有**隐式转换**功能
2. `shuffle`，为了把数据汇总，要进行shuffle，shuffle一定会落盘，在文件中等待所有分区的数据执行完毕，分为不同阶段，一个阶段全部执行完才能执行下一个阶段
3. RDD类似io，底层是**装饰者设计模式**
4. **DAG**是有向无环图，记录了RDD的转换过程和任务的阶段
5. `flatmap`返回List
6. `Nil`代表无数据
7. `isZero`代表初始化
8. `init`表示不包含最后一个
9. `0L`，`1L`，表示转为Long类型
10. scala可以不写main函数，让类继承App类
11. Any=要返回的Unit
12. `rdd.foreach`打印的数据是乱的，因为是分布式打印，不能确定数据顺序
13. **广播变量**：broadcast（）
14. **累加器**：longAccumulator（），也可以自定义
15. spark有两种**调度模式**，**FIFO先进先出**，和**公平调度**，默认是FIFO
    
    #### 五.代码架构
    
    `controller 控制层, service 服务层 , dao持久层`
    应用->控制层(调度)->服务层(逻辑)->
    持久层(读取数据库、文件，给服务层)，文件层再返回控制层

#### 六.groupByKey和reduceByKey的区别

reduceByKey可以预聚合，在shuffle之前减少数据量，提高了性能，应当尽量避免使用groupByKey。

#### 七.aggregate和aggregateByKey的区别

aggregate的初始值会参与分区内和分区间的计算
aggregateByKey只参与分区内计算

#### 八.SparkSQL广播 join

当数据集的大小小于spark.sql.autoBroadcastJoinThreshold 所设置的阈值的时候， SPARK SQL使用广播join来代替hash join来优化join查询。广播join可以非常有效地用于具有相对较小的表和大型表之间的连接，然后可用于执行星型连接。它可以避免通过网络发送大表的所有数据。

**1.dataframe中指明广播join**

    // 1 广播变量参数           
    conf.set( "spark.sql.autoBroadcastJoinThreshold", "104857600" ) //广播表的上限:单位为B,现设置最大广播100M的表;
    conf.set( "spark.sql.broadcastTimeout", "-1" ) //广播超时时间: 单位为ms, -1为永不超时;
    
    // 2 引入需要的包
    import org.apache.spark.sql.functions._
    
    // 3 内关联eid码表(显示指明为广播join)
    val join_eid_df = parse_df
    .join( broadcast( eid_t8_df ), Seq( "tid", "event" ) )

**2.sql语句中显示的指明广播join**

    // 处理语句
    val deal_sql =
      """
        |select /*+ broadcast (a) */ *
        |from range(10) a, range(100) b
        |where a.id = b.id
        |""".stripMargin
    // 执行处理语句
    val res=spark.sql(deal_sql)
    // 打印执行计划
    res.explain()
    // 打印结果
    res.show()
