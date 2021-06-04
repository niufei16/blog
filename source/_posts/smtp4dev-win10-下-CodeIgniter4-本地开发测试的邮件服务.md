---
title: 'smtp4dev: win10 下 CodeIgniter4 本地开发测试的邮件服务'
date: 2021-06-04 21:05:27
tags:
	- smtp4dev
	- CodeIgniter
---
在 win10 系统下开发 CodeIgniter4 应用时，需要一个本地测试邮件的服务。smtp4dev 刚好满足需求，它提供了一个 web 界面实时的查看发送的测试邮件。
## 安装
[smtp4dev](https://github.com/rnwood/smtp4dev) 下载对应的文件，然后解压，运行 .exe 即可。
![](smtp4dev.1.png)
## 配合 CodeIgniter4 使用
```php
$email = \Config\Services::email();
$config = [
	'protocol'   => 'smtp',
	'SMTPCrypto' => '',
];

$email->initialize($config);

$email->setTo('test@admin.com');
$email->setFrom('blog@admin.com');
$email->setSubject('测试');
$email->setMessage("这是一封测试邮件");

$email->send();
```
运行之后就会看到
![](smtp4dev.2.png)

## 注意的一点
CodeIgniter4 中 SMTPCrypto 在配置文件中默认是 tls，因此在配置文件中设置或者在此处设置 ```'SMTPCrypto' => ''``` 都可以，不设置会出现错误 ```Email: sendWithSmtp throwed stream_socket_enable_crypto(): SSL operation failed with code 1. OpenSSL Error messages:
error:1408F10B:SSL routines:ssl3_get_record:wrong version number```，导致 smtp 邮件无法发送成功。