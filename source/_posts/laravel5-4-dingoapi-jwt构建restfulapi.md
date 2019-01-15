---
title: laravel5.4+dingoapi+jwt构建restfulapi
date: 2017-10-01 17:43:29
category: php
tags: [laravel,dingoapi,jwt]
---

### 添加jwt的认证
 在composer.json的reqiure添加如下：
```shell
"tymon/jwt-auth": "1.0.*@dev",
```
运行composer update将jwt装上去

*proc_open(): fork failed - Cannot allocate memory *
```shell
free -m
/bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
/sbin/mkswap /var/swap.1
/sbin/swapon /var/swap.1
composer update
```


**在config/api.php添加内容**
```php
'auth' => [
    'jwt' => Dingo\Api\Auth\Provider\JWT::class
]
```
在config/app.php
```php
'providers' => [
    // 前面很多
    Tymon\JWTAuth\Providers\LaravelServiceProvider::class
],

'aliases' => [
    // 前面很多
    'JWTAuth' => Tymon\JWTAuth\Facades\JWTAuth::class
]
```


在终端运行：
```shell
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```
会生成config/jwt.php 这是jwt的配置文件 
生成jwt的key到.env文件运行：
```shell
php artisan jwt:secret
```
### RegisterApiController

修改User.php
```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Tymon\JWTAuth\Contracts\JWTSubject;

class User extends Authenticatable implements JWTSubject
{
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];

    /**
     * Get the identifier that will be stored in the subject claim of the JWT.
     *
     * @return mixed
     */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims()
    {
        return [];
    }
}


```
添加 /app/Http/Controllers/Api/V1/Auth/RegisterApiController

```php
<?php
/**
 * Created by PhpStorm.
 * User: clyhs
 * Date: 2017/10/1
 * Time: 18:11
 */
namespace App\Http\Controllers\Api\V1\Auth;

use App\User;
use App\Http\Controllers\Api\V1\BaseController;
use Illuminate\Support\Facades\Validator;
use Illuminate\Foundation\Auth\RegistersUsers;
use Illuminate\Http\Request;
use Dingo\Api\Exception\StoreResourceFailedException;
use Tymon\JWTAuth\Facades\JWTAuth;


class RegisterApiController extends BaseController{

    use RegistersUsers;

    public function register(Request $request)
    {
        $valid=$this->valid($request->all());    //验证表单
        if($valid->fails()){
            $this->sendFailResponse($valid->errors());
        }
        else{
            $user=User::create([
                'name'=>$request->name,
                'email'=>$request->email,
                'password'=>bcrypt($request->password)
            ]);
            if($user){
                $token=JWTAuth::fromuser($user);  //获取token
                return $this->response->array([
                    "token" => $token,
                    "message" => "Registration Success",
                    "status_code" => 201
                ]);
            }
            else{
                $this->sendFailResponse("Register Error");
            }
        }
    }
    public function valid($data)
    {
        return Validator::make($data,[
            'name'=>'required|unique:users|max:10',
            'email'=>'required|unique:users|email',
            'password'=>'required|min:6']);
    }
    public function sendFailResponse($message)
    {
        return $this->response->error($message,400);
    }

}
```
routes/api.php中添加路由 
```php
$api->version('v1', ['namespace' => 'App\Http\Controllers\Api\V1'], function ($api) {
    $api->get('user/{id}', 'UserController@show');
    $api->get('user', 'UserController@index');
    $api->get('user2/{id}', 'UserApiController@getUserInfo');
    $api->get('user2/show/{id}', 'UserApiController@show');
    $api->get('user2', 'UserApiController@index');
    $api->get('user2forpage', 'UserApiController@page');

    $api->post('register', 'Auth\RegisterApiController@register');
});
```

POSTMAN测试
![img](https://clyhs.github.io/images/php/jwt01.png)

### LoginApiController

添加 /app/Http/Controllers/Api/V1/Auth/LoginApiController
```php
<?php
/**
 * Created by PhpStorm.
 * User: clyhs
 * Date: 2017/10/1
 * Time: 18:40
 */
namespace App\Http\Controllers\Api\V1\Auth;

use App\User;
use App\Http\Controllers\Api\V1\BaseController;
use Illuminate\Http\Request;
use Tymon\JWTAuth\Facades\JWTAuth;
use Illuminate\Support\Facades\Hash;
use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;
use Illuminate\Foundation\Auth\AuthenticatesUsers;

class LoginApiController extends BaseController{

    use AuthenticatesUsers;

    public function login(Request $request)
    {
        $user=User::where('name',$request->email)->orwhere('email',$request->email)->firstOrFail();
        if($user && Hash::check($request->password,$user->password)){
            $token=JWTAuth::fromuser($user);    //获取token
            $this->clearLoginAttempts($request);  //清除登录次数
            return $this->response->array([
                'token'=>$token,
                'message'=>"Login Success",
                'status_code'=>200
            ]);
        }
        else{
            throw new UnauthorizedHttpException("Login Failed");
        }
    }
    public function logout(){
        JWTAuth::invalidate(JWTAuth::getToken());    //token加入黑名单(注销)
        $this->guard()->logout();
    }
}
```
routes/api.php中添加路由 
```php
$api->post('login', 'Auth\LoginApiController@login');
```
POSTMAN测试
![img](https://clyhs.github.io/images/php/jwt01.png)
