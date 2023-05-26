conda 环境管理常用命令
创建环境：conda create -n env_name python=3.7
查看所有环境：conda info --envs
删除一个环境：conda remove -n env_name --all
3）激活 superset 环境
(base) [atguigu@hadoop102 ~]$ conda activate superset
4）说明：退出当前环境
(superset) [atguigu@hadoop102 ~]$ conda deactivate