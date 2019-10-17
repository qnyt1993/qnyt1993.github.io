---
title: 让你的CSRF Token永不过期
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-17 15:13:54
---

> 转载: [Laravel-caffeine 让你的 CSRF Token 永不过期 ——genealabs/laravel-caffeine](https://learnku.com/courses/laravel-package/2019/laravel-caffeine-makes-your-csrf-token-never-expire-genealabslaravel-caffeine/3974)


CSRF [跨站请求伪造](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)  是一种常见的 Web 攻击方式，通常情况下会在 Form 表单页面增加 CSRF Token 来保护项目安全。

## CSRF 例子

Laravel 中使用时非常方便的，中间件会自动的验证，只有 CSRF Token 正确才能提交表单。


    $ php artisan make:auth


*routes/web.php*

    Route::group(['middleware' => 'auth'], function() {
        Route::get('form', function() {
            return view('form');
        });
    
        Route::post('form', function() {
            dump(app('request')->all());
        });
    });




    $ touch resources/views/form.blade.php


写一个简单的表单。

*resources/views/form.blade.php*

    @extends('layouts.app')
    
    @section('content')
        <form method="POST" action="/form">
            {{ csrf_field()}}
            <textarea name="test"></textarea>
            <button type="submit">submit</button>
        </form>
    @endsection


这是一个非常简单的例子，显示表单页面的时候，`Token` 会保存在` Session` 中，提交表单的时候回自动进行验证。


## 测试 `Session` 失效

但是有的时候 `CSRF Toke`n 会带来一些麻烦，因为这个 `Token` 放在 `Session` 中，它的有效期就是 `Session` 的有效期，默认是 120 分钟。如果我们在填写一个非常大的表单，或者在写一篇很长的博客，可能经常会写到一半，然后去干其他的事情，等我们再次回来编辑表单，`Session` 很有可能过期，必须刷新页面，重新填写表单。

来测试一下，这里要求我们长时间没有操作，导致 `Session` 过期，但是因为普通网站都会有记住用户的功能，`Session` 过期了，但其实用户依然是登录状态。

重新登录一下。

我们可以将 env 中的 `SESSION_LIFETIME` 调小，或者手动删除 `Session` 文件来模拟 `session` 过期。


    $ rm storage/framework/sessions/*



![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/PDMTiZdMB0.png)


为了让用户体验更好，防止上面情况的发生，可以使用这个扩展包 https://github.com/GeneaLabs/laravel-caffeine 。

## 安装


    $ composer require genealabs/laravel-caffeine


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/UMhK63ctBA.png)


## 使用扩展包

安装上扩展包就已经在工作了，它会查看页面中是否存在 CSRF Token，如果存在咋会在页面中添加一段 JS。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/YZDlMK8Iuv.png)


这段 JS 的作用就是不断的发送 ajax 请求，请求不用返回任何内容，扩展包已经注册了一个路由，返回了 204，只是为了，防止了 Session 的过期。每次请求都会记录下来请求的时间，如果上一次请求到现在的时间间隔，接近于 Session 过期时间了，那么就直接刷新页面。

请求的间隔，请求的地址这些都是可以配置的。

*config/genealabs-laravel-caffeine.php*

    'drip-interval' => 5000,


为了看清楚 Session 的过期时间，我们可以使用 redis 作为 Session 驱动，然后间隔调小，5 分钟过期。

*.env*

    SESSION_DRIVER=redis
    SESSION_LIFETIME=5


例如我们修改一下配置，5 秒发送一个请求。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/xArukiBjGZ.png)


现在其实是有问题的，Session 的过期时间依然没有被刷新


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/b5QhQPRsgd.png)



## Bug

这里其实有一个 Bug，扩展包注册路由的时候，因为某些判断的问题，导致 web 中间件组没有被注册上，这样 Session 中间件没有执行，导致 Session 过期时间没有被重置。

<https://github.com/GeneaLabs/laravel-caffeine/issues/89>

你可以关注一下这个 Bug，或者可以手动处理一下，自己写一个路由即可。

*routes/web.php*

    Route::get('drip', function() {
        return response(null, 204);
    });


*config/genealabs-laravel-caffeine.php*

    //'route' => 'genealabs/laravel-caffeine/drip', // Customizable end-point URL
    'route' => 'drip', // Customizable end-point URL


这样就行了，不断请求我们自己的路由，返回 204，再次测试就没有问题了。


## 不全局使用

如果某个页面你不想使用这个扩展包，可以在页面中增加一个标签 

    <meta name="caffeinated" content="false">

如果不想全局使用，可以修改配置。

*config/genealabs-laravel-caffeine.php*

    'use-route-middleware' => true,


这样就可以通过手动使用 `caffeinated` 中间件来使用扩展包。
