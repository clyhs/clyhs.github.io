---
title: laravel5.4集成dingoapi的转化器(Transformer)
date: 2017-10-01 09:40:21
category: laravel
tags: [laravel,dingoapi,php]
---

### dingo/api响应

我们可以自定义一个控制器继承系统的Controller控制器，并在我们自定义的控制器里引入命名空间，我们后续创建的控制器都继承自我们自定义的这一个基控制器
```shell
php artisan make:controller Api/V1/BaseController
```

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

### dingo/api转化器
有些时候我们需要返回用户数据，在web端开发我们可以直接传递一个对象，但在api开发中则是需要把对象转化为标准的json格式响应给客户端。其实我们也可以自己根据对象的属性拼接一个数组，并响应给客户端，但是dingo/api给我们提供了更便捷的方式，也就是转化器(Transformer)。

自定义转化器需要继承TransformerAbstract类，并至少实现transform方法。例如下面是一个User模型的转化：
在App\Http\Controllers目录下手动新建Transformers
还有UserTransformer.php，代码如下
```php
<?php
namespace App\Http\Controllers\Transformers;

use App\User;
use Illuminate\Http\Request;
use App\Http\Requests;
use League\Fractal\TransformerAbstract;

class UserTransformer extends TransformerAbstract
{
    public function transform(User $user)
    {
        return [
            'id' => $user->id,
            'name' => $user->name,
            'email' => $user->email,
        ];
    }
}
```
**编写UserApiController.php**

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Transformers\UserTransformer;

class UserApiController extends BaseController
{
    //
    public function getUserInfo($id)
    {
        $user = User::findOrFail($id);
        return $this->response->item($user, new UserTransformer);
    }
}

```

**修改routers/api.php**
添加：**$api->get('user2/{id}', 'UserApiController@getUserInfo');**

```php
$api->version('v1', ['namespace' => 'App\Http\Controllers\Api\V1'], function ($api) {
    $api->get('user/{id}', 'UserController@show');
    $api->get('user', 'UserController@index');
    $api->get('user2/{id}', 'UserApiController@getUserInfo');
});
```

