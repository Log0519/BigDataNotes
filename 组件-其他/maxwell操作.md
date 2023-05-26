maxwell操作：可以监控mysql的操作，发送到kafka
开启mysql的binlog服务，修改MySQL配置文件 sudo vim /etc/my.cnf
在mysql标签下面增加如下配置
#数据库 id
server-id=1
#启动binlog，该参数的值会作为binlog的文件名
log-bin=mysql-bin
#binlog类型，maxwell要求为row类型
binlog_format=row
#启动binlog的数据库,需根据实际情况做出修改
binlog-do-db=gmall


重启mysql服务
sudo systemctl restart mysqld
1.登录mysql的root用户，在mysql中创建一个maxwell数据库，CREATE DATABASE maxwell
2.创建一个maxwell用户，专门用于maxwell操作
CREATE USER 'maxwell'@'%' IDENTIFIED BY 'maxwell',这句话意思为创建一个叫maxwell的用户，允许连接任意ip地址，密码也是maxwelll
3.赋予权限
GRANT ALL ON maxwell.* TO 'maxwell'@'%';
把maxwell库中的所有权限赋予给maxwell用户，
GRANT SELECT,REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO 'maxwell'@'%';
把任意库任意表的读权限赋予给用户maxwell

拷贝一份config.properties.example以备后续使用
拷贝出来的那一份叫做config.properties
1）修改Maxwell配置文件config.properties
# tl;dr config
log_level=info

producer=kafka
kafka.bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092
#kafka topic动态配置，也可以是静态，该本次maxwell操作涉及到13张表，需要13个topic，所以使用动态配置
kafka_topic=%{table}

#表过滤，只同步特定的13张表
filter= include:gmall.cart_info,include:gmall.comment_info,include:gmall.coupon_use,include:gmall.favor_info,include:gmall.order_detail,include:gmall.order_detail_activity,include:gmall.order_detail_coupon,include:gmall.order_info,include:gmall.order_refund_info,include:gmall.order_status_log,include:gmall.payment_info,include:gmall.refund_payment,include:gmall.user_info

# mysql login info
host=hadoop102
user=maxwell
password=maxwell
jdbc_options=useSSL=false&serverTimezone=Asia/Shanghai

maxwell服务启动,只有在maxwell根目录下才能直接使用
bin/maxwell
当你在其他位置需要使用
/opt/module/maxwell/bin/maxwell --config /opt/module/maxwell/config.properties
后台运行
/opt/module/maxwell/bin/maxwell --config /opt/module/maxwell/config.properties --daemon

使用之前要先启动zookeeper，kafka，并且要在kafka上提前创建主题，maxwell不会自动在kafka上创建,kafka-topics.sh --bootstrap-server hadoop102:9092 --create --topic maxwell --partitions 3 --replication-factor 1

监控kafka-topics.sh --bootstrap-server hadoop102:9092 --topic maxwell

配置maxwell启停脚本

ps -ef | grep maxwell | grep -v grep| awk '{print $2}'  | xargs kill 杀死maxwell

脚本如下
#!/bin/bash
MAXWELL_HOME=/opt/module/maxwell
status_maxwell(){
        result=`ps -ef | grep maxwell | grep -v grep | wc -l`
        return $result
}

start_maxwell(){
status_maxwell
if [[ $? -gt 0 ]];then
echo "==============正在启动maxwell================"    
$MAXWELL_HOME/bin/maxwell --config $MAXWELL_HOME/config.properties --daemon
else
        echo "maxwell之前已经启动过了"
fi
}
stop_maxwell(){
status_maxwell
if [ $? -gt 0 ]; then
        echo "===============正在关闭maxwell，这个过程可能持续5秒================"
        ps -ef | grep maxwell | grep -v grep| awk '{print $2}'  | xargs kill
else
        echo "maxwell没有启动"
fi
}

case $1 in
"start")
start_maxwell
;;
"stop")
stop_maxwell
;;
"restart")
stop_maxwell
start_maxwell
;;
esac

maxwell的全量同步
虽然maxwell用来增量同步，但是通常首日需要一次全量同步，采用bin目录下的maxwell-bootstrap
在~/bin目录创建mysql_to_kafka_inc_init.sh，vim mysql_to_kafka_inc_init.sh，脚本内容例如
#!/bin/bash

# 该脚本的作用是初始化所有的增量表，只需执行一次

MAXWELL_HOME=/opt/module/maxwell

import_data() {
    $MAXWELL_HOME/bin/maxwell-bootstrap --database gmall --table $1 --config $MAXWELL_HOME/config.properties
}

case $1 in
"cart_info")
  import_data cart_info
  ;;
"comment_info")
  import_data comment_info
  ;;
"coupon_use")
  import_data coupon_use
  ;;
"favor_info")
  import_data favor_info
  ;;
"order_detail")
  import_data order_detail
  ;;
"order_detail_activity")
  import_data order_detail_activity
  ;;
"order_detail_coupon")
  import_data order_detail_coupon
  ;;
"order_info")
  import_data order_info
  ;;
"order_refund_info")
  import_data order_refund_info
  ;;
"order_status_log")
  import_data order_status_log
  ;;
"payment_info")
  import_data payment_info
  ;;
"refund_payment")
  import_data refund_payment
  ;;
"user_info")
  import_data user_info
  ;;
"all")
  import_data cart_info
  import_data comment_info
  import_data coupon_use
  import_data favor_info
  import_data order_detail
  import_data order_detail_activity
  import_data order_detail_coupon
  import_data order_info
  import_data order_refund_info
  import_data order_status_log
  import_data payment_info
  import_data refund_payment
  import_data user_info
  ;;
esac
2）为mysql_to_kafka_inc_init.sh增加执行权限
[atguigu@hadoop102 bin]$ chmod +x ~/bin/mysql_to_kafka_inc_init.sh

增量表同步，需要在首日进行一次全量同步，后续每日才是增量同步。首日进行全量同步时，需先启动数据通道，包括Maxwell、Kafka、Flume，然后执行增量表首日同步脚本mysql_to_kafka_inc_init.sh进行同步。后续每日只需保证采集通道正常运行即可，Maxwell便会实时将变动数据发往Kafka。


