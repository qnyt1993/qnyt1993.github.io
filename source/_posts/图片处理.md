---
title: 图片处理——intervention/image
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-17 14:49:57
---

> 转载: [056. 图片处理——intervention/image](https://learnku.com/courses/laravel-package/2019/picture-processing-interventionimage/2493)

如果你的项目中有图片处理相关的功能，相信你一定会安装 [intervention/image](https://github.com/Intervention/image) 这个扩展包，它非常的强大而且方便使用，这也是一个基础的扩展包，之后的课程会介绍一些其他关于图片处理的场景，大多都是利用了这个扩展包，为了方便接下来课程的学习，我们先来快速了解一下 `intervention/image`。

## 安装

安装的文档在这里 http://image.intervention.io/getting_started/installation 。

先安装扩展包：


    $ composer require intervention/image


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/Ep8is0nKWZ.png)


发布配置文件：

    $ php artisan vendor:publish --provider="Intervention\Image\ImageServiceProviderLaravel5"


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/Ai40pRa2NU.png)


最后 `intervention/image` 提供了两种图片驱动，一种是 [GD](http://php.net/manual/zh/book.image.php) 扩展，一个是 [Imagick](http://php.net/manual/zh/book.imagick.php) 扩展，默认使用的是 GD，不过 Imagick 处理的更快，质量更好，我们来安装一下 Imagick。

Homestead 是个 Ubuntu 的虚拟机，所以直接使用 `apt-get` 安装即可：


    $ sudo apt-get install php-imagick

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/ATkFnY3Del.png)



可以使用 `php -m | grep imagick` 查看扩展安装情况，记得重启一下 php-fpm。


    $ sudo service php7.2-fpm restart


查看以下配置文件，只有一个配置，就是切换使用的驱动，我们这里切换到 `Imagick`。

*config/image.php*

    <?php
    return [
        'driver' => 'imagick'
    ];


## 使用

查看以下文档，你会发现扩展包提供了非常多的方法用于图片处理，基本上涵盖了我们所有对于图片处理的需求，这里不可能一个一个介绍，我们从一个真实的场景出发，先学习一些基本功能。

### 图片上传

项目中应该都会遇到图片上传的需求，经常会当度写一个图片资源的接口。先来创建一个图片上传的控制器。


    $ php artisan make:controller Api/ImageController


增加一个对应的接口，用于图片上传。
*routes/api.php*

    .
    .
    .
    Route::post('images', 'Api\ImageController@store')->name('api.images.store');


 我们来尝试完成图片上传的逻辑:
 
*app/Http/Controllers/Api/ImageController.php*
 
     <?php
    
    namespace App\Http\Controllers\Api;
    
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class ImageController extends Controller
    {
        public function store(Request $request)
        {
            $path = $request->file('image')->store('images', 'public');
    
            return response()->json(['path' => \Storage::disk('public')->url($path)]);
        }
    }
 
 
获取表单提交过来的 imag 文件，存储在 public 目录中，记得执行 `storage:link` 设置软连接。最后我们返回文件的 URL。当然真实情况下你还需要验证参数，以及返回更多的内容，这里只是简单的实现功能。

上传一个文件测试一下：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/Mh5P9uX3P5.png)


### 压缩长宽调整

上传的图片大小不一，某些情况下需要统一图片的长宽，利用 `Intervention/image` 可以方便的实现，可以先看看 resize 方法的文档  http://image.intervention.io/api/resize 。

*app/Http/Controllers/Api/ImageController.php*
 
    .
    .
    .
    use Image;
    use Storage;
    .
    .
    .
        public function store(Request $request)
        {
            $path = $request->file('image')->store('images', 'public');
    
            Image::make('storage/'.$path)
               ->resize(400, null, function($constraint) {
                   $constraint->aspectRatio();
                   $constraint->upsize();
               })->save();
    
            return response()->json(['path' => Storage::disk('public')->url($path)]);
        }
    .
    .
    .
 
 
 使用 Image::make 方法来实例化一个对象，参数是一个图片的路径，可以使用相对路径，相对于项目的 public 目录。然后调用 resize 方法调整图片大小，这里可以规定图片最大宽度为 400，高度自动调整，如果图片小于 400，那么保持图片原有的尺寸。


测试一下，可以看到图片被压缩到了制定的宽度。
![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/tWOxx9On2w.png)

 
 ### 上传 base64 格式图片
 
 有的时候前端会上传 Base64 格式的图片，需要将提交过来的 base64 的字符串，存储为一个图片。
 
 *app/Http/Controllers/Api/ImageController.php*
 
     .
     .
     .
            if ($request->image_content) {
                $path = 'images/'.str_random(40).'.png';
    
                $image = Image::make($request->image_content)
                    ->resize(400, null, function($constraint) {
                        $constraint->aspectRatio();
                        $constraint->upsize();
                    })->save('storage/'.$path);
            } else {
                $path = $request->file('image')->store('images', 'public');
    
                $image = Image::make('storage/'.$path)
                    ->resize(400, null, function($constraint) {
                        $constraint->aspectRatio();
                        $constraint->upsize();
                    })->save();
            }
    .
    .
    .
 
 
 可以判断一下提交的数据是否有 `image_content`，如果有，则使用  `Image::make` 实例化这个图片，然后调用相同的逻辑调整图片大小，最后调用 save 方法保存到指定的地方。你会看到代码有一些重复的地方，你可以尝试着优化一下。


### 打水印

接着来尝试着给图片打个水印。其实很简单，使用 `insert` 方法即可，可以将一张图片插入另一张图片，比如我们将某个小图标放在图片的右上角，这里有个 laravel 的 icon 图片 https://laravel.com/favicon.png ：

 *app/Http/Controllers/Api/ImageController.php*

    .
    .
    .
            $image->insert('https://laravel.com/favicon.png', 'top-right', 10, 10)->save();
    .
    .
    .


我们在最后调用 insert 方法，可以传入图片的链接，或者图片地址，或者图片的内容，第二个参数是图片的位置，我们放在右上角，边距为10 ，最后调用 save 保存。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/XjBT5SDXJn.png)



再次测试你应该能看到水印了。

