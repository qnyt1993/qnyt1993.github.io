---
title: 利用 Clockwork 在 Google Chrome 下调试你的 Laravel APP ——itsgoingd/clockwork
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-16 18:55:59
---

> 转载: [利用 Clockwork 在 Google Chrome 下调试你的 Laravel APP ——itsgoingd/clockwork](https://learnku.com/courses/laravel-package/2019/debugging-tool-under-chrome-itsgoingdclockwork/1976)

## 利用 Clockwork 在 Google Chrome 下调试你的 Laravel APP ——itsgoingd/clockwork

[itsgoingd/clockwork](https://github.com/itsgoingd/clockwork) 是浏览器下的调试工具，与 `laravel-debugbar` 类似，不过侵入性更低。

`itsgoingd/clockwork` 不会对页面产生任何影响，它是在服务器端收集 PHP 代码运行过程中产生的性能指标数据，并以 Json 保存在服务器端；浏览器插件 `clockwork` 会访问该数据，展示在开发者工具的 `clockwork` 标签中。

所以首先需要在浏览器端安装 `clockwork` 插件，才能正常使用该扩展包，Chrome 和 Firefox 都可以直接安装该插件。

## 安装

### 扩展包

首先需要在项目中安装扩展包：


    $ composer require itsgoingd/clockwork


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/XodV96kuh2.png)


### 插件:

- 安装 [Chrome 插件](https://chrome.google.com/webstore/detail/clockwork/dmggabnehkmmfmdffgajcflpdjlnoemp)，打开连接即可直接安装。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/2hfgaMFLTH.png)


如果你的网络有问题，可以点击 [这里](https://pan.baidu.com/s/1Alt0z9cQSVta7UWnGI_nYA) 下载插件到本地，然后在 Chrome 中打开 chrome://extensions/ ，将文件拖拽进去即可。
	
访问 http://larabbs.test/ 打开开发者工具，即可看到 `clockwork` 标签：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/nwLpMvbinc.png)


- 安装 [Firefox 插件](https://addons.mozilla.org/en-US/firefox/addon/clockwork-dev-tools/)，打开连接即可直接安装。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/r1fpGLvaZs.png)

	
访问 http://larabbs.test/ 打开开发者工具，即可看到 `clockwork` 标签：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/m5UJXztsJ4.png)


## 使用

`itsgoingd/clockwork` 安装完成后便会在服务器端记录请求相关的日志信息，在浏览器插件中打开即可看到：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/fwYQ4LnGM9.png)


插件中显示的内容大家应该很熟悉，非常像我们经常使用的 `laravel-debugbar`。左侧为请求记录，右侧记录了：

    - Request —— 请求信息；
    - Timeline —— 时间线；
    - Log —— 日志：
    - Events ——触发的事件；
    - Database ——数据库查询记录；
    - Cache —— 缓存使用情况；
    - Session —— Session 记录；

`itsgoingd/clockwork` 还提供了另一种特殊的访问方式，直接打开 [http://larabbs.test/\__clockwork ](http://larabbs.test/__clockwork) 便会看到，上次访问的地址的日志信息，比如我们先访问 http://larabbs.test/users/1 ，在打开 [http://larabbs.test/\__clockwork ](http://larabbs.test/__clockwork)

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/ckjKByUwhB.png)


这种方式显然没有浏览器插件方便，或许你在某些特殊情况下能够用到。

### 添加日志

`itsgoingd/clockwork` 的日志中不仅显示了 Laravel 自身的日志内容 `storage/logs/laravel.log`，还会显示自己的日志，可以借助 `clock` 方法添加日志。

*app/Http/Controllers/TopicsController.php*

    .
    .
    .
        public function index(Request $request, Topic $topic, User $user, Link $link)
        {
            logger('logger log');
            clock('clock log');
            $topics = $topic->withOrder($request->order)->paginate(20);
            $active_users = $user->getActiveUsers();
            $links = $link->getAllCached();
    
            return view('topics.index', compact('topics', 'active_users', 'links'));
        }
    .
    .
    .


上面的代码中，我们分别通过 `logger` 和 `clock` 添加了日志，查看日志内容：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/gF38yzx33p.png)


两个日志都会显示，不过 `clock` 不会记录在 `storage/logs/laravel.log` 文件中，而是记录在了自己的 `storage/clockwork` 目录下的某个 `json` 文件中。

### 添加时间线

Timeline 是一个比较好用的功能，可以记录某段程序，某些执行流程所花费的时间，在首页的控制器中添加一些测试代码：

*app/Http/Controllers/TopicsController.php*

    .
    .
    .
        public function index(Request $request, Topic $topic, User $user, Link $link)
        {
            clock()->startEvent('topic-index', "请求话题相关数据");
    
            $topics = $topic->withOrder($request->order)->paginate(20);
            $active_users = $user->getActiveUsers();
            $links = $link->getAllCached();
    
            clock()->endEvent('topic-index');
    
            return view('topics.index', compact('topics', 'active_users', 'links'));
        }
    .
    .
    .


通过 `clock()->startEvent` 方法可以开启一个 Timeline，第一个参数是自定义的事件名称，第二个参数是描述。在结束的地方通过 `clock()->endEvent('topic-index');` 关闭对应事件的 Timeline，这样我们能很方便的记录下 LaraBBS 首页关于数据查询所花费的时间：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/KbG9JRuoBU.png)


可以看到 `请求话题相关数据` 一共花费了 `540 ms`。

### 调整配置

`itsgoingd/clockwork` 默认情况下会自动记录每个请求的性能指标数据，这是个一件影响性能的事情，当项目上线的时候，需要调整配置 ，停止记录数据。当然我们可以通过 `php artisan vendor:publish` 将配置文件发布出来，直接修改配置文件，不过  `itsgoingd/clockwork` 已经提供了 `env` 参数，直接在 `.env` 文件中调整配置即可。

主要有一下几个配置：

*.env*

    .
    .
    .
    // 是否开启 clockwork （true/false）
    CLOCKWORK_ENABLE
    // 是否允许 HOST/__clockwork 方式访问 （true/false）
    CLOCKWORK_WEB
    // 过期时间，自动清理过期时间之前的文件（单位：分钟）
    CLOCKWORK_STORAGE_EXPIRATION



## 代码版本控制

使用 git status 查看代码状态：

    $ git status


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/tt3HIf540M.png)


发现 `storage/clockwork` 文件没有被忽略，这些记录文件是不需要提交到代码仓库的，所以修改 `.gitignore` 文件忽略这个目录：

*.gitignore*

    .
    .
    .
    /storage/clockwork/

再次使用 `git status` 查看代码状态，`storage/clockwork` 已被忽略。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/SVEjZKeqxs.png)




