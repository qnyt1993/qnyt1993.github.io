---
title: 统计页面访问次数——awssat/laravel-visits
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-17 10:40:43
---


> 转载: [统计页面访问次数——awssat/laravel-visits](https://learnku.com/courses/laravel-package/2019/the-number-of-statistical-visits-awssatlaravel-visits/2057)

本次我们要介绍的项目是 [laravel-visits](https://github.com/awssat/laravel-visits) ，功能如下：

    - 一个模型中支持多个统计字段（使用 tag 来区分）；
    - 支持所有数据模型（不像有些扩展包只是 User 专用）；
    - 支持区分 IP 访问次数，页面多刷几次查看数也不会增加（可配置）；
    - 读取最多和最少访问量排序的模型列表；
    - 获取访问量最高的国家；
    - 获取月份、年份、全部统计访问量模型排序列表。

我们可以使用此项目来轻松实现页面访问量统计，甚至是设置 IP 级别的页面访问去重。本项目是基于 Redis 实现的，因此存储性能远比数据库自增字段要优化好多。

接下来我们以 LaraBBS 为例，详细讲解使用场景。

## 场景分析

LaraBBS 中的话题表 topics 中已经有 `view_count` 字段用于统计，不过未实现统计的功能，我们先来简单的实现一下：

*resources/views/topics/show.blade.php*

    .
    .
    .
                    <div class="article-meta text-center">
                        {{ $topic->created_at->diffForHumans() }}
                        ⋅
                        <span class="glyphicon glyphicon-comment" aria-hidden="true"></span>
                        {{ $topic->reply_count }}
                        ⋅
                        {{ $topic->view_count }} 阅读
                    </div>
    .
    .
    .

在话题详情中显示阅读数 `view_count`，同时在 Controller 中统计访问次数。

*app/Http/Controllers/TopicsController.php*

    .
    .
    .
        public function show(Request $request, Topic $topic)
        {
            // URL 矫正
            if (! empty($topic->slug) && $topic->slug != $request->slug) {
                return redirect($topic->link(), 301);
            }
    
            $topic->increment('view_count');
    
            return view('topics.show', compact('topic'));
        }
    .
    .
    .


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/21bPh5tD3L.png)


每次刷新页面阅读数就会加 1。

上面使用了一种非常简单的方式实现了阅读数的统计，这当然是有问题的：

    - 不停的刷新阅读数就会不断增加，需要根据用户 ip 进行隔离；
    - 在访问数据的时候，进行了一次数据更新，访问量巨大的时候会有性能问题；
    - 需要考虑到代码的复用，进行适当的封装，方便对所有模型进行统计。

今天要学习的扩展包 [awssat/laravel-visits](https://github.com/awssat/laravel-visits) 就很好的解决了上述问题，并且使用起来非常方便。

## 安装

    $ composer require awssat/laravel-visits


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/SMEYfIFhax.png)


发布配置文件：

    $ php artisan vendor:publish --provider="awssat\Visits\VisitsServiceProvider"


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/A2sgDFy61H.png)


访问相关的数据是存储在 `redis` 中的，我们需要确保 `redis` 的链接正常，同时为了区分缓存与访问统计，我们可以新增一个 `redis` 配置：

*config/database.php*

    .
    .
    .
        'redis' => [
    
            'client' => 'predis',
    
            'default' => [
              .
              .
              .
            ],
    
            'laravel-visits' => [
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'password' => env('REDIS_PASSWORD', null),
                'port' => env('REDIS_PORT', 6379),
                'database' => 3, // anything from 1 to 15, except 0 (or what is set in default)
            ],
        ],
    .
    .
    .


增加了 `laravel-visits` 的，与缓存独立开，这样使用 `cache:clear` 命令时就不会清除掉访问数据了。

## 使用

### 基础用法

使用起来很简单，扩展包提供了 `visits` 方法，参数是某个模型的实例，然后就可以使用扩展包提供的一系列方法了，例如 `visits($topic)->count()` 获取访问话题的数量。

*resources/views/topics/show.blade.php*

    .
    .
    .
                    <div class="article-meta text-center">
                        {{ $topic->created_at->diffForHumans() }}
                        ⋅
                        <span class="glyphicon glyphicon-comment" aria-hidden="true"></span>
                        {{ $topic->reply_count }}
                        ⋅
                        {{ visits($topic)->count()}} 阅读
                    </div>
    .
    .
    .


*app/Http/Controllers/TopicsController.php*

    public function show(Request $request, Topic $topic)
    {
        // URL 矫正
        if (! empty($topic->slug) && $topic->slug != $request->slug) {
            return redirect($topic->link(), 301);
        }

        visits($topic)->increment();

        return view('topics.show', compact('topic'));
    }


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/JVSKDVsYYn.png)


### 优化一下代码

*app/Models/Model.php*

    .
    .
    .
        public function visits()
        {
            return visits($this);
        }
    .
    .
    .


我们的模型的基类中增加了上述代码，就可以直接使用 `$model->visits()->increment()` 的方式操作各个模型的统计数据了。

*resources/views/topics/show.blade.php*

    .
    .
    .
                    <div class="article-meta text-center">
                        {{ $topic->created_at->diffForHumans() }}
                        ⋅
                        <span class="glyphicon glyphicon-comment" aria-hidden="true"></span>
                        {{ $topic->reply_count }}
                        ⋅
                        {{ $topic->visits()->count() }} 阅读
                    </div>
    .
    .
    .


*app/Http/Controllers/TopicsController.php*

    public function show(Request $request, Topic $topic)
    {
        // URL 矫正
        if (! empty($topic->slug) && $topic->slug != $request->slug) {
            return redirect($topic->link(), 301);
        }

		
        $topic->visits()->increment();

        return view('topics.show', compact('topic'));
    }


`awssat/laravel-visits` 会将统计数据存储在 redis 中，当然 redis 非常适合做这样的统计，因为 redis 的高性能，在访问量大的情况下也不会有性能问题。同时扩展包会根据用户的 IP 进行隔离，每个 IP 在某段时间内的访问只会使阅读数增加一次，这个时间被放在了配置中，我们尝试修改一下这个配置：

*config/visits.php*

    .
    .
    .
        // 'remember_ip' => 15 * 60,
        'remember_ip' => 1,
    .
    .
    .


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/56nt2Vho53.png)


再次访问一个话题，基本上每次刷新都会增加一次阅读数了。这个配置适用于配置一个过期时间，如果当前时间没有超过上一次操作设置的过期时间，那么就不会增加阅读数，当然我们可以在有需要的地方使用  `$topic->visits()->forceIncrement();` 强制操作。


### 记录访问来源

查看 `composer.json` 文件可以发现，扩展包还依赖了 [spatie/laravel-referer](https://github.com/spatie/laravel-referer)，这个扩展包是用来记录浏览器的 referer 信息的，也就是当前这个页面是从哪里跳转过来的。

*app/Http/Kernel.php*

    .
    .
    .
        // 定义中间件组
        protected $middlewareGroups = [
        .
        .
        .
                // 记录 referer
                \Spatie\Referer\CaptureReferer::class,
        ]
    .
    .
    .


添加测试代码，可以通过 `$topic->visits()->refs()` 查看某个模型的 `referer` 记录：

*app/Http/Controllers/TopicsController.php*

    .
    .
    .
        public function show(Request $request, Topic $topic)
        {
            // URL 矫正
            if (! empty($topic->slug) && $topic->slug != $request->slug) {
                return redirect($topic->link(), 301);
            }
    
            $topic->visits()->increment();
            
            logger('referer =======>');
            logger($topic->visits()->refs());
            logger('<======== referer');
    
            return view('topics.show', compact('topic'));
        }
    .
    .
    .


我们先打开 Google，随便修改一个连接为某个话题详情的地址，然后点击连接跳转过来：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/gkcoNv6jhs.png)


可以看到关于 `referer` 的记录：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/6bN4vfewUK.png)


### 记录访问 IP 地址

再次查看 `composer.json` 文件可以发现，扩展包还依赖了 [torann/geoip](https://github.com/Torann/laravel-geoip)，之前的课程已经介绍过这个扩展包了，你可以点击 [这里](https://learnku.com/courses/laravel-package/2024/get-the-corresponding-geo-location-information-through-ip-toranngeoip) 再回顾一下。

`torann/geoip` 可以根据 IP 地址获取用户的位置信息，所以我们可以同时获取到访问当前页面的用户是来自于哪个国家。增加一些测试代码：

*app/Http/Controllers/TopicsController.php*

    .
    .
    .
        public function show(Request $request, Topic $topic)
        {
            // URL 矫正
            if (! empty($topic->slug) && $topic->slug != $request->slug) {
                return redirect($topic->link(), 301);
            }
    
            $topic->visits()->increment();
            logger('referer =======>');
            logger($topic->visits()->refs());
            logger('<======== referer');
    
            logger('countries =======>');
            logger($topic->visits()->countries());
            logger('<======== countries');
    
            return view('topics.show', compact('topic'));
        }
    .
    .
    .


再次访问话题详情，然后查看日志 `tail -f storage/logs/laravel.log`:

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/LkAwjCXuVp.png)


可以看到关于访问用户所在国家的统计，结果是个数组，键是国家代码，值是访问的次数，可以方便我们进行进一步的统计。



