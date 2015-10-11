title: Git说明与使用教程
date: 2014-05-5 09:07:15
categories: Git使用教程
tags: [Git]
description: 如何将本地文件夹初始化为一个Git仓库，并进行版本回退等管理
---

#初始化git库
初始化一个Git仓库，使用`git init`命令。
## 添加文件到Git仓库
编写文件时，文件在工作区
###第一步，使用命令git add 
注意，可反复多次使用，添加多个文件；
此时存在了暂存区，还没提交
###第二步，使用命令git commit
完成
```
$ git add file1.txt
$ git add file2.txt
$ git add file3.txt
$ git commit -m "add 3 files."
```
## 查看历史记录
###由最近到最远提交的日志信息
```
git log
```
###记录本地的每一个命令
```
git reflog
```
## 查看本地库状态
```
git status
```
## 回退版本
### 回退到上一版本
```
git reset --hard HEAD^
```
### 回退到任一版本
```
git reset --hard commit_id
```
### 撤销修改
#### 场景1
当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

#### 场景2
当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作
***

# 添加远程仓库
现在的情景是，你已经在本地创建了一个Git仓库后，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的仓库可以作为备份。
##本地Git仓库与Github/gitcafe仓库见通信是SSH加密
###创建SSH Key。
在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：
```
$ ssh-keygen -t rsa -C "youremail@example.com"
```
再将密钥添加至自己账户中，详见[github添加密钥](https://help.github.com/articles/generating-ssh-keys)
###在本地创建版本库
#### 在托管的github/gitcafe上新建一个仓库后
在GitHub上的这个learngit仓库还是空的，GitHub告诉我们，可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库。
##### 1.在本地新建一个文件夹，然后通过命令`git init`将其变为Git可以管理的仓库
##### 2.使用以下命令将文件添加进版本库
把文件添加到仓库：
```
git add xxxxx.txt
```
用命令git commit告诉Git，把文件提交到仓库：
```
$ git commit -m "添加自己的comments"
```
##### 3.将远程仓库与本地仓库相关联的
```
$ git remote add origin git@github.com:xxxxx/xxxxxx.git
```
添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
##### 4.把本地库的所有内容推送到远程库上：
```
$ git push -u origin master
```
把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支master推送到远程。

由于远程库是空的，我们第一次推送master分支时，加上了`-u`参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。
## git push出错
### solution:1 强推
即利用强覆盖方式用你本地的代码替代git仓库内的内容
```
git push -f
```
###solution:2 先git pull，再git push
[先把git的东西fetch到你本地然后merge后再push](http://blog.csdn.net/chain2012/article/details/7476493)
[branch "master"]是需要明确(.git/config)如下的内容
[branch "master"]
    remote = origin
    merge = refs/heads/master
这等于告诉git2件事:
1，当你处于master branch, 默认的remote就是origin。
2，当你在master branch上使用git pull时，没有指定remote和branch，那么git就会采用默认的remote（也就是origin）来merge在master branch上所有的改变
如果不想或者不会编辑config文件的话，可以在bush上输入如下命令行
```
$ git config branch.master.remote origin  
$ git config branch.master.merge refs/heads/master  
```
之后再重新git pull下。最后git push你的代码吧。
***
# 更新本地仓库
```
git pull origin master
```