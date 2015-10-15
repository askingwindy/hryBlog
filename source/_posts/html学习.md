title: Html学习
date: 2015-10-05 14:21:15
categories: 学习：前端
tags: [程序员初级修炼,前端学习,html]
description: 如何将本地文件夹初始化为一个Git仓库，并进行版本回退等管理
---
#Html标签

```html
<!DOCTYPE html>
<html lang="zh-hans">
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
		<meta name="description" content="这是说明的文字" />
	</head>
	<body>
		<div id="sidebar" style-"width:300px; float; left;">
			<img src="http://askingwindy-gitcafe.qiniudn.com/touxiang.png"/> <!--插入图片 -->
			<h2>同步</h2> <!--二级标题-->
			<p>这是一个段落</p>
		</div>

		<p>这是有span标签的段落：<span style="color:#00C;font-weight:bold">一个</span>段落</p>
	</body>
</html>


```

###`meta`
```html
<meta name="description" content="这是在搜索时显示的文字"/>
```

###`div`
`div`标签中的CSS样式是独立的

####定义id

```html
<div id="sidebar" style-"width:300px; float; left;"></div>
```

###`a`
`a`标签定义了链接

```html
<a href="http://google.com">google</a>
```

#### 新窗口打开链接
强制在新窗口 
```html
<a href="http://google.com" target="_blank">google</a>
```

###`span`
`span`标签定义了行内元素
```html
<p>这是<span style="color:#00C;font-weight:bold">一个</span>段落</p>
```