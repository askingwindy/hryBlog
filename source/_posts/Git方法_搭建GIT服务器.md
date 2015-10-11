title: 搭建GIT服务器
date: 2015-03-1 09:07:15
categories: Git使用教程
tags: [Git]
description: 如何在本地搭建自己的GIT服务器，来分布式协作代码;以及搭建完成后，客户端如何使用
---
#搭建GIT服务器
假设你已经有sudo权限的用户账号，下面，正式开始安装。
##第一步，安装git：
```
$ sudo apt-get install git
```
##第二步，创建一个git用户，用来运行git服务：
```
$ sudo adduser git
```
##第三步，创建证书登录：
收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到`/home/git/.ssh/authorized_keys`文件里，一行一个（没有.ssh目录需要自己创建）。
##第四步，初始化Git仓库：
先选定一个目录作为Git仓库，假定是/srv/sample.git，在/srv目录下输入命令：
```
$ sudo git init --bare sample.git
```
Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：
```
$ sudo chown -R git:git sample.git
```
##第五步，禁用shell登录：
出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：
```
git:x:1001:1001:,,,:/home/git:/bin/bash
```
改为：
```
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```
这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。
##第六步，克隆远程仓库：
现在，可以通过git clone命令克隆远程仓库了，在各自的电脑上运行：
```
$ git clone git@server:/srv/sample.git
Cloning into 'sample'...
warning: You appear to have cloned an empty repository.
```
剩下的推送就简单了。
***
#客户端如何使用GIT服务器
##第一步：创建自己的公钥，并提交给服务器
将服务器保存客户端的公钥到`/home/git/.ssh/authorized_keys`文件
###公钥是什么
产生公钥：
公钥在`id_rsa.pub`里面，大概如下：
##第二步：与远程仓库连接
###如果没有本地仓库
```
git clone git@10.108.50.122:/origin/origin.git
```
###如果有本地仓库
```
git remote add origin git@10.108.50.122:/origin/origin.git
```
***
#常用的命令行
##添加了新代码，需要提交
```
git add -A
git commit -m "修改了哪些地方"
git push origin master（可修改，提交的分支改变）//创建+切换分支：git checkout -b 分支名称
```
##代码修改错误，需要回溯版本
```
git log (查看提交了的版本号)
git reset --hard 版本号
```
如果想回到未来，使用命令`git reflog`查看历史版本
##同步版本号
###查看远程库信息
```
git remote -v
```
###多人协作
1. 试图用`git push origin branch-name`推送自己的修改
2. 如果推送失败，因为远程分支比本地分支还要新，首先需要`git pull`进行合并
3. 如果合并有冲突，解决冲突，并在本地提交
4. 重复步骤1

#### no tracking information
如果`git pull`时出现这个错误，说明本地分支和远程分支的连接关系没有创捷，先通过命令：
```
git branch --set-upstream branch-name origin/branch-name
```
#参考网站
[廖学锋搭建git](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013744142037508cf42e51debf49668810645e02887691000)