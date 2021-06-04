---
title: CodeIgniter 4 开发（五）
date: 2021-06-04 17:02:45
tags:
	- CodeIgniter
	- php
---
> 密码重置
## 发送重置密码邮件
### 用户数据库
app/Database/Migrations/xxx_Users.php
```php
    ...
    'remember_token' => [
      'type'           => 'VARCHAR',
      'constraint'     => 100,
      'null'           => true,
    ],
+    'reset_hash' => [
+      'type'           => 'VARCHAR',
+      'constraint'     => 191,
+      'null'           => true,
+    ],
+    'reset_expires' => [
+      'type'           => 'BIGINT',
+      'constraint'     => 20,
+      'null'           => true,
+    ],
    'created_at' => [
      'type'           => 'TIMESTAMP',
    ],
    ...
```
重置数据库
```
$ php spark migrate:refresh
```
### 路由
app/Config/Routes.php
```php
// logout
$routes->get('logout', 'Auth\LoginController::logout', ['as' => 'logout']);

// forgot-password
+$routes->get('forgot-password', 'Auth\Password::forgotPassword', ['as' => 'forgot-password']);
+$routes->post('forgot-password', 'Auth\Password::forgotPasswordSendEmail', ['as' => 'forgot-password']);

```
### UserModel
```php
...
protected $allowedFields        = [
  'name',
  'email',
  'password',
+  'reset_hash',
+  'reset_expires',
];
...
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
+'forgot-password' => [
+  'email' => [
+    'label' => 'Email',
+    'rules' => 'required|valid_email'
+  ],
+],
...
protected $beforeUpdate         = ['hashPassword'];
```
### controller
```shell
$ php spark make:controller Auth/PasswordController
```
app/Controllers/Auth/PasswordController.php
```php
protected $session;

public function __construct()
{
  $this->session = Services::session();
}

public function forgotPassword()
{
  if ($this->session->isLoggedIn)
	{
	  return redirect()->to('/')->with('info', '已经登录了，无法进行忘记密码找回操作~');
	}

  return view('auth/forgot-password');
}

public function forgotPasswordSendEmail()
{
  $userModel = new UserModel();
  $rules = $userModel->getRules('forgot-password');

  if (!$this->validate($rules))
  {
    return redirect()->back()->with('errors', $this->validator->getErrors());
  }

  $user = $userModel->where('email', $this->request->getVar('email'))
                    ->first();

  if (is_null($user))
  {
    return redirect()->back()->with('error', '此邮箱尚未注册~');
  }

  if (! empty($user['reset_hash']) && $user['reset_expires'] > time())
  {
    return redirect()->back()->with('error', '链接已发送，请去邮箱查收~');
  }

  helper('text');
  $updateUser['reset_hash'] = random_string('alnum', 32);
  $updateUser['reset_expires'] = time() + HOUR;

  $userModel->update($user['id'], $updateUser);

  // Email 发送文件 app/Helpers/authEmail.php
  helper('authEmail');
  send_password_reset_email($user['email'], $updateUser['reset_hash']);

  return redirect()->back()->with('success', '邮件发送成功~');
}
```
### 页面
app/Views/auth/forgot-password.php
```php
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="main d-flex justify-content-center align-items-center h-100">
  <div class="col-sm-10 col-md-5 col-lg-4">
    <div class="card">
      <h2 class="card-header text-center">
        忘记登录密码
      </h2>
      <div class="card-body">

        <?= view('App\Views\components\messages') ?>

        <form action="<?= route_to('forgot-password') ?>" method="post">
          <?= csrf_field() ?>

          <div class="form-group">
            <label for="email">Email</label>
            <input type="email" name="email" class="form-control" value="<?= old('email') ?>">
          </div>

          <div class="form-group">
            <button type="submit" class="btn form-control theme-bg text-white">发送重置密码的邮件</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</div>

<?= $this->endSection() ?>
```
### 邮件
app/Config/Email.php
```php
public $SMTPCrypto = '';
```
### 新建邮件 helper 文件
app/Helpers/authEmail.php
```php
if (!function_exists('send_password_reset_email'))
{
  function send_password_reset_email($to, $resetHash)
  {
      $htmlMessage = view('emails\header');
      $htmlMessage .= view('emails\reset', ['hash' => $resetHash]);
      $htmlMessage .= view('emails\footer');

      $email = \Config\Services::email();
      $config = [
        'protocol'   => 'smtp',
        'mailType'   => 'html',
      ];

      $email->initialize($config);

      $email->setTo($to);
      $email->setFrom('blog@admin.com');
      $email->setSubject('密码重置');
      $email->setMessage($htmlMessage);

      return $email->send();
  }
}
```
### 邮件依赖的视图文件
app/Views/emails/header.php
```html
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width" />
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
```
app/Views/emails/reset.php
```html
<h2><?= base_url() ?> 重置密码邮件</h2>

<p>您只需点击下面的链接重置您的密码：</p>

<p>
  <a href="<?= base_url('reset-password') . '?token=' . $hash ?>">
    <?= base_url('reset-password') . '?token=' . $hash ?>
  </a>
</p>

<p>(如果上面不是链接形式，请将该地址手工粘贴到浏览器地址栏再访问)</p>

<p>如果您没有进行上述操作，请忽略这封邮件。您不需要退订或进行其他进一步的操作。</p>

```
app/Views/emails/footer.php
```html
</body>
</html>
```
![](5.1.png)
![](5.2.png)

## 重置密码
### 路由
app/Config/Routes.php
```php
 $routes->post('forgot-password', 'Auth\PasswordController::forgotPasswordSendEmail', ['as' => 'forgot-password']);

  // reset-password
+  $routes->get('reset-password', 'Auth\PasswordController::resetPassword', ['as' => 'reset-password']);
+  $routes->post('reset-password', 'Auth\PasswordController::resetPasswordUpdate', ['as' => 'reset-password']);
```
### controller
app/Controllers/Auth/PasswordController.php
```php
...
public function resetPassword()
{
  $userModel = new UserModel();
  $user = $userModel->where('reset_hash', $this->request->getGet('token'))
                    ->where('reset_expires > ', time())
                    ->first();
  if (is_null($user))
  {
    return redirect()->back()->with('error', '重置密码链接错误，请重新发送~');
  }

  return view('auth/reset-password');
}

public function resetPasswordUpdate()
{
  $userModel = new UserModel();
  $rules = $userModel->getRules('reset-password');

  if (!$this->validate($rules))
  {
    return redirect()->back()->with('errors', $this->validator->getErrors());
  }

  $user = $userModel->where('reset_hash', $this->request->getVar('token'))
                    ->where('reset_expires > ', time())
                    ->first();

  if (is_null($user))
  {
    return redirect()->back()->with('error', '重置密码链接错误，请重新发送~');
  }

  $updateUser = [
    'password'      => $this->request->getVar('password'),
    'reset_hash'    => null,
    'reset_expires' => null,
  ];

  $userModel->update($user['id'], $updateUser);

  return redirect()->route('login')->with('success', '密码重置成功~');
}
```
### UserModel
```php
...
'forgot-password' => [
  'email' => [
    'label' => 'Email',
    'rules' => 'required|valid_email'
  ],
],
+'reset-password' => [
+  'password' => [
+    'label' => '新密码',
+    'rules' => 'required|min_length[5]',
+  ],
+  'password_confirm' => [
+    'label' => '确认新密码',
+    'rules' => 'matches[password]',
+  ],
+  'token' => 'required',
+],
...
protected $beforeUpdate         = ['hashPassword'];
```
### 页面
app/Views/auth/reset-password.php
```html
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="main d-flex justify-content-center align-items-center h-100">
  <div class="col-sm-10 col-md-5 col-lg-4">
    <div class="card">
      <h2 class="card-header text-center">
        重置密码
      </h2>

      <div class="card-body">
        <?= view('App\Views\components\messages') ?>

        <form action="<?= route_to('reset-password') ?>" method="post">
          <?= csrf_field() ?>
          <div class="form-group">
            <label for="password">新密码</label>
            <input type="password" name="password" class="form-control">
          </div>
          <div class="form-group">
            <label for="password_confirm">确认新密码</label>
            <input type="password" name="password_confirm" class="form-control">
          </div>

          <input type="hidden" name="token" value="<?= $_GET['token'] ?>">

          <div class="form-group">
            <button type="submit" class="btn form-control theme-bg text-white">重置</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</div>
<?= $this->endSection() ?>
```
![](5.3.png)
![](5.4.png)

## 提交
```shell
$ git add -A
$ git commit -m "发送重置密码邮件 && 重置密码"
```