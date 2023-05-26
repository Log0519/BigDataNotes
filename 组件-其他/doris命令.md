集群启停脚本：mydoris.sh start mydoris.sh stop
启动fe：fe/bin/start_fe.sh --daemon
停止fe：fe/bin/start_fe.sh 
web页面 hadoop102:8030 ，账号root，密码fuxiaoluo

进入mysql客户端 mysql -P9030 -hhadoop102 -uroot -pfuxiaoluo
mysql -P9030 -hhadoop102 -utest -ptest

启动be(每个节点)：be/bin/start_be.sh --daemon

查看broker状态：SHOW PROC "/brokers";
查看FE状态：SHOW PROC '/frontends';
查看BE状态1：SHOW PROC '/backends';        
查看BE状态2：SHOW BACKENDS\G

查看动态分区表调度情况
SHOW DYNAMIC PARTITION TABLES;
SHOW DYNAMIC PARTITION TABLES\G
查看表的分区
SHOW PARTITIONS FROM stufent_dynamic_partition1;
SHOW PARTITIONS FROM stufent_dynamic_partition1\G;
