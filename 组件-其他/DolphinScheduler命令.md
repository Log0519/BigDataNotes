# Dolphinscheduler操作流程
@[toc]
## 1、启动zookeeper
web页面： http://hadoop102:12345/dolphinscheduler
初始用户的用户名为：admin，密码为 dolphinscheduler123
log账户无密码
## 1）一键启停所有服务
./bin/start-all.sh
./bin/stop-all.sh

## 2）启停 Master
./bin/dolphinscheduler-daemon.sh start master-server
./bin/dolphinscheduler-daemon.sh stop master-server
## 3）启停 Worker
./bin/dolphinscheduler-daemon.sh start worker-server
./bin/dolphinscheduler-daemon.sh stop worker-server
## 4）启停 Api
./bin/dolphinscheduler-daemon.sh start api-server
./bin/dolphinscheduler-daemon.sh stop api-server
## 5）启停 Logger
./bin/dolphinscheduler-daemon.sh start logger-server
./bin/dolphinscheduler-daemon.sh stop logger-server
 尚硅谷大数据技术之 DolphinScheduler 
## 6）启停 Alert
./bin/dolphinscheduler-daemon.sh start alert-server
./bin/dolphinscheduler-daemon.sh stop alert-server