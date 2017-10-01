---
title: laravel5.4集成dingoapi的转化器(Transformer)
date: 2017-10-01 09:40:21
category: laravel
tags: [laravel,dingoapi,php]
---
我们可以自定义一个控制器继承系统的Controller控制器，并在我们自定义的控制器里引入命名空间，我们后续创建的控制器都继承自我们自定义的这一个基控制器
```php
<?php

namespace App\Http\Controllers\Api\V1;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Dingo\Api\Routing\Helpers;
use App\Http\Requests;

class BaseController extends Controller
{
    //
    use Helpers;
}

```