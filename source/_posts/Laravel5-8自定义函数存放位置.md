---
title: Laravel5.8自定义函数存放位置
date: 2019-09-23 21:42:24
categories: [后端,php,Laravel]
tags: [php,laravel] 
---


#### 1. 创建文件 app/helpers.php

    <?php
    
    // 示例函数
    function foo() {
        return "foo";
    }
    
#### 2. 修改项目 `composer.json`
在项目 `composer.json` 中 `autoload` 部分里的 `files` 字段加入该文件即可：

    {
        ...
    
        "autoload": {
            "files": [
                "app/helpers.php"
            ]
        }
        ...
    }
    
#### 3. 然后运行:

    $ composer dump-autoload
    
OK，然后你就可以在任何地方用到 `app/helpers.php` 中的函数了。

#### 4. 转载

[ Laravel 目录结构：自定义函数的存放位置](https://learnku.com/laravel/wikis/15903)