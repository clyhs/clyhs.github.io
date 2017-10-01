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
...
'providers' => [
    ...
    Dingo\Api\Provider\LaravelServiceProvider::class,
    ...
],
...
```

生成配置文件config/api.php

```shell
php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
```
