查看docker状态：systemctl status docker
重启docker：systemctl restart docker
查看容器进程：docker ps
查看所有镜像：docker images
运行镜像：docker run ... ... ...(/bin/bash)
删除镜像：docker rmi -f +IMAGE ID，或者REPOSITORY+TAG
停止容器：docker stop 
进入容器：docker exec -it 91616117ced9 /bin/bash
退出容器：exit
删除容器：docker rm -f 容器名字

首次提交私服库过程：
1.启动私库：docker run -d -p 5000:5000 -v /Log/myregistry/:/tmp/registry --privileged=true registry
2.提交生成镜像：docker commit -m="ifconfig cmd add" -a="Log" f7a200681c52 logubuntu:1.2，库名称不能大写
3.可以检查私服库有没有镜像：curl -XGET http://192.168.10.102:5000/v2/_catalog
4.修改为正确的tag格式：docker tag logubuntu:1.2 192.168.10.102:5000/logubuntu:1.2
5.docker默认不允许http方式推送镜像，通过配置来取消，修改后重启：sudo vim /etc/docker/daemon.json,加个逗号，然后添加"insecure-registries": ["192.168.10.102:5000"]
6.推送： docker push 192.168.10.102:5000/logubuntu:1.2
7.检查：curl -XGET http://192.168.10.102:5000/v2/_catalog
8.拉下来：docker pull 192.168.10.102:5000/logubuntu:1.2

容器卷：可以完成容器数据的持久化，重要资料的备份到本地，还可以实现容器间的、容器和主机间的数据共享、传递
1.挂载目录后面＋privileged=true参数
2.docker run -it --privileged=true -v/宿主机绝对路径目录:/容器内目录 镜像名 ，可以实现两个目录的资源互通
3.查看已挂载：docker inspect +容器id，Mounts下为挂载路径
4.容器停止以后，如果主机在挂载目录中有修改，容器挂载目录中的信息依旧会同步
5.权限控制：docker run -it --privileged=true -v/宿主机绝对路径目录:/容器内目录:ro 镜像名,加了ro以后，容器内只能读，不能写
6.另一个容器继承一个容器的卷规则：docker run -it --privileged=true --volumes-from 父类 --name u2 镜像，实现多个容器和主机之间的数据共享，父类停不停不影响子类

