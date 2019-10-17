---
title: 条形码生成工具——milon/barcode
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-17 10:19:25
---

> 转载: [条形码生成工具——milon/barcode](https://learnku.com/courses/laravel-package/2019/bar-code-generation-tool-milonbarcode/2037)


二维码又称二维条码，种类有很多，不同的机构开发出的二维条码具有不同的结构以及编写、读取方法，常见的二维码还有：

    - PDF417 码
    - Data Matrix

二维码中的二维，表示的是在水平和垂直方向的二维空间存储信息的条形码。既然有二维码肯定就有一维码，也就是只在一个方向（一般是水平方向）表达信息，而在垂直方向则不表达任何信息。常见的一维条形码还有：

    - Code 128
    - Code 39
    - EAN 码
    - UPC 码

商品条形码格式主要分为：`EAN-13`、`EAN-8`、`UPC-A`、`UPC-E` 四种，我们在超市购物的时候，见到的商品条码，就是属于 [EAN-13](https://baike.baidu.com/item/EAN-13/1260352) 格式：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/qEBX84kF9p.png)

无论是一维码还是二维码，他们都属于 [条形码](https://zh.wikipedia.org/wiki/%E6%9D%A1%E5%BD%A2%E7%A0%81)，当我们的项目需要生成不同类别的条形码的时候，[milon/barcode](https://github.com/milon/barcode) 就是一个非常方便的工具。

`milon/barcode` 的介绍中提到了，它其实是使用了另一个项目 [TCPDF](https://github.com/tecnickcom/TCPDF) 中的类，将其封装，方便在 Laravel 中使用，它支持非常多的条码格式，能满足我们绝大部分的需求。

它支持的条码格式包括 : `CODE 39`, `ANSI MH10.8M-1983`, `USD-3`, `3 of 9`, `CODE 93`, `USS-93`, `Standard 2 of 5`, `Interleaved 2 of 5`, `CODE 128 A/B/C`, `2 and 5 Digits UPC-Based Extension`, `EAN 8`, `EAN 13`, `UPC-A`, `UPC-E`, `MSI`, `POSTNET`, `PLANET`, `RMS4CC (Royal Mail 4-state Customer Code)`, `CBC (Customer Bar Code)`, `KIX (Klant index - Customer index)`, `Intelligent Mail Barcode, Onecode`, `USPS-B-3200`, `CODABAR`, `CODE 11`, `PHARMACODE`, `PHARMACODE TWO-TRACKS`, `Datamatrix`, `QR-Code`, `PDF417`。

## 使用场景

条形码的使用需要配合项目的业务，LaraBBS 中并没有很多使用条码的场景，不过条码的生成逻辑都是一样的，根据数据 id 或者编号生成项目需要的条形码，我们可以制作一个页面，展示所有用过用户 ID 生成的不同格式的条形码。

## 安装


    $ composer require milon/barcode


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/GANHSb5lfQ.png)


安装完这个扩展包之后你会发现包自动发现好像没起作用，查看项目的 pull request，会发现其实已经有了关于包自动发现的 [提交](https://github.com/milon/barcode/pull/65) 但是作者并没有为这个提交打一个新的 tag，也就是说这个提交还在 mastre 分支上，所以我们现在安装会使用这个包的最新的稳定版 `5.3.6`，需要单独注册一下服务及 Facade。


*config/app.php*

    .
    .
    .
        'providers' => [
            .
            .
            .
            Milon\Barcode\BarcodeServiceProvider::class,
            .
            .
            .
        ]
    .
    .
    .
        'aliases' => [
    
            'View' => Illuminate\Support\Facades\View::class,
    
            // milon/barcode
            'DNS1D' => Milon\Barcode\Facades\DNS1DFacade::class,
            'DNS2D' => Milon\Barcode\Facades\DNS2DFacade::class,
        ]
    .
    .
    .



## 使用

`milon/barcode` 主要有一下几个方法用于生成条形码：

    - getBarcodeSVG —— 生成 SVG 格式的条形码；
    - getBarcodeHTML——通过 HTML 标签及样式，生成二维码；
    - getBarcodePNG ——生成 PNG 格式的条形码；
    - getBarcodePNGPath——保存 PNG 格式的条形码，默认保存在 `public` 目录中，返回文件名称；
    - getBarcodePNGUri ——保存 PNG 格式的条形码，返回文件 URL，其实只是使用 `url` 方法包裹了`getBarcodePNGPath` 方法 。

同时上面我们注册了两个 Facade，DNS1D 和 DNS2D 分别对应一维条形码和二维条形码，它们都包含上述的方法。

同时上述方法都支持 5 个参数：

    - $code —— 生成条形码的文本；
    - $type —— 条形码格式，
        
            DNS1D 支持的参数包括：
            - C39 : CODE 39 - ANSI MH10.8M-1983 - USD-3 - 3 of 9.
            - C39+ : CODE 39 with checksum
            - C39E : CODE 39 EXTENDED
            - C39E+ : CODE 39 EXTENDED + CHECKSUM
            - C93 : CODE 93 - USS-93
            - S25 : Standard 2 of 5
            - S25+ : Standard 2 of 5 + CHECKSUM
            - I25 : Interleaved 2 of 5
            - I25+ : Interleaved 2 of 5 + CHECKSUM
            - C128 : CODE 128
            - C128A : CODE 128 A
            - C128B : CODE 128 B
            - C128C : CODE 128 C
            - EAN2 : 2-Digits UPC-Based Extention
            - EAN5 : 5-Digits UPC-Based Extention
            - EAN8 : EAN 8
            - EAN13 : EAN 13
            - UPCA : UPC-A
            - UPCE : UPC-E
            - MSI : MSI (Variation of Plessey code)
            - MSI+ : MSI + CHECKSUM (modulo 11)
            - POSTNET : POSTNET
            - PLANET : PLANET
            - RMS4CC : RMS4CC (Royal Mail 4-state Customer Code) - CBC (Customer Bar Code)
            - KIX : KIX (Klant index - Customer index)
            - IMB: Intelligent Mail Barcode - Onecode - USPS-B-3200
            - CODABAR : CODABAR
            - CODE11 : CODE 11
            - PHARMA : PHARMACODE
            - PHARMA2T : PHARMACODE TWO-TRACKS
    
            // DNS2D 支持的参数包括：
            - DATAMATRIX : Datamatrix (ISO/IEC 16022)
            - PDF417 : PDF417 (ISO/IEC 15438:2006)
            - PDF417,a,e,t,s,f,o0,o1,o2,o3,o4,o5,o6 : PDF417 with parameters: a = aspect ratio (width/height); e = error correction level (0-8); t = total number of macro segments; s = macro segment index (0-99998); f = file ID; o0 = File Name (text); o1 = Segment Count (numeric); o2 = Time Stamp (numeric); o3 = Sender (text); o4 = Addressee (text); o5 = File Size (numeric); o6 = Checksum (numeric). NOTES: Parameters t, s and f are required for a Macro Control Block, all other parametrs are optional. To use a comma character ',' on text options, replace it with the character 255: "\xff".
            - QRCODE : QRcode Low error correction
            - QRCODE,L : QRcode Low error correction
            - QRCODE,M : QRcode Medium error correction
            - QRCODE,Q : QRcode Better error correction
            - QRCODE,H : QR-CODE Best error correction
    
    - $w —— 每个基本元素（一维码指每个单个的竖条，二维码只每个基本的码元）的宽度；
    - $h —— 每个基本元素（一维码指每个单个的竖条，二维码只每个基本的码元）的高度；
    - $color—— 条码颜色，默认为黑色，`getBarcodeSVG` 和 `getBarcodeHTML` 需使用字符串如 `black` ，`blue` ，联想 css 中的 `background-color`。其他方法是个数组需要传入 RGB 三个值。

增加一个路由，用于显示条形码：

*routes/web.php*

    .
    .
    .
    Route::get('users/{user}/barcode', 'UsersController@barcode')->name('users.barcode');
    Route::resource('users', 'UsersController', ['only' => ['show', 'update', 'edit']]);
    .
    .
    .


在 `UsersController` 中增加对应的 `barcode` 方法。

*app/Http/Controllers/UsersController.php*

    .
    .
    .
    use DNS1D;
    use DNS2D;
    .
    .
    .
        public function barcode(User $user)
        {
            $typesOf1d = [
                'C39', 'C39+', 'C39E', 'C39E+', 'C93', 'S25',
                'S25+', 'I25', 'I25+', 'C128', 'C128A', 'C128B',
                'EAN2', 'EAN5', 'EAN8','EAN13', 'UPCA',
                'UPCE', 'MSI', 'MSI+', 'POSTNET', 'PLANET','RMS4CC',
                'KIX', 'CODABAR', 'CODE11', 'PHARMA', 'PHARMA2T'
            ];
    
            $barcodeOf1d = [];
            foreach ($typesOf1d as $type) {
                $barcodeOf1d[$type] = DNS1D::getBarcodePNG((string) $user->id, $type);
            }
    
            $typesOf2d = [
                'QRCODE', 'QRCODE,L', 'QRCODE,M', 'QRCODE,Q', 'QRCODE,H',
                'DATAMATRIX', 'PDF417', 'PDF417,a,e',
            ];
    
            $barcodeOf2d = [];
            foreach ($typesOf2d as $type) {
                $barcodeOf2d[$type] = DNS2D::getBarcodePNG((string) $user->id, $type);
            }
    
            return view('users.barcode', compact('user', 'barcodeOf1d', 'barcodeOf2d'));
        }
    .
    .
    .


`$typesOf1d` 和 `$typesOf2d` 分别是一维条形码和二维条形码支持的所有格式，我们循环两个数组，通过 `getBarcodePNG((string) $user->id, $type)` ，生成不同类型的 PNG 格式的条形码，存放在 `$barcodeOf1d` 和 `$barcodeOf2d` 两个数组中，传给 `users.barcode` 页面。由于 'C128C', 'IMB' 这两种类型的条码对数据有特殊要求，所以我们不使用用户 id ，单独传入字符串 `11` 作为测试。

创建对应的 `users.barcode` 页面：

    $ touch resources/views/users/barcode.blade.php

---


*resources/views/users/barcode.blade.php*

    @extends('layouts.app')
    
    @section('title', '条形码')
    
    @section('content')
    
    <h3 class="text-center">一维条形码</h3>
    <div class="row">
        @foreach($barcodeOf1d as $type => $barcode)
    
            <div class="col-md-3 text-center" style="min-height: 100px">
                <div><img src="data:image/png; base64, {{ $barcode }}"/></div>
                <div>{{ $type }}</div>
            </div>
        @endforeach
    </div>
    
    <h3 class="text-center">二维条形码</h3>
    <div class="row">
        @foreach($barcodeOf2d as $type => $barcode)
            <div class="col-md-3 text-center" style="min-height: 200px">
                <div><img src="data:image/png; base64, {{ $barcode }}"/></div>
                <div>{{ $type }}</div>
            </div>
        @endforeach
    </div>
    
    @endsection


页面内容很简单，就是显示 `$barcodeOf1d` 和 `$barcodeOf2d` 两个数组中的条形码图片，注意 `getBarcodePNG` 方法最后返回的是 base64_encode 编码的图片，需要使用 `<img src="data:image/png; base64, {{ $barcode }}"/>` 的方式显示。

打开 http://larabbs.test/users/1/barcode 查看所有支持的条形码：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/tNyV0ZD15v.png)


### 调整大小和颜色

对于一维条形码，默认的每个竖条的宽度是 2px，高度为30px，二维条形码每个码元的宽度长宽均为 3px。默认的条码颜色色都是黑色的。

我们调整一下参数，设置背景色为 （220, 220, 220），调整一维条形码基本元素宽高为( 3px, 20px), 调整二维条形码基本元素宽高为 (4px, 8px)。

*app/Http/Controllers/UsersController.php*

    .
    .
    .
    
            $barcodeOf1d = [];
            foreach ($typesOf1d as $type) {
                $barcodeOf1d[$type] = DNS1D::getBarcodePNG((string) $user->id, $type, 3, 20, [220, 220, 220]);
            }
    
            $typesOf2d = [
                'QRCODE', 'QRCODE,L', 'QRCODE,M', 'QRCODE,Q', 'QRCODE,H',
                'DATAMATRIX', 'PDF417', 'PDF417,a,e',
            ];
    
            $barcodeOf2d = [];
            foreach ($typesOf2d as $type) {
                $barcodeOf2d[$type] = DNS2D::getBarcodePNG((string) $user->id, $type, 4, 8, [220, 220, 220]);
            }
    .
    .
    .


打开 http://larabbs.test/users/1/barcode ，查看所有支持的条形码：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/HrNMbdBG9n.png)



一维条形码变矮变胖，二维条形码被拉长，颜色都变为我们设置的 [220, 220, 220] 灰色。这里需要理解宽和高的设置为条码最小组成部分的宽和高。这部分只是为了测试，可以还原这部分代码。

