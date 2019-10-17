---
title: 通过IP获取到对应的地理位置信息
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-17 09:39:57
---

>转载: [通过 IP 获取到对应的地理位置信息——torann/geoip](https://learnku.com/courses/laravel-package/2019/get-the-corresponding-geo-location-information-through-ip-toranngeoip/2024)

[torann/geoip](https://github.com/Torann/laravel-geoip) 是一个通过 IP 获取到对应的地理位置信息的扩展包，有的时候我们需要记录或者显示用户的地理位置信息，那么这个扩展包就非常有用了。

## 场景说明

在 LaraBBS 中我们可以尝试完成在  `footer` 中显示当前用户 IP 对应的位置信息。

## 安装

首先可以看一下扩展包的 [文档](http://lyften.com/projects/laravel-geoip/doc/)，能够快速的了解这个扩展包。


    $ composer require torann/geoip

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/SJHTvv9T78.png)


将配置文件发布出来：

    $ php artisan vendor:publish --provider="Torann\GeoIP\GeoIPServiceProvider" --tag=config


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/4DaCxfVcae.png)


> 注意如果不发布配置文件会遇到报错： `Exception with message 'The GeoIP service is not valid.'`

## 使用

`torann/geoip` 使用起来非常方便，它已经提供了辅助方法和 Facade：

    - `geoip($ip);` ；
    - `GeoIp::getLocation($ip)`。

上面两种方式效果相同，都会根据传入的 IP 返回 `\Torann\GeoIP\Location` 对象。这个对象包含了对应的位置信息。

### 调试

先来 `tinker` 中测试一下：

`php artisan tinker` 打开 Tinker：


    $ip = '119.4.121.109';
    geoip($ip)->toArray();


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/Jvkj64w9Ww.png)


可以看到 IP 对应的地址信息了，但是返回的信息都是英文的，我们需要中文的数据，方便显示。

### 修改为中文

修改语言只需要修改 `geoip.php` 中的配置即可，将 `services.ipapi.lang` 修改为 `zh-CN`：

*config/geoip.php*

	'services' => [
	.
	.
	.
        'ipapi' => [
            'class' => \Torann\GeoIP\Services\IPApi::class,
            'secure' => true,
            'key' => env('IPAPI_KEY'),
            'continent_path' => storage_path('app/continents.json'),
            'lang' => 'zh-CN',
        ],
	]


关闭 `tinker`，任何的代码修改都需要重启 `tinker` 才会生效，同时使用 `cache:clear` 清除缓存，再次打开 tinker，运行上述调试代码：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/oKPIA0vcm5.png)


正确的返回了中文数据。

### 修改页面显示

调试成功以后可以开始开发我们的功能了，我们需要再全局的 `footer` 视图中显示当前用户的位置信息，那么就需要将数据提供给 `footer` 视图，这时候就可以利用 [视图合成器]( https://learnku.com/docs/laravel/5.6/views/1369#b492db) 了，不是太了解的同学可以看一下文档。

首先创建  `ComposerServiceProvider`：

    $ php artisan make:provider ComposerServiceProvider

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/TtWe2QvX57.png)


将 ComposerServiceProvider 增加到 `config/app.php` 中:
*config/app.php*

    .
    .
    .
            App\Providers\EventServiceProvider::class,
            App\Providers\RouteServiceProvider::class,
            App\Providers\ComposerServiceProvider::class,
    .
    .
    .


在 `ComposerServiceProvider` 的 `boot` 方法中为视图提供数据：
*app/Providers/ComposerServiceProvider.php*

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Request;
    
    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap the application services.
         *
         * @return void
         */
        public function boot(Request $request)
        {
            // 在 footer 视图中绑定 location 数据
            view()->composer('layouts._footer', function($view) use ($request) {
                $location = geoip($request->ip());
                $view->with('location', $location->country.' - '.$location->state_name.' - '.$location->city);
            });
        }
    }


我们为 `layouts._footer` 提供了数据 `location`，通过 `$request->ip()` 即可获取当前用户的 IP，然后通过 `geoip` 获取用户的位置信息，最终将 `country`，`state_name`，`city` 三个值拼接赋值给 `$location`。

修改 footer 页面：

*resources/views/layouts/footer.blade.php*

    <footer class="footer">
        <div class="container">
        .
        .
        .
            <p class="pull-right"><a href="mailto:{{ setting('contact_email') }}">联系我们</a></p>
    
            <p class="pull-right" style="margin-right: 20px"> 来自: {{$location}}</p>
        </div>
    </footer>


只是在 最后增加了一行数据  `<p class="pull-right" style="margin-right: 20px"> 来自: {{$location}}</p>` 直接显示了 `$location`。

打开 http://larabbs.test/ 查看底部的 `footer`:

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/JL4N3IQRM8.png)


显示了数据，但是显示的是数据貌似有点问题，这是因为我们在本地的虚拟机中调试，并没有真实的 IP，所以 `torann/geoip` 显示了一个默认的位置信息，这个信息配置在了 `geoip.default_location` 中，我们可以修改一下默认的配置：

*config/geoip.php*

    .
    .
    .
        'default_location' => [
            'ip' => '127.0.0.0',
            'iso_code' => 'CN',
            'country' => '中国',
            'city' => '成都(本地调试)',
            'state' => 'SC',
            'state_name' => '四川省',
            'postal_code' => '',
            'lat' => null,
            'lon' => null,
            'timezone' => 'Asia/Shanghai',
            'continent' => 'NA',
            'default' => true,
            'currency' => 'CNY',
        ],
    .
    .
    .


刷新 http://larabbs.test/ 显示了我们配置的默认信息。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/jxpqAQyIpC.png)


## 使用其他 Service
`torann/geoip` 提供了三种根据 IP 获取位置信息的服务：

    - ipapi —— 默认的方式，利用了 http://ip-api.com/ 的接口来获取位置信息；
    - maxmind_database —— 由 [www.maxmind.com](https://www.maxmind.com/) 提供的开源数据，需要下载数据到本地；
    - maxmind_api —— 利用 [www.maxmind.com](https://www.maxmind.com/) 提供的接口获取位置信息，需要付费，本文不做讲解。

上文中我们使用了默认的方式 `ipapi`，也就是请求了 http://ip-api.com/ 的接口，那么如果处于某些原因，无法请求到接口，那么就可以使用 `maxmind_database` 的方式，将数据同步至本地，进行本地查询。

### 使用 maxmind

要使用 `maxmind_database` 需要先安装 `geoip2/geoip2` 这个扩展包。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/wQQm73abnC.png)


安装成功后，修改一下配置，将使用的 service 切换为 `maxmind_database`，同时修改使用的语言为 `zh-CN`：


    .
    .
    .
        'service' => 'maxmind_database',
        .
        .
        .
        'services' => [
            'maxmind_database' => [
                    'class' => \Torann\GeoIP\Services\MaxMindDatabase::class,
                    'database_path' => storage_path('app/geoip.mmdb'),
                    'update_url' => 'https://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz',
                    'locales' => ['zh-CN'],
                ],
    .
    .
    .


直接调用 `geoip:update` 命令可以将数据信息同步至本地：


    $ php artisan geoip:update

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/uKKbZ90mpe.png)


数据文件下载到了 `storage/app/geoip.mmdb` 中。

清空缓存，打开tinker，随便输入一个 IP 进行调试：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/1WXJU1t21w.png)


可以得到正确的结果。



