---
title: 哈希数据 id -- vinkla/hashids
categories: [后端,php,Laravel]
tags: [php,laravel] 
date: 2019-10-14 10:52:41
---

> 转载: [哈希数据 ID —— vinkla/hashids](https://learnku.com/courses/laravel-package/2019/hash-data-id-vinklahashids/1945)

## 哈希数据 ID —— vinkla/hashids

普通项目中，我们使用的数据 ID 通常为数据库的自增 ID，自增 ID 有一些问题：

- 数据的 ID 是暴露的；
- 有可能被别人恶意采集；
- 容易根据 ID 猜测数项目中的数据量；
- 随着数据的增长，ID 会越来越大，无法统一长度。

当然解决方案有很多种，例如 UUID，使用自己生成的 ID 等。今天我们来了解一下一种方便的解决方案——  [vinkla/hashids](https://github.com/vinkla/laravel-hashids) 。

[vinkla/hashids](https://github.com/vinkla/laravel-hashids) 是 [hashids](https://hashids.org/) 在 Laravel 中的封装，[hashids](https://hashids.org/) 是一个可以通过数字生成简短，唯一，无序字符串的开源库，它不仅能将数字转换为字符串，还能将转换后的结果转换回数字，利用这一点，我们可以很好的解决上述问题。

## 场景分析

打开 LaraBBS 的某个用户详情页面，例如 http://larabbs.test/users/4 。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/smXA6gO5fy.png)

我们现在希望用户的 ID 无序，长度统一，不直接暴露。所以需要通过 `vinkla/hashids` 将用户 ID 装换为字符串显示在 URL 中。为了实现这样的功能，先来想想我们需要做些什么：

- 将所有用户相关的链接地址由  `/users/4` 修改为 `hashids` 转换后的值 `/users/N1QLZOoGM`，也就是任何有关用户的链接都需要修改；
- 我们获取的参数是一个字符串，在用户查询之前需要将字符串转换回用户 ID，然后再查询用户；
- 由于现在是通过 [路由模型绑定](https://learnku.com/docs/laravel/5.5/routing/1293#route-model-binding) 的方式获取的用户，我们要重新实现一下模型绑定的中间件——bindings? 或者是不使用这个中间件，所有用户查询的地方都自己实现；
- 我们将来不仅仅会使用在用户相关的 URL 中，其他数据也有可能做类似的处理，得考虑代码封装和重用。

乍一想你可能会觉得实现起来会很麻烦，有太多的地方需要修改，工作量似乎有些大。不过大家请放心，我们使用的是 `Laravel` 框架，真正实现起来其实很简单。

## 安装

首先来安装这个扩展包


     composer require vinkla/hashids

发现安装过程中报错了，学会看报错是成为一个优秀程序员的必备条件，通过报错我们很快能定位到是我们自身环境的问题还是这个扩展包的问题。例如报错中有一句很重要的话：

> vinkla/hashids 5.0.0 requires illuminate/contracts 5.6.*

`5.0.0` 版本的 `vinkla/hashids` 需要 `5.6.*` 版本的 `illuminate/contracts`，也就是需要 `5.6` 版本的 `Laravel`。因为我们安装 `vinkla/hashids` 的时候并未指定版本，所以默认安装最新的稳定版 `5.0.0`，这个版本并不适配我们当前的 Laravel 5.5。

得到了这些信息，我们应该去 [项目](https://github.com/vinkla/laravel-hashids) 中找找适配 Laravel 5.5 的版本，最后发现我们应该安装 `3.3` 版版本 `vinkla/hashids`。

安装并且指定版本：


    composer require vinkla/hashids:~3.3


安装成功后需要将 `vinkla/hashids` 的配置文件发布出来：

    php artisan vendor:publish --provider="Vinkla\Hashids\HashidsServiceProvider"


`vinkla/hashids` 的思路是将数字转换为字符串，同时又可以将字符串再转换回数字。为了保证字符串不被轻易的破解，我们需要配置盐（`salt`），也就是给目标数字额外增加一些变量，保证他人在没有相同盐（`salt`）的情况下，无法将符串再转换回数字，增加了安全性。同时我们还可以指定转换后字符串的最小长度，这样能保证 ID 尽量统一美观。

在配置文件中修改 `salt` 和 `length` 信息

*config/hashids.php*


    .
    .
    .
        'default' => 'main',
        'connections' => [
    
            'main' => [
                'salt' => 'larabbs',
                'length' => '9',
            ],
            
            'alternative' => [
                'salt' => 'your-salt-string',
                'length' => 'your-length-integer',
            ],
        ]
    .
    .
    .


`vinkla/hashids` 提供了多套配置信息，默认使用的是 `main` 中的配置，方便我们根据不同场景做不同的转换。我们现在的需求使用默认的即可，所以修改 `main` 中的 `salt` 为 `larabbs`, 修改 `lentgh` 为 `9`。

## 使用

### 测试
我们先使用 Tinker 来验证一下相关的功能是否好用，打开 Tinker：


    php artisan tinker


输入如下内容：

    $hashId = Hashids::encode(10);
    Hashids::decode($hashId);


结果正确，可以看到对 10 `encode` 的结果为 9 位，将转换后的字符串进行 `decode` 可以正确得到数字 10，不过这里要注意，`decode` 后的结果为数组。转换后的结果为什么是数组呢？因为 `hashids` 不仅支持数字和字符串，还支持将数组进行 `encode`。

### 封装逻辑

其实我们要处理的主要有两点：

1. 通过 `route` 方法生成链接的时候，不使用用户 ID，而是 `hashids` 转换后的字符串；
	
	我们通常都会通过 `route` 方法生成链接，例如 `route('users.show',  $user->id)` ，生成用户详情页的链接，结果为 http://larabbs.test/users/4 ，但是其实我们可以**将 Eloquent 模型 作为参数值传给 `route` 方法，它会自动提取模型的主键来生成 URL**，回顾一下[文档](https://learnku.com/docs/laravel/5.5/urls/1300#7a2a59)。看一下生成 URL 的代码：
	
	*vendor/laravel/framework/src/Illuminate/Routing/UrlGenerator.php*
	
	
        .
        .
        .
            public function formatParameters($parameters)
            {
                $parameters = Arr::wrap($parameters);
    
                foreach ($parameters as $key => $parameter) {
                    if ($parameter instanceof UrlRoutable) {
                        $parameters[$key] = $parameter->getRouteKey();
                    }
                }
    
                return $parameters;
            }
        .
        .
        .

---

	
如果传入的参数实现了 `UrlRoutable` 接口，那么就会通过 `getRouteKey` 方法获取参数的值。Eloquent 当然已经实现了 `UrlRoutable` 接口，`getRouteKey` 返回的就是模型的主键 `primaryKey`，也就是 `id` 字段。
	
所以只需要在模型中重写 `getRouteKey` 方法，返回 `hashids` 转换后的字符串，生成 URL 的时候就能获得想要的结果。
	
2. 路由模型绑定的查询用户的时候，先将字符串转换为用户 ID，再进行查询。
	
	路由模型绑定是 `Laravel` 提供的中间件，为了在数据查询前先将字符串转换回 ID，难道要自己实现一个中间件吗？当然不用，中间件通过参数查找模型的时候会调用模型的 `resolveRouteBinding` 方法，当然这个方法 Eloquent 默认也是通过模型主键查询的，可以看一下源码：
	
	*vendor/laravel/framework/src/Illuminate/Database/Eloquent/Model.php*
	
	
	
	.
	.
	.
	public function resolveRouteBinding($value)
    {
        return $this->where($this->getRouteKeyName(), $value)->first();
    }
	.
	.
	.
	
	
同样我们在模型中重写 `resolveRouteBinding` 方法，先进行 `ID` 转换即可。


既然这两步都可以通过在模型中重写来实现，那么就可以为模型实现一个 `Trait`，来实现 `URL` 生成以及路由模型绑定自动使用 `hashids` 的功能。是为模型实现的 `Trait`，当然需要放在 `Models/Traits` 目录中：

创建一个 Trait 文件。
```
$ touch app/Models/Traits/HashIdHelper.php
```

填入如下内容：

*app/Models/Traits/HashIdHelper.php*


    <?php
    
    namespace App\Models\Traits;
    
    use Hashids;
    
    trait HashIdHelper
    {
        private $hashId;
    
        // 调用 $model->hash_id 时触发
        public function getHashIdAttribute()
        {
            if (!$this->hashId) {
                $this->hashId = Hashids::encode($this->id);
            }
    
            return $this->hashId;
        }
    
        // 先将参数 decode 为模型id，再调用父类的 resolveRouteBinding 方法
        public function resolveRouteBinding($value)
        {
            $value = current(Hashids::decode($value));
            if (!$value) {
                return;
            }
            return parent::resolveRouteBinding($value);
        }
    
        // 使用 hash_id 生成 URL
        public function getRouteKey()
        {
            return $this->hash_id;
        }
    }


我们重写了 `resolveRouteBinding` 和 `getRouteKey` 两个方法。其中 `getHashIdAttribute` 是我们定义的一个访问器，当访问模型的 `hash_id` 属性时会通过该方法返回数据，如果你对这部分代码有疑问，可以再看看文档中的[修改器](https://learnku.com/docs/laravel/5.5/eloquent-mutators/1335#f3b389) 这部分 。逻辑很简单，可以根据注释再理解一下代码逻辑。这样任何使用了这个 Trait 的 Eloquent 模型都能直接使用 `hashids` 了。

在用户模型中使用这个 `Trait`：
*app\Models\User.php*

    .
    .
    .
    class User extends Authenticatable implements JWTSubject
    {
        use Traits\HashIdHelper;
    .
    .
    .


因为 LaraBBS 中使用 `route` 的地方大部分情况下是直接传入的 `id`，而不是传入模型实例，所以我们先来修改首页的两处链接：

*resources/views/topics/topiclist.blade.php*


    .
    .
    .
                    <div class="media-left">
                        <a href="{{ route('users.show', $topic->user) }}">
                            <img class="media-object img-thumbnail" style="width: 52px; height: 52px;" src="{{ $topic->user->avatar }}" title="{{ $topic->user->name }}">
                        </a>
                    </div>
                    .
                    .
                    .
                            <span> • </span>
                            <a href="{{ route('users.show', $topic->user) }}" title="{{ $topic->user->name }}">
                                <span class="glyphicon glyphicon-user" aria-hidden="true"></span>
                                {{ $topic->user->name }}
                            </a>
    .
    .
    .


直接使用 `route('users.show', $topic->user)` 生成用户详情链接。

### 测试
打开首页 http://larabbs.test/topics ，观察话题用户相关的链接。
已经变成了我们想要的样子。访问某个用户详情：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/ObAz1HcTas.png)

用户详情可以正确访问，再次通过用户 ID 访问用户详情，例如 http://larabbs.test/users/4 。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/6wEU6PlYls.png)


已经不能通过用户 ID 访问详情页面了。

### 兼容性修改

仅仅增加了一个 `Trait`，我们就完成了整个功能，不过需要注意的是，因为我们重写了用户模型的生成 `URL` 和路由模型绑定的相关方法，这是全局的修改，需要将所有使用 `route` 方法生成用户相关 `URL` 的地方都修改为传入用户模型。我们只修改了首页列表，还有例如 `首页活跃用户`，`用户个人信息`，`用户个人信息编辑`，`回复列表`，`管理后台` 等等地方都涉及到用户 `URL`。

可以通过 Sed 命令或者编辑器批量替换的方法将这些地方统一修改，不过依然需要做一些功能测试。这其实也告诉我们，尽量使用对象，当我们做一些统一修改的时候会更加的便利。

下面我们可以做一些兼容性的修改：

*app/Models/Traits/HashIdHelper.php*

    .
    .
    .
        public function resolveRouteBinding($value){
            if (!is_numeric($value)) {
                $value = current(Hashids::decode($value));
                if (!$value) {
                    return;
                }
            }
            return parent::resolveRouteBinding($value);
        }
    .
    .
    .


增加了 `is_numeric` 的判断，当参数不是数字的时候，才通过 `hashids` 转换，这样现在所有的链接都是可以正确访问的，当做完了相关链接修正，进行了测试之后即可恢复上述代码。

