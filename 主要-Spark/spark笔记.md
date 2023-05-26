# Spark知识点
@[toc]
//rdd.map是一个一个对数据操作，类似于串行，mapPartitions是对一个分区内的数据操作
//scala和spark都是两两聚合,先执行前面两个相加，再把结果和后面一个相加
byKey和newRDD.mapValues都是针对key的value操作

## 1、RDD 任务划分
RDD 任务切分中间分为：Application、Job、Stage 和 Task
1. Application：初始化一个 SparkContext 即生成一个 Application； 
2. Job：一个 Action 算子就会生成一个 Job； 
3. Stage：Stage 等于宽依赖(ShuffleDependency)的个数加 1；
4. Task：一个 Stage 阶段中，最后一个 RDD 的分区个数就是 Task 的个数。
注意：Application->Job->Stage->Task 每一层都是 1 对 n 的关系

## 2、知识点
spark有隐式转换功能
shuffle，为了把数据汇总，要进行shuffle，shuffle一定会落盘，在文件中等待所有分区的数据执行完毕，分为不同阶段，一个阶段全部执行完才能执行下一个阶段
RDD类似io，底层是装饰者设计模式
DAG有向无环图，记录了RDD的转换过程和任务的阶段
flatmap返回List
Nil代表无数据
isZero代表初始化
init不包含最后一个
0L，1L，表示转为Long类型
scala可以不写main函数，让类继承App类
Any=要返回的Unit
rdd.foreach打印的数据是乱的，因为是分布式打印，不能确定数据顺序

## 3、持久化：
Cache和Persist方法将前面的计算结果缓存，默认情况下会把数据以缓存在JVM的堆内存中，不是在这两个方法被调用时立即缓存，而是触发后面的action算子时，该rdd将会被缓存在计算节点的内存中，并供后面使用，使用完后会进行删除
检查点：CheckPoint，将RDD中间结果写入磁盘，由于血缘依赖过长会造成容错成本过高，这样不如在中间阶段做检查点容错，如果检查点之后有节点出现问题，可以从检查点开始重做血缘，减少开销，也是触发后面的action算子时才会使用，会落盘，执行完后不会删除

## 4、广播变量：broadcast（）
## 累加器：longAccumulator（），也可以自定义


## 5、工作中代码架构
三层架构：controller 控制层, service 服务层 , dao持久层
应用->控制层(调度)->服务层(逻辑)->持久层(读取数据库、文件，给服务层)，文件层再返回控制层

## 6、groupByKey,reduceByKey的区别
reduceByKey可以预聚合，在shuffle之前减少数据量，提高了性能

## 7、aggregate的初始值会参与分区内和分区间的计算
aggregateByKey只参与分区内计算



## 8、RDD 序列化
### 1) 闭包检查
从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor
端执行。那么在 scala 的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就
形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给 Executor
端执行，就会发生错误，所以需要在执行任务计算前，检测闭包内的对象是否可以进行序列
化，这个操作我们称之为闭包检测。Scala2.12 版本后闭包编译方式发生了改变
### 2) 序列化方法和属性
从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor
端执行
### Kryo 序列化框架
参考地址: https://github.com/EsotericSoftware/kryo
Java 的序列化能够序列化任何的类。但是比较重（字节多），序列化后，对象的提交也
比较大。Spark 出于性能的考虑，Spark2.0 开始支持另外一种 Kryo 序列化机制。Kryo 速度
是 Serializable 的 10 倍。当 RDD 在 Shuffle 数据的时候，简单数据类型、数组和字符串类型
已经在 Spark 内部使用 Kryo 来序列化。
注意：即使使用 Kryo 序列化，也要继承 Serializable 接口。