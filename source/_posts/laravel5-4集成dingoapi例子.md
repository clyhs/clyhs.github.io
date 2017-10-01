---
title: laravel5.4集成dingoapi例子
date: 2017-10-01 09:01:47
category: laravel
tags: [laravel,php,dingoapi]
---
### 安装dingo api
 在composer.json的reqiure添加如下：
```shell
"dingo/api":"1.0.*@dev",
```
然后执行
```shell
composer update
```
### 配置

在config/app.php 注册服务提供者
```php
'providers' => [
    ...
    Dingo\Api\Provider\LaravelServiceProvider::class,
    ...
],
```

生成配置文件config/api.php

```shell
php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
```

设置.env

API_STANDARDS_TREE=vnd
API_SUBTYPE=myapp
API_PREFIX=api
API_VERSION=v1

创建UserController
```shell
php artisan make:controller Api/V1/UserController
```

修改app/Http/Controller/Api/V1/UserController

```php
<?php
namespace App\Http\Controllers\Api\V1;

use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function index()
    {
        return User::all();
    }

    public function show($id)
    {
        return User::findOrFail($id);
    }
}
```
在routes/api.php注册路由
```php
...
$api = app('Dingo\Api\Routing\Router');
$api->version('v1', ['namespace' => 'App\Http\Controllers\Api\V1'], function ($api) {
    $api->get('user/{id}', 'UserController@show');
    $api->get('user', 'UserController@index');
});
```

查看路由列表
```shell
php artisan api:routes
```








