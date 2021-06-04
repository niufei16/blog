---
title: CodeIgniter 4 开发（二）
date: 2021-06-03 21:05:03
tags:
	- CodeIgniter
	- php
---
> 一些设置、下载用到的库和基础页面
## 基本设置
### 代码风格
新建 .editorconfig
```
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 4
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.{yml,yaml}]
indent_size = 2

[*.{js,html,php,css}]
indent_style = space
indent_size = 2
```
### APP key
```
$ php spark key:generate
```
在应用根目录下生成了 .env 文件。
```
# 开发环境
CI_ENVIRONMENT = development
...
# 数据库设置
database.default.hostname = localhost
database.default.database = ci
database.default.username = root
database.default.password =
database.default.DBDriver = MySQLi
database.default.DBPrefix =
```
访问 [localhost:8080](http://localhost:8080/) 发现右下角出现 debug 小图标，点击查看如下图详情
![](2.1.png)
### 语言
app/Config/App.php
```
public $defaultLocale = 'zh_CN';
```
下载[语言文件](https://github.com/codeigniter4/translations)，将对应的文件夹复制到 app/Language 里。
```
$ tree app/Language
├─Language
│  │  ├─en
│  │  └─zh-CN
```
### Timezone
app/Config/App.php
```
public $appTimezone = 'Asia/Shanghai';
```
## 页面布局
### 下载库
- [Bootstrap 4](https://v4.bootcss.com/docs/getting-started/download/)
- [jQuery](https://jquery.com/download/)
- [Font Awesome](https://fa5.dashgame.com/common/fontawesome-free-5.11.2-web.zip)

在文件夹 public 下新建文件夹 vendor，将三个库复制进去。
```
$ tree public
│─vendor
│   ├─bootstrap
│   ├─fontawesome-free
│   |─jquery
```
测试一下
app/Views/welcome_message.php
```
+ <link rel="stylesheet" href="<?= base_url('vendor/bootstrap/css/bootstrap.min.css') ?>">
+ <link rel="stylesheet" href="<?= base_url('vendor/fontawesome-free/css/all.css') ?>">
...
+ <div class="container">
+   <div class="card">
+     <div class="card-body">
+       <i class="fa fa-ad"></i>
+       test
+     </div>
+   </div>
+ </div>
```
访问 localhost:8080
![](2.2.png)
一切顺利。
### 页面
在 app/Views 下新建文件夹 auth、components、layouts
```
├─auth
├─components
├─errors
└─layouts
```
在 app/Views/layouts 下新建文件 app.php 
app/Views/layouts/app.php
```
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <meta name="description" content="">
  <meta name="author" content="">
  <meta name="generator" content="">
  <title>Blog</title>

  <!-- load extended styles -->
  <?= $this->renderSection('style') ?>

  <!-- default styles  -->
  <link rel="stylesheet" href="<?= base_url('vendor/fontawesome-free/css/fontawesome.css'); ?>">
  <link rel="stylesheet" href="<?= base_url('vendor/fontawesome-free/css/solid.css'); ?>">
  <link rel="stylesheet" href="<?= base_url('vendor/fontawesome-free/css/brands.css'); ?>">
  <link rel="stylesheet" href="<?= base_url('vendor/bootstrap/css/bootstrap.css'); ?>">
  <link rel="stylesheet" href="<?= base_url('css/app.css'); ?>">
</head>

<body class="bg-light">

<!-- navbar -->
<?= view('App\Views\layouts\navbar') ?>

<main role="main" class="container main">
  <!-- load content from other views -->
  <?= $this->renderSection('main') ?>
</main>

<!-- footer -->
<?= view('App\Views\layouts\footer') ?>

<script src="<?= base_url("vendor/jquery/jquery.js") ?>" type="text/javascript"></script>
<script src="<?= base_url("vendor/bootstrap/js/bootstrap.bundle.min.js") ?>" type="text/javascript"></script>

<!-- load extended scripts -->
<?= $this->renderSection('script') ?>
</body>
</html>
```
在 app/Views/layouts 下新建文件 footer.php 
app/Views/layouts/footer.php
```
<footer>
  <ul class="nav justify-content-center bg-white py-5">
    <li class="nav-item">
      <a class="nav-link theme-text" href="#">关于我</a>
    </li>
    <li class="nav-item">
      <a class="nav-link theme-text" href="#">帮助</a>
    </li>
    <li class="nav-item">
      <a class="nav-link theme-text" href="#">联系我</a>
    </li>
  </ul>
  <div class="w-100 my-2">
    <p class="text-center">Copyright &copy; <?= date('Y') ?> Blog All Rights Reserved. </p>
  </div>
</footer>
```
在 app/Views/layouts 下新建文件 navbar.php 
app/Views/layouts/navbar.php，
```
<nav class="navbar navbar-expand-md theme-bg">
  <div class="container">
    <a class="navbar-brand logo-brand" href="/">
      <img height="44" title="blog Logo" alt="blog" src="<?= base_url('logo.png') ?>">
    </a>
    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarsExampleDefault" aria-controls="navbarsExampleDefault" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarsExampleDefault">
      <ul class="navbar-nav mr-auto">
        <li class="nav-item">
          <a class="nav-link text-white" href="/">首页</a>
        </li>
      </ul>

      <ul class="navbar-nav flex-row ml-md-auto d-none d-md-flex">
        <li class="nav-item dropdown">
          <a class="nav-item nav-link dropdown-toggle mr-md-2 text-white" href="#" id="bd-versions" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
            <span class="text-capitalize">admin</span>
          </a>
          <div class="dropdown-menu dropdown-menu-right" aria-labelledby="bd-versions">
            <a class="dropdown-item" href="<?= site_url('profile') ?>">个人中心</a>
            <div class="dropdown-divider"></div>
            <a class="dropdown-item" href="<?= site_url('logout') ?>">退出</a>
          </div>
        </li>
      </ul>
    </div>
  </div>
</nav>
```
删除 app/Views/welcome_message.php，新建 app/Views/home.php
```
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<h1>首页</h1>
<?= $this->endSection() ?>
```
至此，页面布局基本完成。
![](2.3.png)

## 提交
```
$ git add -A
$ git commit -m "设置 && 页面布局"
```
