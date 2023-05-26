### Kafka基本命令和操作
@[toc]
##### 一.创建一个主题

    bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --topic first --create --partitions 1 --replication-factor 3
    bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --topic first --alter --partitions 3
    bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --topic first --describe

注意：分区只能增加，不能减少

##### 二.生产者

连接

    bin/kafka-console-producer.sh --bootstrap-server hadoop102:9092
     --topic first

##### 三.消费者

连接

    bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092
     --topic first

##### 四.基本语法

1.后台启动kafka

    kafka-server-start.sh -daemon config/server.properties

2.创建topic

    kafka-topics.sh --create --zookeeper hadoop1:2181 --replication-factor 1 --partitions 1 --    topic test20200818

3.查看topic

    kafka-topics.sh  --zookeeper hadoop1:2181 --list

4.查看topic详情

    kafka-topics.sh --zookeeper hadoop1:2181 --describe --topic kb07demo

5.删除topic

    kafka-topics.sh --zookeeper hadoop1:2181 --delete --topic test3

6.启动product

    kafka-console-producer.sh --topic kb07demo --broker-list hadoop1:9092

7.启动consumer

    kafka-console-consumer.sh --bootstrap-server hadoop1:9092 --topic kb07demo --from-beginning

8.查看分区数据

    kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list hadoop1:9092 --topic kb07demo -time -1 --offsets 1

##### 五.零散知识

1.一个log文件默认大小为`1G`

2.一般把mysql的表名作为key用来分区，将一个表分到同一个分区中

3.可以配置一个kafka2(脱离zookeeper)

    修改/opt/module/kafka2/config/kraft/server.properties
    
    初始化bin/kafka-server-start.sh -daemon config/kraft/server.properties
