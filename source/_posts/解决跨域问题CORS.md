---
title: 解决跨域问题CORS
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-17 08:43:13
---

## 转载
> 原文 :[解决跨域问题（ CORS ）——barryvdh/laravel-cors](https://learnku.com/courses/laravel-package/2019/solving-cross-domain-problems-cors-barryvdhlaravel-cors/2026)

## 解决跨域问题（ CORS ）——barryvdh/laravel-cors

[barryvdh/laravel-cors](https://github.com/barryvdh/laravel-cors) 是一个帮助我们解决浏览器跨域问题的扩展包，那么什么是跨域问题，什么时候需要使用这个扩展包呢？

## 同源策略

首先需要了解一下浏览器 [同源策略](https://baike.baidu.com/item/%E5%90%8C%E6%BA%90%E7%AD%96%E7%95%A5)，它是浏览器最核心也最基本的安全功能，同源的意思是，域名，协议，端口都相同。如果两个地址不同源，那么：

    1. Cookie、LocalStorage 和 IndexDB 无法相互读取；
    2. DOM 无法相互获得；
    3. AJAX 请求不能相互发送。

举个例子，http://larabbs.test  和 https://laravel-china.org 是无法读取对方的 Cookie 等数据，无法获取对方的 DOM，无法相互发送 AJAX 请求的。

这是出于安全的考虑，但是有的时候我们又确实需要跨域，比如要实现前后端分离，前端的域名是 http://www.larabbs.test 服务器接口的域名是 http://api.larabbs.test ，因为子域不同，依然会被同源策略限制，所以前端的 JS 就无法请求到后端接口的数据，我们需要解决这样的跨域问题。

## CORS

CORS 是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing），就是其中一种用来解决浏览器跨域的问题方法。这种方法的关键是在于服务器，只要服务器端返回了合适的头信息（Header），前端 JS 发送 AJAX 的时候就能访问跨域的资源你，代码和同源时一致，无需做任何修改，用户也不会有感知。

对于 CORS 请求时，又会分为两种，简单请求和非简单请求。

只要同时满足以下两大条件，就属于 [简单请求](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)：

### 1. 请求方法是以下三种方法之一：

        - HEAD
        - GET
        - POST
        
### 2. HTTP的头信息不超出以下几种字段：

        - Accept
        - Accept-Language
        - Content-Language
        - Last-Event-ID
        - Content-Type：只限于三个值 `application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

那么这两种请求有什么区别呢？我们在浏览器中测试一下加深了解。

### 测试简单请求

LaraBBS 中已经有了一些接口，比如 话题列表接口 `http://larabbs.test/api/topics`，我们可以测试访问一些接口，理解 `CORS` 请求。打开 http://api.jquery.com/  ，注意我们使用的协议是 HTTP 而不是 HTTPS，因为我们要调试的是本地的 http://larabbs.test ，使用 `HTTPS` 的网站请求 `HTTP` 会被浏览器拒绝。

打开 `Chrome` 开发者工具，选择` Console`，输入如下内容，点击回车：

    $.ajax({
      url: 'http://larabbs.test/api/topics'
    }).done(function(data) {
      console.log(data);
    });


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/3rpOa13fuD.png)


发送成功后，会看到浏览器报错了，打开 `Network`，可以看到一次网络请求，浏览器发现这次跨源 AJAX 请求，而且是简单请求，所以自动在头信息之中，添加一个Origin字段  `Origin: http://api.jquery.com` 也就是当前请求的网页地址。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/YpgvF5vIZC.png)


针对简单请求的 `CORS`，服务器需要返回的字段有：

- `Access-Control-Allow-Origin` —— 允许的域名，这个字段是必须的，`*` 代表允许所有域名；
- `Access-Control-Allow-Credentials` ——CORS 请求是否发送 Cookie；
- `Access-Control-Expose-Headers` ——除了 6 个基本的头字段 `Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`，`XMLHttpRequest` 对象的 `getResponseHeader()` 可以获取的额外的头。

### 测试非简单请求

不符合上述两个条件的都是非简单请求，例如 `PUT`，`DELETE`，或者请求头中有额外的 `Header`，我们还是用话题列表接口，这次增加一个头信息。

    $.ajax({
      url: 'http://larabbs.test/api/topics',
      headers: {
        'foo':'bar',
      }
    }).done(function(data) {
      console.log(data);
    });


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/9xk0DR04mW.png)


请求结果依然是报错，这次打开 `Network` 查看网络请求，点击网络请求，这次请求的方法是 `OPTIONS`，是 `HTTP` 的查询请求，称为『预检』请求，对于非简单请求，只有通过了『预检』请求，才会真正的执行请求。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/d4K7HrqKXy.png)


针对非简单请求的 CORS，服务器需要返回的字段有：

- `Access-Control-Allow-Origin` —— 允许的域名，这个字段是必须的，`*` 代表允许所有域名；
- `Access-Control-Allow-Methods`——该字段必需，表示支持的所有 HTTP 请求方法，如 `GET, POST, PUT`；
- `Access-Control-Allow-Headers`——支持的所有头信息字段；
- `Access-Control-Allow-Credentials`——CORS 请求是否发送 Cookie；
- `Access-Control-Max-Age`——本次预检请求的有效期。


好了，上面提到了很多概念，看着可能会有点懵，总之我们需要知道的就是，由于同源策略的限制，浏览器请求是会做一些检测，当服务器没有响应的 `CORS` 头信息的时候，浏览器就会报错了，我们就请求不到接口数据。要解决这个问题，就需要在响应中添加 `CORS` 头信息，`barryvdh/laravel-cors` 就是方便我们添加这些头信息的扩展包，下面我们看看添加了扩展包是否能解决问题。

## 安装


    $ composer require barryvdh/laravel-cors


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/awXiZDu42z.png)


将配置文件发布出来：

    $ php artisan vendor:publish --provider="Barryvdh\Cors\ServiceProvider"


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/utLgLxyMZw.png)


## 使用

扩展包的使用是非常简单的，在需要的地方增加中间件即可。

如果需要全局使用，可以在 `app/Http/Kernel.php` 的 `$middleware` 中增加 `\Barryvdh\Cors\HandleCors::class`，但是 LaraBBS 中只有接口部分设计到 CORS 问题，我们只添加到 API 相关的路由中。

由于 LaraBBS 中使用了 DingoApi，路由部分被接管了，所以需要去 `routes/api.php` 中单独设置中间件。

首先需要设置一下该中间件的别名，方便使用：

*app/Http/Kernel.php*

    protected $routeMiddleware = [
	.
	.
	.
        // CORS
        'cors' => \Barryvdh\Cors\HandleCors::class,
    ];


在接口中增加 `cors` 中间件：

*routes/api.php*

    $api->version('v1', [
        'namespace' => 'App\Http\Controllers\Api',
        'middleware' => ['serializer:array', 'bindings', 'change-locale', 'cors']
    ], function ($api) {


再次发送简单请求进行测试：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/LivdOl9uqM.png)


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/M3zVLWAkGr.png)


可以正确的获取到数据了，查看网络请求会发现，响应头中多了一条关键信息 `Access-Control-Allow-Origin: http://api.jquery.com`，只有有了这个头字段，并且域名是当前域名，才能请求成功。

再次发送非简单请求测试：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/em20rp0YcI.png)


预检请求中，多出了 `CORS` 相关的字段，请求成功。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/MNClCPau8a.png)


### 配置

可以看到只要使用了中间件，所有的请求都是可以通过的，也就是说，`barryvdh/laravel-cors` 的默认配置都是允许所有请求，当然我们可以修改这些配置：

*config/cors.php*

    'supportsCredentials' => false,
    'allowedOrigins' => ['*'],
    'allowedOriginsPatterns' => [],
    'allowedHeaders' => ['*'],
    'allowedMethods' => ['*'],
    'exposedHeaders' => [],
    'maxAge' => 0,


分别对应如下配置：

| 配置                   | 对应的 Header                    | 说明                                  |
| ---------------------- | -------------------------------- | ------------------------------------- |
| supportsCredentials    | Access-Control-Allow-Credentials | 是否携带 Cookie                       |
| allowedOrigins         | Access-Control-Allow-Origin      | 允许的域名                            |
| allowedOriginsPatterns | Access-Control-Allow-Origin      | 通过正则匹配允许的域名                |
| allowedHeaders         | Access-Control-Allow-Headers     | 允许的 Header                         |
| allowedMethods         | Access-Control-Allow-Methods     | 允许的 HTTP 方法                      |
| exposedHeaders         | Access-Control-Expose-Headers    | 除了 6 个基本的头字段，额外允许的字段 |
| maxAge                 | Access-Control-Max-Age           | 预检请求的有效期                      |


根据实际需要修改配置即可。
