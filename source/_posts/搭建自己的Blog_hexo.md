title: "搭建自己的Blog:hexo"
date: 2015-04-16 16:36:30
tags: [Blog, 程序员初级修炼]
categories: Personal
description: 主要介绍了Hexo，以及如何利用Hexo在windows和Ubuntu系统下搭建环境,以及将本地blog托管在github/gitcafe上
---
# hexo简介
[Hexo](http://hexo.io/zh-cn/)可以将本地生成的静态网页托管到github或者gitcafe上，作者是台湾的[@tommy351](https://github.com/tommy351/hexo)
根据网上教程，目前看来常使用的命令只有：`hexo g`, `hexo s`, `hexo d`和`hexo new ...`，十分简单方便
## hexo管理blog的流程
### 创建新的文章
```ruby
hexo new "新文章名字"
```
利用[MarkdownPad](http://markdownpad.com/)在本地打开新建的文章，输入内容，并保存。
### 更新本地blog
```ruby
hexo g
```
### 本地查看blog
```ruby
hexo s
```
此时，在浏览器中输入网址[localhost:4000](http://127.0.0.1:4000/)，就可以查看最新的blog。
###在网上更新自己的blog
```ruby
hexo d
```
为了使自己的系统能顺利执行以上命令并发表blog，需要做一些准备工作。
***
#使用hexo的前期准备
##win8下搭建hexo环境
### 安装Git bash
下载 [git](https://code.google.com/p/msysgit/) 最新版，安装即可
### 安装Node.js
下载 [Node.js](http://nodejs.org/) 最新版，安装即可
### 安装hexo
在计算机任一位置点击鼠标右键，选择*Git bash*，再在打开的窗口中输入以下命令来安装hexo
```ruby
npm install -g hexo
```
安装成功后，再运行以下命令来更新hexo
```ruby
npm update -g hexo
```
到此为止已经搭建好hexo环境
##ubuntu下搭建hexo环境
### 安装git
```ruby
 sudo apt-get install git-core
```
### 安装Node.js
nvm的github主页https://github.com/creationix/nvm
使用以下两个命令中的一个就可以：
```ruby
curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```
或者
```ruby
wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```
等待nvm安装完成后重启终端后运行一下命令安装Node.js:
```ruby
#通过nvm ls查看nvm版本号。这里是0.10.32
nvm install 0.10.32
```
#### command not found
重启虚拟机后，使用`hexo`可能会遇见command not found的报错，此时使用`nvm ls`检查Node.js的版本，并使用以下命令运行刚刚安装好的nvm。
```ruby
nvm use 0.10.32
```
或者直接设置全局的默认Node.js版本
```ruby
nvm alias default 0.10.32
```
或者在/etc/rc.local文件中加入`nvm use 0.10.32`
### 安装hexo
输入以下命令来安装hexo
```ruby
npm install -g hexo
```
安装成功后，再运行以下命令来更新hexo
```ruby
npm update -g hexo
```
#在本地利用hexo搭建blog
##初始化本地blog
选择任意一个文件夹（以后所有的博客信息都在此文件夹中存储），在terminal中输入以下命令来初始化blog（**以后关于此blog的所有操作多需要在这个文件夹开启Git bash，输入命令完成**）。
```ruby
hexo init
```
hexo会自动在这个文件夹里建立网站所需要的所有文件，此时一个基本的框架已经搭建完毕
### 本地查看自己博客
输入以下命令可以在[本地](127.0.0.7:4000)看自己的blog，此时blog只在本地搭建完毕，其他人不能通过浏览器查看这个blog。
```ruby
hexo generate
hexo server
```
以后如果更新了blog,可通过以上两个命令在本地查看自己的blog。
其中`hexo generate`是blog的生成命令，每次更新自己的blog后必须执行此命令才能更新信息。
***
#本地Blog发布到网上
## Github
国内访问github速度太渣，这里用gitcafe，一个国内的代码托管网站。
### 注册
注册[github](https://github.com/),再次假设注册名在此假设`whoami`
### 进入账户设置
进入[SSH公钥管理](https://gitcafe.com/account/public_keys),添加主机公钥。产生公钥和配置公钥参考github下的[教程](https://help.github.com/articles/generating-ssh-keys)，几乎一致。再次贴出关键代码。
#### 产生一个新的SSH
```ruby
ssh-keygen -t rsa -C "注册的邮箱“
```
一直按回车后，WINDOWS中在`c:\user\用户名\.ssh`（隐藏文件夹）文件夹/LINUX下是~/.ssh的隐藏文件夹中找到新的两个文件:`id_rsa`和`id_rsa.pub`。`id_rsa.pub`利用记事本打开后就是本机公钥。复制到网站的添加公钥中。
####测试
可以输入下面的命令，看看设置是否成功，git@github.com的部分不要修改：
```ruby
ssh -T git@github.com
```
如果是下面的反馈：
```
The authenticity of host 'github.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
```
不要紧张，输入yes就好，然后会看到：
```
Hi cnfeat! You've successfully authenticated, but GitHub does not provide shell access.
```
#####设置用户信息
现在你已经可以通过SSH链接到GitHub了，还有一些个人信息需要完善的。
Git会根据用户的名字和邮箱来记录提交。GitHub也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的，名字必须是你的真名，而不是GitHub的昵称。
```ruby
git config --global user.name "cnfeat"//用户名
git config --global user.email  "cnfeat@gmail.com"//填写自己的邮箱
```
### 网站上新建项目
取名一定为`whoami.github.io`，否则**无法托管**成功。
项目新建成功后，进入项目管理，点击顶部的抓取，获取SSH的url
### 回到本地blog
在本地blog的主目录下打开`_config.yml`，拖到文件尾部，参考一下命令配置
```ruby
deploy:
  type: git #gitcafe托管也是github
  repository: #刚才复制的url，前面记得空一格，格式类似git@gitcafe.com:whoami/whoami.github.iogit
  branch: master #git项目的分支
```
### 最后部署
在git bash命令行中输入`hexo deploy`即可秒速布置成功。最后就可以访问主页`whoami.gitcafe.com`
### 注意
每次更新blog后，都要先`hexo generate`再`hexo deploy`，否则无法看到最新成果
## 购买域名
参考[zipperary的blog：如何绑定二级域名](http://zipperary.com/2013/06/19/secondary-dns/)，先留着，以后自己看
***
#hexo的优化
##添加评论框
静态博客要使用第三方评论系统，hexo默认集成的是Disqus，国内的话还是建议用多说。直接用你的微博/豆瓣/人人/百度/开心网帐号登录多说，做一下基本设置。如果你是有的其他第三方评论系统，将通用代码粘贴到`themes\主题\layout\_partial\comment.ejs`里面，如下：
```ruby
<% if (config.disqus_shortname && page.comments){ %>
<section id="comment">
  #你的通用代码
<% } %>
```
###其他主题添加评论框
如果使用非官方默认的其他主题，具体参考其README.MD来更改评论框配置，以上仅供参考。
##更换主题
在本地blog的主目录下打开`_config.yml`，在`theme: `后添加自己新下载的主题。
###其他bug（新版的hexo已修正）
首先 `hexo g` 本地服务器theme是新的，deploy到 Github 样式乱七八糟的
上传前要 `rm db.json`
###landscape主题
#### 最新的landscape中，已经没有comment.ejs，这个评论功能合并到了同目录下的`article.ejs`
找到`<section id="comments">`，更改上一行的代码为：
```ruby
<% if (!index && post.comments){ %> 
```
之后再里面添加多说代码：
```ruby
<section id="comments">
  <!--多说代码-->
</section>
```
#### 引用的格式修改
今天发现landscape的引用时居中、斜体，中文不太美观，可以在`themes/landscape/source/css/_partial/article.styl`将引用`blockquote`的格式设置更改为左对齐等，并修改字体颜色，参考如下代码
```json
font-size: 0.9em
margin: line-height 0
text-align: left
color:#838B8B
```
###其他主题
####TKL主题
十分漂亮的主题，git：https://github.com/askingwindy/TKL，以及博主的博客：http://go.kieran.top/
####NEXT主题
本blog采用的主题，git：https://github.com/askingwindy/hexo-theme-next
***
#Hexo中的特殊格式
## 写文章
利用hexo自带命令，可以再`sourec/_posts`里按照模板生成一个`.md`的markdown文件，编辑此文件再保存就可以形成一篇漂亮的文档了~
```ruby
hexo new "新文章"
```
生成的新文章中，默认格式如下：

```ruby
title: { { title } }
date: { { date } }
tags: #多标签用格式[tag1,tag2,tag3]
---
```
在hexo中，每个`:`后都必须空一格，否在在`hexo generate`时会报错
### 自定义模板
如果想在自己的文章中增加分类，需要在文章首部添加`categories:`，添加后的格式为：
```ruby
title: { { title } }
date: { { date } }
tags: #空一格写标签
categories: #空一格写分类
---
```
为了避免每次添加，可以再`scaffolds/post.md`添加一行`categories:`。`scaffolds`文件夹中其余的`.md`文件机理相同。
### 插入图片
使用markdown写文章，插入图片的格式为`![图片名称](链接地址)`。
对于Hexo，可以支持两种方式的链接地址：
1. 使用本地路径：在`hexo/sourc`e目录下新建一个img文件夹，将图片放入该文件夹下，插入图片时链接即为`/img/图片名称`。
2. 使用图床，将图片拖入图床中，会生成图片的URL，这就是链接地址。*（网上推荐的有七牛、dropbox）*

### 插入超链接
与插入图片类似，格式为`[链接名字](链接地址)`
### 文章摘要
书写完正文后，页面会显示整篇文章，如果太长，可以再*摘要*(这个摘要自定义...)下添加`<!--more-->`，这样首页显示的只有题目和摘要，查看全文需要点击进去。
```ruby
#显示的摘要
<!--more-->
#余下正文
```
### fancybox
在文章`.md`的首部添加`photos:`后，可以有意想不到的效果~添加后的格式为：
```ruby
title: { { title } }
date: { { date } }
tags: #空一格写标签
categories: #空一格写分类
photos:
- http:// #图片网址
- http:// #图片网址
```
***
# 参考资料：
1. [Zipperary的hexo系列教程](http://zipperary.com/categories/hexo/)
2. [不如](http://ibruce.info/2013/11/22/hexo-your-blog/ )
3. [tommy：Hexo 颯爽登場](http://zespia.tw/blog/2012/10/11/hexo-debut/)
4. [tommy351的github](https://github.com/tommy351/hexo)