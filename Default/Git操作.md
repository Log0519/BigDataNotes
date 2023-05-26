# git更新fork
@[toc]
## 1、clone fork的分支到本地
git clone git@github.com:*/flutter.git

## 2、增加源分支(被自己fork的项目)地址到自己git项目远程分支列表中，将原来的仓库命名为upstream，命令为：
git remote add upstream git@github.com:flutter/flutter.git

## 3、核实远程分支列表（optional）
git remote -v
如：origin git@github.com:*/flutter.git (fetch)
origin git@github.com:*/flutter.git (push)
upstream git@github.com:flutter/flutter.git (fetch)
upstream git@github.com:flutter/flutter.git (push)

核实后，发现不如意，想删除，可以用 git remote remove name
name 为远程分支的命名，如上面例子，可以删除 upstream
git remote remove upstream
或者直接删除之前fork的原始分支 origin
git remote remove origin (直接删除了原始分支后，再fork，
也能达到更新了最新代码的需求，后面的步骤就不用了。)

## 4、fetch源分支的新版本到本地
git fetch upstream

## 5、合并两个版本的代码
git merge upstream/master

## 6、将合并后的代码push到github上自己的fork中去
git push origin master


## 7、一些问题
LF will be replaced by CRLF the next time Git touches it，window（CRLF）和linux（LF）的换行符自动转换，代码换行符转换成当前系统的换行符

## 8、更多操作
文件在工作区中-红色，文件添加到暂存区后-绿色，修改后又变红

文件--git add 文件名-->暂存区（不想让文件保存历史版本，可以删掉）

删除暂存区文件 git rm --cached 文件名，工作区中的不会删除

暂存区文件--git commit -m “日志信息”文件名-->本地库（加-m“可以写日志信息”）

git log 详细历史信息

git使一行一行操作的，修改一行，实际上是删除这一行，再添加修改后的这一行

版本穿梭：git reset --hard 版本号

查看当前库别名，git-remote -v
起别名：git remote add 别名 命名的东西(网址)
git push git-dame(远程库地址别名) master(分支)
git pull git-dame master
git clone 网址
克隆不需要登录账号，加地址就可以
push需要登录网页账号
克隆会拉取代码、初始化本地库、可以看到克隆库里面的别名地址，自动叫origin