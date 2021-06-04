---
title: CodeIgniter 4 开发（三）
date: 2021-06-04 00:09:10
tags:
	- CodeIgniter
	- php
---
> 注册用户功能
## 用户
### 创建模型
```
$ php spark make:model UserModel
```
app/Models/UserModel.php
```php
protected $allowedFields        = [
  	'name',
	'email',
	'password',
];
...
protected $useTimestamps        = true;
...
protected $dynamicRules = [
  'register' => [
       'name' => [
	      'label' => '名称',
	      'rules' => 'required|min_length[2]',
	  ],
	  'email' => [
	    'label' => 'Email',
	    'rules' => 'required|valid_email|is_unique[users.email, id, {id}]',
	  ],
	  'password' => [
	    'label' => '密码',
	    'rules' => 'required|min_length[5]',
	  ],
	  'password_confirm' => [
	    'label' => '确认密码',
	    'rules' => 'matches[password]',
	  ],
  ],
];
...
protected $beforeInsert         = ['hashPassword'];
...
/**
* return dynamicRules
*/
public function getRules(string $rule)
{
	return $this->dynamicRules[$rule];
}

protected function hashPassword(array $data)
{
	if (! isset($data['data']['password'])) return $data;

	$data['data']['password'] = password_hash($data['data']['password'], PASSWORD_DEFAULT);
	unset($data['data']['password_confirm']);

	return $data;
}
```
### 创建数据库模型
```
$ php spark make:migration Users
```
app/Database/Migrations/xxx_Users.php
```php
public function up()
{
	$this->forge->addField([
      'id' => [
        'type'           => 'BIGINT',
        'unsigned'       => true,
        'auto_increment' => true,
      ],
      'name' => [
        'type'           => 'VARCHAR',
        'constraint'     => 255,
      ],
      'email' => [
        'type'           => 'VARCHAR',
        'constraint'     => 255,
        'unique'         => true,
      ],
      'email_verified_at' => [
        'type'           => 'TIMESTAMP',
        'null'           => true,
      ],
      'password' => [
        'type'           => 'VARCHAR',
        'constraint'     => 255,
      ],
      'remember_token' => [
        'type'           => 'VARCHAR',
        'constraint'     => 100,
        'null'           => true,
      ],
      'created_at' => [
        'type'           => 'TIMESTAMP',
      ],
      'updated_at' => [
        'type'           => 'TIMESTAMP',
      ],
    ]);

	$this->forge->addKey('id');
	$this->forge->createTable('users');
}

public function down()
{
	$this->forge->dropTable('users');
}
```
生成表
```
$ php spark migrate
```
### 数据填充
```
$ php spark make:seeder UsersTableSeeder
```
app/Database/Seeds/UsersTableSeeder
```php
public function run()
{
    // 清空
	$this->db->table('users')->truncate();

	helper('text');

	for ($i = 0; $i < 50; $i++)
	{
	  $data = [
	    'name' => random_string('alpha'),
	    'email' => random_string('alpha') . "@qq.com",
	    'password' => password_hash('password', PASSWORD_DEFAULT),
	    'created_at' => date('Y-m-d H:i:s'),
	    'updated_at' => date('Y-m-d H:i:s'),
	  ];

	  $this->db->table('users')->insert($data);
	}
}
```

## 注册
### 路由
app/Config/Routes.php
```php
$routes->group('', ['namespace' => 'App\Controllers' ], function ($routes) {
  // register
  $routes->get('register', 'Auth\RegisterController::register', ['as' => 'register']);
  $routes->post('register', 'Auth\RegisterController::attemptRegister', ['as' => 'register']);
});
```
### 创建 controller
```
$ php spark make:controller Auth/RegisterController
```
app/Controllers/Auth/RegisterController.php
```php
public function register()
{
	return view('auth/register');
}

public function attemptRegister()
{
	// 验证
	$userModel = new UserModel();
	$rules = $userModel->getRules('register');
	$userModel->setValidationRules($rules);

	$user = [
	  'name' => $this->request->getVar('name'),
	  'email' => $this->request->getVar('email'),
	  'password' => $this->request->getVar('password'),
	  'password_confirm' => $this->request->getVar('password_confirm'),
	];

	if (!$userModel->insert($user))
	{
	  return redirect()->back()->withInput()->with('errors', $userModel->errors());
	}

	return redirect()->to('/')->with('success', '用户注册成功，请及时进行邮箱验证~');
}
```
### 创建页面
app/Views/auth/register
```php
<?= $this->extend('layouts/app') ?>
<?= $this->section('main') ?>
<div class="main d-flex justify-content-center align-items-center h-100">
  <div class="col-sm-10 col-md-5 col-lg-4">
    <div class="card">
      <h2 class="card-header text-center">
        注册
      </h2>
      <div class="card-body">

      	<!-- error messages -->
      	<?= view('App\Views\components\messages') ?>

        <form action="<?= route_to('register') ?>" method="post">
          <div class="form-group">
            <label for="name">名称</label>
            <input type="text" name="name" class="form-control" value="<?= old('name') ?>">
          </div>
          <div class="form-group">
            <label for="email">Email</label>
            <input type="email" name="email" class="form-control" value="<?= old('email') ?>">
          </div>
          <div class="form-group">
            <label for="password">密码</label>
            <input type="password" name="password" class="form-control">
          </div>
          <div class="form-group">
            <label for="password_confirm">确认密码</label>
            <input type="password" name="password_confirm" class="form-control">
          </div>

          <div class="form-group">
            <button type="submit" class="btn form-control theme-bg text-white">注册</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</div>
<?= $this->endSection() ?>
```
app/Views/components/messages.php
```php
<?php if (session()->has('errors')) : ?>
  <div class="alert alert-danger w-100" role="alert">
    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
      <span aria-hidden="true">&times;</span>
    </button>

    <ul>
      <?php foreach (session('errors') as $error) : ?>
        <li><?= $error ?></li>
      <?php endforeach ?>
    </ul>
  </div>
<?php endif; ?>

<?php if (session()->has('success')) : ?>
<div class="alert alert-success w-100" role="alert">
  <button type="button" class="close" data-dismiss="alert" aria-label="Close">
    <span aria-hidden="true">&times;</span>
  </button>

  <p><?= session('success') ?></p>
</div>
<?php endif; ?>
```
![](3.1.png)
![](3.2.png)

### 注意的地方
- 验证规则
- 插入前的密码加密

## 提交
```shell
$ git add -A
$ git commit -m "用户注册"
```