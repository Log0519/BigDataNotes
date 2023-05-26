# 1 组件定义
 a1.sources = r1
 a1.sinks = k1
 a1.channels = c1
 # 2 配置 source
 a1.sources.r1.type = TAILDIR
 a1.sources.r1.filegroups = f1
 a1.sources.r1.filegroups.f1 = /opt/module/applog/app.*
# 1 组件定义
 a1.sources = r1
 a1.sinks = k1
 a1.channels = c1
 # 2 配置 source
 a1.sources.r1.type = TAILDIR
 a1.sources.r1.filegroups = f1
 a1.sources.r1.filegroups.f1 = /opt/module/applog/app.*
 a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json
 # 3 配置 channel
 a1.channels.c1.type = memory
 a1.channels.c1.capacity = 1000
 a1.channels.c1.transactionCapacity = 100
 # 4 配置 sink
 a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
 a1.sinks.k1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
 a1.sinks.k1.kafka.topic = first
 a1.sinks.k1.kafka.flumeBatchSize = 20
 a1.sinks.k1.kafka.producer.acks = 1
 a1.sinks.k1.kafka.producer.linger.ms = 1
# 5 拼接组件
 a1.sources.r1.channels = c1
 a1.sinks.k1.channel = c1
