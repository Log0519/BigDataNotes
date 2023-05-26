# Kafka操作
@[toc]
## 创建一个主题
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --topic first --create --partitions 1 --replication-factor 3
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --topic first --alter --partitions 3
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --topic first --describe
分区只能增加，不能减少

## 查看所有topic
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --list 
## 查看topic详情    
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --describe --topic first
## 修改分区数
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --alter --topic first --partitions 3 只能增加不能减小
## 删除topic
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --delete --topic first

## 把主题中所有的数据都读取出来
bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --from-beginning --topic first

## 生产者
连接
bin/kafka-console-producer.sh --bootstrap-server hadoop102:9092 --topic first

## 消费者
连接
bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic first

一般把mysql的表名作为key用来分区，将一个表分到同一个分区中

## kafka2
修改/opt/module/kafka2/config/kraft/server.properties
初始化bin/kafka-server-start.sh -daemon config/kraft/server.properties

## 命令
--bootstrap-server <String: server toconnect to> 连接的 Kafka Broker 主机名称和端口号。
--topic <String: topic> 操作的 topic 名称。
--create 创建主题。
--delete 删除主题。
--alter 修改主题。
--list 查看所有主题。
--describe 查看主题详细描述。
--partitions <Integer: # of partitions> 设置分区数。
--replication-factor<Integer: replication factor> 设置分区副本。
--config <String: name=value> 更新系统默认的配置。

--topic 定义 topic 名
--replication-factor 定义副本数
--partitions 定义分区数