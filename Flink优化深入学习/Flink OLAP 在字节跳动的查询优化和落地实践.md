# Flink OLAP 在字节跳动的查询优化和落地实践


实践

@[toc]


## 一、总体建构&业务架构

### 1、总体架构

&ensp;&ensp;&ensp;&ensp;用户在client通过接口，提交一个query，经过SQL Geteway的解析和优化，生成作业的执行计划，通过高效的socket接口(NettySocketServer)提供给Flink Session Cluster上的 JobManager,随后JobManager上的Dispatcher会创建一个JobMaster，最后，JobMaster通过集群对应的TaskManager，按照一定的调度策略，将task部署到对应的TaskManager上并执行。

#### 1.1 Flink SQL Geteway
parse,Validate,Optimize
#### 1.2 Flink Session Cluster
JobManager、TaskManager
### 2、业务架构(双机房容灾)
Client->MysqlProxy->Flink SQL Gateway=>FlinkSession Cluster->HTAP Store


## 二、优化实践
### 1、查询优化
#### 1.1、Query Optimizer(优化器)优化
##### 1.1.1、Plan缓存
&ensp;&ensp;&ensp;&ensp;业务上有很多重复的Query，查询耗时要求亚秒级，其中Plan耗时几十到几百毫秒占比较高，因此进行Plan缓存。
①对Query的Plan结果Transformations进行缓存，避免相同Query的重复Plan
②支持catlog cache，加速了元信息的访问
③exccload的并行Translate
**结果：Plan耗时降低了10%**
##### 1.1.2、算子下推(TopN)
在存算分离架构下，尽可能将算子下推到存储层来计算，减少scan的数据量，降低外部的io，明显提升性能。
把local的topN算子下推到Scan节点，最后在存储层进行topN的计算，大大降低了从存储读取的数据量
**结果：从存储读取的数据量降低了99.9%；query的Latency降低了90.4%** 
##### 1.1.3、跨union all的下推
字节有一个业务是典型的分库分表存放，用户如果需要查询全量数据，需要进行union all，导致需要读取大量的数据，并且占用大量资源处理aggrgate，sortlimit(top N)
**结果：E2E Latency降低42%，Flink的CPU消耗降低30%**
##### 1.1.4 Join Filter传递
#### 1.2、Query Executor(执行器)优化



