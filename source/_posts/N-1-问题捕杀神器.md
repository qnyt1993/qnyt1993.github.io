---
title: ' N+1 问题捕杀神器 ——beyondcode/laravel-query-detector'
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-17 11:01:03
---

> 文章转载: [023. N+1 问题捕杀神器 ——beyondcode/laravel-query-detector](https://learnku.com/courses/laravel-package/2019/n1-problem-killing-artifact-beyondcodelaravel-query-detector/2153)

## N+1 问题捕杀神器 ——beyondcode/laravel-query-detector

[N + 1 问题](https://learnku.com/docs/laravel/5.6/eloquent-relationships/1404#012e7e) 是我们开发过程中必须避免的问题，通常情况下我们都会根据经验在必要的地方进行预加载，防止 N + 1 问题，有些时候则是根据查询日志或者 Laravel-Debugbar 等工具发现并及时修改掉 N + 1 问题。

开发总是会有疏忽的时候，通过经验或者日志工具都不够靠谱，有没有工具能及时的发现 N + 1 问题，并提示我们及时解决呢，今天我们要学习的 [beyondcode/laravel-query-detector](https://github.com/beyondcode/laravel-query-detector)，就是这样的一个工具，我们一起安装一下。

## 安装


    $ composer require beyondcode/laravel-query-detector --dev


注意我们使用了 `--dev` 参数，指定是开发环境的依赖。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/4dlQ0jNWk9.png)


我们这里安装的是 `0.6.0`，因为扩展包还在不断的更新当中，1.0 发布之前，版本可能发布的会很快，如果后续发布的版本有新的功能，我们会及时更新这篇文章。现在的版本已经可以很好的满足需求了。

将配置文件发布出来：


    $ php artisan vendor:publish --provider=BeyondCode\\QueryDetector\\QueryDetectorServiceProvider


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/Kpb6NmYXg7.png)


## 使用

先修改一下代码，制造一个 N + 1 问题，修改话题列表关于预加载的处理，在 Topic 模型中：

*app/Models/Topic.php*

    .
    .
    .
        public function scopeWithOrder($query, $order)
        {
            // 不同的排序，使用不同的数据读取逻辑
            switch ($order) {
                case 'recent':
                    $query = $this->recent();
                    break;
    
                default:
                    $query = $this->recentReplied();
                    break;
            }
            return $query;
            // 预加载防止 N+1 问题
            // return $query->with('user', 'category');
        }
    .
    .
    .


访问 LaraBBS 的话题列表 http://larabbs.test/topics 

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/xz1Z9Ca5OG.png)


浏览器弹出了一个 Alert 框，提示我们 Topic 模型出现了 N + 1 问题，需要使用 with('user') 和 with('category')。

查看一下日志 `tail -n 100 storage/logs/laravel.log`，也能看到关于  N + 1 问题的提示：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/YxVkBfTn2k.png)


### 其他的提示方式

这里一下配置中所有的提示方式：

    - \BeyondCode\QueryDetector\Outputs\Alert::class —— 浏览器 Alert 提示；
    - \BeyondCode\QueryDetector\Outputs\Console::class —— 浏览器 console 提示；
    - \BeyondCode\QueryDetector\Outputs\Clockwork::class —— 浏览器 clockwork插件提示，该扩展包的安装建 [课程](https://learnku.com/courses/laravel-package/1976/debugging-tool-under-chrome-itsgoingdclockwork)；
    - \BeyondCode\QueryDetector\Outputs\Debugbar::class —— Laravel-Debugbar 提示；
    - \BeyondCode\QueryDetector\Outputs\Json::class —— json 响应提示，由于 LaraBBS 中使用了 Dingo 扩展包完成接口数据的返回，该配置对现有接口没有作用。任何继承了 `Illuminate\Http\JsonResponse` 的响应都会自动添加提示；
    - \BeyondCode\QueryDetector\Outputs\Log::class —— 日志提示；

提示在哪里是可以配置的，尝试修改一下配置：

*config/querydetector.php*

    .
    .
    .
        'output' => [
            \BeyondCode\QueryDetector\Outputs\Alert::class,
            \BeyondCode\QueryDetector\Outputs\Log::class,
            \BeyondCode\QueryDetector\Outputs\Console::class,
            \BeyondCode\QueryDetector\Outputs\Clockwork::class,
            \BeyondCode\QueryDetector\Outputs\Debugbar::class,
        ]
    .
    .
    .


查看 `clockwork` 中的日志：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/500VBMqs5w.png)



查看 `debugbar` 中的数据查询：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/umI7e2WxZs.png)


查看浏览器 `console` 中的输出：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/nhdlK7eh56.png)



## 白名单功能

扩展包还提供了白名单的功能，也就当某个模型的的某个关系出现了 N + 1问题的时候，不进行报错。相当于手动取消了对某个模型预加载的监控，或许你会在某个特殊的场合遇到这样的白名单使用场景，我们只是来测试一下功能。调整一下白名单的配置：

*config/querydetector.php*

    .
    .
    .
        'except' => [
            App\Models\Topic::class => [
                App\Models\Category::class,
                'category',
            ]
        ],
    .
    .
    .


设置为不出了 Topic 模型关于 Category 预加载的检测，再次访问话题列表页面：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/xay6YsdoXn.png)


已经没有了关于 `with('category')` 的提示。


## 代码版本控制

可以根据需求调整相关的输出位置，我们将代码还原为在页面中进行 alert 提示以及在日志中提示。

*config/querydetector.php*

    .
    .
    .
        'except' => [
        ],
        'output' => [
            \BeyondCode\QueryDetector\Outputs\Alert::class,
            \BeyondCode\QueryDetector\Outputs\Log::class,
        ]
    .
    .
    .

