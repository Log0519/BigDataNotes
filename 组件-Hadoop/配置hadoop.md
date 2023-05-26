先配置第一台虚拟机的IP，主机名称
第一步：
删除自带java，装入自己的java
装入hadoop在/opt/module中
/opt/software存压缩包
配置环境变量

第二步：
第一台作为版机，克隆三台虚拟机
更改ip和hostname

第三步：
连接finalshell

第四步：
配置xsync分发脚本,用户目录下创建一个bin目录，写一个xsync在里面==============
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if [ $pcount -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
# 也可以采用：
# for host in hadoop{102..104};
for host in hadoop102 hadoop103 hadoop104
do
    echo ====================    $host    ====================
    #3. 遍历所有目录，挨个发送
    for file in $@
    do
        #4 判断文件是否存在
        if [ -e $file ]
        then
            #5. 获取父目录
            pdir=$(cd -P $(dirname $file); pwd)
            echo pdir=$pdir
            
            #6. 获取当前文件的名称
            fname=$(basename $file)
            echo fname=$fname
            
            #7. 通过ssh执行命令：在$host主机上递归创建文件夹（如果存在该文件夹）
            ssh $host "mkdir -p $pdir"
            
                        #8. 远程同步文件至$host主机的$USER用户的$pdir文件夹下
            rsync -av $pdir/$fname $USER@$host:$pdir
        else
            echo $file does not exists!
        fi
    done
done


然后配置各个主机、账户免密登录============================
ssh-keygen -t rsa生成公钥私钥
ssh-copy-id hadoop10.拷贝公钥

第五步：集群配置opt/modle/hadoop3.3.2/etc/hadoop下
配置核心配置文件core-site.xml===================================
<configuration>
     <!--指定NameNode的地址-->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://hadoop102:8020</value>
	</property>
     <!--指定hadoop数据的存储目录-->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/module/hadoop-3.3.2/data</value>
	</property>
</configuration>

配置HDFS配置文件hdfs-site.xml==================================
<configuration>
<!--nn web端访问地址-->
        <property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop102:9870</value>
        </property>
<!--2nn web端访问地址-->
         <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:9868</value>
        </property>
</configuration>

配置MapReduce-site.xml==================================
<configuration>

<!--指定MapReduce运行在Yarn上-->
<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
</property>

</configuration>

配置yarnsite.xml===========================================
<configuration>

<!-- Site specific YARN configuration properties -->
<!--指定MR走shuffle-->
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<!--指定ResourceManager地址-->
<property>
<name>yarn.resourcemanager.hostname</name>
<value>hadoop103</value>
</property>

</configuration>

第六步
pwd=/opt/module/hadoop3.3.2/etc
xsync分发hadoop/这个目录给其他虚拟机

第七步
配置works
vim /opt/module/hadoop-3.3.2/etc/hadoop/workers
添加
hadoop102
hadoop103
hadoop104
本结尾不能有空格，文件不能有空行
xsync同步配置

第八步
启动集群
在/opt/module/hadoop-3.3.2中先进行初始化执行hdfs namenode -format初始化
启动
 sbin/start-dfs.sh
在hadoop103上启动yarn-dfs.sh

第九步
配置windows的hosts映射，给当前window用户修改host文件的权限，然后添加192.168.10.102  hadoop102
192.168.10.103  hadoop103
192.168.10.104  hadoop104

第十步
通过浏览器访问hadoop102:9870和hadoop103:8088
添加yarn配置hadoop classpath

第十一步
配置历史服务器
重启yarn
启动历史服务器
mapred --daemon start historyserver

退出安全模式hadoop dfsadmin -safemode leave
测试
hadoop fs -mkdir /input
hadoop fs -put wcinput/word.txt /input
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.2.jar wordcount /input /output

第十二步
配置日志聚集到HDFS
</property>
<!--日志聚集功能使能-->
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>
<!--设置日志聚集服务器地址-->
<property>
        <name>yarn.log.server.url</name>
        <value>http://hadoop102:19888/jobhistory/logs</value>
</property>
<!--日志保留时间设置7天-->
<property>
<name>yarn.log-aggregation.retain-seconds</name>
<value>604800</value>
</property>
</configuration>

重启yarn和历史服务器
mapred --daemon stop historyserver

第十三步
编写hadoop常用集群脚本
#!/bin/bash

if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi

case $1 in
"start")
        echo " =================== 启动 hadoop集群 ==================="

        echo " --------------- 启动 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.3.2/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.3.2/sbin/start-yarn.sh"
        echo " --------------- 启动 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.3.2/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " =================== 关闭 hadoop集群 ==================="

        echo " --------------- 关闭 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.3.2/bin/mapred --daemon stop historyserver"
        echo " --------------- 关闭 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.3.2/sbin/stop-yarn.sh"
        echo " --------------- 关闭 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.3.2/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac
配置查看所有jps
#!/bin/bash
for host in hadoop102 hadoop103 hadoop104
do
 echo =============== $host ===============
 ssh $host jps
done

第十四步
可以配置时间服务器，方便不能连接外网的生产环境同步时间

第十五步
在core-site.xml中
<!--配置HDFS网页登录使用的静态用户为log-->
<property>
<name>hadoop.http.staticuser.user</name>
<value>log</value>
</property>
</configuration>

-----------------------------
如果namenode关不掉，根据收藏配置pid











