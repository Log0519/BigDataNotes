load data local inpath 'xxxxxxxxxx' into table xxx;4

如果使用jdbc连接，要先启动服务
先bin/hive --service metastore
后bin/hive --service hiveserver2
启动hive
启动hadoop
bin/hive