---
title: CodeIgniter 4 开发（八）
date: 2021-06-06 16:24:58
tags:
	- php
	- CodeIgniter
---
> 限流

针对用户相关的页面进行限流，每秒不超过1个请求。

## 步骤

1. 新建限流文件

```shell
$ php spark make:filter Throttle
```
2. 进行限流设置

app/Filters/Throttle.php

```php
public function before(RequestInterface $request)
{
    $throttle = Services::throttler();
    if ($throttle->check($request->getIPAddress(), 60, MINUTE) === false)
    {
      return Services::response()->setStatusCode(429);
    }
}
```

3. 限流别名

app/Config/Filters.php

```php
    'loggedIn'     => LoggedInAuthFilter::class,
+    'throttle' => Throttle::class,
```
4. 对路由进行限流

app/Config/Routes.php

```php
$routes->group('', ['namespace' => 'App\Controllers', 'filter' => 'throttle' ], function ($routes) {
  ...
});
```

![](8.1.png)



## 提交

```shell
$ git add -A
$ git commit -m "限流"
```

