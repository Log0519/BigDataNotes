# Hadoop搭建流程
@[toc]
 1.注：本文只是hadoop搭建大概流程，供参考和复习使用，可以用来检验自己的步骤某些具体操作可以自行上网搜索

 2.前置操作：先安装好第一台虚拟机，最好分配50G一台，方便后续学习使用，这个50G是虚拟机的内存上限，实际使用不会占到那么多，首先配置第一台虚拟机的IP，主机名称（参考:192.168.10.102,hadoop102）

 3.使用工具 Finalshell，VMwareWorkstation Pro

## 第一步：安装jdk

1.删除自带java，装入自己的java
2.装入hadoop在`/opt/module`中，
3.`/opt/module`用来存软件，`/opt/software`存压缩包
 4.配置环境变量

### 第二步：克隆虚拟机

1. 第一台作为版机，克隆三台虚拟机
   2. 记得克隆的虚拟机也需要更改`ip`和`hostname`

### 第三步：虚拟机连接界面

 连接`finalshell`(虚拟机连接界面)

### 第四步：配置分发脚本

 配置`xsync`分发脚本,用户目录下创建一个`bin`目录，写一个`xsync`在里面,内容如下
 使用方式，`xsync 文件名(路径)`

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

### 第五步:配置各个主机、账户免密登录

 1.使用`ssh-keygen -t rsa`生成公钥私钥
 2.使用`ssh-copy-id hadoop102(主机名)`拷贝公钥

### 第六步：集群配置信息

在`opt/modle/hadoop3.3.2/etc/hadoop`下
 1.配置核心配置文件`core-site.xml`

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

2.配置HDFS配置文件`hdfs-site.xml`

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

3.配置`MapReduce-site.xml`

    <configuration>
    
    <!--指定MapReduce运行在Yarn上-->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    
    </configuration>

4.配置`yarn-site.xml`

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

### 第七步：分发配置

1.在路径`/opt/module/hadoop3.3.2/etc`下
2.使用编写好的脚本`xsync`分发刚才修改配置的目录给其他虚拟机

### 第八步：配置works

##### 1.配置works

 `vim /opt/module/hadoop-3.3.2/etc/hadoop/workers`

 添加:

     hadoop102
    
     hadoop103
    
     hadoop104

本结尾不能有空格，文件不能有空行

##### 2.使用`xsync`分发同步配置

### 第九步：初始化

 1.启动hadoop集群`sbin/start-dfs.sh`,再在hadoop`103`上启动`yarn-dfs.sh`
 2.在`/opt/module/hadoop-3.3.2`中先进行初始化,执行`hdfs namenode -format`初始化

### 第十步：添加windows的hosts映射

 配置`windows`的`hosts映射`，给当前window用户修改host文件的权限，然后添加

    192.168.10.102  hadoop102
    192.168.10.103  hadoop103
    192.168.10.104  hadoop104

### 第十一步：访问

 1.通过浏览器访问`hadoop102:9870`和`hadoop103:8088`
 2.添加yarn配置`hadoop classpath`

### 第十二步：历史服务器

1.配置历史服务器
2.重启yarn
3.启动历史服务器`mapred --daemon start historyserver`

### 第十三步：hdfs相关操作

 `注意：hadoop集群刚启动的时候会处于安全模式`
手动退出安全模式`hadoop dfsadmin -safemode leave`

#### 测试

使用`hadoop fs -(指令)`，对HDFS进行操作

    hadoop fs -mkdir /input
    hadoop fs -put wcinput/word.txt /input
    hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.2.jar wordcount /input /output

### 第十四步：日志聚集

 1.在`core-site.xml`配置日志聚集到HDFS

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

 2.重启yarn和历史服务器，停止命令`mapred --daemon stop historyserver`

### 第十五步：hadoop集群启停脚本

编写hadoop常用集群脚本，写在家目录的bin目录下面（建议文件名后面加上.sh），写完后记得要赋予执行权限
1.hadoop集群`启停`脚本
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

 2.在用户家目录的bin目录下，创建`jpsall`文件，配置查看所有虚拟机上的jps

    #!/bin/bash
    for host in hadoop102 hadoop103 hadoop104
    do
     echo =============== $host ===============
     ssh $host jps
    done

#### 第十六步：配置时间服务器(选配)

可以配置时间服务器，方便不能连接外网的生产环境同步时间

#### 第十七步：让用户可以在HDFS页面上进行操作

需要在在`core-site.xml`中添加

    <!--配置HDFS网页登录使用的静态用户为log-->
    <property>
    <name>hadoop.http.staticuser.user</name>
    <value>log</value>
    </property>
    </configuration>

#### 到此hadoop搭建完成!

`个人遇到的其中一个问题，namenode关不掉：此时需要配置pid！`
