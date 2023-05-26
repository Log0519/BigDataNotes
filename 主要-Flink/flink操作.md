命令行提交任务引用
flink run -m hadoop102:8081(指定任务master) -c com.log.StreamWordCount(类路劲) -p 2(并行度) ./FlinkTutorial-1.0-SNAPSHOT.jar(任务jar包) --host hadoop102(任务参数) --port 7777(任务参数)

查看作业运行状态
flink list 
查看所有作业状态，包括之前取消的
flink list -a
取消作业
flink cancel +jobID、

web页面haodop102:8081

在 hadoop102 节点服务器上执行 start-cluster.sh 启动 Flink 集群ster.sh 启动 Flink 集群