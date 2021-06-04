---
title: CodeIgniter 4 开发（一）
date: 2021-06-03 20:43:56
tags:
	- CodeIgniter
	- php
---
> 安装与hello,ci
## 安装应用
首先，在 [CodeIgniter](https://codeigniter.com/download) or [CodeIgniter 中国](https://codeigniter.org.cn/) 官网下载框架，解压到 ci 文件夹，进入该文件夹，打开命令行，输入下面命令：
```shell
$ php spark serve
```
完成之后，访问 [localhost:8080](http://localhost:8080/) 就能看到如下界面，这是 CodeIgniter 的默认界面。
![](1.1.png)
打开 ci 文件夹，就可以看到应用的目录结构了，至于各文件夹及文件的内容，请访问[应用结构](https://codeigniter.org.cn/user_guide/concepts/structure.html)查看详细的介绍。
## hello ci
将文件修改为：
app/Views/welcome_message.php
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Blog</title>
	<meta name="description" content="The small framework with powerful features">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link rel="shortcut icon" type="image/png" href="/favicon.ico"/>
</head>
<body>
<h1>hello, ci~</h1>
</body>
</html>
```
再次打开 [localhost:8080](http://localhost:8080/) ，就可以看到如下界面了。
![](1.2.png)
## git
应用根目录下运行
```shell
$ git init
$ git add -A
$ git commit -m "初始化"
```