启动：
start-hbase
关闭：
方案一：stop-hbase
方案二：先hbase-daemons.sh stop master
再hbase-daemons.sh stop regionserver，如果pid目录有问题，就重新配置pid路径，我重新配过了