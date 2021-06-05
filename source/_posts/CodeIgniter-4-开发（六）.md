---
title: CodeIgniter 4 开发（六）
date: 2021-06-04 23:51:35
tags:
    - CodeIgniter
    - php
---
> 发送 Email 验证与登录下的优化
## 注册用户 Email 验证

### userModel

app/Models/UserModel.php

```php
protected $allowedFields        = [
    'name',
    'email',
    'password',
    'reset_hash',
    'reset_expires',
+    'email_verified_at',
+    'email_verified_hash',
];
```

### database

app/Database/Migrations/xxx_Users.php

```php
'email' => [
    'type'           => 'VARCHAR',
    'constraint'     => 255,
    'unique'         => true,
],
+'email_verified_at' => [
+    'type'           => 'TIMESTAMP',
+    'null'           => true,
+],
+'email_verified_hash' => [
+    'type'           => 'VARCHAR',
+    'constraint'     => 191,
+    'null'           => true,
+],
```

重置数据库

```shell
$ php spark migrate:refresh
```

### 路由

app/Config/Routes.php

```php
  $routes->post('register', 'Auth\RegisterController::attemptRegister', ['as' => 'register']);

  // email verified
+  $routes->get('email-verified', 'Auth\RegisterController::emailVerified', ['as' => 'email-verified']);
```

### 邮箱

app/Helpers/authEmails.php

```php
if (!function_exists('send_verified_email'))
{
  function send_verified_email($to, $verifiedHash)
  {
    $htmlMessage = view('emails\header');
    $htmlMessage .= view('emails\verified', ['hash' => $verifiedHash]);
    $htmlMessage .= view('emails\footer');

    $email = \Config\Services::email();
    $config = [
      'protocol'   => 'smtp',
      'mailType'   => 'html',
    ];

    $email->initialize($config);

    $email->setTo($to);
    $email->setFrom('blog@admin.com');
    $email->setSubject('Email 地址验证');
    $email->setMessage($htmlMessage);

    return $email->send();
  }
}
```

### 邮件页面

app/Views/emails/verified.php

```html
<h2>Email地址验证</h2>

<p>您只需点击下面的链接即可激活您的帐号：</p>

<p>
  <a href="<?= base_url('email-verified') . '?token=' . $hash ?>">
    <?= base_url('email-verified') . '?token=' . $hash ?>
  </a>
</p>

<p>(如果上面不是链接形式，请将该地址手工粘贴到浏览器地址栏再访问)</p>

<p>感谢您的访问，祝您使用愉快！</p>

```

### controller

app/Controllers/Auth/RegisterController.php

```php
public function attemptRegister()
{
    // 验证
    $userModel = new UserModel();
    $rules = $userModel->getRules('register');
    $userModel->setValidationRules($rules);

    helper('text');
    $user = [
        'name' => $this->request->getVar('name'),
        'email' => $this->request->getVar('email'),
        'password' => $this->request->getVar('password'),
        'password_confirm' => $this->request->getVar('password_confirm'),
+        'email_verified_hash' => random_string('alnum', 32),
    ];

    if (!$userModel->insert($user))
    {
        return redirect()->back()->withInput()->with('errors', $userModel->errors());
    }

    // 发送验证邮箱
+    helper('authEmail');
+    send_verified_email($user['email'], $user['email_verified_hash']);

    return redirect()->to('/')->with('success', '用户注册成功，请及时进行邮箱验证~');
}
```

![](6.1.png)

![](6.2.png)

![](6.3.png)



## 路由过滤

用户在登录时，不能进行注册、登录、密码重置和更新密码操作，因此需要对这些路由进行过滤， 运行命令生成过滤文件。

```
$ php spark make:filter LoggedInAuthFilter
```

app/Filters/LoggedInAuthFilter.php

```php
public function before(RequestInterface $request, $arguments = null)
{
    $session = Services::session();

    if ($session->isLoggedIn)
    {
        return redirect()->to('/')->with('info', '您已经登录了，无法进行此操作~');
    }
}
```
删除 app/Controllers/Auth 文件夹下各文件里方法中包含的以下代码
```php
if ($this->session->isLoggedIn)
{
    return redirect()->to('/')->with('info', '您已经登录了，无法进行此操作~');
}
```
app/Config/Filters.php

```php
public $aliases = [
    'csrf'     => CSRF::class,
    'toolbar'  => DebugToolbar::class,
    'honeypot' => Honeypot::class,
+    'loggedInAuth'     => LoggedInAuthFilter::class,
];
...
public $filters = [
    'loggedInAuth' => [
        'before' => ['register', 'login', 'forgot-password', 'reset-password'],
    ],
];
```
![](6.4.png)

## 提交
```php
$ git add -A
$ git commit -m "验证Email与优化"
```