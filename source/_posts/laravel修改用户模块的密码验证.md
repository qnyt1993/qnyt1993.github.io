---
title: laravel修改用户模块的密码验证
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-16 17:28:15
---


>做项目的时候，用户认证几乎是必不可少的，如果我们的项目由于一些原因不得不使用 `users` 之外的用户表进行认证，那么就需要多做一点工作来完成这个功能。

>现在假设我们只需要修改登录用户的表，表名和表结构都与框架默认的表`users`不同，文档没有教我们如何去做，但是别慌，稍微看下框架实现用户认证的源码就能轻松实现。

首先，自定义一张表用来登录，表结构和模拟数据如下：
> 表 admins

|  id |  login_name | login_pass |
| -------- | -------- | -------- |
| 1     | admin     | $2y$10$2MUhp7b6ghVOngb/.b/x6uuEW/yL3FqPKJztawrM0U577Clf07xda     |

#### 从配置文件入手

用户认证相关的配置都保存在`config/auth.php`文件中，先来看看配置文件的内容：

        <?php
        
        return [
        
            /*
            |--------------------------------------------------------------------------
            | Authentication Defaults
            |--------------------------------------------------------------------------
            |
            | This option controls the default authentication "guard" and password
            | reset options for your application. You may change these defaults
            | as required, but they're a perfect start for most applications.
            |
            */
        
            'defaults' => [
                'guard' => 'web',
                'passwords' => 'users',
            ],
        
            /*
        |--------------------------------------------------------------------------
        | Authentication Guards
        |--------------------------------------------------------------------------
        |
        | Next, you may define every authentication guard for your application.
        | Of course, a great default configuration has been defined for you
        | here which uses session storage and the Eloquent user provider.
        |
        | All authentication drivers have a user provider. This defines how the
        | users are actually retrieved out of your database or other storage
        | mechanisms used by this application to persist your user's data.
        |
        | Supported: "session", "token"
        |
        */
    
        'guards' => [
            'web' => [
                'driver' => 'session',
                'provider' => 'users',
            ],
    
            'api' => [
                'driver' => 'passport',
                'provider' => 'users',
            ],
        ],
    
        /*
        |--------------------------------------------------------------------------
        | User Providers
        |--------------------------------------------------------------------------
        |
        | All authentication drivers have a user provider. This defines how the
        | users are actually retrieved out of your database or other storage
        | mechanisms used by this application to persist your user's data.
        |
        | If you have multiple user tables or models you may configure multiple
        | sources which represent each model / table. These sources may then
        | be assigned to any extra authentication guards you have defined.
        |
        | Supported: "database", "eloquent"
        |
        */
    
        'providers' => [
            'users' => [
                'driver' => 'eloquent',
                'model' => App\User::class,
            ],
    
            // 'users' => [
            //     'driver' => 'database',
            //     'table' => 'users',
            // ],
        ],
    
        /*
        |--------------------------------------------------------------------------
        | Resetting Passwords
        |--------------------------------------------------------------------------
        |
        | You may specify multiple password reset configurations if you have more
        | than one user table or model in the application and you want to have
        | separate password reset settings based on the specific user types.
        |
        | The expire time is the number of minutes that the reset token should be
        | considered valid. This security feature keeps tokens short-lived so
        | they have less time to be guessed. You may change this as needed.
        |
        */
    
        'passwords' => [
            'users' => [
                'provider' => 'users',
                'table' => 'password_resets',
                'expire' => 60,
            ],
        ],
    
    ];


默认使用的守卫是`web`，而`web`守卫使用的认证驱动是`session`，用户提供器是`users`。假设我们的需求只是将用户的提供器由`users`改为`admins`，那么我们需要做两步操作：
* 修改默认的用户提供器，将`provider=>'users'`改为`provider=>'admins'`


          'guards' => [
                'web' => [
                    'driver' => 'session',
                    'provider' => 'users',
                ],
            ],
            

* 配置`admins`提供器，假设依旧使用`eloquent`作为驱动，并创建好了`admins`表的模型

    'providers' => [
            'admins' => [
                'driver' => 'eloquent',
                'model' => App\Admin::class
            ]
        ],


#### 使用`Auth`门面的`attempt`方法进行登录
`SessionGuard` 中的`attempt`方法：

    //Illuminate\Auth\SessionGuard
     public function attempt(array $credentials = [], $remember = false)
        {
            $this->fireAttemptEvent($credentials, $remember);
    
            $this->lastAttempted = $user = $this->provider->retrieveByCredentials($credentials);
    
            // If an implementation of UserInterface was returned, we'll ask the provider
            // to validate the user against the given credentials, and if they are in
            // fact valid we'll log the users into the application and return true.
            if ($this->hasValidCredentials($user, $credentials)) {
                $this->login($user, $remember);
    
                return true;
            }
    
            // If the authentication attempt fails we will fire an event so that the user
            // may be notified of any suspicious attempts to access their account from
            // an unrecognized user. A developer may listen to this event as needed.
            $this->fireFailedEvent($user, $credentials);
    
            return false;
        }

该方法中调用 `UserProvider` 接口的`retrieveByCredentials`方法检索用户，根据我们的配置，`UserProvider`接口的具体实现应该是`EloquentUserProvider`，因此，我们定位到`EloquentUserProvider`的`retrieveByCredentials`方法：

    //Illuminate\Auth\EloquentUserProvider
    public function retrieveByCredentials(array $credentials)
        {
            if (empty($credentials) ||
               (count($credentials) === 1 &&
                array_key_exists('password', $credentials))) {
                return;
            }
    
            // First we will add each credential element to the query as a where clause.
            // Then we can execute the query and, if we found a user, return it in a
            // Eloquent User "model" that will be utilized by the Guard instances.
            $query = $this->createModel()->newQuery();
    
            foreach ($credentials as $key => $value) {
                if (Str::contains($key, 'password')) {
                    continue;
                }
    
                if (is_array($value) || $value instanceof Arrayable) {
                    $query->whereIn($key, $value);
                } else {
                    $query->where($key, $value);
                }
            }
    
            return $query->first();
        }

该方法会使用传入的参数（不包含`password`）到我们配置的数据表中搜索数据，查询到符合条件的数据之后返回对应的用户信息，然后`attempt`方法会进行密码校验，校验密码的方法为：

    //Illuminate\Auth\SessionGuard
    /**
         * Determine if the user matches the credentials.
         *
         * @param  mixed  $user
         * @param  array  $credentials
         * @return bool
         */
        protected function hasValidCredentials($user, $credentials)
        {
            return ! is_null($user) && $this->provider->validateCredentials($user, $credentials);
        }

进一步查看`EloquentUserProvider`中的`validateCredentials`方法

    //Illuminate\Auth\EloquentUserProvider
    public function validateCredentials(UserContract $user, array $credentials)
    {
        $plain = $credentials['password'];
    
        return $this->hasher->check($plain, $user->getAuthPassword());
    }

通过`validateCredentials`可以看出，提交的认证数据中密码字段名必须是`password`，这个无法自定义。同时可以看到，入参`$user`必须实现`Illuminate\Contracts\Auth\Authenticatable`接口（`UserContract`是别名）。

####  修改 `Admin` 模型
Admin模型必须实现`Illuminate\Contracts\Auth\Authenticatable`接口，可以借鉴一下`User`模型，让`Admin`直接继承`Illuminate\Foundation\Auth\User` 就可以，然后重写`getAuthPassword`方法，正确获取密码字段：
    
    // App\Admin
    public function getAuthPassword()
    {
        return $this->login_pass;
    }

不出意外的话，这个时候就能使用`admins`表进行登录了。

---

Larval 5.4的默认Auth登陆传入邮件和用户密码到attempt 方法来认证，通过email 的值获取，如果用户被找到，经哈希运算后存储在数据中的password将会和传递过来的经哈希运算处理的passwrod值进行比较。如果两个经哈希运算的密码相匹配那么将会为这个用户开启一个认证Session。

参考上面的分析，我们就需要对`EloquentUserProvider`中的`validateCredentials`方法进行重写,步骤如下

#### 1. 修改 App\Models\User.php 添加如下代码

    
    public function getAuthPassword()
        {
            return ['password' => $this->attributes['password'], 'salt' => $this->attributes['salt']];
        }


#### 2. 建立一个自己的UserProvider.php 的实现


    <?php namespace App\Foundation\Auth;
    
    use Illuminate\Auth\EloquentUserProvider;
    use Illuminate\Contracts\Auth\Authenticatable;
    use Illuminate\Support\Str;
    
    /**
     * 重写用户密码校验逻辑
     * Class GfzxEloquentUserProvider
     * @package App\Foundation\Auth
     */
    class GfzxEloquentUserProvider extends EloquentUserProvider
    {
        /**
         * Validate a user against the given credentials.
         *
         * @param  \Illuminate\Contracts\Auth\Authenticatable $user
         * @param  array $credentials
         * @return bool
         */
        public function validateCredentials(Authenticatable $user, array $credentials)
        {
            $plain = $credentials['password'];
            $authPassword = $user->getAuthPassword();
            return md5($plain . $authPassword['salt']) == $authPassword['password'];
        }
    }


#### 3. 将User Providers换成我们自己的GfzxEloquentUserProvider

修改 `app/Providers/AuthServiceProvider.php`


    <?php
    
    namespace App\Providers;
    
    use App\Foundation\Auth\GfzxEloquentUserProvider;
    use Auth;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    
    class AuthServiceProvider extends ServiceProvider
    {
        .
        .
        .
    
        /**
         * Register any authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();
    
            Auth::provider('gfzx-eloquent', function ($app, $config) {
                return new GfzxEloquentUserProvider($this->app['hash'], $config['model']);
            });
        }
    }



#### 4. 修改 config/auth.php


       'providers' => [
            'users' => [
                'driver' => 'gfzx-eloquent',
                'model' => App\Models\User::class,
            ],
        ],



这是就可以用过salt+passwrod的方式密码认证了

## 文章参考

[laravel 修改用户模块密码验证](https://blog.csdn.net/wepe12/article/details/78749326)

[Laravel 中自定义用户登录的数据表](https://learnku.com/index.php/articles/27806)
