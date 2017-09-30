---
title: laravel5.4简单入门
date: 2017-09-30 17:10:37
category: laravel
tags: [php,laravel]
---
### 安装 Composer

Composer 需要 PHP 5.3.2+ 才能运行。
```shell
$ curl -sS https://getcomposer.org/installer | php
```

这个命令会将 composer.phar 下载到当前目录。PHAR（PHP 压缩包）是一个压缩格式，可以在命令行下直接运行。
```shell
mv composer.phar /usr/local/bin/composer
```
### 安装 Laravel

composer global require "laravel/installer=~1.1"

请确保把 ~/.composer/vendor/bin 路径添加到 PATH 环境变量里, 这样laravel 可执行文件才能被命令行找到, 以后您就可以在命令行下直接使用 laravel 命令.

```shell
export PATH=$PATH:/root/.composer/vendor/bin
```


通过 Composer 的 create-project 命令安装 Laravel

还可以通过在命令行执行 Composer 的 create-project 命令来安装Laravel：
```shell
 composer create-project laravel/laravel laravelweb --prefer-dist

composer create-project laravel/laravel=5.4.* laravelweb 
```

### demo

#### 注册登录
php artisan make:auth 
php artisan migrate:refresh --seed 

#### 建立文章后台

##### 建立model

php artisan make:model Article

model文件目录是（基于Laravel5.4）app。目前简单例子中并不会在model中写点什么，但是没它不行。
##### 建立数据表
```shell
php artisan make:migration create_article_table
```

执行以上命令后，会在database/migrations下生成2*_create_article_table.php文件。修改它的up方法：

```php
public function up()
{
    Schema::create('articles', function(Blueprint $table)
    {
        $table->increments('id');
        $table->string('title');
        $table->text('body')->nullable();
        $table->integer('user_id');
        $table->timestamps();
    });
}

```
然后执行以下命令
```shell
php artisan migrate
```
数据库中会出现article表了。

##### 建立controller
```shell
php artisan make:controller ArticleController
```
controller目录是app/Http/ 。以上语句执行成功后，会在该目录建立ArticleControllser.php