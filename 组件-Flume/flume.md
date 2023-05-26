开启监听端口
bin/flume-ng agent -n a1 -c conf/ -f job/net-flume-logger.conf -Dflume.root.logger=INFO,console
另一个地方 开启客户端
 nc localhost 44444

防止晚上文件名字更改，导致文件重复上传一次
源码修改
H:\Flume资料\资料\资料\apache-flume-1.9.0-src\apache-flume-1.9.0-src\flume-ng-sources\flume-taildir-source\src\main\java\org\apache\flume\source\taildir
TailFile
第122行
ReliableTaildirEventReader.java
第254行

打印到控制台
-Dflume.root.logger=INFO,console

使用memoryChannel通常把容量参数100改为1000

hdfsSink：
hdfs.rollInterval	30	=>1h
hdfs.rollSize	1024	=>128
hdfs.rollCount	10  	=>0


