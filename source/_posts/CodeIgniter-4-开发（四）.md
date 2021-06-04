---
title: CodeIgniter 4 开发（四）
date: 2021-06-04 14:10:37
tags:
	- CodeIgniter
	- php
---
> 用户登录功能
## 登录
### 路由
app/Config/Routes.php
```
  $routes->post('register', 'Auth\RegisterController::attemptRegister', ['as' => 'register']);

  // login
+  $routes->get('login', 'Auth\LoginController::login', ['as' => 'login']);
+  $routes->post('login', 'Auth\LoginController::attemptLogin', ['as' => 'login']);

```
### model
app/Models/UserModel.php
```
protected $dynamicRules = [
 'register' => [
    ...
  ],
  'login' => [
    'email' => [
      'label' => 'Email',
      'rules' => 'required|valid_email',
    ],
    'password' => [
      'label' => '密码',
      'rules' => 'required|min_length[5]',
    ],
  ],
];
```
### controller
```
$ php spark make:controller Auth/LoginController
```
app/Controllers/Auth/LoginController.php
```
protected $session;

public function __construct()
{
	$this->session = Services::session();
}

public function login()
{
	return view('auth/login');
}

public function attemptLogin()
{
  $userModel = new UserModel();
  $rules = $userModel->getRules('login');

  if (!$this->validate($rules))
  {
    return redirect()->back()->withInput()->with('errors', $this->validator->getErrors());
  }

  $user = $userModel->where('email', $this->request->getVar('email'))
                    ->first();

  if (is_null($user) || ( !password_verify($this->request->getVar('password'), $user['password'])))
  {
    return redirect()->back()->withInput()->with('error', '邮箱或者密码错误');
  }

  $this->session->set('isLoggedIn', true);
  $this->session->set('blog_user', [
    'id' => $user['id'],
    'name' => $user['name'],
    'email' => $user['email']
  ]);

  return redirect()->to('/')->with('success', '登录成功，欢迎回来~');
}
```
### 页面
app/Views/auth/login.php
```
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="main d-flex justify-content-center align-items-center h-100">
  <div class="col-sm-10 col-md-5 col-lg-4">
    <div class="card">
      <h2 class="card-header text-center">
        登录
      </h2>
      <div class="card-body">

        <?= view('App\Views\components\messages') ?>

        <form action="<?= route_to('login') ?>" method="post">
          <?= csrf_field() ?>
          <div class="form-group">
            <label for="email">Email</label>
            <input type="email" name="email" class="form-control" value="<?= old('email') ?>">
          </div>
          <div class="form-group">
            <label for="password">密码</label>
            <input type="password" name="password" class="form-control">
          </div>

          <div class="form-group">
            <button type="submit" class="btn form-control theme-bg text-white">登录</button>
          </div>

          <a class="w-100 d-block text-center text-muted" href="#"><small>忘记密码？</small></a>
        </form>
      </div>
    </div>
  </div>
</div>

<?= $this->endSection() ?>
```
app/View/layouts/navbar.php
```
<ul class="navbar-nav mr-auto">
  <li class="nav-item">
    <a class="nav-link text-white" href="/">首页</a>
  </li>
</ul>

<?php if (session('isLoggedIn')) : ?>
  <ul class="navbar-nav flex-row ml-md-auto d-none d-md-flex">
    <li class="nav-item dropdown">
      <a class="nav-item nav-link dropdown-toggle mr-md-2 text-white" href="#" id="bd-versions" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
        <span class="text-capitalize">
          <?= session('blog_user')['name'] ?>
        </span>
      </a>
      <div class="dropdown-menu dropdown-menu-right" aria-labelledby="bd-versions">
        <a class="dropdown-item" href="<?= route_to('profile') ?>">个人中心</a>
        <div class="dropdown-divider"></div>
        <a class="dropdown-item" href="<?= route_to('logout') ?>">退出</a>
      </div>
    </li>
  </ul>
<?php else : ?>
  <ul class="navbar-nav flex-row ml-md-auto d-none d-md-flex">
    <li class="nav-item">
      <a class="nav-link text-white" href="<?= route_to('login') ?>">
        登录
      </a>
    </li>
    <li class="nav-item">
      <a class="nav-link text-white" href="<?= route_to('register') ?>">
        注册
      </a>
    </li>
  </ul>
<?php endif; ?>
```
![](4.1.png)
![](4.2.png)

## 登出
### 路由
app/Config/Routes.php
```
  $routes->post('login', 'Auth\LoginController::attemptLogin', ['as' => 'login']);

  // logout
+  $routes->get('logout', 'Auth\LoginController::logout', ['as' => 'logout']);
```
### controller
app/Controllers/Auth/LoginController.php
```
public function logout()
{
  $this->session->remove(['isLoggedIn', 'blog_user']);

  return redirect()->route('login');
}
```
## 细节完善
用户登录之后不能进行注册和登录操作
app/Controllers/Auth/RegisterController.php
```
+protected $session;

+public function __construct()
+{
+  $this->session = Services::session();
+}

public function register()
{
+  if ($this->session->isLoggedIn)
+  {
+    return redirect()->to('/')->with('info', '您已经登录了，无需注册~');
+  }

	return view('auth/register');
}
```
app/Controllers/Auth/LoginController.php
```
public function login()
{
+  if ($this->session->isLoggedIn)
+  {
+    return redirect()->to('/')->with('info', '您已经登录了~');
+  }

  return view('auth/login');
}
```
app/Views/components/messages.php
```
<?php if (session()->has('info')) : ?>
  <div class="alert alert-info w-100" role="alert">
    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
      <span aria-hidden="true">&times;</span>
    </button>

    <p><?= session('info') ?></p>
  </div>
<?php endif; ?>
```
访问 login 和 register，会自动跳转到首页。
![](4.3.png)
![](4.4.png)

## 提交
```
$ git add -A
$ git commit -m "用户登录"
```