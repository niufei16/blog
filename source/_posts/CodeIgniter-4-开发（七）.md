---
title: CodeIgniter 4 开发（七）
date: 2021-06-06 01:18:13
tags:
    - CodeIgniter
    - php
---
> 用户个人页面和用户设置，设置里包括密码修改、头像上传、个人资料修改、重发Email验证
## 个人页面
### userModel
app/Models/UserModel.php
```php
protected $allowedFields        = [
    'name',
    'email',
    'password',
    'reset_hash',
    'reset_expires',
    'email_verified_at',
    'email_verified_hash',
+    'avatar',
+    'introduction',
];
...
+protected $afterFind            = ['userSetting'];
...
+protected function userSetting(array $data)
+{
+
+//    dd(is_null($data['data']['avatar']));
+    if (is_null($data['data']['avatar']))
+    {
+      $data['data']['avatar'] = 'avatar.png';
+    }
+
+    if (is_null($data['data']['introduction'])) {
+      $data['data']['introduction'] = '暂无简介';
+    }
+
+    if (! is_null($data['data']['email_verified_at'])) {
+      $time = Time::parse($data['data']['email_verified_at']);
+      $timeNow = Time::parse(date('Y-m-d H:i:s'));
+      if ($time->isBefore($timeNow))
+      {
+        $data['data']['email_verified'] = true ;
+      }
+    }
+
+    $data['data']['avatar'] = "image/" . $data['data']['avatar'];
+
+    return $data;
+}
```
### database

app/Databse/Migrations/xxx_Users.php

```php
'reset_expires' => [
    'type'           => 'BIGINT',
    'constraint'     => 20,
    'null'           => true,
],
+'avatar' => [
+    'type'           => 'VARCHAR',
+    'constraint'     => 255,
+    'null'           => true,
+],
+'introduction' => [
+    'type'           => 'VARCHAR',
+    'constraint'     => 255,
+    'null'           => true,
+],
```

### 路由

app/Config/Routes.php

```php
  // users
+  $routes->get('users/(\d+)', 'Auth\UsersController::show/$1', ['as' => 'users.show']);
```

### controller

创建 Auth/UsersController

```
$ php spark make:controller Auth/UsersController
```

app/Controllers/Auth/UsersController

```php
protected $userModel;

protected $session;

public function __construct()
{
    $this->userModel = new UserModel();
    $this->session = Services::session();
}

public function show($id)
{
    $user = $this->userModel->find($id);

    if (is_null($user))
    {
        return $this->notFoundUser();
    }

    return view('users/show', ['user' => $user]);
}
```

app/Controllers/BaseController

```php
protected function notFoundUser()
{
	return redirect()->to('/')->with('error', '没有发现该用户或者您的权限不足~');
}
```

### 页面

app/Views/users/show

```php
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="container main mt-4">
  <div class="row">
    <div class="col-md-4">
      <div class="card">
        <div class="card-body text-center">
          <img src="<?= base_url($user['avatar']) ?>" alt="avatar" width="60">
          <h3 class="card-title">
            <?= $user['name'] ?>
          </h3>
          <span class="text-muted">
            <?= $user['email'] ?>
          </span>
          <p class="card-text">
            <?= $user['introduction'] ?>
          </p>

          <a href="<?= route_to('users.edit', $user['id']) ?>" class="btn theme-bg text-white">
            <i class="fa fa-user-cog mr-2"></i>
            个人设置
          </a>
        </div>
      </div>
    </div>
    <div class="col-md-8">
      <div class="card">
        <?= view('App\Views\components\messages') ?>
    
        <h3>个人页面</h3>
      </div>
    </div>
  </div>
</div>
<?= $this->endSection() ?>
```

![](7.1.png)



## 用户设置

### 路由

app/Config/Routes.php

```php
  // users
  $routes->get('users/(\d+)', 'Auth\UsersController::show/$1', ['as' => 'users.show']);
    
+  $routes->get('users/(\d+)/edit', 'Auth\UsersController::edit/$1', ['as' => 'users.edit']);
+  $routes->post('users/(\d+)', 'Auth\UsersController::update/$1', ['as' => 'users.update']);
```

### userModel

app/Models/UserModel.php

```php
'reset-password' => [
    'password' => [
        'label' => '新密码',
        'rules' => 'required|min_length[5]',
    ],
    'password_confirm' => [
        'label' => '确认新密码',
        'rules' => 'matches[password]',
    ],
    'token' => 'required',
],
+'user-update' => [
+    'name' => [
+        'label' => '名称',
+        'rules' => 'required|min_length[2]',
+    ],
+    'email' => [
+        'label' => 'Email',
+        'rules' => 'required|valid_email|is_not_unique[users.email,id,{id}]',
+        'errors' => [
+            'is_not_unique' => 'Email 错误',
+        ],
+    ],
+    'introduction' => [
+        'label' => '简介',
+        'rules' => 'max_length[255]',
+    ],
+],
```

### controller

app/Controller/Auth/UsersController.php

```php
public function edit($id)
{
    $user = $this->userModel->find($id);

    if (is_null($user))
    {
        return $this->notFoundUser();
    }

    return view('users/edit', ['user' => $user]);
}

public function update($id)
{
    $rules = $this->userModel->getRules('user-update');
    if (!$this->validate($rules))
    {
        return redirect()->back()->with('errors', $this->validator->getErrors());
    }

    $user = $this->userModel;
    $user = [
        'name' => $this->request->getPost('name'),
        'introduction' => $this->request->getPost('introduction'),
    ];

    if (! $this->userModel->update($id, $user))
    {
        return redirect()->back()->with('errors', $this->userModel->errors());
    }

    // 更新 session 中的用户信息
    $this->session->set('blog_user', [
        'id' => $id,
        'name' => $user['name'],
        'email' => $this->request->getPost('email'),
    ]);

    return redirect()->back()->with('success', '更新成功~');
}
```

### 页面

app/Views/users/show

```php
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="container main mt-4">
  <div class="row">
    <div class="col-md-4">
      <?= view('App\Views\layouts\_user_sidebar') ?>
    </div>
    <div class="col-md-8">
      <div class="card">
        <h2 class="card-header bg-white text-center">
          个人资料设置
        </h2>

        <div class="card-body">
          <?= view('App\Views\components\messages') ?>

          <form action="<?= route_to('users.update', $user['id']) ?>" method="post">
            <?= csrf_field() ?>
            <div class="form-group">
              <label for="name">名称</label>
              <input type="text" name="name" class="form-control" value="<?= $user['name'] ?>">
            </div>
            <div class="form-group">
              <label for="email">Email</label>
              <input type="email" name="email" class="form-control" value="<?= $user['email'] ?>" readonly>
            </div>
            <div class="form-group">
              <label for="introduction">简介</label>
              <textarea name="introduction" cols="30" rows="10" class="form-control">
                <?= $user['introduction'] ?>
              </textarea>
            </div>

            <div class="form-group">
              <button type="submit" class="btn form-control theme-bg text-white">更新</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>
<?= $this->endSection() ?>
```

app/Views/layouts/_user_sidebar.php

```php
<div class="card">
  <h4 class="card-header">
    <img src="<?= base_url($user['avatar']) ?>" class="rounded-circle mr-2" width="50" alt="avatar" >
    <?= $user['name'] ?>
  </h4>
  <div class="card-body">
    <div class="card border-0">
      <h5 class="card-header bg-white">
        消息中心
      </h5>
      <div class="card-body">
        <nav class="nav flex-column">
          <li>
            <a href="" class="nav-link text-black-50">
              <i class="fa fa-bell mr-2"></i>
              通知信息
            </a>
          </li>
        </nav>
      </div>
    </div>

    <div class="card border-0">
      <h5 class="card-header bg-white">
        设置
      </h5>
      <div class="card-body">
        <nav class="nav flex-column">
          <li>
            <a href="<?= route_to('users.email-verified', $user['id']) ?>" class="nav-link text-black-50">
              <i class="fa fa-envelope mr-2"></i>
              Email 验证
            </a>
          </li>
          <li>
            <a href="<?= route_to('users.password-setting', $user['id']) ?>" class="nav-link text-black-50">
              <i class="fa fa-user-lock mr-2"></i>
              密码重置
            </a>
          </li>
          <li>
            <a href="<?= route_to('users.edit', $user['id']) ?>" class="nav-link text-black-50">
              <i class="fa fa-address-card mr-2"></i>
              个人资料
            </a>
          </li>
          <li>
            <a href="<?= route_to('users.avatar', $user['id']) ?>" class="nav-link text-black-50">
              <i class="fa fa-image mr-2"></i>
              头像
            </a>
          </li>
        </nav>
      </div>
    </div>
  </div>
</div>
```

app/Views/layouts/navbar.php

```php
<a class="dropdown-item" href="<?= route_to('users.show', session('blog_user')['id']) ?>">
    <i class="fa fa-user mr-2"></i>
    个人中心
</a>
<a class="dropdown-item" href="<?= route_to('users.edit', session('blog_user')['id']) ?>">
    <i class="fa fa-user-cog mr-2"></i>
    设置
</a>
<div class="dropdown-divider"></div>
<a class="dropdown-item" href="<?= route_to('logout') ?>">
    <i class="fa fa-user-slash mr-2"></i>
    退出
</a>
```

![](7.2.png)

![](7.3.png)



## 密码重置

### 路由

app/Config/Routes.php

```php
$routes->get('users/(\d+)/edit', 'Auth\UsersController::edit/$1', ['as' => 'users.edit']);
$routes->post('users/(\d+)', 'Auth\UsersController::update/$1', ['as' => 'users.update']);

+$routes->get('users/(\d+)/password-setting', 'Auth\UsersController::passwordSetting/$1', ['as' => 'users.password-setting']);
+$routes->post('users/(\d+)/password-setting', 'Auth\UsersController::passwordSettingUpdate/$1', ['as' => 'users.password-setting']);
```

### controller

app/Controllers/Auth/UsersController.php

```php
public function passwordSetting($id)
{
    $user = $this->userModel->find($id);

    if (is_null($user))
    {
        return $this->notFoundUser();
    }

    $user['password'] = '******';

    return view('users/password-setting', ['user' => $user]);
}

public function passwordSettingUpdate($id)
{
    $user = $this->userModel->find($id);

    if (is_null($user))
    {
        return $this->notFoundUser();
    }

    // 发送密码重置邮件
    helper('text');
    $updateUser['reset_hash'] = random_string('alnum', 32);
    $updateUser['reset_expires'] = time() + HOUR;

    $this->userModel->update($user['id'], $updateUser);

    helper('authEmail');
    send_password_reset_email($user['email'], $updateUser['reset_hash']);

    return redirect()->back()->with('success', '密码修改邮件已发送~');
}
```

### 页面

app/Views/users/password-setting.php

```php
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="container main mt-4">
  <div class="row">
    <div class="col-md-4">
      <?= view('App\Views\layouts\_user_sidebar') ?>
    </div>
    <div class="col-md-8">
      <div class="card">
        <h2 class="card-header bg-white text-center">
          密码设置
        </h2>

        <div class="card-body">
          <?= view('App\Views\components\messages') ?>

          <form action="<?= route_to('users.password-setting', $user['id']) ?>" method="post">
            <?= csrf_field() ?>
            <div class="form-group">
              <label for="password">密码</label>
              <div class="form-inline">
                <input type="password" name="password" class="form-control" value="<?= $user['password'] ?>">
                <button type="submit" class="btn ml-2 bg-light">修改密码</button>
              </div>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>
<?= $this->endSection() ?>
```

![](7.4.png)

![](7.5.png)

![](7.6.png)



## Email 验证

### 路由

app/Config/Routes.php

```php
$routes->get('users/(\d+)/password-setting', 'Auth\UsersController::passwordSetting/$1', ['as' => 'users.password-setting']);
$routes->post('users/(\d+)/password-setting', 'Auth\UsersController::passwordSettingUpdate/$1', ['as' => 'users.password-setting']);

+$routes->get('users/(\d+)/email-verified', 'Auth\UsersController::emailVerified/$1', ['as' => 'users.email-verified']);
+$routes->post('users/(\d+)/email-verified', 'Auth\UsersController::sendVerifiedEmail/$1', ['as' => 'users.email-verified']);
```

### controller

app/Controller/Auth/UsersController.php

```php
public function emailVerified($id)
{
    $user = $this->userModel->find($id);

    if (is_null($user))
    {
        return $this->notFoundUser();
    }

    return view('users/email-verified', ['user' => $user]);
}

public function sendVerifiedEmail($id)
{
    $user = $this->userModel->find($id);

    if (is_null($user))
    {
        return $this->notFoundUser();
    }

    // 重新发送邮箱验证
    helper('text');
    $userUpdate = [
        'email_verified_hash' => random_string('alnum', 32),
    ];

    if (! $this->userModel->update($user['id'], $userUpdate))
    {
        return redirect()->back()->with('error','邮箱验证发送失败，请重新尝试~');
    }

    helper('authEmail');
    send_verified_email($user['email'], $userUpdate['email_verified_hash']);

    return redirect()->back()->with('success','邮箱验证发送成功~');
}
```

### 页面

app/Views/users/email-verified.php

```php
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="container main mt-4">
  <div class="row">
    <div class="col-md-4">
      <?= view('App\Views\layouts\_user_sidebar') ?>
    </div>
    <div class="col-md-8">
      <div class="card">
        <h2 class="card-header bg-white text-center">
          重新发送邮箱验证
        </h2>

        <div class="card-body">
          <?php if (isset($user['email_verified']) && $user['email_verified']) : ?>
            <div class="form-group">
              <label for="email">Email <span class="badge badge-danger">已验证</span></label>
              <input type="email" name="email" class="form-control" value="<?= $user['email'] ?>" readonly>
            </div>
          <?php else : ?>
            <?= view('App\Views\components\messages') ?>
            <form action="<?= route_to('users.email-verified', $user['id']) ?>" method="post">
              <?= csrf_field() ?>
              <div class="form-group">
                <label for="email">Email <span class="badge badge-danger">没验证</span></label>
                <div class="form-inline">
                  <input type="email" name="email" class="form-control" value="<?= $user['email'] ?>" readonly>
                  <button type="submit" class="btn ml-2 theme-bg text-white">发送邮箱验证</button>
                </div>
              </div>
            </form>
          <?php endif; ?>
        </div>
      </div>
    </div>
  </div>
</div>
<?= $this->endSection() ?>
```

![](7.7.png)

![](7.8.png)

![](7.9.png)

![](7.10.png)

![](7.11.png)



## avatar

### 路由

app/Config/Rputes.php

```php
$routes->get('users/(\d+)/email-verified', 'Auth\UsersController::emailVerified/$1', ['as' => 'users.email-verified']);
$routes->post('users/(\d+)/email-verified', 'Auth\UsersController::sendVerifiedEmail/$1', ['as' => 'users.email-verified']);

+$routes->get('users/(\d+)/avatar', 'Auth\UsersController::avatar/$1', ['as' => 'users.avatar']);
+$routes->post('users/(\d+)/avatar', 'Auth\UsersController::avatarUpload/$1', ['as' => 'avatar']);
```

### controller

app/Controllers/Auth/UsersController.php

```php
public function avatar($id)
{
    $user = $this->userModel->find($id);

    if (is_null($user))
    {
        return $this->notFoundUser();
    }

    return view('users/avatar', ['user' => $user]);
}

public function avatarUpload($id)
{
    $rules = [
        'avatar' => [
            'label' => '头像',
            'rules' => 'uploaded[avatar]|max_size[avatar,200]|ext_in[avatar,jpg,png,gif]',
        ]
    ];

    if (!$this->validate($rules))
    {
        return redirect()->back()->with('errors', $this->validator->getErrors());
    }

    $file = $this->request->getFile('avatar');

    if (!$file->isValid() || $file->hasMoved())
    {
        return redirect()->back()->with('error', $file->getError());
    }

    $ext = $file->getClientExtension();

    helper('text');
    $fileName = date('Y-m-d') . '_' . $id .'_' . random_string() .'.' . $ext;
    $path = FCPATH . 'image\avatar';

    if ( $file->move($path, $fileName, true))
    {
        $user = [
            'avatar' => 'avatar/' . $fileName,
        ];

        $this->userModel->update($id, $user);

        return redirect()->back()->with('success', '头像上传成功~');
    }

    return redirect()->back()->with('error', '头像上传失败~');
}
```

### 页面

app/Views/users/avatar.php

```php
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="container main mt-4">
  <div class="row">
    <div class="col-md-4">
      <?= view('App\Views\layouts\_user_sidebar') ?>
    </div>
    <div class="col-md-8">
      <div class="card">
        <h2 class="card-header bg-white text-center">
          头像
        </h2>

        <div class="card-body">
          <img src="<?= base_url($user['avatar']) ?>" width="80" alt="avatar">
        </div>

        <div class="card-body">
          <?= view('App\Views\components\messages') ?>

          <form action="<?= route_to('users.avatar', $user['id']) ?>" method="post" enctype="multipart/form-data">
            <?= csrf_field() ?>

            <div class="form-group">
              <label for="avatar">头像</label>
              <input type="file" name="avatar" id="avatar" class="form-control-file">
            </div>

            <div class="form-group">
              <button type="submit" class="btn theme-bg text-white">上传</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>
<?= $this->endSection() ?>
```

![](7.12.png)

![](7.13.png)

![](7.14.png)



## 细节优化

目前有2个问题，未登录用户也可以访问用户设置，用户可以修改其他用户的设置。

CodeIgniter4 在 **URI 路由** 里提供了[应用过滤器](https://codeigniter.org.cn/user_guide/incoming/routing.html#id43)可以解决这2个问题。

生成 filters 文件

```php
$ php spark make:filter LoggedInAuthFilter
```

app/Filters/LoggedInAuthFilter.php

```php
public function before(RequestInterface $request, $arguments = null)
{
    $session = Services::session();

    // 登录
    if (! $session->isLoggedIn)
    {
        return redirect()->route('login');
    }

    // 登录用户只能更改自己的信息
    $id = $request->uri->getSegments(3)[1];

    if ($id !== $session->blog_user['id'])
    {
        return redirect()->to('/')->with('error', '无权限~');
    }
}
```

app/Config/Filters.php

```php
'loggedInNotHandle'     => LoggedInNotHandleFilter::class,
+'loggedIn'     => LoggedInAuthFilter::class,
```

app/Config/Routes.php

```php
// users
$routes->get('users/(\d+)', 'Auth\UsersController::show/$1', ['as' => 'users.show']);

// filter
$routes->group('', ['filter' => 'loggedIn'], function ($routes) {

    $routes->get('users/(\d+)/edit', 'Auth\UsersController::edit/$1', ['as' => 'users.edit']);
    $routes->post('users/(\d+)', 'Auth\UsersController::update/$1', ['as' => 'users.update']);

    $routes->get('users/(\d+)/password-setting', 'Auth\UsersController::passwordSetting/$1', ['as' => 'users.password-setting']);
    $routes->post('users/(\d+)/password-setting', 'Auth\UsersController::passwordSettingUpdate/$1', ['as' => 'users.password-setting']);

    $routes->get('users/(\d+)/email-verified', 'Auth\UsersController::emailVerified/$1', ['as' => 'users.email-verified']);
    $routes->post('users/(\d+)/email-verified', 'Auth\UsersController::sendVerifiedEmail/$1', ['as' => 'users.email-verified']);

    $routes->get('users/(\d+)/avatar', 'Auth\UsersController::avatar/$1', ['as' => 'users.avatar']);
    $routes->post('users/(\d+)/avatar', 'Auth\UsersController::avatarUpload/$1', ['as' => 'avatar']);
});
```

未登录用户访问用户设置页面，会发现跳转到 login 页面。

登录用户访问其他用户设置页面，会提示无权限。

![](7.15.png)



## 提交

```shell
$ git add -A
$ git commit -m "用户个人页面 && 用户设置"
```

