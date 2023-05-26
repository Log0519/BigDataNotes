# Docker学习笔记
@[toc]

前置：安装好docker，配置好相关操作，配置阿里云镜像加速，

## 一.docter的三大组件

1.镜像（image）

2.容器（container）

3.仓库（repository）

## 二.常用网站

`https://www.docker.com/ `

`https://hub.docker.com/`

## 三.常用命令

1.搜索镜像：docker search 镜像名字

2.拉取镜像：docker pull 镜像名字

3.删除容器：docker rm -f 容器名字

4.查看docker状态：systemctl status docker

5.重启docker：systemctl restart docker

6.查看容器进程：docker ps

7.查看所有镜像：docker images

8.运行镜像：docker run ... ... ...(/bin/bash)

9.删除镜像：docker rmi -f +IMAGE ID，或者REPOSITORY+TAG

10.停止容器：docker stop 

11.进入容器：docker exec -it 91616117ced9 /bin/bash

12.退出容器：exit

13:docker帮助命令：

(1) 查看docker版本：
docker version

(2) 查看docker信息：
docker info

(3) docker帮助命令：
`docker --help`

14.更多命令：

    attach    Attach to a running container                 # 当前 shell 下 attach 连接指定运行镜像
    
    build     Build an image from a Dockerfile              # 通过 Dockerfile 定制镜像
    
    commit    Create a new image from a container changes   # 提交当前容器为新的镜像
    
    cp        Copy files/folders from the containers filesystem to the host path   #从容器中拷贝指定文件或者目录到宿主机中
    
    create    Create a new container                        # 创建一个新的容器，同 run，但不启动容器
    
    diff      Inspect changes on a container's filesystem   # 查看 docker 容器变化
    
    events    Get real time events from the server          # 从 docker 服务获取容器实时事件
    
    exec      Run a command in an existing container        # 在已存在的容器上运行命令
    
    export    Stream the contents of a container as a tar archive   # 导出容器的内容流作为一个 tar 归档文件[对应 import ]
    
    history   Show the history of an image                  # 展示一个镜像形成历史
    
    images    List images                                   # 列出系统当前镜像
    
    import    Create a new filesystem image from the contents of a tarball # 从tar包中的内容创建一个新的文件系统映像[对应export]
    
    info      Display system-wide information               # 显示系统相关信息
    
    inspect   Return low-level information on a container   # 查看容器详细信息
    
    kill      Kill a running container                      # kill 指定 docker 容器
    
    load      Load an image from a tar archive              # 从一个 tar 包中加载一个镜像[对应 save]
    
    login     Register or Login to the docker registry server    # 注册或者登陆一个 docker 源服务器
    
    logout    Log out from a Docker registry server          # 从当前 Docker registry 退出
    
    logs      Fetch the logs of a container                 # 输出当前容器日志信息
    
    port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT    # 查看映射端口对应的容器内部源端口
    
    pause     Pause all processes within a container        # 暂停容器
    
    ps        List containers                               # 列出容器列表
    
    pull      Pull an image or a repository from the docker registry server   # 从docker镜像源服务器拉取指定镜像或者库镜像
    
    push      Push an image or a repository to the docker registry server    # 推送指定镜像或者库镜像至docker源服务器
    
    restart   Restart a running container                   # 重启运行的容器
    
    rm        Remove one or more containers                 # 移除一个或者多个容器
    
    rmi       Remove one or more images       # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
    
    run       Run a command in a new container              # 创建一个新的容器并运行一个命令
    
    save      Save an image to a tar archive                # 保存一个镜像为一个 tar 包[对应 load]
    
    search    Search for an image on the Docker Hub         # 在 docker hub 中搜索镜像
    
    start     Start a stopped containers                    # 启动容器
    
    stop      Stop a running containers                     # 停止容器
    
    tag       Tag an image into a repository                # 给源中镜像打标签
    
    top       Lookup the running processes of a container   # 查看容器中运行的进程信息
    
    unpause   Unpause a paused container                    # 取消暂停容器
    
    version   Show the docker version information           # 查看 docker 版本号
    
    wait      Block until a container stops, then print its exit code   # 截取容器停止时的退出状态值

## 四.私服库

### 首次提交私服库过程

1.启动私库：docker run -d -p 5000:5000 -v /Log/myregistry/:/tmp/registry --privileged=true registry

2.提交生成镜像：docker commit -m="ifconfig cmd add" -a="Log" f7a200681c52 logubuntu:1.2，库名称不能大写

3.可以检查私服库有没有镜像：curl -XGET http://192.168.10.102:5000/v2/_catalog

4.修改为正确的tag格式：docker tag logubuntu:1.2 192.168.10.102:5000/logubuntu:1.2

5.docker默认不允许http方式推送镜像，通过配置来取消，修改后重启：sudo vim /etc/docker/
daemon.json,加个逗号，然后添加"insecure-registries"：["192.168.10.102:5000"]

6.推送： docker push 192.168.10.102:5000/logubuntu:1.2

7.检查：curl -XGET http://192.168.10.102:5000/v2/_catalog

8.拉下来：docker pull 192.168.10.102:5000/logubuntu:1.2

## 五.容器卷

作用：可以完成容器数据的`持久化`，重要资料的备份到本地，还可以实现容器间的、容器和主机间的`数据共享、传递`
1.挂载目录后面的时候要＋privileged=true参数

2.`docker run -it --privileged=true -v/宿主机绝对路径目录:/容器内目录 镜像名 `，可以实现两个目录的资源互通

3.查看已挂载：`docker inspect +容器id`，Mounts下为挂载路径

4.容器停止以后，如果主机在挂载目录中有修改，容器挂载目录中的信息依旧会同步

5.权限控制：`docker run -it --privileged=true -v/宿主机绝对路径目录:/容器内目录:ro 镜像名`,加了ro以后，容器内只能读，不能写

6.另一个容器继承一个容器的卷规则：`docker run -it --privileged=true --volumes-from 父类 --name u2 镜像`，实现多个容器和主机之间的数据共享，父类停不停不影响子类
